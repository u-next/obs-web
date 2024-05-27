<script>
  // Get from App.svelte
  export let overlaySystemUrl
  export let overlaySystemPreviewPrefix
  export let overlayId

  let previewIdArray = [1,2,3,4,5,6,7,8]
</script>

{#each previewIdArray as previewId}
  <div class="previewFrame">
    <iframe
      title='Overlay {previewId}'
      src="{overlaySystemUrl}{overlayId}{overlaySystemPreviewPrefix}{previewId}"
      width="1920"
      height="1080"
      style="transform: scale(0.20313); transform-origin: top left;"
    ></iframe>
  </div>
{/each}

<style>
  .previewFrame {
    width: 390px;
    height: 221px;
    overflow: hidden;
    border: 0.25rem solid;
    box-sizing: border-box; 
    display: inline-block; 
  }
</style>