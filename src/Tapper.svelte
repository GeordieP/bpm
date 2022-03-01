<script lang="ts">
  import { onMount } from "svelte";
  import { createEventDispatcher } from "svelte";

  const dispatch = createEventDispatcher();

  export let id;
  export let value;

  type Transform = { x: number; y: number; w: number; h: number };
  type BoundsCheck = { oob?: boolean };
  type Text = { text: string };
  type Box = Transform & BoundsCheck & Partial<Text>;

  let root: HTMLDivElement;
  let canvas: HTMLCanvasElement;
  let boxes: Box[] = [];

  const BOX_SIZE = 30;
  $: STARTING_POS =
    canvas == null
      ? { x: 0, y: 0 }
      : { x: canvas.width + BOX_SIZE / 2, y: canvas.height / 2 - BOX_SIZE / 2 };
  const spawnBox = (text?: string) =>
    boxes.push({ ...STARTING_POS, w: BOX_SIZE, h: BOX_SIZE, text });

  onMount(() => {
    const ctx = canvas.getContext("2d");
    ctx.font = "63px Arial";
    let frame = requestAnimationFrame(loop);
    let lastTick = 0;

    function loop(tick) {
      frame = requestAnimationFrame(loop);

      const deltaTime = (tick - lastTick) / 1000;
      lastTick = tick;

      ctx.clearRect(0, 0, canvas.width, canvas.height);
      ctx.fillStyle = "white";

      if (boxes.length > 0) {
        for (const box of boxes) {
          box.h -= 40 * deltaTime;
          box.y += 20 * deltaTime;
          box.x -= 800 * deltaTime;

          if (box.x + BOX_SIZE < -BOX_SIZE) {
            box.oob = true;
            continue;
          }

          // if (box.text) {
          //   ctx.fillText(box.text, box.x, box.y + 20);
          // } else {
          ctx.fillRect(box.x, box.y, box.w, box.h);
          // }
        }

        boxes = boxes.filter((x) => !x.oob);
      }
    }

    return () => {
      cancelAnimationFrame(frame);
    };
  });

  const onTap = (evt: any) => {
    spawnBox(evt.currentTarget.value);
    dispatch("tap", evt);
  };

  // UTIL

  const parseCssPx = (value: string | number) => {
    console.log(value);
    return 40 as number;
  };
</script>

<div id="tapper-root" class="hStack gap-sm" bind:this={root}>
  <canvas bind:this={canvas} width={300} height={100} />
  <input {id} type="text" on:keydown={onTap} bind:value autofocus />
</div>

<style>
  #tapper-root {
    --canvas-width: 300px;
    --canvas-height: 100px;
  }

  canvas {
    width: var(--canvas-width);
    height: var(--canvas-height);
    background-color: #222426;
  }

  input {
    width: var(--canvas-height);
    height: var(--canvas-height);
  }
</style>
