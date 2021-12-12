<script setup lang="ts">
import * as web3 from '@solana/web3.js'
import * as anchor from '@project-serum/anchor'
import { ref } from 'vue'
import {
  CandyMachine,
  getCandyMachineState,
  awaitTransactionSignatureConfirmation,
  mintOneToken,
} from '../candyMachine'
import { onMounted, defineProps } from '@vue/runtime-core'
import { useAnchorWallet, useWallet } from '@solana/wallet-adapter-vue'


const props = defineProps({
  candyMachineId: anchor.web3.PublicKey,
  treasury: anchor.web3.PublicKey,
  config: anchor.web3.PublicKey,
  txTimeout: Number,
})

// eslint-disable-next-line no-unused-vars
let connection: web3.Connection | null = null
const anchorWallet = useAnchorWallet();
const {connected} = useWallet();
let isSoldOut = ref(false)
let isMinting = ref(false)
let isActive = ref(false)
//
let candyMachine: CandyMachine | undefined
let itemsAvailable = ref(0)
let itemsRedeemed = ref(0)
let itemsRemaining = ref(0)
let goLiveDate = ref(0)

onMounted(() => {
  connection = connectChain()
})

const connectChain = () => {
  return new anchor.web3.Connection(process.env.VUE_APP_SOLANA_RPC_HOST!)
}

const refreshCandyMachineState = async () => {
  try {
    await getCandyMachineState(
      anchorWallet.value! as anchor.Wallet,
      props.candyMachineId,
      connection!,
    )?.then((data) => {
      candyMachine = data?.candyMachine
      itemsAvailable.value = data?.itemsAvailable!
      itemsRedeemed.value = data?.itemsRedeemed!
      itemsRemaining.value = data?.itemsRemaining!
      goLiveDate.value = data?.goLiveDate!

      isSoldOut.value = itemsRemaining.value === 0
    })
  } catch (e) {
    console.log(e)
  }
}

const onMint = async () => {
  // check if wallet address is whitelisted
  // add toast on return to tell people they are not on the whitelist
  await refreshCandyMachineState();
  try {
    isMinting.value = true
    if (anchorWallet.value && candyMachine?.program) {
      const mintTxId = await mintOneToken(
        candyMachine,
        props.config,
        anchorWallet.value.publicKey,
        props.treasury,
      )

      const status = await awaitTransactionSignatureConfirmation(
        mintTxId,
        props.txTimeout!,
        connection!,
        'singleGossip',
        false,
      )

      if (!status?.err) {
        console.log('MINT SUCCESS')
      }
    }
  } catch (error: any) {
    // TODO: blech:
    // eslint-disable-next-line no-unused-vars
    let message = error.msg || 'Minting failed! Please try again!'
    if (!error.msg) {
      // eslint-disable-next-line no-empty
      if (error.message.indexOf('0x138')) {
      } else if (error.message.indexOf('0x137')) {
        message = `SOLD OUT!`
      } else if (error.message.indexOf('0x135')) {
        message = `Insufficient funds to mint. Please fund your wallet.`
      }
    } else {
      if (error.code === 311) {
        message = `SOLD OUT!`
        isSoldOut.value = true
      } else if (error.code === 312) {
        message = `Minting period hasn't started yet.`
      }
    }
  } finally {
    isMinting.value = false
    refreshCandyMachineState()
  }
}
</script>

<template>
  <div class="right">
    <button
      v-if="connected"
      :disabled="isSoldOut || isMinting || isActive"
      @click="onMint"
    >
      <div v-if="isSoldOut">Sold Out</div>
      <div v-else-if="isActive">Loading</div>
      <div v-else-if="!isActive">Mint</div>
      <div v-else-if="Date.now() / 1000 > goLiveDate">Has Not</div>
    </button>
  </div>
</template>
