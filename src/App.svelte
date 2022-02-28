<script lang="ts">
  import { onMount } from "svelte";
  const ONE_MINUTE_MS = 1000 * 60;

  const reset = () => {
    field = "";
    count = 0;
    first = 0;
    bpm = fmtBPM(0);
    document.getElementById("tapInputField").focus();
  };
  onMount(reset);

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

  const tap = () => {
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
  }; ///tap

  // STATE
  let field = "";
  let count = 0;
  let first = 0;
  let bpm = fmtBPM(0);

  $: active = count >= 2;
  $: halftime = fmtBPM(bpm.asFloat / 2);
  $: doubletime = fmtBPM(bpm.asFloat * 2);
</script>

<main>
  <div class="vStack gap-sm">
    {#if count == 0}
      <button on:click={tap} class="accent">tap a key to start</button>
    {:else if count == 1}
      <button on:click={tap} class="accent">keep tapping</button>
    {:else}
      <button on:click={tap} class="txt-dim">{count} taps</button>
    {/if}

    <input
      id="tapInputField"
      type="text"
      on:keydown={tap}
      bind:value={field}
      autofocus
    />
    <button on:click={reset} class:txt-dim={!active} class:accent={active}
      >RESET</button
    >

    <hr />

    <div class="vStack">
      <table class:txt-dim={!active}>
        <tr>
          <td>
            <h1>
              <span class:accent={active}>BPM</span>
            </h1>
          </td>
          <td class="text-right">
            <h1>{bpm.whole}<span class="txt-dim">.{bpm.decimal}</span></h1>
          </td>
        </tr>

        <tr>
          <td>
            <span class:accent={active}>&divide; 2</span>
          </td>
          <td class="text-right">
            <h3>
              {halftime.whole}<span class="txt-dim">.{halftime.decimal}</span>
            </h3>
          </td>
        </tr>

        <tr>
          <td>
            <span class:accent={active}>&times; 2</span>
          </td>
          <td class="text-right">
            <h3>
              {doubletime.whole}<span class="txt-dim"
                >.{doubletime.decimal}</span
              >
            </h3>
          </td>
        </tr>
      </table>
    </div>

    <hr />
    <a
      href="https://github.com/GeordieP/bpm"
      target="_blank"
      class:txt-dim={active}>src code</a
    >
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

  button {
    cursor: pointer;
  }

  button,
  input {
    margin: 0;
    color: white;
    background-color: rgba(0, 0, 0, 0.2);
    border: 1px solid black;
  }
  input {
    background-color: rgba(0, 0, 0, 0.1);
    border: 1px solid rgba(40, 150, 255, 0.3);
  }

  h1,
  h3 {
    margin: 0;
  }

  hr {
    width: 100%;
    border-color: #333;
  }

  /*  */
  .hStack {
    display: flex;
    flex-direction: row;
  }

  .vStack {
    display: flex;
    flex-direction: column;
  }

  .gap-sm {
    gap: 0.2em;
  }

  .center {
    justify-content: center;
    align-items: center;
  }

  .txt-dim {
    color: rgba(255, 255, 255, 0.3);
  }

  .spaceBetween {
    justify-content: space-between;
  }

  .text-right {
    text-align: right;
  }

  /*  */
  .accent {
    color: dodgerblue;
    width: 100%;
  }
</style>
