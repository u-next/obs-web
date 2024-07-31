<!-- Popup.svelte -->
<script>
    export let show = false;
    export let onConfirm = () => {};
    export let onCancel = () => {};
  
    function handleYes() {
      onConfirm();
    }
  
    function handleNo() {
      onCancel();
    }
  </script>
  
  <style>
    .box {
      position: fixed;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      padding: 20px;
      border-radius: 15px;
      z-index: 1000;
    }
    .overlay {
      position: fixed;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      background-color: rgba(0, 0, 0, 0.5);
      z-index: 999;
    }
  </style>
  
  {#if show}
    <div class="overlay"></div>
    <div class="box has-background-dark">
      <p class="has-text-centered has-text-white mb-3">配信を停止します。よろしいでしょうか？</p>
      <div class="buttons is-right">
        <button class="button is-danger is-dark" on:click={handleYes}>はい</button>
        <button class="button is-primary is-dark" on:click={handleNo}>いいえ</button>
      </div>
    </div>
  {/if}