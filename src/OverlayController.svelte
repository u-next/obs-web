<script>
  import { onMount } from 'svelte'
  import IdSetupMenu from './IdSetupMenu.svelte';

  const overlaySystemPrefix = 'https://tatooine-e1db8.web.app/'
  const overlaySystemControllerPrefix = '/controller'
  const overlaySystemPreviewPrefix = '/presets'
  export let overlayId
  let overlayPreviewUrl
  let overlayControllerUrl
  let isOverlayIdActive

  onMount(async () => {
    if (overlayId) {
      overlayControllerUrl = `${overlaySystemPrefix}${overlayId}${overlaySystemControllerPrefix}`
      overlayPreviewUrl = `${overlaySystemPrefix}${overlayId}${overlaySystemPreviewPrefix}`
      isOverlayIdActive = true
    } else {
      isOverlayIdActive = false
    }
  });

  function setCurrentPreviewId(previewId){
    currentPreviewId = previewId;
    overlayPreviewUrl = `${overlaySystemPrefix}${overlayId}${overlaySystemPreviewPrefix}${currentPreviewId}`
    console.log(overlayPreviewUrl)
  }
</script>

{#if isOverlayIdActive}
  <div class="columns pb-0">
    <div class="column is-three-fifths">
      <iframe
        title="Overlay Preview"
        src={overlayPreviewUrl}
        width="765"
        height="965">
    </div>

    <div class="column">
      <iframe
        title="Overlay Control"
        src={overlayControllerUrl}
        width="500"
        height="1015"></iframe>
    </div>
  </div>
{:else}
  <IdSetupMenu overlayId = {overlayId}/>
{/if}

<style>
  .button{
    margin-left: 60px;
  }
</style>