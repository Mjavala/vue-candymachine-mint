<template>
  <WalletProvider :wallets="wallets" auto-connect>
    <wallet-modal-provider>
        <wallet-multi-button></wallet-multi-button>
    </wallet-modal-provider>
    <Home
      :treasury="treasury"
      :config="config"
      :candyMachineId="candyMachineId"
      :txTimeout="txTimeout"
    />
  </WalletProvider>
</template>

<script setup lang="ts">
import Home from './components/Home.vue'
import * as anchor from '@project-serum/anchor'
import { WalletProvider } from '@solana/wallet-adapter-vue'
import {
  getPhantomWallet,
  getSolletWallet,
} from '@solana/wallet-adapter-wallets'
import { WalletMultiButton, WalletModalProvider } from '@solana/wallet-adapter-vue-ui';

const wallets = [getPhantomWallet(), getSolletWallet()]

const treasury = new anchor.web3.PublicKey(
  process.env.VUE_APP_TREASURY_ADDRESS!,
)

const config = new anchor.web3.PublicKey(
  process.env.VUE_APP_CANDY_MACHINE_CONFIG!,
)

const candyMachineId = new anchor.web3.PublicKey(
  process.env.VUE_APP_CANDY_MACHINE_ID!,
)

const txTimeout = 30000 // milliseconds (confirm this works for your project)
</script>
