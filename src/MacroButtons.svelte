<script>
  import { onMount } from 'svelte';
  import { obs, sendCommand } from './obs.js';

  let macroCommandArray = [];

  onMount(async () => {
    await sendCommand('CallVendorRequest', {
      vendorName: 'AdvancedSceneSwitcher',
      requestType: 'AdvancedSceneSwitcherMessage',
      requestData: { message: 'get-macro-list' },
    });

    obs.once('VendorEvent', async (data) => {
      macroCommandArray = ('VendorEvent', data.eventData.message).split(',');
    });
  });
</script>

<div class="buttons has-text-centered px-6 my-5">
  {#each macroCommandArray as macroCommand}
    <button
      class="macro button has-text-dark ml-1 is-medium {macroCommand.includes('pg-audio') ? 'is-info' : 'is-warning'}"
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
  .button {
    width: 8rem;
    justify-content: center;
    align-items: center;
    text-align: center;
  }
</style>
