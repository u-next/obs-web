<script>
  import { onMount } from 'svelte'

  //Get from App.svelte
  export let overlaySystemUrl
  export let overlaySystemControllerPrefix
  export let overlayId

  let overlayControllerUrl

  onMount(async () => {
    overlayControllerUrl = `${overlaySystemUrl}${overlayId}${overlaySystemControllerPrefix}`;
  });
</script>

<iframe
  title="Overlay Control Sidebar"
  src={overlayControllerUrl}
  width="500"
  height="1400"></iframe>