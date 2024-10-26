<script lang="ts">
    import { add } from "@scope/common/models";

    /******** VARIABLE INSTANTIATION  *********/
    let imageSrcs: string[] = [];
    let galleryContainer: HTMLDivElement | null = null;
    let fileInput: HTMLInputElement | null = null;
    let playerStatus: "play" | "pause" = "pause";
    let scrollInterval: ReturnType<typeof setInterval> | null = null;
    let scrollSpeed: number = 8;
  
    /******** FILE UPLOAD MANAGEMENT  *********/
    function handleDrop(event: DragEvent) {
      event.preventDefault();
      const files = event.dataTransfer?.files;
      if (files) {
        Array.from(files).forEach((file) => {
          if (file.type.startsWith("image/")) {
            const reader = new FileReader();
            reader.onload = () => {
              imageSrcs = [...imageSrcs, reader.result as string];
            };
            reader.readAsDataURL(file);
          }
        });
      }
    }
  
    function handleDragOver(event: DragEvent) {
      event.preventDefault();
    }
  
    function handleClickUpload() {
      fileInput?.click();
    }
  
    function handleFileChange(event: Event) {
      const files = (event.target as HTMLInputElement).files;
      if (files) {
        Array.from(files).forEach((file) => {
          if (file.type.startsWith("image/")) {
            const reader = new FileReader();
            reader.onload = () => {
              imageSrcs = [...imageSrcs, reader.result as string];
            };
            reader.readAsDataURL(file);
          }
        });
      }
    }
  
    /********   PREVENT ZOOM FUNCTION   *******/
    let lastTouchEnd: number = 0;
    function preventZoomOnDoubleTap(event: TouchEvent) {
      const now = new Date().getTime();
      const timeSinceLastTouch = now - lastTouchEnd;
      if (timeSinceLastTouch <= 300) {
        event.preventDefault();
      }
  
      lastTouchEnd = now;
    }
  
    /********  PLAY PAUSE SCROLLING   *********/
    function handlePlayPause() {
      if (playerStatus === "pause") {
        play();
      } else {
        pause();
      }
    }
  
    function scrollGallery() {
      if (galleryContainer) {
        galleryContainer.scrollTop += 1;
  
        // Reset scroll to top when reaching the end
        if (
          galleryContainer.scrollTop + galleryContainer.clientHeight >=
          galleryContainer.scrollHeight
        ) {
          galleryContainer.scrollTop = 0;
        }
      }
    }
  
    function play() {
      playerStatus = "play";
      scrollInterval = setInterval(scrollGallery, 700 / scrollSpeed);
    }
  
    function pause() {
      playerStatus = "pause";
      if (scrollInterval) clearInterval(scrollInterval);
    }
  
    /********  REACTIVE STATEMENT FOR SCROLL SPEED  *********/
    $: {
      if (playerStatus === "play") {
        if (scrollInterval) clearInterval(scrollInterval);
        scrollInterval = setInterval(scrollGallery, 700 / scrollSpeed);
      }
    }
  </script>
  
  <svelte:window on:touchend={preventZoomOnDoubleTap} />
  <span>Ignore this number: {add(3,7)}</span>
  
  <div class="flex items-center justify-center">
    <button
      class="m-3 p-3 w-full items-center text-center cursor-pointer border-2 border-gray-300 border-dashed"
      aria-label="Drop images here or click to upload"
      tabindex="0"
      on:drop={handleDrop}
      on:dragover={handleDragOver}
      on:click={handleClickUpload}
    >
      Upload images
    </button>
    <input
      type="file"
      accept="image/*"
      class="hidden"
      bind:this={fileInput}
      multiple
      on:change={handleFileChange}
    />
  </div>
  <div class="flex items-center w-full justify-center space-x-3">
    <button
      class="material-symbols-outlined icon-hover"
      on:click={handlePlayPause}
      >{playerStatus === "play" ? "pause_circle" : "play_circle"}</button
    >
    <div
      class="flex items-center space-x-2 bg-gray-200 border border-gray-300 rounded-md"
    >
      <button on:click={() => scrollSpeed--} class="material-symbols-outlined">
        keyboard_arrow_down
      </button>
      <span>Tempo: {scrollSpeed}</span>
      <button on:click={() => scrollSpeed++} class="material-symbols-outlined">
        keyboard_arrow_up
      </button>
    </div>
  </div>
  
  <!-- THE IMAGE GALLERY-->
  <div
    class="h-[500px] m-3 border border-black overflow-scroll"
    bind:this={galleryContainer}
  >
    <div class="flex flex-col overflow-auto">
      {#each imageSrcs as imageSrc}
        <img src={imageSrc} alt="uploaded" class="image" />
      {/each}
    </div>
  </div>
  
  <style>
    .image {
      max-width: 100%;
      height: auto;
    }
  
    .icon-hover {
      font-size: 48px;
    }
  </style>
  