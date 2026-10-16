<script>
  import { onMount, onDestroy } from 'svelte';

  // Props
  export let numBoids = 80;
  export let numBoidsMobile = 40;
  export let opacity = 0.7;
  export let colors = [
    'rgba(0, 255, 136, 0.4)',   // primary
    'rgba(255, 107, 53, 0.4)',   // secondary
    'rgba(139, 92, 246, 0.4)',   // accent
    'rgba(229, 229, 229, 0.3)',  // white
  ];
  export let avoidanceSelector = '.boid-repel';
  export let avoidanceBuffer = 0.5;

  let canvas;
  let animationId;
  let boids = [];
  let frameCount = 0;
  let mouse = null;
  let repelRects = [];
  const cellSize = 70; // matches perception
  const mouseRadius = 220;

  class Boid {
    constructor(x, y, canvasEl) {
      this.pos = { x, y };
      this.vel = {
        x: (Math.random() - 0.5) * 1.2,
        y: (Math.random() - 0.5) * 1.2
      };
      this.acc = { x: 0, y: 0 };
      this.maxSpeed = 1.0;
      this.maxForce = 0.018;
      this.perception = 70;
      this.canvas = canvasEl;
      this.trail = [];
      this.maxTrailLength = 18;
      this.color = this.getRandomColor();
      this.size = 0.9;
      // Per-boid noise offset for organic wandering
      this.noiseOffset = Math.random() * 1000;
      this.noiseSpeed = 0.0006 + Math.random() * 0.0004;
    }

    getRandomColor() {
      return colors[Math.floor(Math.random() * colors.length)];
    }

    edges() {
      const margin = 50;
      if (this.pos.x > this.canvas.width + margin) this.pos.x = -margin;
      if (this.pos.x < -margin) this.pos.x = this.canvas.width + margin;
      if (this.pos.y > this.canvas.height + margin) this.pos.y = -margin;
      if (this.pos.y < -margin) this.pos.y = this.canvas.height + margin;
    }

    avoidElements() {
      let avoidForce = { x: 0, y: 0 };

      repelRects.forEach(rect => {
        const buffer = avoidanceBuffer;
        if (this.pos.x > rect.left - buffer &&
            this.pos.x < rect.right + buffer &&
            this.pos.y > rect.top - buffer &&
            this.pos.y < rect.bottom + buffer) {

          const distLeft = this.pos.x - rect.left;
          const distRight = rect.right - this.pos.x;
          const distTop = this.pos.y - rect.top;
          const distBottom = rect.bottom - this.pos.y;

          const minDist = Math.min(distLeft, distRight, distTop, distBottom);

          if (minDist === distLeft) avoidForce.x -= 1;
          else if (minDist === distRight) avoidForce.x += 1;
          else if (minDist === distTop) avoidForce.y -= 1;
          else if (minDist === distBottom) avoidForce.y += 1;
        }
      });

      return this.limit(avoidForce, this.maxForce);
    }

    // Noise-based wandering using overlapping sine waves
    wander(frame) {
      const t = frame * this.noiseSpeed + this.noiseOffset;
      const angle = Math.sin(t) * Math.PI * 2
                  + Math.cos(t * 0.73) * Math.PI;
      return {
        x: Math.cos(angle) * this.maxForce * 0.4,
        y: Math.sin(angle) * this.maxForce * 0.4
      };
    }

    align(boidsList) {
      let steering = { x: 0, y: 0 };
      let total = 0;

      for (let other of boidsList) {
        let d = this.distance(this.pos, other.pos);
        if (other !== this && d < this.perception) {
          steering.x += other.vel.x;
          steering.y += other.vel.y;
          total++;
        }
      }

      if (total > 0) {
        steering.x /= total;
        steering.y /= total;
        let mag = Math.sqrt(steering.x * steering.x + steering.y * steering.y);
        if (mag > 0) {
          steering.x = (steering.x / mag) * this.maxSpeed;
          steering.y = (steering.y / mag) * this.maxSpeed;
        }
        steering.x -= this.vel.x;
        steering.y -= this.vel.y;
        return this.limit(steering, this.maxForce);
      }
      return steering;
    }

    cohesion(boidsList) {
      let steering = { x: 0, y: 0 };
      let total = 0;

      for (let other of boidsList) {
        let d = this.distance(this.pos, other.pos);
        if (other !== this && d < this.perception) {
          steering.x += other.pos.x;
          steering.y += other.pos.y;
          total++;
        }
      }

      if (total > 0) {
        steering.x /= total;
        steering.y /= total;
        steering.x -= this.pos.x;
        steering.y -= this.pos.y;
        let mag = Math.sqrt(steering.x * steering.x + steering.y * steering.y);
        if (mag > 0) {
          steering.x = (steering.x / mag) * this.maxSpeed;
          steering.y = (steering.y / mag) * this.maxSpeed;
        }
        steering.x -= this.vel.x;
        steering.y -= this.vel.y;
        return this.limit(steering, this.maxForce);
      }
      return steering;
    }

    separation(boidsList) {
      let steering = { x: 0, y: 0 };
      let total = 0;

      for (let other of boidsList) {
        let d = this.distance(this.pos, other.pos);
        if (other !== this && d < 40) {
          let diff = {
            x: this.pos.x - other.pos.x,
            y: this.pos.y - other.pos.y
          };
          if (d > 0) {
            diff.x /= d;
            diff.y /= d;
          }
          steering.x += diff.x;
          steering.y += diff.y;
          total++;
        }
      }

      if (total > 0) {
        steering.x /= total;
        steering.y /= total;
        let mag = Math.sqrt(steering.x * steering.x + steering.y * steering.y);
        if (mag > 0) {
          steering.x = (steering.x / mag) * this.maxSpeed;
          steering.y = (steering.y / mag) * this.maxSpeed;
        }
        steering.x -= this.vel.x;
        steering.y -= this.vel.y;
        return this.limit(steering, this.maxForce);
      }
      return steering;
    }

    // Push nearby boids away from the cursor, harder the closer they are
    repelFromMouse() {
      if (!mouse) return { x: 0, y: 0 };
      const dx = this.pos.x - mouse.x;
      const dy = this.pos.y - mouse.y;
      const d = Math.sqrt(dx * dx + dy * dy);
      if (d > mouseRadius || d < 1) return { x: 0, y: 0 };

      const falloff = 1 - d / mouseRadius;
      const strength = this.maxForce * 4 * falloff;
      return {
        x: (dx / d) * strength,
        y: (dy / d) * strength
      };
    }

    flock(boidsList, frame) {
      let alignment = this.align(boidsList);
      let cohesion = this.cohesion(boidsList);
      let separation = this.separation(boidsList);
      let avoidance = this.avoidElements();
      let wander = this.wander(frame);
      let mouseForce = this.repelFromMouse();

      alignment.x *= 1.2;
      alignment.y *= 1.2;
      cohesion.x *= 0.6;
      cohesion.y *= 0.6;
      separation.x *= 1.4;
      separation.y *= 1.4;
      avoidance.x *= 1.0;
      avoidance.y *= 1.0;

      this.acc.x += alignment.x + cohesion.x + separation.x + avoidance.x + wander.x + mouseForce.x;
      this.acc.y += alignment.y + cohesion.y + separation.y + avoidance.y + wander.y + mouseForce.y;
    }

    update() {
      this.vel.x += this.acc.x;
      this.vel.y += this.acc.y;
      this.vel = this.limit(this.vel, this.maxSpeed);

      this.pos.x += this.vel.x;
      this.pos.y += this.vel.y;

      this.trail.push({ x: this.pos.x, y: this.pos.y });
      if (this.trail.length > this.maxTrailLength) {
        this.trail.shift();
      }

      this.acc.x = 0;
      this.acc.y = 0;
    }

    draw(ctx) {
      if (this.trail.length < 2) return;

      ctx.lineCap = 'round';
      ctx.lineJoin = 'round';

      // Draw the trail as a few tapering, fading polylines (cheaper than one stroke per point)
      const n = this.trail.length;
      const bands = 4;
      for (let k = 0; k < bands; k++) {
        const start = Math.floor((k * (n - 1)) / bands);
        const end = Math.floor(((k + 1) * (n - 1)) / bands);
        const t = (k + 1) / bands;
        ctx.globalAlpha = Math.min(1, t * t * 1.2);
        ctx.strokeStyle = this.color;
        ctx.lineWidth = t * 1.4 + 0.2;
        ctx.beginPath();
        ctx.moveTo(this.trail[start].x, this.trail[start].y);
        for (let i = start + 1; i <= end; i++) {
          const a = this.trail[i - 1];
          const b = this.trail[i];
          // Break the line if the boid wrapped around the canvas edge
          if (Math.abs(b.x - a.x) > 100 || Math.abs(b.y - a.y) > 100) {
            ctx.moveTo(b.x, b.y);
          } else {
            ctx.lineTo(b.x, b.y);
          }
        }
        ctx.stroke();
      }

      // Bright head dot
      ctx.globalAlpha = 1;
      ctx.fillStyle = this.color;
      ctx.beginPath();
      ctx.arc(this.pos.x, this.pos.y, this.size, 0, Math.PI * 2);
      ctx.fill();

      ctx.globalAlpha = 1;
    }

    distance(a, b) {
      let dx = a.x - b.x;
      let dy = a.y - b.y;
      return Math.sqrt(dx * dx + dy * dy);
    }

    limit(vector, max) {
      let mag = Math.sqrt(vector.x * vector.x + vector.y * vector.y);
      if (mag > max) {
        return {
          x: (vector.x / mag) * max,
          y: (vector.y / mag) * max
        };
      }
      return vector;
    }
  }

  function resizeCanvas() {
    if (!canvas) return;
    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;
  }

  function initBoids() {
    if (!canvas) return;
    boids = [];
    const count = window.innerWidth < 768 ? numBoidsMobile : numBoids;
    for (let i = 0; i < count; i++) {
      boids.push(new Boid(
        Math.random() * canvas.width,
        Math.random() * canvas.height,
        canvas
      ));
    }
  }

  function animate() {
    if (!canvas) return;
    const ctx = canvas.getContext('2d');

    ctx.clearRect(0, 0, canvas.width, canvas.height);

    // Screen blending makes overlapping trails glow luminously
    ctx.globalCompositeOperation = 'screen';

    frameCount++;

    // Read layout once per frame instead of once per boid
    repelRects = Array.from(document.querySelectorAll(avoidanceSelector))
      .map(el => el.getBoundingClientRect());

    // Spatial hash so each boid only checks nearby boids, not all of them
    const grid = new Map();
    for (let boid of boids) {
      const key = Math.floor(boid.pos.x / cellSize) + ',' + Math.floor(boid.pos.y / cellSize);
      const cell = grid.get(key);
      if (cell) cell.push(boid); else grid.set(key, [boid]);
    }

    for (let boid of boids) {
      boid.edges();
      const cx = Math.floor(boid.pos.x / cellSize);
      const cy = Math.floor(boid.pos.y / cellSize);
      const neighbours = [];
      for (let gx = cx - 1; gx <= cx + 1; gx++) {
        for (let gy = cy - 1; gy <= cy + 1; gy++) {
          const cell = grid.get(gx + ',' + gy);
          if (cell) for (let o of cell) neighbours.push(o);
        }
      }
      boid.flock(neighbours, frameCount);
      boid.update();
      boid.draw(ctx);
    }

    ctx.globalCompositeOperation = 'source-over';

    animationId = requestAnimationFrame(animate);
  }

  onMount(() => {
    resizeCanvas();
    initBoids();
    animate();

    window.addEventListener('resize', handleResize);
    window.addEventListener('mousemove', handleMouseMove);
    window.addEventListener('mouseleave', handleMouseLeave);
  });

  onDestroy(() => {
    if (animationId) {
      cancelAnimationFrame(animationId);
    }
    window.removeEventListener('resize', handleResize);
    window.removeEventListener('mousemove', handleMouseMove);
    window.removeEventListener('mouseleave', handleMouseLeave);
  });

  function handleMouseMove(e) {
    mouse = { x: e.clientX, y: e.clientY };
  }

  function handleMouseLeave() {
    mouse = null;
  }

  function handleResize() {
    resizeCanvas();
    initBoids();
  }
</script>

<canvas
  bind:this={canvas}
  style="
    position: fixed;
    top: 0;
    left: 0;
    width: 100vw;
    height: 100vh;
    z-index: 0;
    pointer-events: none;
    opacity: {opacity};
  "
></canvas>

<style>
  /* Canvas styles are inline for better prop control */
</style>