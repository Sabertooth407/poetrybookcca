<script>
  import { onMount, onDestroy } from "svelte";
  import { PageFlip } from "page-flip";
  import "page-flip/dist/page-flip.css";

  let flipbookEl;
  let pageFlip;

  const totalPages = 10;
  const images = Array.from({ length: totalPages }, (_, i) => `images/${i + 1}.png`);

  function createPages() {
    images.forEach((src, index) => {
      const pageDiv = document.createElement("div");
      pageDiv.className = "page";

      const img = document.createElement("img");
      img.src = src;
      img.alt = `Page ${index + 1}`;
      img.style.maxWidth = "100%";
      img.style.maxHeight = "100%";
      img.style.display = "block";

      pageDiv.appendChild(img);
      flipbookEl.appendChild(pageDiv);
    });
  }

  function addKeyboardControls() {
    const onKey = (e) => {
      if (!pageFlip) return;
      if (e.key === "ArrowRight") pageFlip.flipNext();
      if (e.key === "ArrowLeft") pageFlip.flipPrev();
    };
    window.addEventListener("keydown", onKey);
    return () => window.removeEventListener("keydown", onKey);
  }

  function addTouchControls() {
    let startX = 0;
    const threshold = 50;

    const onTouchStart = (e) => {
      startX = e.touches[0].clientX;
    };

    const onTouchEnd = (e) => {
      const endX = e.changedTouches[0].clientX;
      const diff = endX - startX;

      if (!pageFlip) return;

      if (diff > threshold) pageFlip.flipPrev();
      else if (diff < -threshold) pageFlip.flipNext();
    };

    flipbookEl.addEventListener("touchstart", onTouchStart);
    flipbookEl.addEventListener("touchend", onTouchEnd);

    return () => {
      flipbookEl.removeEventListener("touchstart", onTouchStart);
      flipbookEl.removeEventListener("touchend", onTouchEnd);
    };
  }

  onMount(() => {
    createPages();

    pageFlip = new PageFlip(flipbookEl, {
      width: 800,
      height: 1100,
      size: "stretch",
      minWidth: 300,
      minHeight: 400,
      maxShadowOpacity: 0.5,
      showCover: true,      
      useMouseEvents: true,
      drawShadow: true,
    });

    pageFlip.loadFromHTML(flipbookEl.querySelectorAll(".page"));

    const cleanupKeyboard = addKeyboardControls();
    const cleanupTouch = addTouchControls();

    onDestroy(() => {
      cleanupKeyboard();
      cleanupTouch();
    });
  });
</script>

<style>
  #flipbook {
    width: 100vw;
    height: 100vh;
    display: flex;
    justify-content: center;
    align-items: center;
    background: #222;
    overflow: hidden;
  }

  .page {
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .page img {
    width: 100%;
    height: 100%;
    object-fit: contain; 
    user-select: none;
    pointer-events: none;
  }
</style>

<div id="flipbook" bind:this={flipbookEl}></div>
