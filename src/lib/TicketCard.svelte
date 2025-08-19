<script>
  import { onMount, onDestroy } from 'svelte';

  export let eventTitle = "Early Bird Ticket";
  export let eventDate = "September 23rd, 2025";
  export let eventTime = "5:30 - 8:00 PM";
  export let eventDateTime = "2025-09-23T17:30:00"; // ISO format for countdown
  export let venue = "Network Eagle Labs";
    let countdown = { days: 0, hours: 0, minutes: 0, seconds: 0 };
  let interval;

  export let location = "Southampton";
  export let format = "6 × 6min talks";
  export let price = "Free";
  export let eventbriteUrl = "https://www.eventbrite.co.uk/e/rsa-creative-catalyst-make-tickets-1567351664019?aff=oddtdtcreator";
  export let perks = [
    "6 creative tech flash talks",
    "Networking with local makers & creatives", 
    "Light refreshments & drinks",
  ];
  let mapsUrl = "https://maps.app.goo.gl/uYA7BFMGPaaeM15Y6";
    function updateCountdown() {
    const now = new Date().getTime();
    const eventTime = new Date(eventDateTime).getTime();
    const difference = eventTime - now;
    
    if (difference > 0) {
      countdown = {
        days: Math.floor(difference / (1000 * 60 * 60 * 24)),
        hours: Math.floor((difference % (1000 * 60 * 60 * 24)) / (1000 * 60 * 60)),
        minutes: Math.floor((difference % (1000 * 60 * 60)) / (1000 * 60)),
        seconds: Math.floor((difference % (1000 * 60)) / 1000)
      };
    } else {
      countdown = { days: 0, hours: 0, minutes: 0, seconds: 0 };
    }
  }
    onMount(() => {
    updateCountdown();
    interval = setInterval(updateCountdown, 1000);
  });
  
  onDestroy(() => {
    if (interval) clearInterval(interval);
  });
  function openTickets() {
    window.open(eventbriteUrl, '_blank');
  }
  function openDirections() {
    window.open(mapsUrl, '_blank');
  }
</script>

