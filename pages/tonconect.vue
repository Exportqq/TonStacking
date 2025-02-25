<template>
    <header>
      <span>My App with Vue UI</span>
      <TonConnectButton/>
    </header>
  </template>
  
  <script>
  import { TonConnectButton } from '@townsquarelabs/ui-vue';
  
  export default {
    components: {
      TonConnectButton
    }
  }
  </script>