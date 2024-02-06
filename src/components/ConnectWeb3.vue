<script setup>
import {Contract, ethers} from 'ethers';
import { ref, onMounted } from 'vue';
import tkVaultSecretUIABI from "../contracts/TokenVaultSecretUI.sol/TokenVaultSecretUI.json";
import tkABI from "../contracts/TKXToken.sol/TKXToken.json";


var connected = ref(false);
var address =  ref('');
var provider = ref(null);
var signer = ref(null);
var tkVaultSecretContractUIAddr = ref('0xB8a1bb8A96D5A1c65CECaD0aCDE5DC4C02622766');
var tkContractAddr = ref('0x392C03945A0D8809Ccc50656a4fA694C67162786');
var tokenAmount = ref(0);
var depositAmount = ref(0);
var withdrawAmount = ref(0);
var randomCode = ref('');
var tokenVaultBalance = ref(0);

onMounted(() => {
  initWeb3();
})
async function initWeb3() {
    try {
      if (window.ethereum == null) {
        console.log("MetaMask not installed; using read-only defaults")
        provider = ethers.getDefaultProvider();
      }
      else {
        provider = new ethers.BrowserProvider(window.ethereum)
        signer = await provider.getSigner();
        connected.value = true;
        address.value = await signer.getAddress();

        await getTokenBalance();
        await getTokenVaultBalance();
      }
    }
    catch (error) {
      console.error('Connection error:', error);
    }
}
async function deposit() {
  var tkContract = new Contract(tkContractAddr.value, tkABI.abi, signer);
  var approveAmount = await tkContract.approve(tkVaultSecretContractUIAddr.value, depositAmount.value);
  await approveAmount.wait();

  var tkVaultSecretUIContract = new Contract(tkVaultSecretContractUIAddr.value, tkVaultSecretUIABI.abi, signer);
  var deposit = await tkVaultSecretUIContract.deposit(address.value, depositAmount.value, tkContractAddr.value)
  await deposit.wait();

  var userInfo = await tkVaultSecretUIContract.getUserInfo(address.value, tkContractAddr.value);
  randomCode.value = await userInfo[0];

  await getTokenBalance();
  await getTokenVaultBalance();
}
async function withDraw() {
  var tkVaultSecretUIContract = new Contract(tkVaultSecretContractUIAddr.value, tkVaultSecretUIABI.abi, signer);
  var wdAmt = await tkVaultSecretUIContract.withdraw(address.value, randomCode.value, withdrawAmount.value, tkContractAddr.value);
  await wdAmt.wait();

  await getTokenBalance();
  await getTokenVaultBalance();
}
async function getTokenVaultBalance() {
  var tkVaultSecretUIContract = new Contract(tkVaultSecretContractUIAddr.value, tkVaultSecretUIABI.abi, signer);
  tokenVaultBalance.value = await tkVaultSecretUIContract.getBalance(address.value, tkContractAddr.value);
}
async function getTokenBalance() {
  var tkContract = new Contract(tkContractAddr.value, tkABI.abi, signer);
  tokenAmount.value = await tkContract.balanceOf(address.value);
}
</script>

<template>
  <section class="walletApp">
    <div class="container">
      <h1>Vựa Token</h1>
      <hr>
      <div class="row">
        <div class="col-md-6">
          <div class="card">
            <div class="card-header">Nạp tiền</div>
            <div class="card-body">
              <form @submit.prevent="deposit()">
                <div class="mb-3">
                  <label for="deposit-amount" class="form-label">Số tiền</label>
                  <input type="number" class="form-control" id="deposit-amount" v-model="depositAmount" placeholder="Nhập số tiền">
                </div>
                <button type="submit" class="btn btn-primary">Nạp tiền</button>
              </form>
            </div>
          </div>
        </div>
        <div class="col-md-6">
          <div class="card">
            <div class="card-header">Rút tiền</div>
            <div class="card-body">
              <form @submit.prevent="withDraw()">
                <div class="mb-3">
                  <label for="withdraw-amount" class="form-label">Số tiền</label>
                  <input type="number" class="form-control" id="withdraw-amount" v-model="withdrawAmount" placeholder="Nhập số tiền">
                </div>
                <button type="submit" class="btn btn-primary">Rút tiền</button>
              </form>
            </div>
          </div>
        </div>
      </div>
      <hr>
      <div class="row">
        <div class="col-md-12">
          <div class="card">
            <div class="card-header">Thông tin ví</div>
            <div class="card-body" v-if="connected">
              <p>Địa chỉ ví: {{ address }}</p>
              <p>Số dư Token Vault: {{ tokenVaultBalance }}</p>
              <p>Số lượng Token: {{ tokenAmount }}</p>
              <div class="mb-3">
                <label for="secret-key" class="form-label">Secret Key</label>
                <input type="text" class="form-control" id="secret-key" v-model="randomCode" placeholder="Nhập Secret Key">
              </div>
<!--              <form @submit.prevent="getBalance">-->
<!--                <div class="mb-3">-->
<!--                  <label for="secret-key" class="form-label">Secret Key</label>-->
<!--                  <input type="text" class="form-control" id="secret-key" v-model="randomCode" placeholder="Nhập Secret Key">-->
<!--                </div>-->
<!--                <button type="submit" class="btn btn-primary">Lấy số dư</button>-->
<!--              </form>-->
            </div>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>
