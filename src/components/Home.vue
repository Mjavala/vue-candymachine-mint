<script setup lang="ts">
import * as web3 from '@solana/web3.js'
import * as anchor from '@project-serum/anchor'
import { useWallet, useAnchorWallet } from '@solana/wallet-adapter-vue'
import { ref } from 'vue'
import {
  CandyMachine,
  getCandyMachineState,
  awaitTransactionSignatureConfirmation,
  mintOneToken,
} from '../candyMachine'
import { WalletName } from '@solana/wallet-adapter-wallets'
import { onMounted, defineProps } from '@vue/runtime-core'

// @ts-ignore
import { whitelist } from '../constants/whitelist.ts'

const props = defineProps({
  candyMachineId: anchor.web3.PublicKey,
  treasury: anchor.web3.PublicKey,
  config: anchor.web3.PublicKey,
  txTimeout: Number,
})

// eslint-disable-next-line no-unused-vars
let connection: web3.Connection | null = null
let walletConnected = ref(false)
let wallet = useAnchorWallet()
let walletStore = useWallet()
let walletChoice = 'Phantom'
let phantom: string | null = null
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

const connectWallet = async () => {
  if (walletChoice === 'Phantom') await walletStore.select(WalletName.Phantom)
  else if (walletChoice === 'Sollet')
    await walletStore.select(WalletName.Sollet)

  if (walletStore.adapter.value) {
    walletStore.adapter.value.on('connect', async () => {
      walletConnected.value = true
      await refreshCandyMachineState()
    })
    walletStore.adapter.value.on('disconnect', () => {
      walletConnected.value = false
    })
  }
  await walletStore.connect()
}

const refreshCandyMachineState = async () => {
  ;(async () => {
    console.log(wallet.value)
    if (!wallet.value) return
    if (!connection) return
    await getCandyMachineState(
      wallet.value as anchor.Wallet,
      props.candyMachineId,
      connection,
    )?.then((data) => {
      candyMachine = data?.candyMachine
      itemsAvailable.value = data?.itemsAvailable!
      itemsRedeemed.value = data?.itemsRedeemed!
      itemsRemaining.value = data?.itemsRemaining!
      goLiveDate.value = data?.goLiveDate!

      isSoldOut.value = itemsRemaining.value === 0
    })
  })()
}
const isWhiteListed = () => {
  if (wallet) return whitelist.includes(wallet.value?.publicKey.toString())
  else return false
}

const onMint = async () => {
  // check if wallet address is whitelisted
  // add toast on return to tell people they are not on the whitelist
  if (!isWhiteListed()) {
    return
  }
  try {
    isMinting.value = true
    if (wallet.value && candyMachine?.program) {
      const mintTxId = await mintOneToken(
        candyMachine,
        props.config,
        wallet.value?.publicKey!,
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
    <span v-if="!walletConnected">
      <button @click="connectWallet()">connect</button>
    </span>

    <div>
      <span>
        <input
          type="radio"
          id="sollet-rad"
          value="Sollet"
          v-model="walletChoice"
        />
        <label for="sollet-rad">sollet</label>
        <input
          type="radio"
          id="phantom-rad"
          value="Phantom"
          v-model="walletChoice"
          :disabled="!phantom"
        />
        <label for="phantom-rad" :class="{ 'label-disabled': !phantom }"
          >phantom</label
        >
      </span>
      <br />
    </div>
    <button
      v-if="walletConnected"
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
