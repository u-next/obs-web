<script>
  import { sendCommand } from './obs.js'

  const overlaySystemPrefix = 'https://tatooine-e1db8.web.app/'
  const overlaySystemCreatePrefix = 'create'
  export let overlayId

  async function setBrowserInputSetting() {
    try {
      await sendCommand('SetInputSettings', {
        inputName: 'Overlay',
        inputSettings: {
          url: `${overlaySystemPrefix}${overlayId}`,
        },
        overlay: true,
      })
      alert('テロップID設定のしました。')
      location.reload();
    } catch (error) {
      overlayControllerUrl = ''
      alert('エラー: テロップ設定の更新は失敗しました。')
    }
  }
</script>

<div class="columns pb-0">
  <div class="column is-three-fifths">
    <p class="mb-2 has-text-white">テロップ ID</p>
    <input
      class="input is-info"
      type="text"
      placeholder="テロップ ID"
      bind:value={overlayId}
    />
    <div class="buttons mt-4 is-right">
      <button class="button is-danger" on:click={setBrowserInputSetting}>適用</button>
    </div>
  </div>
  <div class="column">
    <iframe
    title="Overlay ID Create"
    src="{overlaySystemPrefix}{overlaySystemCreatePrefix}"
    width="500"
    height="345"
    style="overflow: auto; border: none;"
    frameborder="0"
    ></iframe>
  </div>
</div>