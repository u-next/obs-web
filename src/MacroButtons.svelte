<script>
  import { onMount } from 'svelte';
  import { obs, sendCommand } from './obs.js';

  onMount(async () => {
    await sendCommand('CallVendorRequest', {
      vendorName: 'AdvancedSceneSwitcher',
      requestType: 'AdvancedSceneSwitcherMessage',
      requestData: { message: 'get-macro-list' },
    });
  });

  let macroCommandArray = [];

  obs.on('VendorEvent', async (data) => {
    macroCommandArray = ('VendorEvent', data.eventData.message).split(',');
  });
</script>

<div class="buttons has-text-centered px-6 my-5">
  {#each macroCommandArray as macroCommand}
    <button
      class="macro button has-text-dark is-medium {macroCommand.includes('pg-audio') ? 'is-info' : 'is-warning'}"
      on:click={async () => {
        await sendCommand('CallVendorRequest', {
          vendorName: 'AdvancedSceneSwitcher',
          requestType: 'AdvancedSceneSwitcherMessage',
          requestData: { message: macroCommand },
        });
      }}>{macroCommand}</button
    >
  {/each}
</div>

<style>
  .macro {
    width: 8rem;
    margin-left: 5rem;
  }
</style>
