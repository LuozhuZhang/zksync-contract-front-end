<template>
  <meta name="google-site-verification" content="3NHhqgFIEb5TTfIk9DmCiVkRUbi3qkPIFtjLtrdJ0x0" />
  <div id="app" v-if="!mainLoading">
    <h1> Greeter says: {{greeting}} 👋</h1>
    <div>
      This a simple dApp, which can choose fee token and interact with the `Greeter` smart contract.
    </div>
    <div class="main-box">
      <div>
        Select token: <select v-model="selectedTokenAddress" v-on:change="changeToken">
          <option v-for="token in tokens" v-bind:value="token.address" v-bind:key="token.address" >
            {{ token.symbol }}
          </option>
        </select>
      </div>
      <div class="balance" v-if="selectedToken">
        <p>Balance: <span v-if="retreivingBalance">Loading...</span>
        <span v-else>{{currentBalance}} {{selectedToken.symbol}}</span></p>
        <p>Expected fee: <span v-if="retreivingFee">Loading...</span>
        <span v-else>{{currentFee}} {{selectedToken.symbol}}</span>
        <button class="refresh-button" v-on:click="updateFee">Refresh</button>
        </p>
      </div>
      <div class="greeting-input">
        <input v-model="newGreeting" :disabled="!selectedToken || txStatus!=0" placeholder="Write new greeting here..." >

        <button class="change-button" :disabled="!selectedToken || txStatus!=0 || retreivingFee" v-on:click="changeGreeting">
          <span v-if="selectedToken && !txStatus">Change greeting</span>
          <span v-else-if="!selectedToken">Select token to pay fee first</span>
          <span v-else-if="txStatus == 1">Sending tx...</span>
          <span v-else-if="txStatus == 2">Waiting until tx is committed...</span>
          <span v-else-if="txStatus == 3">Updating the page...</span>
          <span v-else-if="retreivingFee">Updating the fee...</span>
        </button>
      </div>
    </div>
  </div>
  <div id="app" v-else>
    <div class="start-screen">
      <h1>Welcome to Greeter dApp!</h1>
      <button v-on:click="connectMetamask">Connect Metamask</button>
    </div>
  </div>
</template>

<script>
import { Contract, Web3Provider, Provider } from "zksync-web3";
import { ethers } from "ethers";

// eslint-disable-next-line
const GREETER_CONTRACT_ADDRESS = '0x99b6022120302cF091d3eDf7Ed7a1b7Cb3CB8f61'; // contract Address
// eslint-disable-next-line
const GREETER_CONTRACT_ABI = require('./abi.json'); // contract ABI  

const allowedTokens = require("./tokens.json");

