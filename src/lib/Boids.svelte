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
  
  class Boid {
    constructor(x, y, canvasEl) {
      this.pos = { x, y };
      this.vel = {
        x: (Math.random() - 0.5) * 0.5,
        y: (Math.random() - 0.5) * 0.5
      };
      this.acc = { x: 0, y: 0 };
      this.maxSpeed = 0.8;
      this.maxForce = 0.02;
      this.perception = 80;
      this.canvas = canvasEl;
      this.trail = [];
      this.maxTrailLength = 3;
      this.color = this.getRandomColor();
      this.size = 1;
      this.glowSize = 4;
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
      const elements = document.querySelectorAll(avoidanceSelector);
      let avoidForce = { x: 0, y: 0 };
      
      elements.forEach(element => {
        const rect = element.getBoundingClientRect();
        
        // Check if boid is near/inside element bounds with small buffer
        const buffer = avoidanceBuffer;
        if (this.pos.x > rect.left - buffer && 
            this.pos.x < rect.right + buffer &&
            this.pos.y > rect.top - buffer && 
            this.pos.y < rect.bottom + buffer) {
          
          // Calculate gentle deflection based on closest edge
          const distLeft = this.pos.x - rect.left;
          const distRight = rect.right - this.pos.x;
          const distTop = this.pos.y - rect.top;
          const distBottom = rect.bottom - this.pos.y;
          
          // Find minimum distance to edge
          const minDist = Math.min(distLeft, distRight, distTop, distBottom);
          
          // Apply gentle force away from closest edge
          if (minDist === distLeft) avoidForce.x -= 1;
          else if (minDist === distRight) avoidForce.x += 1;
          else if (minDist === distTop) avoidForce.y -= 1;
          else if (minDist === distBottom) avoidForce.y += 1;
        }
      });
      
      return this.limit(avoidForce, this.maxForce);
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

    flock(boidsList) {
      let alignment = this.align(boidsList);
      let cohesion = this.cohesion(boidsList);
      let separation = this.separation(boidsList);
      let avoidance = this.avoidElements();
      
      // Weight the forces
      alignment.x *= 1.5;
      alignment.y *= 1.5;
      cohesion.x *= 0.8;
      cohesion.y *= 0.8;
      separation.x *= 1.2;
      separation.y *= 1.2;
      avoidance.x *= 1.0;
      avoidance.y *= 1.0;
      
      this.acc.x += alignment.x + cohesion.x + separation.x + avoidance.x;
      this.acc.y += alignment.y + cohesion.y + separation.y + avoidance.y;
    }

    update() {
      // Update velocity and position
      this.vel.x += this.acc.x;
      this.vel.y += this.acc.y;
      this.vel = this.limit(this.vel, this.maxSpeed);
      
      this.pos.x += this.vel.x;
      this.pos.y += this.vel.y;
      
      // Update trail (very short)
      this.trail.push({ x: this.pos.x, y: this.pos.y });
      if (this.trail.length > 3) {
        this.trail.shift();
      }
      
      // Reset acceleration
      this.acc.x = 0;
      this.acc.y = 0;
    }

    draw(ctx) {
      // Draw very subtle trail

      // Draw boid dot
      ctx.globalAlpha = 0.6;
      ctx.fillStyle = this.color;
      ctx.beginPath();
      ctx.arc(this.pos.x, this.pos.y, this.size, 0, Math.PI * 2);
      ctx.fill();
      
      // Draw subtle glow
      ctx.globalAlpha = 0.1;
      ctx.beginPath();
      ctx.arc(this.pos.x, this.pos.y, this.glowSize, 0, Math.PI * 2);
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
    
    // Clear completely each frame
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    
    for (let boid of boids) {
      boid.edges();
      boid.flock(boids);
      boid.update();
      boid.draw(ctx);
    }
    
    animationId = requestAnimationFrame(animate);
  }
  
  onMount(() => {
    resizeCanvas();
    initBoids();
    animate();
    
    // Add resize listener
    window.addEventListener('resize', handleResize);
  });
  
  onDestroy(() => {
    if (animationId) {
      cancelAnimationFrame(animationId);
    }
    window.removeEventListener('resize', handleResize);
  });
  
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
/>

<style>
  /* Canvas styles are inline for better prop control */
</style>