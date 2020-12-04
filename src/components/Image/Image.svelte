<script>
  import Waypoint from "svelte-waypoint";
  import { fade } from "svelte/transition";
  import { Dialog } from '../Dialog'

  export let alt = "";
  export let width = "";
  export let height = "";
  export let src = "";
  export let thumbnail = "";
  export let title = "Click to zoom in.";

  let showDialog = false;

  let loaded = false;
  let loading = false;

  function load() {
    const img = new Image();
    img.src = src;
    loading = true;

    img.onload = () => {
      loading = false;
      loaded = true;
    };
  }

  function openInNewWindow() {
    window.open(src);
    showDialog = false;
  }
</script>

<Waypoint class={$$props.class} once on:enter={load} style="height: {height}px" offset="0">
  {#if loaded}
    <img class={$$props.class} style="cursor: pointer" on:click={() => showDialog = true} {title} {src} {alt} {width} {height} />
  {:else if thumbnail}
    <img class={$$props.class} style="cursor: pointer" on:click={() => showDialog = true} {title} src={thumbnail} {alt} {width} {height} />
  {:else if loading}
    <slot name="loading" />
  {/if}
</Waypoint>

<Dialog bind:value={showDialog}>
  <div class="text-gray-700"><img class={$$props.class} {src} /></div>
  <div slot="actions">
    <Button text small color="secondary" on:click={openInNewWindow}>Open in new window</Button>
    <Button text small on:click={() => showDialog = false}>Close</Button>
  </div>
</Dialog>
