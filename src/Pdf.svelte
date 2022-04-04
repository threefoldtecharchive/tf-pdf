<svelte:options tag={null} />

<script lang="ts">
  import { onMount } from "svelte";

  export let url: string;

  // Icons
  const prev_icon =
    "https://fonts.gstatic.com/s/i/materialiconsround/arrow_back/v13/24px.svg";
  const next_icon =
    "https://fonts.gstatic.com/s/i/materialiconsround/arrow_forward/v13/24px.svg";
  const full_icon =
    "https://fonts.gstatic.com/s/i/materialiconsround/fullscreen/v12/24px.svg";

  // let canvas: HTMLCanvasElement;
  let canvas_holder: HTMLElement;
  let pageNo: number;
  let height: number;
  let width: number;
  let n: number = 1;
  let loading: boolean = false;
  let style: string = "";

  onMount(() => {
    function renderPage(page) {
      var viewport = page.getViewport({ scale: 1 });
      var wrapper = document.createElement("div");
      wrapper.className = "canvas-wrapper";
      var canvas = document.createElement("canvas");
      var ctx = canvas.getContext("2d");
      var renderContext = {
        canvasContext: ctx,
        viewport: viewport,
      };

      height = canvas.height = viewport.height;
      width = canvas.width = viewport.width;
      style =
        "--page-height: " +
        viewport.height +
        "px;" +
        "--page-width: " +
        viewport.width +
        "px;";
      wrapper.appendChild(canvas);
      canvas_holder.appendChild(wrapper);
      page.render(renderContext);
    }

    function renderPages(pdfDoc) {
      for (var num = 1; num <= pdfDoc.numPages; num++)
        pdfDoc
          .getPage(num)
          .then(renderPage)
          .finally(() => {
            loading = false;
          });
    }

    loading = true;
    pdfjsLib.disableWorker = true;
    pdfjsLib.getDocument(url).promise.then((doc) => {
      pageNo = doc.numPages;
      renderPages(doc);
    });
  });

  function updateCurrentPage() {
    n = Math.round(canvas_holder.scrollTop / height + 1);
  }

  function prev() {
    let prev_page_at = (n - 2) * height;
    canvas_holder.scrollTo(0, prev_page_at);
    n--;
  }

  function next() {
    let next_page_at = n * height;
    canvas_holder.scrollTo(0, next_page_at);
    n++;
  }

  function full() {
    canvas_holder.requestFullscreen();
  }
</script>

{#if loading}
  <p class="loading">loading ...</p>
{/if}
<div class="pdf-viewer" style={"--page-width: " + width + "px"}>
  {#if pageNo}
    <div class="control">
      <button on:click={prev} disabled={loading || n === 1}>
        <img
          src={prev_icon}
          alt="previous page"
          title="previous page"
          width="24px"
        />
      </button>
      <p>page {n} from {pageNo}</p>
      <button on:click={next} disabled={loading || pageNo === n}>
        <img src={next_icon} alt="next page" title="next page" width="24px" />
      </button>
      <button on:click={full} disabled={loading}>
        <img src={full_icon} alt="full page" title="full page" width="24px" />
      </button>
    </div>
  {/if}
  <div
    class="canvas_holder"
    bind:this={canvas_holder}
    {style}
    on:scroll={updateCurrentPage}
  />
</div>

<style>
  .loading {
    text-align: center;
  }

  .canvas_holder {
    position: relative;
    right: 0;
    left: 0;
    margin-right: auto;
    margin-left: auto;
    height: var(--page-height);
    width: var(--page-width);
    justify-content: center;
    align-items: center;
    text-align: center;
    overflow-x: hidden;
    overflow-y: auto;
    scroll-behavior: smooth;
  }

  .control {
    position: relative;
    right: 0;
    left: 0;
    margin-right: auto;
    margin-left: auto;
    width: var(--page-width);
    height: 30px;
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .control p {
    margin: 10px;
  }

  .control button {
    border: none;
    background-color: transparent;
    margin: 10px;
    cursor: pointer;
  }
</style>
