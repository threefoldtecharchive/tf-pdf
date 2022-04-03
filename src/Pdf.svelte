<svelte:options tag={null} />

<script lang="ts">
  import { onMount } from "svelte";

  export let url: string;

  let canvas: HTMLCanvasElement;
  let pageNo: number;
  let width: number;
  let height: number;
  let n: number = 1;
  let loading: boolean = false;

  let _render: any;

  onMount(() => {
    let ctx = canvas.getContext("2d");
    let page: any;
    let viewport: any;

    loading = true;
    pdfjsLib
      .getDocument(url)
      .promise.then((doc) => {
        pageNo = doc.numPages;
        return doc;
      })
      .then((doc: any) => () => {
        loading = true;
        return doc
          .getPage(n)
          .then((_page) => (page = _page))
          .then(() => page.getViewport({ scale: 1 }))
          .then((_viewport) => (viewport = _viewport))
          .then(() => {
            width = viewport.width;
            height = viewport.height;
            return page.render({
              canvasContext: ctx,
              viewport,
            });
          })
          .finally(() => {
            loading = false;
          });
      })
      .then((x) => {
        _render = x;
        return _render();
      })
      .catch((err) => {
        console.log("Error", err);
      });
  });

  $: hidden = !canvas || !width || !height || !pageNo;

  function prev() {
    n--;
    _render();
  }

  function next() {
    n++;
    _render();
  }
</script>

<p>
  loading: {loading} - {pageNo}
</p>
<div {hidden}>
  <button on:click={prev} disabled={loading || n === 1}>Prev</button>
  <button on:click={next} disabled={loading || pageNo === n}>Next</button>
  <canvas bind:this={canvas} {width} {height} />
</div>
