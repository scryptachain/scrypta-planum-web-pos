<template>
  <div class="home">
    <div v-if="!showScan" style="padding:20px">
      Controlla il saldo di una card wallet direttamente da qui.<br><br>
      <div v-if="!userBalance" style="padding:5px 10px">
        <b-button type="is-primary" v-on:click="showScanQR()" size="is-large">CONTROLLA ORA</b-button>
      </div>
      <div v-if="userBalance !== false">
        <hr>
        <h1>Il saldo è {{ userBalance }} {{ ticker }}</h1>
      </div>  
    </div>
    <div class="fullscreen" v-if="showScan">
      <b-button v-on:click="hideScanModal" type="is-primary" style="width:50px; position:fixed; z-index:999; top:10px; right:10px;">X</b-button>
      <qrcode-stream @decode="onDecode"></qrcode-stream>
    </div>
  </div>
</template>

<script>
const ScryptaCore = require("@scrypta/core");
const axios = require('axios')
export default {
  data() {
    return {
      scrypta: new ScryptaCore(true),
      address: "",
      wallet: "",
      axios: axios,
      qrcode: "",
      ticker: "",
      amountLyra: 0,
      amountFIAT: 0,
      currency: 'eur',
      userBalance: false,
      chains: {},
      amountSidechain: 0,
      focus: 'lyra',
      chain: '6RQ54yHx2dARWkN8Biiw3gDjb4sB5hSHSH',
      showScan: false,
      guestwallet: ''
    };
  },
  async mounted() {
    const app = this;
    app.wallet = await app.scrypta.importBrowserSID();
    app.wallet = await app.scrypta.returnDefaultIdentity();
    if(localStorage.getItem('currency') !== null){
      app.currency = localStorage.getItem('currency')
    }

    if(localStorage.getItem('chain') !== null){
      app.chain = localStorage.getItem('chain')
    }
    if(app.chain !== 'main'){
      let sidechains = await app.scrypta.get('/sidechain/list')
      for(let x in sidechains.data){
        let sidechain = sidechains.data[x]
        if(sidechain.address === app.chain){
          app.ticker = sidechain.genesis.symbol
        }
      }
    }
    if (app.wallet.length > 0) {
      let SIDS = app.wallet.split(":");
      app.address = SIDS[0];
      let identity = await app.scrypta.returnIdentity(app.address);
      app.wallet = identity;
      app.isLogging = false;
    } else {
      app.isLogging = false;
    }
  },
  methods: {
    hideScanModal() {
      const app = this
      app.showScan = false
    },
    showScanQR(){
      const app = this
      app.showScan = true
    },
    async onDecode (decodedString) {
      const app = this
      app.showScan = false
      app.guestwallet = decodedString
      let exp = app.guestwallet.split(':')
      let balance = await app.scrypta.post('/sidechain/balance',{
        dapp_address: exp[0],
        sidechain_address: app.chain
      })
      app.userBalance = balance.balance
    },
  }
};
</script>