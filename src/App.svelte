<script>
  import svelteLogo from './assets/svelte.svg'
  import viteLogo from '/vite.svg'
  import Counter from './lib/Counter.svelte'

  function redirect(obj) {
    var goTo= obj.getAttribute
  }
  var location = 'https://forms.gle/HZbQd3WQsqmzTByAA'
</script>

<main>
  <div id='background'></div>
      <div class="url-display mono">
        <span on:click={()=>window.location=location} id="url-text">~ Creative Catalyst Tech Talks ~</span>
    </div>
    <nav class="nav">
        <a href="#home" class="nav-item">home</a>
        <a href="#about" class="nav-item">about</a>
        <a href="#talks" class="nav-item">talks</a>
        <a href="#speak" class="nav-item">speak</a>
        <a href="#contact" class="nav-item">contact</a>
    </nav>
    <section id="home" class="header">
        <h1 class="title">make~</h1>
        <p class="subtitle">creative tech flash talks in southampton</p>
        <p class="subtitle"> 23rd September 2025 6-8pm</p>
        <p class="subtitle">Register Your Interest</p>
    </section>

    <section id="about" class="section">
        <div class="content">
            <h2>What is make~?</h2>
            <p><strong>make~ is an evening of flash talks about creative technology, tools, ideas, and unfinished projects.</strong> It's made for artists, designers, hackers, programmers, students, tinkerers, writers. 
              
              Basically, anyone who wants to share something they've made, thought about, or can't stop tinkering with.</p>
            <p class="interactive-text">6 speakers. 6 minutes each. Then time for drinks, food and idea sharing</p>
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
    <section id="speak" class="section">
        <div class="content">
            <h2>Want to Speak?</h2>
            <p>We'd love that. You can talk about something you made, something you're making, or just something you think. We're especially excited by first-time speakers, non-coders, weird formats, and strange metaphors.</p>
            <p><strong>Date:</strong> September 23rd, 2025<br>
            <strong>Location:</strong> Network Eagle Labs, Southampton<br>
            <strong>Time:</strong> Doors at 6:00PM. Talks 6:30–8:00.</p>
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



















</main>