export default {
  name: 'App',
  data() {
    return {
      newGreeting: "",
      greeting: "unknown",
      tokens: allowedTokens,
      selectedToken: null,
      selectedTokenAddress: "",
      mainLoading: true,
      provider: null,
      signer: null,
      contract: null,
      canSubmit: true,
      // 0 stands for no status, i.e no tx has been sent
      // 1 stands for tx is beeing submitted to the operator
      // 2 stands for tx awaiting commit
      // 3 stands for updating the balance and greeting on the page
      txStatus: 0,
      retreivingFee: false,
      retreivingBalance: false,

      currentBalance: "",
      currentFee: ""
    }
  },
  methods: {
    // * 连接钱包的provider，Web3Provider和Provider用于和zkSync network交互，Contract用于和Greeter.sol合约交互
    // * initialize provider and signer based on `window.ethereum`
    initializeProviderAndSigner() {
      // * provider用于发送RPC请求 与节点通信（这里使用的zksync节点的rpc url），signer是电子签名 用于签发交易（这里使用metamask的signer）
      // * window.ethereum是一个全局可用的以太坊API，兼容大部分浏览器
      // * https://www.preethikasireddy.com/post/the-architecture-of-a-web-3-0-application
      this.provider = new Provider('https://zksync2-testnet.zksync.dev');
      this.signer = (new Web3Provider(window.ethereum)).getSigner();

      // * 连接Greeter.sol合约（通过address和ABI）
      this.contract = new Contract(
        GREETER_CONTRACT_ADDRESS,
        GREETER_CONTRACT_ABI,
        this.signer
      );
    },
    async getGreeting() {
      // * 调用合约，返回消息内容
      return await this.contract.greet();
    },
    async getFee() {
      // * 预估一笔交易需要的gas费
      const feeInGas = await this.contract.estimateGas.setGreeting(this.newGreeting);
      const gasPriceInUnits = await this.provider.getGasPrice();
      return ethers.utils.formatUnits(feeInGas.mul(gasPriceInUnits), this.selectedToken.decimals);
    },
    async getBalance() {
      const balanceInUnits = await this.signer.getBalance(this.selectedToken.address);
      // * 转换格式：wei -> eth，USD的格式还有点问题
      return ethers.utils.formatUnits(balanceInUnits, this.selectedToken.decimals);
    },
    async changeGreeting() {
      this.txStatus = 1;
      try {
        const txHandle = await this.contract.setGreeting(this.newGreeting, {
          customData: {
            // * 选择支付tx fee的token
            feeToken: this.selectedToken.address
          }
        });
        this.txStatus = 2;

        // * 等待交易完成
        await txHandle.wait();
        this.txStatus = 3;

        // * 更新greeting
        this.greeting = await this.getGreeting();

        this.retreivingFee = true;
        this.retreivingBalance = true;
        // * 更新 balance 和 fee
        this.currentBalance = await this.getBalance();
        this.currentFee = await this.getFee();
      } catch (e) {
        alert(JSON.stringify(e));
      }

      this.txStatus = 0;
      this.retreivingFee = false;
      this.retreivingBalance = false;
    },

    updateFee() {
      this.retreivingFee = true;
      // * 调用getFee，计算交易费用
      this.getFee().then((fee) => {
        this.currentFee = fee;
      })
      .catch(e => console.log(e))
      .finally(() => {
        this.retreivingFee = false;
      });
    },
    updateBalance() {
      this.retreivngBalance = true;
      // * 调用getBalance，计算余额
      this.getBalance().then((balance) => {
        this.currentBalance = balance;
      })
      .catch(e => console.log(e))
      .finally(() => {
        this.retreivngBalance = true;
      });
    },
    // * 3.更改token时，自动计算balance和交易fee
    changeToken() {
      this.selectedToken = this.tokens.filter(t => t.address == this.selectedTokenAddress)[0];
      this.updateFee();
      this.updateBalance();
    },
    // * 2.加载主页面
    loadMainScreen() {
      this.initializeProviderAndSigner();
      alert("debug：Successful connection");

      if(!this.provider || !this.signer) {
        alert("Follow the tutorial to learn how to connect to Metamask!");
        return;
      }

      // * 调用合约，获取消息；将mainloading的状态改为false，加载出主页面
      // * v-if条件渲染很方便，react的稍微复杂一些：https://zh-hans.reactjs.org/docs/conditional-rendering.html
      this.getGreeting().then((greeting) => {
        this.greeting = greeting;
        this.mainLoading = false;
      });
    },  
    connectMetamask() {
      // * 1.发送eth_requestAccounts rpc请求，连接metamask钱包
      window.ethereum.request({ method: 'eth_requestAccounts' })
        .then(() => {
          if (+window.ethereum.networkVersion == 280) {
            // * 连接成功后加载主页面
            this.loadMainScreen();
          } else {
            alert("Please switch network to zkSync!");
          }
        })
        .catch((e) => console.log(e)); 
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 30px;
}

#app ul{
  display: inline-block;
}

.main-box {
  text-align: left;
  width: 400px;

  margin: auto;
  margin-top: 40px;
}

.greeting-input {
  margin-top: 20px;
}

.change-button {
  margin-left: 20px;
}

.start-screen {
  margin-top: 100px;
}

.balance {
  margin-top: 10px;
}

.refresh-button {
  margin-left: 15px;
}
</style>
