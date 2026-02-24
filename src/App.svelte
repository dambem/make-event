<script>
  import { onMount, onDestroy } from 'svelte';
  import Boids from './lib/Boids.svelte';
  import TicketCard from './lib/TicketCard.svelte';
  import BottomTicketDrawer from './lib/BottomTicketDrawer.svelte';

  import imgNetworking from './assets/1758730076988.jpeg'
  import imgVR from './assets/1758730080901.jpeg'
  import imgTwoSpeakers from './assets/1758730076939.jpeg'
  import imgAudience from './assets/1758730080993.jpeg'

  const customColors = [
    'rgba(0, 255, 136, 0.4)',
    'rgba(0, 255, 136, 0.4)',
    'rgba(139, 92, 246, 0.4)',
    'rgba(229, 229, 229, 0.3)',
  ];

  let isHovered = false;
  var location = 'https://forms.gle/HZbQd3WQsqmzTByAA'

  let showDrawer = false;
  let observer;

  onMount(() => {
    const aboutEl = document.querySelector('#about');
    observer = new IntersectionObserver(
      ([entry]) => { showDrawer = !entry.isIntersecting; },
      { threshold: 0.1 }
    );
    if (aboutEl) observer.observe(aboutEl);
  });

  onDestroy(() => {
    if (observer) observer.disconnect();
  });
</script>

