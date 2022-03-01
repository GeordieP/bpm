<script lang="ts">
  import { onMount } from "svelte";
  import Tapper from "./Tapper.svelte";

  // UTIL
  const fmtBPM = (bpm: number) => {
    const str = (Math.round(bpm * 100) / 100).toString();
    const [whole, decimal] = str.split(".");
    return {
      whole,
      decimal:
        decimal == null ? "00" : decimal.length < 2 ? "0" + decimal : decimal,
      asStr: str,
      asFloat: parseFloat(str),
    };
  };

  // CONSTANTS
  const ONE_MINUTE_MS = 1000 * 60;

  // MUTABLE STATE
  let field = "";
  let count = 0;
  let first = 0;
  let bpm = fmtBPM(0);

  // REACTIVE STATE
  $: active = count >= 2;
  $: halftime = fmtBPM(bpm.asFloat / 2);
  $: doubletime = fmtBPM(bpm.asFloat * 2);

  // EFFECTS
  const onReset = () => {
    field = "";
    count = 0;
    first = 0;
    bpm = fmtBPM(0);
    document.getElementById("tapInputField").focus();
  };

  // ACTIONS
  const onTap = () => {
    const now = Date.now();

    if (count === 0) {
      first = now;
      count++;
      return;
    }

    const tmp = (ONE_MINUTE_MS * count) / (now - first);
    bpm = fmtBPM(tmp);

    field = "";
    count++;
  };

  // LIFECYCLE
  onMount(onReset);
</script>

<main>
  <div class="vStack gap-sm">
    {#if count == 0}
      <button on:click={onTap}>tap a key to start</button>
    {:else if count == 1}
      <button on:click={onTap}>keep tapping</button>
    {:else}<button on:click={onTap} class="txt-dim">{count} taps</button>{/if}

    <Tapper id="tapInputField" on:tap={onTap} bind:value={field} />
    <button
      on:click={onReset}
      class:txt-dim={!active}
      class:accent={active}
    >RESET</button>

    <hr />

    <div class="vStack">
      <table class:txt-dim={!active}>
        <tr>
          <td>
            <h1><span class:accent={active}>BPM</span></h1>
          </td>
          <td class="text-right">
            <h1>{bpm.whole}<span class="txt-dim">.{bpm.decimal}</span></h1>
          </td>
        </tr>

        <tr>
          <td><span class:accent={active}>&divide; 2</span></td>
          <td class="text-right">
            <h3>
              {halftime.whole}<span class="txt-dim">.{halftime.decimal}</span>
            </h3>
          </td>
        </tr>

        <tr>
          <td><span class:accent={active}>&times; 2</span></td>
          <td class="text-right">
            <h3>
              {doubletime.whole}<span class="txt-dim">.{doubletime.decimal}</span>
            </h3>
          </td>
        </tr>
      </table>
    </div>

    <hr />
    <a
      href="https://github.com/GeordieP/bpm"
      target="_blank"
      class:txt-dim={active}
    >src code</a>
  </div>
</main>

<style>
  main {
    display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: column;
    height: 100%;
    width: 100%;
    margin: 0 auto;
    font-size: 2em;
  }
</style>
