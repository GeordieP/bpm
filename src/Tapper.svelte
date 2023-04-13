<script lang="ts">
  import { onMount } from "svelte";
  import { createEventDispatcher } from "svelte";
  const dispatch = createEventDispatcher();

  // PROPS
  export let id;
  export let value;

  // TYPES
  type Transform = { x: number; y: number; w: number; h: number };
  type BoundsCheck = { oob?: boolean };
  type Text = { text: string };
  type Graphic = { bgColor?: string; opacity?: number };
  type Box = Transform & BoundsCheck & Partial<Text> & Graphic;

  // CONSTANTS
  const BOX_W = 30;
  $: BOX_H = canvas == null ? 30 : canvas.height;

  // MUTABLE STATE
  let canvas: HTMLCanvasElement;
  let boxes: Box[] = [];

  // REACTIVE STATE
  $: STARTING_POS =
    canvas == null
      ? { x: 0, y: 0 }
      : { x: canvas.width + BOX_W / 2, y: canvas.height / 2 - BOX_H / 2 };

  // EFFECTS
  const spawnBox = (text?: string) =>
    boxes.push({
      ...STARTING_POS,
      w: BOX_W,
      h: BOX_H,
      text,
      opacity: 1,
    });

  // ACTIONS
  const onTap = (evt: any) => {
    spawnBox(evt?.currentTarget?.value ?? "");
    dispatch("tap", evt);
  };

  // LIFECYCLE

  onMount(function demoMode() {
    if (true) return; // Un-comment me
    const interval = setInterval(onTap, 344.4);
    return () => clearInterval(interval);
  });

  onMount(function startRenderLoop() {
    const ctx = canvas.getContext("2d");
    ctx.font = "63px Arial";
    let frame = requestAnimationFrame(loop);
    let lastTick = 0;

    function loop(tick) {
      const deltaTime = (tick - lastTick) / 1000;
      lastTick = tick;

      ctx.clearRect(0, 0, canvas.width, canvas.height);

      if (boxes.length > 0) {
        for (const box of boxes) {
          box.h -= 40 * deltaTime;
          box.y += 20 * deltaTime;
          box.x -= 800 * deltaTime;
          box.opacity == null ? null : (box.opacity -= 2.8 * deltaTime);

          if (box.x + BOX_W < -BOX_W) {
            box.oob = true;
            continue;
          }

          // if (box.text) {
          // ctx.fillText(box.text, box.x, box.y + 20);
          // } else {
          ctx.fillStyle = `rgba(255, 255, 255, ${
            box.opacity == null ? 1 : box.opacity
          })`;
          ctx.fillRect(box.x, box.y, box.w, box.h);
          // }
        }

        boxes = boxes.filter((x) => !x.oob);
      }

      frame = requestAnimationFrame(loop);
    }

    return () => {
      cancelAnimationFrame(frame);
    };
  });
</script>

<div class="vStack gap-sm">
  <slot appliedOnTap={onTap} />
  <div id="tapper-root" class="hStack gap-sm">
    <canvas bind:this={canvas} width={300} height={100} />
    <!-- `on:keyup` seems to give better bpm measurements, as long as the counting process is started correctly -->
    <input {id} type="text" on:keyup={onTap} bind:value autofocus />
  </div>
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
