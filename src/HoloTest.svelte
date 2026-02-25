<script>
  import Card from "./lib/components/CardProxy.svelte";
  import { createEventDispatcher } from "svelte";

  const dispatch = createEventDispatcher();

  let imageUrl = "";
  let appliedUrl = "";
  let darkBg = true;
  let fileObjectUrl = null;

  // placeholder: solid gray data-URI
  const placeholder = "data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='660' height='921' viewBox='0 0 660 921'%3E%3Crect fill='%23708090' width='660' height='921' rx='24'/%3E%3Ctext x='330' y='460' text-anchor='middle' fill='white' font-size='48' font-family='sans-serif'%3ETest Card%3C/text%3E%3C/svg%3E";

  $: cardImg = appliedUrl || placeholder;

  function applyImage() {
    const v = imageUrl.trim();
    if (!v) {
      appliedUrl = "";
      return;
    }
    // If user pasted raw SVG markup, convert to a safe data URI
    if (v.indexOf("<svg") !== -1) {
      appliedUrl = "data:image/svg+xml;utf8," + encodeURIComponent(v);
      return;
    }
    // If they pasted a data:image/svg+xml but it's unencoded, encode the payload
    if (v.startsWith("data:image/svg+xml") && v.indexOf("<") !== -1) {
      const comma = v.indexOf(",");
      if (comma !== -1) {
        const header = v.slice(0, comma + 1);
        const rest = v.slice(comma + 1);
        appliedUrl = header + encodeURIComponent(rest);
        return;
      }
    }
    appliedUrl = v;
  }

  function onFileChange(e) {
    const f = e.target.files && e.target.files[0];
    if (!f) return;
    if (fileObjectUrl) URL.revokeObjectURL(fileObjectUrl);
    fileObjectUrl = URL.createObjectURL(f);
    appliedUrl = fileObjectUrl;
    imageUrl = fileObjectUrl;
  }

  function clearImage() {
    appliedUrl = "";
    imageUrl = "";
    if (fileObjectUrl) {
      URL.revokeObjectURL(fileObjectUrl);
      fileObjectUrl = null;
    }
  }

  function handleKey(e) {
    if (e.key === "Enter") applyImage();
  }

  /**
   * Each entry defines the props that differ per rarity variant.
   * Every card shares the same `img`, and disables CDN foil via foil={false} mask={false}.
   */
  const cards = [
    {
      label: "Common",
      props: { rarity: "Common", number: "001", supertype: "pokémon", subtypes: "basic" }
    },
    {
      label: "Reverse Holo",
      props: { rarity: "Common", number: "002", supertype: "pokémon", subtypes: "basic", isReverse: true }
    },
    {
      label: "Rare Holo",
      props: { rarity: "Rare Holo", number: "003", set: "swsh1", supertype: "pokémon", subtypes: "basic" }
    },
    {
      label: "Galaxy / Cosmos",
      props: { rarity: "Rare Holo Cosmos", number: "004", set: "swsh1", supertype: "pokémon", subtypes: "basic" }
    },
    {
      label: "Radiant",
      props: { rarity: "Radiant Rare", number: "005", set: "swsh1", supertype: "pokémon", subtypes: "Radiant" }
    },
    {
      label: "Trainer Gallery",
      props: { rarity: "Rare Holo", number: "tg001", set: "swsh1", supertype: "pokémon", subtypes: "basic" }
    },
    {
      label: "Pokemon V",
      props: { rarity: "Rare Holo V", number: "006", set: "swsh1", supertype: "pokémon", subtypes: "V" }
    },
    {
      label: "Pokemon V Full Art",
      props: { rarity: "Rare Ultra", number: "007", set: "swsh1", supertype: "pokémon", subtypes: "V" }
    },
    {
      label: "Pokemon V Alt Art",
      props: { rarity: "Rare Ultra", number: "008", set: "swsh1", supertype: "pokémon", subtypes: "V" }
    },
    {
      label: "VMAX",
      props: { rarity: "Rare Holo VMAX", number: "009", set: "swsh1", supertype: "pokémon", subtypes: "VMAX" }
    },
    {
      label: "VMAX Alt / Rainbow",
      props: { rarity: "Rare Rainbow", number: "010", set: "swsh1", supertype: "pokémon", subtypes: "VMAX" }
    },
    {
      label: "VSTAR",
      props: { rarity: "Rare Holo VSTAR", number: "011", set: "swsh1", supertype: "pokémon", subtypes: "VSTAR" }
    },
    {
      label: "Trainer Full Art",
      props: { rarity: "Rare Ultra", number: "012", set: "swsh1", supertype: "trainer", subtypes: "Supporter" }
    },
    {
      label: "Rainbow Rare",
      props: { rarity: "Rare Rainbow", number: "013", set: "swsh1", supertype: "pokémon", subtypes: "VSTAR" }
    },
    {
      label: "Secret Rare (Gold)",
      props: { rarity: "Rare Secret", number: "014", set: "swsh1", supertype: "pokémon", subtypes: "basic" }
    },
    {
      label: "Shiny Vault V",
      props: { rarity: "Rare Shiny V", number: "sv001", set: "swsh1", supertype: "pokémon", subtypes: "V" }
    },
    {
      label: "Amazing Rare",
      props: { rarity: "Amazing Rare", number: "015", set: "swsh1", supertype: "pokémon", subtypes: "basic" }
    },
  ];
