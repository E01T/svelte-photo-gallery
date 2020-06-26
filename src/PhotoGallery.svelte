<script>
  import { crossfade, scale } from "svelte/transition";
  import images from "./images.js";

  const [send, receive] = crossfade({
    duration: 700,
    fallback: scale,
  });

  let selected = null;

  const load = image => {
    const img = new Image();

    img.onload = () => (selected = image);

    img.src = image.src;
  };

  function handleKeyDown(e) {
    const key = e.key;
    const image = document.querySelector("[id^='img']");
    const imgLastChar = image ? parseInt(image.id.split("-")[1]) : null;

    if (key === "Escape" && selected !== null) {
      // get the current opened image id,
      // find the equivalent button and focus on it

      // const imgLastChar = parseInt(image.id.split("-")[1]);

      selected = null;

      let timeout = setTimeout(() => {
        const activeBtn = document.querySelector(
          "[id='btn-" + imgLastChar + "']"
        );
        activeBtn.focus();
        clearTimeout(timeout);
      }, 700);
    }

    if (key === "ArrowRight" && selected !== null) {
      // get the last characher(s) of the image.id (e.g img-1 -> 1)
      // and turn it into an integer
      // const imgLastChar = parseInt(image.id.split("-")[1]);

      if (imgLastChar >= images.length) {
        const index = 0;
        load(images[index]);
      } else {
        // image last character is also the array index
        // note that array starts at zero put the first id is one
        const index = imgLastChar;
        load(images[index]);
      }
    }

    if (key === "ArrowLeft" && selected !== null) {
      // const imgLastChar = parseInt(image.id.split("-")[1]);

      if (imgLastChar === 1) {
        const index = images.length - 1;
        load(images[index]);
      } else {
        // image last character is also the array index
        // NOTE: array starts at zero but the first id is one
        // I take the imgLastChar, which is one more already
        // and I subtrack 2 to get the right index
        const index = imgLastChar - 2;
        load(images[index]);
      }
    }
  }
</script>

<svelte:body on:keydown="{handleKeyDown}" />

<div class="container">
  <div class="phone">
    <h1>Photo gallery</h1>
    <div class="grid">
      {#each images as image}
      <div class="square">
        {#if selected !== image}
        <button
          id="btn-{image.id}"
          style="background: lightblue url({image.src}) center/cover;"
          on:click="{() => load(image)}"
          in:receive="{{key:image.id}}"
          out:send="{{key:image.id}}"
        ></button>
        {/if}
      </div>
      {/each}
    </div>

    {#if selected} {#await selected then d}
    <div
      role="img"
      class="photo"
      in:receive="{{key:d.id}}"
      out:send="{{key:d.id}}"
    >
      <img
        id="img-{d.id}"
        alt="{d.alt}"
        src="{d.src}"
        on:click="{() => selected = null}"
      />
    </div>
    {/await} {/if}
  </div>
</div>

<style>
  .container {
    position: absolute;
    display: flex;
    align-items: center;
    justify-content: center;
    width: 100%;
    height: 100%;
    top: 0;
    left: 0;
  }

  .phone {
    position: relative;
    display: flex;
    flex-direction: column;
    width: 52vmin;
    height: 76vmin;
    border: 2vmin solid #ccc;
    border-bottom-width: 10vmin;
    padding: 3vmin;
    border-radius: 2vmin;
  }

  h1 {
    font-weight: 300;
    text-transform: uppercase;
    font-size: 5vmin;
    margin: 0.2em 0 0.5em 0;
  }

  .grid {
    flex: 1;
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    grid-template-rows: repeat(4, 1fr);
    grid-gap: 2vmin;
  }

  button {
    width: 100%;
    height: 100%;
    color: white;
    font-size: 5vmin;
    border: none;
    margin: 0;
    will-change: transform;
    transition: all 0.6s;
  }

  button:focus,
  button:hover {
    box-shadow: 0px 0px 5px 10px #ccc;
  }

  .photo,
  img {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    overflow: hidden;
  }

  .photo {
    display: flex;
    align-items: stretch;
    justify-content: flex-end;
    flex-direction: column;
    will-change: transform;
  }

  img {
    object-fit: cover;
    cursor: pointer;
  }
</style>
