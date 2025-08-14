<template>
  <client-only>
    <div class="relative w-full h-screen m-0 p-0 overflow-hidden">
      <!-- canvas -->
      <canvas
        ref="canvasRef"
        id="drawing-board"
        class="absolute top-0 left-0 w-full h-full"
      ></canvas>

      <!-- toolbar  -->
      <div
        ref="toolbarRef"
        id="toolbar"
        class="fixed top-0 left-0 flex flex-col p-1.5 w-20 bg-black z-10 cursor-move select-none"
      >
        <h1 class="mb-1.5 text-white">Draw.</h1>
        <label class="mb-1.5 text-xs text-white" for="stroke">Stroke</label>
        <input class="mb-1.5 w-full bg-white" id="stroke" name="stroke" type="color" />
        <label class="mb-1.5 text-xs text-white" for="lineWidth">Line Width</label>
        <input
          class="mb-1.5 w-full bg-white"
          id="lineWidth"
          name="lineWidth"
          type="number"
          value="5"
        />
        <button class="mb-1.5 bg-blue-300 border-0 rounded-sm text-white p-0.5" id="clear">
          Clear
        </button>
      </div>
    </div>
  </client-only>
</template>

<script setup>
import { ref, onMounted, nextTick } from "vue";

const canvasRef = ref(null);
const toolbarRef = ref(null);

onMounted(async () => {
  await nextTick();

  const canvas = canvasRef.value;
  const toolbar = toolbarRef.value;
  const ctx = canvas.getContext("2d");

  //  fill viewport
  const resizeCanvas = () => {
    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;
  };
  resizeCanvas();
  window.addEventListener("resize", resizeCanvas);

  // do the drawing
  let isPainting = false;
  let lineWidth = 5;
  let startX, startY;

  function handleToolChange(e) {
    if (e.target.id === "stroke") {
      ctx.strokeStyle = e.target.value;
    }
    if (e.target.id === "lineWidth") {
      lineWidth = e.target.value;
    }
  }

  toolbar.addEventListener("input", handleToolChange);
  toolbar.addEventListener("change", handleToolChange);

  toolbar.addEventListener("click", (e) => {
    if (e.target.id === "clear") {
      ctx.clearRect(0, 0, canvas.width, canvas.height);
    }
  });

  const draw = (e) => {
    if (!isPainting) return;

    ctx.lineWidth = lineWidth;
    ctx.lineCap = "round";

    let clientX, clientY;
    if (e.touches) {
      clientX = e.touches[0].clientX;
      clientY = e.touches[0].clientY;
    } else {
      clientX = e.clientX;
      clientY = e.clientY;
    }

    ctx.lineTo(clientX, clientY);
    ctx.stroke();
  };

  const touching = (e) => {
    e.preventDefault();
    isPainting = true;

    if (e.touches) {
      startX = e.touches[0].clientX;
      startY = e.touches[0].clientY;
    } else {
      startX = e.clientX;
      startY = e.clientY;
    }

    ctx.beginPath();
    ctx.moveTo(startX, startY);
  };

  const nottouching = (e) => {
    e.preventDefault();
    isPainting = false;
    ctx.stroke();
    ctx.beginPath();
  };

  // mouse events
  canvas.addEventListener("mousedown", touching);
  canvas.addEventListener("mouseup", nottouching);
  canvas.addEventListener("mousemove", draw);

  // touch events
  canvas.addEventListener("touchstart", touching);
  canvas.addEventListener("touchend", nottouching);
  canvas.addEventListener("touchmove", draw);

  // prevent scrolling with touch (until resize is added..)
  const preventIfCanvas = (e) => {
    if (e.target === canvas) e.preventDefault();
  };
  document.body.addEventListener("touchstart", preventIfCanvas, { passive: false });
  document.body.addEventListener("touchend", preventIfCanvas, { passive: false });
  document.body.addEventListener("touchmove", preventIfCanvas, { passive: false });

  // dragging the toolbar
  let isDragging = false;
  let dragOffsetX = 0;
  let dragOffsetY = 0;

  const startDrag = (e) => {
    e.preventDefault();
    isDragging = true;

    const rect = toolbar.getBoundingClientRect();
    const clientX = e.touches ? e.touches[0].clientX : e.clientX;
    const clientY = e.touches ? e.touches[0].clientY : e.clientY;

    dragOffsetX = clientX - rect.left;
    dragOffsetY = clientY - rect.top;

    document.addEventListener("mousemove", dragMove);
    document.addEventListener("mouseup", endDrag);
    document.addEventListener("touchmove", dragMove, { passive: false });
    document.addEventListener("touchend", endDrag);
  };

  const dragMove = (e) => {
    if (!isDragging) return;

    const clientX = e.touches ? e.touches[0].clientX : e.clientX;
    const clientY = e.touches ? e.touches[0].clientY : e.clientY;

    toolbar.style.left = clientX - dragOffsetX + "px";
    toolbar.style.top = clientY - dragOffsetY + "px";
  };

  const endDrag = () => {
    isDragging = false;
    document.removeEventListener("mousemove", dragMove);
    document.removeEventListener("mouseup", endDrag);
    document.removeEventListener("touchmove", dragMove);
    document.removeEventListener("touchend", endDrag);
  };

  toolbar.addEventListener("mousedown", startDrag);
  toolbar.addEventListener("touchstart", startDrag, { passive: false });
});
</script>
