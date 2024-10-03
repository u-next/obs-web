<script>
  import { onMount, onDestroy } from 'svelte';
  import { obs, sendCommand } from './obs.js';
  import MacroButtons from './MacroButtons.svelte';

  onMount(async () => {
    sendCommand('GetInputList').then((data) => {
      // console.log('Mixer GetInputList', data);
      for (let i = 0; i < data.inputs.length; i++) {
        if (data.inputs[i].inputName.endsWith('@ah')) {
          // hide if input has @ah suffix
          continue;
        }
        sendCommand('GetInputVolume', {
          inputName: data.inputs[i].inputName,
        }).then((vol) => {
          // console.log('Mixer GetInputVolume', vol);
          if ( "inputVolumeDb" in vol) {;
            inputs[data.inputs[i].inputName] = {
              ...data.inputs[i],
              ...vol,
            };
          }
        });

        sendCommand('GetInputAudioSyncOffset', {
          inputName: data.inputs[i].inputName,
        }).then((offset) => {
          // console.log('Mixer GetInputAudioSyncOffset', offset);
          if ( "inputAudioSyncOffset" in offset) {;
            inputs[data.inputs[i].inputName] = {
              ...inputs[data.inputs[i].inputName],
              ...offset,
            };
          }
        });
      }
    });
  });

  onDestroy(() => {});

  let inputs = {};

  obs.on('StudioModeStateChanged', async (data) => {
    console.log('Mixer StudioModeStateChanged', data.studioModeEnabled);
  });

  obs.on('CurrentPreviewSceneChanged', async (data) => {
    console.log('Mixer CurrentPreviewSceneChanged', data.sceneName);
  });

  obs.on('CurrentProgramSceneChanged', async (data) => {
    console.log('Mixer CurrentProgramSceneChanged', data.sceneName);
  });

  obs.on('InputVolumeChanged', async (data) => {
    // console.log('Mixer InputVolumeChanged', data)
    if (inputs[data.inputName]) {
      inputs[data.inputName] = { ...inputs[data.inputName], ...data };
    }
  });

  obs.on('InputAudioSyncOffsetChanged', async (data) => {
    // console.log('Mixer InputVolumeChanged', data)
    if (inputs[data.inputName]) {
      inputs[data.inputName] = { ...inputs[data.inputName], ...data };
    }
  });

  // obs.on('InputVolumeMeters', async (data) => {
  //   // console.log('Mixer InputVolumeChanged', data.inputs)
  //   inputMeter = data.inputs
  //   if (inputMeter && inputMeter.length > 0) {
  //     tempval = Math.round(inputMeter[0].inputLevelsMul[0][2] * 100)
  //   }
  // })

  // obs.on('SceneNameChanged', async (data) => {
  //   if (data.oldSceneName === programScene) programScene = data.sceneName
  //   if (data.oldSceneName === previewScene) previewScene = data.sceneName
  // })

  function updateVolume(e) {
    // console.log('updateVolume', e.target.name, e.target.value)
    sendCommand('SetInputVolume', {
      inputName: e.target.name,
      inputVolumeDb: parseFloat(e.target.value),
    });
  }

  function updateVolumeDelta(inputName, db) {
    const newVolume = db === 0 ? 0 : inputs[inputName].inputVolumeDb + db;
    sendCommand('SetInputVolume', {
      inputName: inputName,
      inputVolumeDb: parseFloat(newVolume),
    });
  }

  async function updateSyncOffset(input) {    
    // console.log('updateSyncOffset', input)

    try {
      await sendCommand('SetInputAudioSyncOffset', {
      inputName: input.inputName,
      inputUuid: input.inputUuid,
      inputAudioSyncOffset: parseFloat(input.inputAudioSyncOffset)
    })
      alert('遅延値を更新しました。')
    } catch (error) {
      alert('エラー: 遅延値の更新は失敗しました。')
    }
  }
</script>


<div class="mixer">
  <MacroButtons />

  <ol class="mt-5">
    {#if inputs && Object.keys(inputs).length > 0}
      {#each Object.keys(inputs).sort() as iname}
        <li class="box is-marginless has-background-dark">
          <div class="is-relative">
            <span class="tag is-dark is-small mixer-label"
              >{inputs[iname].inputName}</span
            >

            <div class="buttons are-small mixer-buttons">
              <button
                class="button is-white is-outlined"
                on:click={() => updateVolumeDelta(inputs[iname].inputName, 1)}
                >+1</button
              >
              <button
                class="button is-white is-outlined"
                on:click={() => updateVolumeDelta(inputs[iname].inputName, 0)}
                >=0</button
              >
              <button
                class="button is-white is-outlined"
                on:click={() => updateVolumeDelta(inputs[iname].inputName, -1)}
                >-1</button
              >
            </div>
            
            <input
              orient="vertical"
              class="slider mixer-slider"
              step="0.1"
              min="-60"
              max="12"
              value={inputs[iname].inputVolumeDb}
              type="range"
              on:input={updateVolume}
              name={inputs[iname].inputName}
            />
          </div>

          <span class="tag is-info is-small is-marginless is-centered has-background-dark mixer-value"
            >{typeof inputs[iname].inputVolumeDb === 'number'
              ? inputs[iname].inputVolumeDb.toFixed(1)
              : inputs[iname].inputVolumeDb} dB
          </span>

          <div class="has-text-white">
            <p class="mb-1">遅延値</p>
              <input class="input is-info mb-1" type="text" bind:value={inputs[iname].inputAudioSyncOffset}/>  ms
              <button class="button is-warning" on:click={updateSyncOffset(inputs[iname])}>更新</button>
          </div>
        </li>
      {/each}
    {/if}
  </ol>
</div>

<style>
  ol {
    list-style: None;
    display: flex;
    flex-wrap: wrap;
    justify-content: space-between;
    gap: 0.5rem;
    row-gap: 0.5rem;
  }
  .mixer-slider {
    width: 4rem;
    height: 10rem;
  }
  /* patch bulma-slider bug on chrome */
  input[type='range'].slider[orient='vertical']::-webkit-slider-thumb {
    position: relative;
    left: 0.25rem;
  }
  .mixer-value {
    width: 4rem;
  }
  .mixer-buttons {
    position: absolute;
    top: 0;
    right: 0rem;
    width: 1.5rem;
  }
  .mixer-label {
    position: absolute;
    transform-origin: 0 0;
    left: 0.5rem;
    transform: rotate(90deg);
  }
  
  .input.is-info {
    border-radius: 4px;
    width: 80px;
  }

</style>