</script>

<div class="holo-test" class:dark={darkBg} class:light={!darkBg}>

  <header class="holo-header">
    <div class="holo-nav">
      <button class="back-btn" on:click={() => dispatch("back")}>← Back</button>
      <h1>Holo Effect Showcase</h1>
      <button class="theme-toggle" on:click={() => darkBg = !darkBg}>
        {darkBg ? "☀️ Light" : "🌙 Dark"}
      </button>
    </div>

    <div class="image-input">
      <input
        type="text"
        bind:value={imageUrl}
        on:keydown={handleKey}
        placeholder="Paste card image URL…"
      />
      <button on:click={applyImage}>Apply</button>
      <input type="file" accept="image/*" on:change={onFileChange} />
      <button on:click={clearImage}>Clear</button>
    </div>
    {#if appliedUrl}
      <p class="caption">Using: {appliedUrl}</p>
    {:else}
      <p class="caption">Using placeholder test image</p>
    {/if}
  </header>

  <section class="holo-grid">
      {#each cards as { label, props } (label)}
        <div class="holo-cell">
          <Card
            id={props.id || `test-${label.toLowerCase().replace(/\s+/g, "-")}`}
            name={label}
            number={props.number}
            set={props.set || ""}
            types={props.types || "Colorless"}
            supertype={props.supertype}
            subtypes={props.subtypes}
            rarity={props.rarity}
            isReverse={props.isReverse || false}
            img={cardImg}
            foil={false}
            mask={false}
          />
          <span class="holo-label">{label}</span>
        </div>
      {/each}
  </section>
</div>

<style>
  .holo-test {
    min-height: 100vh;
    padding: 24px;
    transition: background 0.3s ease, color 0.3s ease;
  }
  .holo-test.dark {
    background: #1a1a2e;
    color: #e0e0e0;
  }
  .holo-test.light {
    background: #f0f0f0;
    color: #222;
  }

  .holo-header {
    max-width: 1200px;
    margin: 0 auto 32px;
    text-align: center;
  }

  .holo-nav {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 16px;
    flex-wrap: wrap;
    margin-bottom: 16px;
  }
  .holo-nav h1 {
    margin: 0;
    font-size: 1.6rem;
  }

  .back-btn,
  .theme-toggle {
    background: rgba(255,255,255,0.12);
    border: 1px solid rgba(255,255,255,0.2);
    color: inherit;
    padding: 8px 16px;
    border-radius: 8px;
    cursor: pointer;
    font-size: 0.9rem;
    transition: background 0.2s;
  }
  .light .back-btn,
  .light .theme-toggle {
    background: rgba(0,0,0,0.06);
    border-color: rgba(0,0,0,0.15);
  }
  .back-btn:hover,
  .theme-toggle:hover {
    background: rgba(255,255,255,0.22);
  }
  .light .back-btn:hover,
  .light .theme-toggle:hover {
    background: rgba(0,0,0,0.12);
  }

  .image-input {
    display: flex;
    gap: 8px;
    justify-content: center;
    flex-wrap: wrap;
  }
  .image-input input {
    width: min(100%, 460px);
    padding: 10px 14px;
    border-radius: 8px;
    border: 1px solid rgba(255,255,255,0.2);
    background: rgba(255,255,255,0.08);
    color: inherit;
    font-size: 0.95rem;
  }
  .light .image-input input {
    background: #fff;
    border-color: #ccc;
  }
  .image-input button {
    padding: 10px 20px;
    border-radius: 8px;
    border: none;
    background: #6366f1;
    color: #fff;
    font-weight: 600;
    cursor: pointer;
    transition: background 0.2s;
  }
  .image-input button:hover {
    background: #4f46e5;
  }

  .caption {
    font-size: 0.8rem;
    opacity: 0.6;
    margin-top: 6px;
    word-break: break-all;
  }

  .holo-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(260px, 1fr));
    gap: 48px 24px;
    max-width: 1400px;
    margin: 0 auto;
    padding: 16px;
    transform-style: preserve-3d;
  }

  .holo-cell {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 12px;
  }

  .holo-label {
    font-size: 0.85rem;
    font-weight: 600;
    letter-spacing: 0.03em;
    text-transform: uppercase;
    opacity: 0.8;
    padding: 4px 12px;
    border-radius: 6px;
    background: rgba(255,255,255,0.06);
  }
  .light .holo-label {
    background: rgba(0,0,0,0.05);
  }
</style>
