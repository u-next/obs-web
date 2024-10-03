<script>
  import { onMount } from 'svelte'
  import { sendCommand } from './obs.js'
  import IdSetupMenu from './IdSetupMenu.svelte';

  export let overlayId
  export let activeControlTab = 'audio'

  const overlaySystemPrefix = 'https://tatooine-e1db8.web.app/'
  let idSetupMenuActive

  onMount(async () => {
    const data = await sendCommand('GetInputSettings', {
      inputName: 'Overlay',
    });
    let overlayUrl = data.inputSettings.url;
    overlayId = extractIdFromURL(overlayUrl);
  });

  function extractIdFromURL(url) {
    if (!url.startsWith(overlaySystemPrefix) || url === overlaySystemPrefix) {
      alert('エラー: テロップIDが間違っています。入力直しください。');
      return null;
    }
    return url.substring(overlaySystemPrefix.length);
  }

  function setControlTabSelected(tab) {
    activeControlTab = tab;
    console.log(activeControlTab)
  }

  function setIdSetupMenuActive() {
    idSetupMenuActive = !idSetupMenuActive;
  }
</script>

<div class="columns mt-1 pb-0">
  <div class="column is-four-fifths">
    <p class="has-text-centered has-text-white">制御中テロップID</p>
    <p class="title is-4 has-text-centered has-text-white">{overlayId}</p>
  </div>
  <div class="column">
    <div class="buttons">
      <button class="button is-danger"
      on:click={setIdSetupMenuActive}>テロップID再設置・新規発行</button>
    </div>
  </div>
</div>

{#if idSetupMenuActive}
  <IdSetupMenu bind:overlayId = {overlayId}/>
{/if}

<div class="tabs is-large is-boxed is-centered">
  <ul>
    <li class={activeControlTab === 'audio' ? 'is-active' : ''}>
      <a on:click={() => setControlTabSelected('audio')}>オーディオ設定</a>
    </li>
    <li class={activeControlTab === 'overlay' ? 'is-active' : ''}>
      <a on:click={() => setControlTabSelected('overlay')}>テロップ設定</a>
    </li>
  </ul>
</div>

<style>
  .tabs li a{
    color: #fff
  }

  .tabs a:hover,
  .tabs li.is-active a {
    color: hsl(229, 53%, 53%);
  }
</style>