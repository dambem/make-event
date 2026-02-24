<script>
  import TicketCard from './TicketCard.svelte';

  export let visible = false;

  let expanded = false;

  function toggle() {
    expanded = !expanded;
  }

  function close() {
    expanded = false;
  }

  function handleKeydown(e) {
    if (e.key === 'Escape') close();
  }
</script>

<svelte:window on:keydown={handleKeydown} />

{#if expanded}
  <div class="drawer-backdrop" on:click={close} aria-hidden="true" />
{/if}

<div class="drawer" class:visible class:expanded>
  <button class="drawer-handle" on:click={toggle} aria-expanded={expanded} aria-label="Toggle ticket drawer">
    <span class="drawer-grip">
      <span></span>
      <span></span>
      <span></span>
    </span>
    <span class="drawer-meta">make~&nbsp;&nbsp;·&nbsp;&nbsp;10 Mar 2026&nbsp;&nbsp;·&nbsp;&nbsp;Free</span>
    <span class="drawer-cta" class:flipped={expanded}>get tickets ↑</span>
  </button>

  {#if expanded}
    <div class="drawer-body">
      <TicketCard />
    </div>
  {/if}
</div>

<style>
  .drawer-backdrop {
    position: fixed;
    inset: 0;
    z-index: 149;
    background: rgba(0, 0, 0, 0.4);
    backdrop-filter: blur(2px);
    -webkit-backdrop-filter: blur(2px);
  }

  .drawer {
    position: fixed;
    bottom: 0;
    left: 0;
    right: 0;
    z-index: 150;
    transform: translateY(100%);
    transition: transform 0.4s cubic-bezier(0.32, 0.72, 0, 1);
    border-radius: 16px 16px 0 0;
    background: rgba(8, 8, 8, 0.95);
    backdrop-filter: blur(20px);
    -webkit-backdrop-filter: blur(20px);
    border-top: 1px solid rgba(255, 255, 255, 0.08);
  }

  .drawer.visible {
    transform: translateY(calc(100% - 56px));
  }

  .drawer.expanded {
    transform: translateY(0);
  }

  .drawer-handle {
    display: flex;
    align-items: center;
    justify-content: space-between;
    height: 56px;
    width: 100%;
    padding: 0 1.25rem;
    background: none;
    border: none;
    cursor: pointer;
    font-family: 'JetBrains Mono', 'Monaco', 'Menlo', monospace;
    color: inherit;
    gap: 1rem;
  }

  .drawer-handle:focus-visible {
    outline: 2px solid rgba(0, 255, 136, 0.5);
    outline-offset: -2px;
  }

  .drawer-grip {
    display: flex;
    flex-direction: column;
    gap: 3px;
    flex-shrink: 0;
  }

  .drawer-grip span {
    display: block;
    width: 18px;
    height: 2px;
    background: rgba(255, 255, 255, 0.4);
    border-radius: 2px;
  }

  .drawer-meta {
    font-size: 0.8rem;
    color: #00ff88;
    letter-spacing: 0.04em;
    flex: 1;
    text-align: center;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }

  .drawer-cta {
    font-size: 0.75rem;
    color: rgba(255, 255, 255, 0.5);
    letter-spacing: 0.04em;
    flex-shrink: 0;
    transition: transform 0.3s ease;
    display: inline-block;
  }

  .drawer-cta.flipped {
    transform: rotate(180deg);
  }

  .drawer-body {
    max-height: 85vh;
    overflow-y: auto;
    padding: 0 1rem 1.5rem;
  }
</style>