<main>
    <Boids 
    numBoids={500}
    numBoidsMobile={40}
    opacity={0.7}
    colors={customColors}
    avoidanceSelector=".boid-repel"
    avoidanceBuffer={50}
    />
  <div id='background'></div>
      <div class="url-display mono">
        <span on:click={()=>window.location=location} id="url-text">~ Creative Catalyst Tech Talks ~</span>
    </div>
    <nav class="nav">
        <a href="#home" class="nav-item">home</a>
        <a href="#about" class="nav-item">about</a>
        <a href="#talks" class="nav-item">talks</a>
        <a href="#previous" class="nav-item">make~ 01</a>
        <a href="#tips" class="nav-item">speaker tips</a>
        <a href="#speak" class="nav-item">speak</a>
        <a href="#contact" class="nav-item">contact</a>
    </nav>
    <section id="home" class="header">
        <h1 class="title" class:boid-repel={isHovered} on:mouseover={() => isHovered = true} on:mouseout={() => isHovered = false}>make~</h1>
        <p class="subtitle">creative tech flash talks in southampton</p>
    </section>

    <section id="about" class="section">
        <div class="about-inner">
            <div class="content">
                <h2>What is make~?</h2>
                <p><strong>make~ is an evening of flash talks about creative technology, tools, ideas, and unfinished projects.</strong> It's made for artists, designers, hackers, programmers, students, tinkerers, writers. Basically, anyone who wants to share something they've made, thought about, or can't stop tinkering with.</p>
                <p class="interactive-text">6 speakers. 6 minutes each. Then time for drinks, food and idea sharing</p>
            </div>
            <div class="about-ticket">
                <TicketCard/>
            </div>
        </div>
    </section>

    <section id="talks" class="section">
        <div class="content">
            <h2>What kinds of talks?</h2>
            <ul style="list-style: none; padding-left: 0;">
                <li class="interactive-text" style="margin: 1rem 0; padding-left: 2rem; position: relative;">
                    <span style="position: absolute; left: 0; color: var(--primary);">*</span>
                    a generative music experiment made with code
                </li>
                <li class="interactive-text" style="margin: 1rem 0; padding-left: 2rem; position: relative;">
                    <span style="position: absolute; left: 0; color: var(--secondary);">*</span>
                    an obsession with old keyboards
                </li>
                <li class="interactive-text" style="margin: 1rem 0; padding-left: 2rem; position: relative;">
                    <span style="position: absolute; left: 0; color: var(--accent);">*</span>
                    a philosophy of open tools
                </li>
                <li class="interactive-text" style="margin: 1rem 0; padding-left: 2rem; position: relative;">
                    <span style="position: absolute; left: 0; color: var(--primary);">*</span>
                    a speculative design concept
                </li>
                <li class="interactive-text" style="margin: 1rem 0; padding-left: 2rem; position: relative;">
                    <span style="position: absolute; left: 0; color: var(--secondary);">*</span>
                    a failed side project with lessons that stuck
                </li>
            </ul>
            <p><em>We welcome the creative, the short lessons in tech. You don't need a polished pitch deck. You just need an idea to share.</em></p>
        </div>
    </section>

    <section id="previous" class="section previous-section">
        <div class="content">
            <h2>make~ 01</h2>

            <div class="event-hero">
                <img src={imgNetworking} alt="People talking and networking at make~ 01" />
            </div>

            <p>The first make~ went better than I could have hoped. Getting that many interesting people in the same room; makers, designers, developers, educators — and having it feel genuinely warm rather than just another networking event. </p>

            <div class="event-photo-strip">
                <img src={imgVR} alt="Holly New presenting through a VR headset" />
                <img src={imgTwoSpeakers} alt="Megan Watkins and Fiona Bavinton presenting" />
                <img src={imgAudience} alt="Speaker at make~ 01" />
            </div>

            <p class="section-subheading">speakers</p>
            <div class="speaker-grid">
                <div class="speaker-card">
                    <h4>Tracy Gardner PhD</h4>
                    <p>Demoed flock3D live. It's a genuinely fun tool — worth trying yourself, whether you're in education or just curious about what it can do.</p>
                </div>
                <div class="speaker-card">
                    <h4>Simon Strong FRSA</h4>
                    <p>Six minutes on how we might fundamentally rethink the way people learn. </p>
                </div>
                <div class="speaker-card">
                    <h4>Emma Callow</h4>
                    <p>Mixed a live workshop with her talk — people actually made things. Her case for bringing more play and tinkering into development landed.</p>
                </div>
                <div class="speaker-card speaker-card--highlight">
                    <h4>Holly New</h4>
                    <p>First VR-native fashion designer. She did the entire talk live through her VR headset — not simulated, not pre-recorded, in-person in the headset for six straight minutes. The workflow was something else.</p>
                </div>
                <div class="speaker-card">
                    <h4>Alex Bainbridge</h4>
                    <p>Came down between flights to Washington DC to show what personalised AI tour guides can actually look like in practice. </p>
                </div>
                <div class="speaker-card speaker-card--duo">
                    <h4>Megan Watkins &amp; Fiona Bavinton</h4>
                    <p>A full costume design workflow using AI while keeping period-era accuracy intact. Two people on stage made the whole process visible in a way a solo talk wouldn't.</p>
                </div>
                <div class="speaker-card">
                    <h4>Jen Wright</h4>
                    <p>Closed the night talking about the future of creative work and how to stay oriented in it without losing hope. </p>
                </div>
            </div>

            <div class="event-thanks">
                <p>Thank you to Barclays Eagle Labs and Southampton City Council — and specifically Luke Hampson and Catherine Anderson — for the space, the pizzas, and the drinks. And to Fred Bassett (FRSA) for pulling everyone together, and to Sian Campbell, Chris Cloke Browne PhD FRSA, Paul Woodford, Natalie Haigh, Kane Watkinson and everyone else who made the evening run.</p>
            </div>
        </div>
    </section>

    <section id="tips" class="section">
        <div class="content">
            <h2>Speaker Tips</h2>
            <p class="section-intro">6 minutes goes fast. Here's how to make the most of it.</p>
            <div class="tips-grid">
                <div class="tip-card">
                    <span class="tip-number">01</span>
                    <h3>Pick one idea</h3>
                    <p>Don't try to cover everything. One thing, explored well, is more memorable than five things rushed.</p>
                </div>
                <div class="tip-card">
                    <span class="tip-number">02</span>
                    <h3>Show, don't explain</h3>
                    <p>A working demo beats a thousand slides. If you can show it live, show it live.</p>
                </div>
                <div class="tip-card">
                    <span class="tip-number">03</span>
                    <h3>Half-finished is fine</h3>
                    <p>You don't need a polished product. Rough edges and open questions make for better conversation afterwards.</p>
                </div>
                <div class="tip-card">
                    <span class="tip-number">04</span>
                    <h3>Time yourself once</h3>
                    <p>6 minutes is shorter than you think. Do one run-through. Cut anything that isn't essential to the point.</p>
                </div>
                <div class="tip-card">
                    <span class="tip-number">05</span>
                    <h3>Start with the thing</h3>
                    <p>Skip the preamble. Don't open with "my talk is about...". Just show it. Context follows naturally.</p>
                </div>
                <div class="tip-card">
                    <span class="tip-number">06</span>
                    <h3>Arrive 15 mins early</h3>
                    <p>We'll have HDMI and USB-C adapters. If you're demoing something live, test your setup before the room fills up.</p>
                </div>
            </div>
            <div class="tips-footer">
                <p>Slides are optional. Demos are encouraged. First-time speakers very welcome.</p>
            </div>
        </div>
    </section>

    <section id="speak" class="section">
        <div class="content">
            <h2>Want to Speak?</h2>
            <p>We'd love that. You can talk about something you made, something you're making, or just something you can't stop thinking about. We're especially excited by first-time speakers, weird formats, and strange metaphors.</p>
            <p><strong>Date:</strong> 10 March 2026<br>
            <strong>Location:</strong> Network Eagle Lab, Portland Terrace, Southampton<br>
            <strong>Time:</strong> Doors at 5:30 PM. Talks from 6:00 PM.</p>
            <div style="margin-top: 2rem;">
                <a href="{location}" class="interactive-text" style="color: var(--primary); text-decoration: none; font-weight: 600;">→ Apply to speak here</a>
            </div>
        </div>
    </section>
    <section id="contact" class="section">
        <div class="content">
            <h2>Contact</h2>
            <p>Want to sponsor, provide drinks or just have some questions, email us!</p>
            <a> Damian Bemben | Organiser | damianbemben@geochip.uk</a>
        </div>
    </section>

  <BottomTicketDrawer visible={showDrawer} />
</main>

<style>
      .class-on-hover {
    color: red;
  }
</style>