<div class="ticket-section">
  <div class="ticket-card">
    <div class="ticket-header">
      <h3 class="ticket-title">{eventTitle}</h3>
      
      <p class="ticket-date">{eventDate}</p>
            <div class="countdown">
          <div class="countdown-item">
            <span class="countdown-number">{countdown.days}</span>
            <span class="countdown-label">days</span>
          </div>
          <div class="countdown-item">
            <span class="countdown-number">{countdown.hours}</span>
            <span class="countdown-label">hrs</span>
          </div>
          <div class="countdown-item">
            <span class="countdown-number">{countdown.minutes}</span>
            <span class="countdown-label">min</span>
          </div>
          <div class="countdown-item">
            <span class="countdown-number">{countdown.seconds}</span>
            <span class="countdown-label">sec</span>
          </div>
        </div>
    </div>
    
    <div class="ticket-details">
      <div class="ticket-detail">
        <span class="detail-label">Time</span>
        <span class="detail-value">{eventTime}</span>
      </div>
      <div class="ticket-detail">
        <span class="detail-label">Venue</span>
        <span class="detail-value">{venue}</span>
      </div>
      <div class="ticket-detail">
        <span class="detail-label">Format</span>
        <span class="detail-value">{format}</span>
      </div>
      <div class="ticket-detail">
        <span class="detail-label">Location</span>
        <span class="detail-value">{location}</span>
      </div>
    </div>
    
    <div class="ticket-price">
      <p class="price-label">Ticket Price</p>
      <p class="price-value">{price}</p>
    <div class="availability">
      <p class="availability-text">Limited seats available • Book now</p>
    </div>
    </div>
    
    <button class="eventbrite-btn" on:click={openTickets}>
      Get Tickets Now
    </button>
    <button class="directions-btn" on:click={openDirections}>
      📍 Get Directions
    </button>
    <div class="ticket-perks">
      <p class="perks-title">What's Included</p>
      <ul class="perks-list">
        {#each perks as perk}
          <li class="perk-item">
            <span class="perk-icon">✓</span>
            {perk}
          </li>
        {/each}
      </ul>
    </div>
    

  </div>
</div>

<style>
  :global(:root) {
    --primary: #00ff88;
    --secondary: #ff6b35;
    --accent: #4ecdc4;
    --bg: #0a0a0a;
    --text: #ffffff;
    --text-dim: #cccccc;
    --card-bg: rgba(255, 255, 255, 0.05);
    --border: rgba(255, 255, 255, 0.1);
  }

  .ticket-section {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 2rem;
    margin: 2rem 0;
  }

  .ticket-card {
    background: var(--card-bg);
    border: 1px solid var(--border);
    border-radius: 16px;
    padding: 2rem;
    backdrop-filter: blur(10px);
    position: relative;
    overflow: hidden;
    max-width: 500px;
    width: 100%;
    transition: all 0.3s ease;
    font-family: 'Monaco', 'Menlo', 'Ubuntu Mono', monospace;
  }

  .ticket-card:hover {
    transform: translateY(-5px);
    box-shadow: 0 20px 40px rgba(0, 255, 136, 0.1);
    border-color: var(--primary);
  }

  .ticket-card::before {
    content: '';
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    height: 2px;
    background: linear-gradient(90deg, var(--primary), var(--secondary), var(--accent));
  }

  .ticket-header {
    text-align: center;
    margin-bottom: 1.5rem;
  }

  .ticket-title {
    font-size: 1.25rem;
    color: var(--primary);
    margin-bottom: 0.5rem;
    font-weight: 600;
  }

  .ticket-date {
    font-size: 0.9rem;
    color: var(--text-dim);
  }

  .ticket-details {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 1rem;
    margin-bottom: 2rem;
    font-size: 0.9rem;
  }

  .ticket-detail {
    display: flex;
    flex-direction: column;
    gap: 0.25rem;
  }

  .detail-label {
    color: var(--accent);
    font-size: 0.8rem;
    text-transform: uppercase;
    letter-spacing: 0.5px;
  }

  .detail-value {
    color: var(--text);
  }

  .ticket-price {
    text-align: center;
    margin-bottom: 2rem;
  }

  .price-label {
    font-size: 0.9rem;
    color: var(--text-dim);
    margin-bottom: 0.5rem;
  }

  .price-value {
    font-size: 2rem;
    color: var(--primary);
    font-weight: 600;
  }

  .eventbrite-btn {
    width: 100%;
    padding: 1rem 2rem;
    background: linear-gradient(135deg, var(--primary), var(--accent));
    color: var(--bg);
    border: none;
    border-radius: 12px;
    font-family: inherit;
    font-size: 1rem;
    font-weight: 600;
    cursor: pointer;
    transition: all 0.3s ease;
    position: relative;
    overflow: hidden;
    text-transform: uppercase;
    letter-spacing: 1px;
  }

  .eventbrite-btn::before {
    content: '';
    position: absolute;
    top: 0;
    left: -100%;
    width: 100%;
    height: 100%;
    background: linear-gradient(135deg, var(--secondary), var(--primary));
    transition: left 0.3s ease;
    z-index: -1;
  }

  .eventbrite-btn:hover::before {
    left: 0;
  }

  .eventbrite-btn:hover {
    transform: translateY(-2px);
    box-shadow: 0 10px 25px rgba(0, 255, 136, 0.3);
  }
  .directions-btn {
    width: 100%;
    padding: 0.875rem 2rem;
    background: transparent;
    color: var(--accent);
    border: 1px solid var(--accent);
    border-radius: 12px;
    font-family: inherit;
    font-size: 0.95rem;
    font-weight: 500;
    cursor: pointer;
    transition: all 0.3s ease;
    margin-top: 1rem;
    text-transform: uppercase;
    letter-spacing: 1px;
    position: relative;
    overflow: hidden;
  }

  .directions-btn::before {
    content: '';
    position: absolute;
    top: 0;
    left: -100%;
    width: 100%;
    height: 100%;
    background: var(--accent);
    transition: left 0.3s ease;
    z-index: -1;
  }

  .directions-btn:hover::before {
    left: 0;
  }

  .directions-btn:hover {
    color: var(--bg);
    transform: translateY(-2px);
    box-shadow: 0 8px 20px rgba(78, 205, 196, 0.3);
  }

  .ticket-perks {
    margin-top: 1.5rem;
    padding-top: 1.5rem;
    border-top: 1px solid var(--border);
  }

  .perks-title {
    font-size: 0.9rem;
    color: var(--accent);
    margin-bottom: 1rem;
    text-align: center;
  }

  .perks-list {
    list-style: none;
    display: grid;
    gap: 0.5rem;
    padding: 0;
  }

  .perk-item {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    font-size: 0.85rem;
    color: var(--text-dim);
  }

  .perk-icon {
    width: 16px;
    height: 16px;
    background: var(--primary);
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 0.7rem;
    color: var(--bg);
    flex-shrink: 0;
  }

  .availability {
    text-align: center;
    margin-top: 1rem;
  }

  .availability-text {
    font-size: 0.8rem;
    color: var(--secondary);
    animation: pulse 2s infinite;
  }

  @keyframes pulse {
    0%, 100% { opacity: 1; }
    50% { opacity: 0.7; }
  }

  @media (max-width: 768px) {
    .ticket-details {
      grid-template-columns: 1fr;
    }
    
    .ticket-card {
      padding: 1.5rem;
    }
  }
  .countdown {
    display: flex;
    justify-content: center;
    gap: 1rem;
    margin-top: 1rem;
  }

  .countdown-item {
    display: flex;
    flex-direction: column;
    align-items: center;
    background: rgba(255, 255, 255, 0.03);
    border: 1px solid var(--border);
    border-radius: 8px;
    padding: 0.5rem;
    min-width: 50px;
    transition: all 0.3s ease;
  }

  .countdown-item:hover {
    border-color: var(--primary);
    background: rgba(0, 255, 136, 0.05);
  }

  .countdown-number {
    font-size: 1.25rem;
    font-weight: 600;
    color: var(--primary);
    line-height: 1;
  }

  .countdown-label {
    font-size: 0.7rem;
    color: var(--text-dim);
    text-transform: uppercase;
    letter-spacing: 0.5px;
    margin-top: 0.25rem;
  }
</style>