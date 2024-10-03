<script>
  import { onMount } from 'svelte';
  import { obs, sendCommand } from './obs.js';
  let macroCommandArray = [];
  let isAudioCommandArray = [];
  let pgmAudioCommandArray = [];

  onMount(async () => {
    await sendCommand('CallVendorRequest', {
      vendorName: 'AdvancedSceneSwitcher',
      requestType: 'AdvancedSceneSwitcherMessage',
      requestData: { message: 'get-macro-list' },
    });

    obs.once('VendorEvent', async (data) => {
      macroCommandArray = ('VendorEvent', data.eventData.message).split(',');
      
      isAudioCommandArray = macroCommandArray.filter(command => 
        command.includes('is-audio') && command !== 'is-audio=0'
      );
      pgmAudioCommandArray = macroCommandArray.filter(command => 
        command.includes('pgm-audio') && command !== 'pgm-audio=0'
      );
    });
  });
</script>

<div class="columns is-vcentered is-centered">
  <div class="column is-one-fifth">
    <div class="title is-3 has-text-white has-text-centered">IS Audio</div>
  </div>

  <div class="column is-narrow">
    <button
        class="macro button has-text-dark is-medium is-info"
        on:click={async () => {
          await sendCommand('CallVendorRequest', {
            vendorName: 'AdvancedSceneSwitcher',
            requestType: 'AdvancedSceneSwitcherMessage',
            requestData: { message: "is-audio=0" },
          });
        }}>0</button>
  </div>

  <div class="column is-three-fifths">
    <div class="buttons has-text-centered px-6 my-5">
      {#each isAudioCommandArray as commands}
        <button
          class="macro button has-text-dark ml-1 is-medium is-info"
          on:click={async () => {
            await sendCommand('CallVendorRequest', {
              vendorName: 'AdvancedSceneSwitcher',
              requestType: 'AdvancedSceneSwitcherMessage',
              requestData: { message: commands },
            });
          }}>{commands.split("is-audio").pop()}</button>
      {/each}
    </div>
  </div>
</div>

<div class="columns is-vcentered is-centered">
  <div class="column is-one-fifth">
    <div class="title is-3 has-text-white has-text-centered">PGM Audio</div>
  </div>

  <div class="column is-narrow">
    <button
        class="macro button has-text-dark is-medium is-warning"
        on:click={async () => {
          await sendCommand('CallVendorRequest', {
            vendorName: 'AdvancedSceneSwitcher',
            requestType: 'AdvancedSceneSwitcherMessage',
            requestData: { message: "pgm-audio=0" },
          });
        }}>0</button>
  </div>

  <div class="column is-three-fifths">
    <div class="buttons has-text-centered px-6 my-5">
      {#each pgmAudioCommandArray as commands}
      <button
        class="macro button has-text-dark ml-1 is-medium is-warning"
        on:click={async () => {
          await sendCommand('CallVendorRequest', {
            vendorName: 'AdvancedSceneSwitcher',
            requestType: 'AdvancedSceneSwitcherMessage',
            requestData: { message: commands },
          });
        }}>{commands.split("pgm-audio").pop()}</button>
      {/each}
    </div>
  </div>
</div>

<style>
  .button {
    width: 8rem;
    justify-content: center;
    align-items: center;
    text-align: center;
  }
</style>
