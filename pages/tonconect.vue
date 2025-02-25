<template>
    <TonConnectUIProvider :options="options">
      <!-- Your app -->
    </TonConnectUIProvider>
  </template>
  
  <script>
  import { TonConnectUIProvider } from '@townsquarelabs/ui-vue';
  
  export default {
    components: {
      TonConnectUIProvider
    },
    setup(){
      const options = {
        manifestUrl:"https://ton-staker.netlify.app/tonconnect-manifest.json",
      };
      return {
        options
      }
    }
  }
  </script>