<template>
  <div class="hello">
    <button @click="queryAccounts">get account balance for Superone</button>
    <div v-if="scatter">
      <p v-if="!currentAccount">
        <button @click="login">Login</button>
      </p>
      
      <p v-else>
        {{currentAccount}}
        <button @click="logout">Logout</button>
        <br>
        <br>
          <button @click="send">send EOS to Superone</button>
      </p>
      
    </div>
    
  </div>
</template>

<script>

import Eos from "eosjs";
import ScatterJS from "scatterjs-core";
import ScatterEOS from "scatterjs-plugin-eosjs";

ScatterJS.plugins(new ScatterEOS());

const network = {
  blockchain: "eos",
  host: "mainnet.eoshenzhen.io",
  port: 8888,
  protocol: "http",
  chainId: "aca376f206b8fc25a6ed44dbdc66547c36c6c33e3a119ffbeaef943642f0e906"
};

// Put eosClient in data will case a weird problem in scatter-desktop.
let eosClient = null;

const requiredFields = { accounts: [network] };

export default {
  name: "HelloWorld",
  data() {
    return {
      currentAccount: "",
      scatter: null,
      readOnlyEos: null
    };
  },
  created() {
    let chainId = network.chainId;
    let httpEndpoint =
      network.protocol + "://" + network.host + ":" + network.port;

    this.readOnlyEos = Eos({
      chainId,
      httpEndpoint
    });

    ScatterJS.scatter.connect("scatter-demo").then(connected => {
      if (!connected) return false;

      this.scatter = ScatterJS.scatter;

      window.scatter = null;
      window.ScatterJS = null;
    });
  },
  methods: {
    // 获取账号信息
    login() {

      this.scatter.getIdentity(requiredFields).then(() => {
        const account = this.scatter.identity.accounts.find(x => x.blockchain === 'eos');
        this.currentAccount = account.name;
        
      }).catch(error => {
          alert(JSON.stringify(error));
      });
    },
    // 退出账号
    logout() {
      this.scatter.forgetIdentity();
      this.currentAccount = null;
    },
    // 调用智能合约 eosio.token 的 transfer 操作
    send() {
      eosClient.transfer(this.currentAccount, 'superone', '0.0001 EOS', 'from scatter-demo').then(data => {
        alert(JSON.stringify(data));
      }).catch(error => {
        alert(JSON.stringify(error));
      })
    },
    queryAccounts() {
      this.readOnlyEos.getTableRows({code:"eosio.token",scope:"superoneiobp",table:"accounts",json:true}).then( (data)=> {
        alert(JSON.stringify(data));
      });
    }
  }
}
</script>
