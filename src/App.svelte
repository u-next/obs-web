<script>
  /* eslint-env browser */
  const OBS_WEBSOCKET_LATEST_VERSION = '5.0.1'; // https://api.github.com/repos/Palakis/obs-websocket/releases/latest

  // Imports
  import { onMount } from 'svelte';
  import {
    mdiSquareRoundedBadge,
    mdiSquareRoundedBadgeOutline,
    mdiImageEdit,
    mdiImageEditOutline,
    mdiFullscreen,
    mdiFullscreenExit,
    mdiBorderVertical,
    mdiAccessPoint,
    mdiAccessPointOff,
    mdiRecord,
    mdiStop,
    mdiPause,
    mdiPlayPause,
    mdiConnection,
    mdiCameraOff,
    mdiCamera,
    mdiMotionPlayOutline,
    mdiMotionPlay,
  } from '@mdi/js';
  import Icon from 'mdi-svelte';
  import { compareVersions } from 'compare-versions';

  import './style.scss';
  import { obs, sendCommand } from './obs.js';
  import FramerateSelect from './FramerateSelect.svelte';
  import ProgramPreview from './ProgramPreview.svelte';
  import SceneSwitcher from './SceneSwitcher.svelte';
  import SourceSwitcher from './SourceSwitcher.svelte';
  import ProfileSelect from './ProfileSelect.svelte';
  import SceneCollectionSelect from './SceneCollectionSelect.svelte';
  import Mixer from './Mixer.svelte';
  import Status from './Status.svelte';
  import StreamDestinationInput from './StreamDestinationInput.svelte';
  import OverlayController from './OverlayController.svelte';
  import ControlTab from './ControlTab.svelte';
  import OverlaySidebar from './OverlaySidebar.svelte';
  import OverlayPreviews from './OverlayPreviews.svelte';
  import MacroButtons from './MacroButtons.svelte';

  onMount(async () => {
    if ('serviceWorker' in navigator) {
      navigator.serviceWorker.register('/service-worker.js');
    }

    // Request screen wakelock
    if ('wakeLock' in navigator) {
      try {
        await navigator.wakeLock.request('screen');
        // Re-request when coming back
        document.addEventListener('visibilitychange', async () => {
          if (document.visibilityState === 'visible') {
            await navigator.wakeLock.request('screen');
          }
        });
      } catch (e) {}
    }

    // Toggle the navigation hamburger menu on mobile
    const navbar = document.querySelector('.navbar-burger');
    navbar.addEventListener('click', () => {
      navbar.classList.toggle('is-active');
      document
        .getElementById(navbar.dataset.target)
        .classList.toggle('is-active');
    });

    // Listen for fullscreen changes
    document.addEventListener('fullscreenchange', () => {
      isFullScreen = document.fullscreenElement;
    });

    document.addEventListener('webkitfullscreenchange', () => {
      isFullScreen = document.webkitFullscreenElement;
    });

    document.addEventListener('msfullscreenchange', () => {
      isFullScreen = document.msFullscreenElement;
    });

    if (document.location.hash !== '') {
      // Read address from hash
      address = document.location.hash.slice(1);

      // This allows you to add a password in the URL like this:
      // http://obs-web.niek.tv/#ws://localhost:4455#password
      if (address.includes('#')) {
        [address, password, displayName] = address.split('#');
      }
      await connect();
    }

    // Export the sendCommand() function to the window object
    window.sendCommand = sendCommand;
  });

  // State
  let connected;
  let heartbeat = {};
  let heartbeatInterval;
  let isFullScreen;
  let isStudioMode;
  let isVirtualCamActive;
  let isIconMode = window.localStorage.getItem('isIconMode') || false;
  let isReplaying;
  let isStreaming;
  let isRecording;
  let editable = false;
  let address;
  let password;
  let scenes = [];
  let replayError = '';
  let errorMessage = '';
  let imageFormat = 'jpg';
  let isLocked = false;
  let displayName = '';
  let connectionPreset = '';
  let activeControlTab;
  let overlayId;
  let avalonMode = '';

  // Overlay System Related
  const overlaySystemUrl = 'https://tatooine-e1db8.web.app/';
  const overlaySystemControllerPrefix = '/controller';
  const overlaySystemPreviewPrefix = '/presetSync/';

  $: [displayName, address, password] = connectionPreset.split(' ');

  $: isIconMode
    ? window.localStorage.setItem('isIconMode', 'true')
    : window.localStorage.removeItem('isIconMode');

  function formatTime(secs) {
    secs = Math.round(secs / 1000);
    const hours = Math.floor(secs / 3600);
    secs -= hours * 3600;
    const mins = Math.floor(secs / 60);
    secs -= mins * 60;
    return hours > 0
      ? `${hours}:${mins < 10 ? '0' : ''}${mins}:${secs < 10 ? '0' : ''}${secs}`
      : `${mins < 10 ? '0' : ''}${mins}:${secs < 10 ? '0' : ''}${secs}`;
  }

  function toggleFullScreen() {
    if (isFullScreen) {
      if (document.exitFullscreen) {
        document.exitFullscreen();
      } else if (document.webkitExitFullscreen) {
        document.webkitExitFullscreen();
      } else if (document.msExitFullscreen) {
        document.msExitFullscreen();
      }
    } else {
      if (document.documentElement.requestFullscreen) {
        document.documentElement.requestFullscreen();
      } else if (document.documentElement.webkitRequestFullscreen) {
        document.documentElement.webkitRequestFullscreen();
      } else if (document.documentElement.msRequestFullscreen) {
        document.documentElement.msRequestFullscreen();
      }
    }
  }

  function setOperatorMode(event) {
    event.preventDefault();
    avalonMode = 'operator';
    connect();
  }

  function setDirectorMode(event) {
    event.preventDefault();
    avalonMode = 'director';
    connect();
  }

  function extractIdFromURL(url) {
    return url.substring(overlaySystemUrl.length);
  }

  async function toggleStudioMode() {
    await sendCommand('SetStudioModeEnabled', {
      studioModeEnabled: !isStudioMode,
    });
  }

  async function toggleReplay() {
    const data = await sendCommand('ToggleReplayBuffer');
    console.debug('ToggleReplayBuffer', data.outputActive);
    if (data.outputActive === undefined) {
      replayError = 'Replay buffer is not enabled.';
      setTimeout(function () {
        replayError = '';
      }, 5000);
    } else isReplaying = data.outputActive;
  }

  async function startStream() {
    await sendCommand('StartStream');
  }

  async function stopStream() {
    await sendCommand('StopStream');
  }

  async function startRecording() {
    await sendCommand('StartRecord');
  }

  async function stopRecording() {
    await sendCommand('StopRecord');
  }

  async function startVirtualCam() {
    await sendCommand('StartVirtualCam');
  }

  async function stopVirtualCam() {
    await sendCommand('StopVirtualCam');
  }

  async function pauseRecording() {
    await sendCommand('PauseRecord');
  }

  async function resumeRecording() {
    await sendCommand('ResumeRecord');
  }

  async function connect() {
    address = address || 'ws://localhost:4455';
    if (address.indexOf('://') === -1) {
      const secure = location.protocol === 'https:' || address.endsWith(':443');
      address = secure ? 'wss://' : 'ws://' + address;
    }
    console.log('Connecting to:', address, '- using password:', password);
    console.log('Avalon Mode: ', avalonMode);

    await disconnect();
    try {
      const { obsWebSocketVersion, negotiatedRpcVersion } = await obs.connect(
        address,
        password,
        // {
        //   eventSubscriptions: eventSubscription.InputVolumeMeters,
        //   rpcVersion: 1
        // }
      );
      console.log(
        `Connected to obs-websocket version ${obsWebSocketVersion} (using RPC ${negotiatedRpcVersion})`,
      );
    } catch (e) {
      console.log(e);
      errorMessage = e.message;
    }
  }

  async function disconnect() {
    await obs.disconnect();
    clearInterval(heartbeatInterval);
    connected = false;
    errorMessage = '切断された';
  }

  // OBS events
  obs.on('ConnectionClosed', () => {
    connected = false;
    window.history.pushState(
      '',
      document.title,
      window.location.pathname + window.location.search,
    ); // Remove the hash
    console.log('Connection closed');
  });

  obs.on('Identified', async () => {
    console.log('Connected');
    connected = true;
    document.location.hash = address + '#' + password + '#' + displayName; // For easy bookmarking
    let data = await sendCommand('GetVersion');
    const version = data.obsWebSocketVersion || '';
    console.log('OBS-websocket version:', version);
    if (compareVersions(version, OBS_WEBSOCKET_LATEST_VERSION) < 0) {
      alert(
        'You are running an outdated OBS-websocket (version ' +
          version +
          '), please upgrade to the latest version for full compatibility.',
      );
    }
    if (
      data.supportedImageFormats.includes('webp') &&
      document
        .createElement('canvas')
        .toDataURL('image/webp')
        .indexOf('data:image/webp') === 0
    ) {
      imageFormat = 'webp';
    }

    data = await sendCommand('GetInputSettings', {
      inputName: 'Overlay',
    });
    overlayId = extractIdFromURL(data.inputSettings.url);

    heartbeatInterval = setInterval(async () => {
      const stats = await sendCommand('GetStats');
      const streaming = await sendCommand('GetStreamStatus');
      const recording = await sendCommand('GetRecordStatus');
      heartbeat = { stats, streaming, recording };
    }, 1000); // Heartbeat
    isStudioMode =
      (await sendCommand('GetStudioModeEnabled')).studioModeEnabled || false;
    isVirtualCamActive =
      (await sendCommand('GetVirtualCamStatus')).outputActive || false;
  });

  obs.on('ConnectionError', async () => {
    errorMessage = 'Please enter your password:';
    document.getElementById('password').focus();
    if (!password) {
      connected = false;
    } else {
      await connect();
    }
  });

  obs.on('VirtualcamStateChanged', async (data) => {
    console.log('VirtualcamStateChanged', data.outputActive);
    isVirtualCamActive = data && data.outputActive;
    isLocked = isVirtualCamActive;
  });

  obs.on('StudioModeStateChanged', async (data) => {
    console.log('StudioModeStateChanged', data.studioModeEnabled);
    isStudioMode = data && data.studioModeEnabled;
  });

  obs.on('ReplayBufferStateChanged', async (data) => {
    console.log('ReplayBufferStateChanged', data);
    isReplaying = data && data.outputActive;
  });

  obs.on('StreamStateChanged', async (data) => {
    console.log('StreamStateChanged', data);
    isStreaming = data && data.outputActive;
    isLocked = isStreaming;
  });

  obs.on('RecordStateChanged', async (data) => {
    console.log('ReplayBufferStateChanged', data);
    isRecording = data && data.outputActive;
    isLocked = isRecording;
  });
