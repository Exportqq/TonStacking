<template>
    <div>
      <button v-if="!connected" @click="connectWallet">Connect Wallet</button>
      <div v-else>
        <p>Connected Wallet: {{ walletAddress }}</p>
        <p>Chain: {{ chain }}</p>
        <button @click="disconnectWallet">Disconnect</button>
        <slot></slot> <!-- Added a slot for content when connected -->
      </div>
    </div>
  </template>
  
  <script>
  import { TonConnectUI } from '@tonconnect/ui';  // Use TonConnectUI (recommended)
  // OR: import { TonConnect } from '@tonconnect/sdk';  // If you need more manual control
  
  export default {
    data() {
      return {
        tonConnectUI: null, //  For TonConnectUI
        // tonConnect: null,     // For TonConnect (manual control option)
        walletAddress: null,
        chain: null, // Add chain
        connected: false,
        connectorOptions: { // Options for the connector, moved for clarity
            manifestUrl: 'https://ton-staker.netlify.app/tonconnect-manifest.json',  //  REQUIRED:  Replace with your actual manifest URL
            // jsBridgeKey:  // Only needed for the Injected wallet (TON Wallet extension).  Get from your wallet provider.
            // Other options as needed (see below)
        }
      };
    },
    mounted() {
      this.initTonConnect();
    },
    beforeUnmount() {
      //  Important:  Disconnect on component unmount to avoid memory leaks and errors.
      if (this.tonConnectUI) {
        this.tonConnectUI.disconnect();  //  For TonConnectUI
      }
      // else if (this.tonConnect) {
      //    this.tonConnect.disconnect();  // For TonConnect (manual)
      // }
    },
    methods: {
      async initTonConnect() {
          //  TonConnectUI (Recommended, simpler approach)
          this.tonConnectUI = new TonConnectUI(this.connectorOptions);
  
          this.tonConnectUI.onStatusChange((wallet) => {
              if (wallet) {
                  this.walletAddress = wallet.account.address;
                  this.chain = wallet.account.chain; // Set the chain
                  this.connected = true;
              } else {
                  this.walletAddress = null;
                  this.chain = null;
                  this.connected = false;
              }
          }, (error) => {
            console.error("TonConnectUI Error:", error);
          });
  
  
         //  ----  Alternative:  Manual TonConnect initialization (more control, but more complex) ----
        /*
        this.tonConnect = new TonConnect(this.connectorOptions);
  
        this.tonConnect.onStatusChange((wallet) => {
          if (wallet) {
            this.walletAddress = wallet.account.address;
            this.connected = true;
          } else {
            this.walletAddress = null;
            this.connected = false;
          }
        }, (error) => {console.error("TonConnect Error:", error);});
  
        // Restore the session if one exists.
        if (await this.tonConnect.restoreConnection()) {
           this.connected = true;
           // No need to get address again, it's handled by onStatusChange
        }
        */
      },
      async connectWallet() {
         if (this.tonConnectUI) {
             await this.tonConnectUI.connectWallet(); //  For TonConnectUI, this is all you need.
         }
         // ---- Alternative for TonConnect (manual control) ----
         /*
         else if (this.tonConnect) {
              try {
                  // Request connection.  This will open the wallet selection modal.
                  const walletsList = await this.tonConnect.getWallets();  // Get list of available wallets.
                  await this.tonConnect.connect(walletsList[0]); // Connect to the *first* wallet in the list (for simplicity).  You'll likely want to show a list to the user.
              } catch (error) {
                   console.error("Wallet connection error:", error);
              }
         }
         */
      },
        disconnectWallet() {
            if (this.tonConnectUI) {
                this.tonConnectUI.disconnect();
            }
            // --- Alternative for TonConnect (manual) ---
            // else if(this.tonConnect) {
            //    this.tonConnect.disconnect();
            //}
        },
        //  Add other methods as needed, such as for sending transactions
    },
  };
  </script>
  
  <style lang="scss" scoped>
  /* Add any necessary styling here */
  button {
    padding: 10px 15px;
    background-color: #4CAF50; /* Example style */
    color: white;
    border: none;
    cursor: pointer;
    border-radius: 5px;
  
    &:hover {
      background-color: #3e8e41;
    }
  }
  
  div {
    margin: 10px 0;
  }
  </style>