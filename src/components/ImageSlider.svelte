<script>
  import { getButton, getOutermostParent } from "../lib/utils.js";
  import { onMount } from "svelte";

  export let slider;
  export let thumbsWrapper;
  export let previewWrapper;

  let thumbs;

  function makeSwap(img) {
    const swap = img.cloneNode();
    swap.style.position = "absolute";
    swap.style.top = 0;
    swap.style.left = 0;
    swap.style.zIndex = "-1";
    swap.ariaHidden = true;
    return swap;
  }

  let swap;
  let preview;
  function init(previewWrapper) {
    const previewCandidate = thumbs[0];
    const data = previewCandidate.dataset;

    thumbs.map((node, id) => {
      const data = node.dataset;
      node === previewCandidate ? (data.active = true) : (data.active = false);
      data.id = id;
    });

    const img = previewWrapper.querySelector("img");
    swap = makeSwap(img);
    img.dataset.id = data.id;

    preview = img;
    previewWrapper.appendChild(preview);
    previewWrapper.appendChild(swap);
  }

  function updateSwap(id, bool = false) {
    let fallbackID = 0;
    if (bool) fallbackID = thumbs.length - 1;
    bool ? id-- : id++;

    if (!thumbs[id]) id = fallbackID;
    swap.src = thumbs[id].dataset.src;
    preview.alt = thumbs[id].dataset.alt;
    preview.dataset.id = id;
  }

  let timeout = 0;
  function handlePreviewVFX(direction, inlineCSS, time = 0) {
    direction = direction.toLowerCase();
    const flag = (direction === "left") | (direction === "l");

    if (timeout === 0) {
      let id = parseInt(preview.dataset.id);
      updateSwap(id, flag);

      inlineCSS += `; transition: all ${time}ms`;
      preview.setAttribute("style", inlineCSS);
    }

    const _f = () => {
      preview.src = swap.src;
      preview.removeAttribute("style");
      timeout = 0;
    };

    if (timeout > 0) return;
    if (time === 0) return _f();
    timeout = setTimeout(_f, time);
  }

  function onClick(e) {
    const target = e.target;
    const button = getButton(target, e.currentTarget);

    if (button) {
      const action = button.dataset.action;
      if (!action) return;

      const direction = action === "next" ? "r" : "l";
      const x = action == "next" ? "-100%" : "100%";
      handlePreviewVFX(direction, `transform: translateX(${x})`, 500);
    }

    const slide = getOutermostParent(
      target,
      (node) => node.dataset && "slide" in node.dataset,
      thumbsWrapper
    );

    if (slide) {
      const state = slide.dataset.active === "true" ? true : false;
      if (state) return;

      const id = parseInt(slide.dataset.id);
      updateSwap(id - 1);
      preview.src = swap.src;

      const item = thumbs.filter((v) => v.dataset.active === "true")[0];
      item.dataset.active = false;
      slide.dataset.active = true;
      return;
    }
  }

  onMount(() => {
    const sel = "." + slider.classList[0];
    thumbs = Array.from(slider.querySelector(`${sel}__thumbs`).children);
    init(previewWrapper);
  });
</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<div class="image-slider" bind:this={slider} on:click={onClick}>
  <div class="image-slider__wrapper">
    <div class="image-slider__controls">
      <button class="button  button--round" data-action="previous">
        <span class="sr-only">Previous Image</span>
        <svg aria-hidden="true" width="15" height="18" viewBox="0 0 15 18">
          <use href="#svg_icon-previous" />
        </svg>
      </button>
      <button class="button button--round" data-action="next">
        <span class="sr-only">Next Image</span>
        <svg aria-hidden="true" width="13" height="18" viewBox="0 0 13 18">
          <use href="#svg_icon-next" />
        </svg>
      </button>
    </div>
    <div class="image-slider__preview" bind:this={previewWrapper}>
      <img
        src="./images/image-product-1.jpg"
        alt="Preview"
        width="425"
        height="425"
      />
    </div>
  </div>
  <div
    class="image-slider__thumbs"
    aria-hidden="true"
    bind:this={thumbsWrapper}
  >
    <div
      data-slide
      class="image-slider__item"
      data-src="./images/image-product-1.jpg"
      data-alt="123"
    >
      <img
        src="./images/image-product-1-thumbnail.jpg"
        alt=""
        width="64"
        height="64"
      />
    </div>
    <div
      data-slide
      class="image-slider__item"
      data-src="./images/image-product-2.jpg"
      data-alt="456"
    >
      <img
        src="./images/image-product-2-thumbnail.jpg"
        alt=""
        width="64"
        height="64"
      />
    </div>
    <div
      data-slide
      class="image-slider__item"
      data-src="./images/image-product-3.jpg"
      data-alt="789"
    >
      <img
        src="./images/image-product-3-thumbnail.jpg"
        alt=""
        width="64"
        height="64"
      />
    </div>
    <div
      data-slide
      class="image-slider__item"
      data-src="./images/image-product-4.jpg"
      data-alt="abc"
    >
      <img
        src="./images/image-product-4-thumbnail.jpg"
        alt=""
        width="64"
        height="64"
      />
    </div>
  </div>
</div>