</script>

<svelte:head>
  <title>Avalonリモートコントロール</title>
</svelte:head>

<nav class="navbar is-dark" aria-label="main navigation">
  <div class="navbar-brand">
    <!-- svelte-ignore a11y-missing-attribute -->
    <button
      class="navbar-burger burger"
      aria-label="menu"
      aria-expanded="false"
      data-target="navmenu"
    >
      <span aria-hidden="true" />
      <span aria-hidden="true" />
      <span aria-hidden="true" />
    </button>
  </div>

  <div id="navmenu" class="navbar-menu">
    <div class="navbar-start">
      {#if connected}
        <div class="navbar-item">
          <div class="buttons">
            <!-- svelte-ignore a11y-missing-attribute -->
            <button
              class:is-light={!isFullScreen}
              class="button is-link"
              on:click={toggleFullScreen}
              title="Toggle Fullscreen"
            >
              <span class="icon">
                <Icon path={isFullScreen ? mdiFullscreenExit : mdiFullscreen} />
              </span>
            </button>
            <button
              class:is-light={!isStudioMode}
              class="button is-link"
              on:click={toggleStudioMode}
              title="Toggle Studio Mode"
            >
              <span class="icon"><Icon path={mdiBorderVertical} /></span>
            </button>
            <button
              class:is-light={!editable}
              class="button is-link is-hidden"
              title="Edit Scenes"
              on:click={() => (editable = !editable)}
            >
              <span class="icon">
                <Icon path={editable ? mdiImageEditOutline : mdiImageEdit} />
              </span>
            </button>
            <button
              class:is-light={!isIconMode}
              class="button is-link is-hidden"
              title="Show Scenes as Icons"
              on:click={() => (isIconMode = !isIconMode)}
            >
              <span class="icon">
                <Icon
                  path={isIconMode
                    ? mdiSquareRoundedBadgeOutline
                    : mdiSquareRoundedBadge}
                />
              </span>
            </button>
            <button
              class:is-light={!isReplaying}
              class:is-danger={replayError}
              class="button is-link is-hidden"
              title="Toggle Replay Buffer"
              on:click={toggleReplay}
            >
              <span class="icon">
                <Icon
                  path={isReplaying ? mdiMotionPlayOutline : mdiMotionPlay}
                />
              </span>
              {#if replayError}<span>{replayError}</span>{/if}
            </button>
            <div class="block">
              <h1 class="title has-text-centered has-text-white">
                {displayName}
              </h1>
            </div>
          </div>
        </div>
      {:else}
        <img class="logo" src="/u-next-logo.png" alt="U-NEXT logo" />
      {/if}
    </div>
    <div class="navbar-end">
      <div class="navbar-item">
        <div class="buttons">
          <!-- svelte-ignore a11y-missing-attribute -->
          {#if connected}
            {#if avalonMode === 'operator'} <!-- Avalon O -->
              <ProfileSelect uiLock={isLocked} />
              <FramerateSelect uiLock={isLocked} />
              <SceneCollectionSelect uiLock={isLocked} />
              <StreamDestinationInput uiLock={isLocked} />
              {#if heartbeat && heartbeat.streaming && heartbeat.streaming.outputActive}
                <button
                  class="button is-danger"
                  on:click={stopStream}
                  title="Stop Stream"
                >
                  <span class="icon"><Icon path={mdiAccessPointOff} /></span>
                  <span>{formatTime(heartbeat.streaming.outputDuration)}</span>
                </button>
              {:else}
                <button
                  class="button is-danger is-light {isLocked
                    ? 'is-locked'
                    : ''}"
                  on:click={startStream}
                  title="Start Stream"
                >
                  <span class="icon"><Icon path={mdiAccessPoint} /></span>
                </button>
              {/if}
              {#if heartbeat && heartbeat.recording && heartbeat.recording.outputActive}
                {#if heartbeat.recording.outputPaused}
                  <button
                    class="button is-danger"
                    on:click={resumeRecording}
                    title="Resume Recording"
                  >
                    <span class="icon"><Icon path={mdiPlayPause} /></span>
                  </button>
                {:else}
                  <button
                    class="button is-success"
                    on:click={pauseRecording}
                    title="Pause Recording"
                  >
                    <span class="icon"><Icon path={mdiPause} /></span>
                  </button>
                {/if}
                <button
                  class="button is-danger"
                  on:click={stopRecording}
                  title="Stop Recording"
                >
                  <span class="icon"><Icon path={mdiStop} /></span>
                  <span>{formatTime(heartbeat.recording.outputDuration)}</span>
                </button>
              {:else}
                <button
                  class="button is-danger is-light {isLocked
                    ? 'is-locked'
                    : ''}"
                  on:click={startRecording}
                  title="Start Recording"
                >
                  <span class="icon"><Icon path={mdiRecord} /></span>
                </button>
              {/if}
              {#if isVirtualCamActive}
                <button
                  class="button is-danger"
                  on:click={stopVirtualCam}
                  title="Stop Virtual Webcam"
                >
                  <span class="icon"><Icon path={mdiCameraOff} /></span>
                </button>
              {:else}
                <button
                  class="button is-danger is-light {isLocked
                    ? 'is-locked'
                    : ''}"
                  on:click={startVirtualCam}
                  title="Start Virtual Webcam"
                >
                  <span class="icon"><Icon path={mdiCamera} /></span>
                </button>
              {/if}
            {/if}
            <button
              class="button is-danger is-light {isLocked ? 'is-locked' : ''}"
              on:click={disconnect}
              title="Disconnect"
            >
              <span class="icon"><Icon path={mdiConnection} /></span>
            </button>
          {:else}
            <button class="button is-danger" disabled
              >{errorMessage || '切断された'}</button
            >
          {/if}
        </div>
      </div>
    </div>
  </div>
</nav>

<section id="main-body" class="section has-background-black-ter">
  <div class="container">
    {#if connected}
      {#if avalonMode === 'operator'}
        <!-- Avalon O -->
        <Status bind:heartbeat />
        <SceneSwitcher
          bind:scenes
          buttonStyle={isIconMode ? 'icon' : 'text'}
          {editable}
        />
        <ProgramPreview {imageFormat} />
        {#each scenes as scene}
          {#if scene.sceneName.indexOf('(switch)') > 0}
            <SourceSwitcher
              name={scene.sceneName}
              {imageFormat}
              buttonStyle="screenshot"
            />
          {/if}
        {/each}
        <div class="box has-background-dark py-2 px-5">
          <ControlTab bind:activeControlTab bind:overlayId />
          {#if activeControlTab === 'audio'}
            <Mixer />
          {:else if activeControlTab === 'overlay'}
            <OverlayController bind:overlayId />
          {/if}
        </div>
      {:else}
        <!-- Avalon D -->
        <div class="columns">
          <div class="column is-two-fifths">
            <p class="has-text-centered has-text-white">制御中テロップID</p>
            <p class="title is-3 has-text-centered has-text-white">
              {overlayId}
            </p>

            {#if overlayId}
              <OverlaySidebar
                {overlaySystemUrl}
                {overlaySystemControllerPrefix}
                {overlayId}
              />
            {/if}
          </div>

          <div class="column">
            <ProgramPreview {imageFormat} />
            <MacroButtons />
            {#if overlayId}
              <OverlayPreviews
                {overlaySystemUrl}
                {overlaySystemPreviewPrefix}
                {overlayId}
              />
            {/if}
          </div>
        </div>
      {/if}
    {:else}
      <p class="title is-3 mt-3 has-text-centered has-text-white">
        Avalonに接続する
      </p>
      <form on:submit|preventDefault={connect}>
        <p class="has-text-white">プリセット</p>
        <div class="field is-grouped">
          <div class="select">
            <select bind:value={connectionPreset}>
              <option selected>501-PRIMARY ws://Avalon-501:4455Y </option>
              <option>501-BACKUP ws://Avalon-502:4455 </option>
              <option>502-PRIMARY ws://Avalon-503:4455 </option>
              <option>502-BACKUP ws://Avalon-504:4455 </option>
              <option>DEV ws://10.231.102.227:4455 </option>
              <option>LOCAL ws://localhost:4455 </option>
            </select>
          </div>
        </div>
        <p class="has-text-white">接続情報：</p>
        <div class="field is-grouped">
          <p class="control is-expanded">
            <input
              id="host"
              bind:value={address}
              class="input mb-3"
              type="text"
              autocomplete=""
              placeholder="ws://IPアドレス:4455"
            />
            <input
              id="password"
              bind:value={password}
              class="input mb-3"
              type="password"
              autocomplete="current-password"
              placeholder="パスワード (認証を無効にしている場合は空のままにしてください)"
            />
            <input
              id="displayName"
              bind:value={displayName}
              class="input mb-3"
              type="text"
              autocomplete=""
              placeholder="表示名"
            />
          </p>
        </div>
        <div class="buttons is-right">
          <button
            class="button is-success {address ? '' : 'is-locked'}"
            on:click={setOperatorMode}
          >
            Operator モード
          </button>
          <button
            class="button is-info {address ? '' : 'is-locked'}"
            on:click={setDirectorMode}
          >
            Director モード
          </button>
        </div>
      </form>
    {/if}
  </div>
</section>

<footer class="footer my-3 mx-2 p-0 has-background-black-ter">
  <div class="has-text-right has-text-white">Version: avalon_web_20240528</div>
</footer>
