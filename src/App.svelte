<script lang="ts">
  const ONE_MINUTE_MS = 1000 * 60;

  let field = "";
  let count = 0;
  let first = 0;
  let bpm = "000.00";
  $: [whole, dec] = bpm.split(".");

  const tap = () => {
    const now = Date.now();

    if (count === 0) {
      first = now;
      count++;
      return;
    }

    field = "";
    count++;

    const tmp = (ONE_MINUTE_MS * count) / (now - first);
    bpm = (Math.round(tmp * 100) / 100).toString();
  };
</script>

<style>
  main {
    display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: column;
    height: 100%;
    width: 100%;
    margin: 0 auto;
  }

  .hStack {
    display: flex;
    flex-direction: row;
  }

  .vStack {
    display: flex;
    flex-direction: column;
  }

  .bpm {
    font-size: 124px;
  }

  .dec {
    color: gray;
    font-size: 0.3em;
    font-weight: normal;
  }

  .accent {
    color: dodgerblue;
    font-size: 0.2em;
    text-align: right;
    width: 100%;
  }

  .center {
    justify-content: center;
    align-items: center;
  }
</style>

<main>
  <div class="vStack">
    <h1 class="bpm hStack">
      <b>{whole}</b>
      <div class="vStack center">
        <span class="accent">BPM</span>
        <span class="dec">.{dec || '00'}</span>
      </div>
    </h1>
    <div class="hStack">
      <input
        type="text"
        on:keydown={tap}
        bind:value={field}
        placeholder="Tap btns on your keyboard"
        autofocus />
      <button on:click={tap}>{count}</button>
    </div>
    <a href="https://github.com/GeordieP/bpm">src</a>
  </div>
</main>
