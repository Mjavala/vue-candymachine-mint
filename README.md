# vue-candymachine

This repo is a modified version of Exiled Apes [candymachine](https://github.com/exiled-apes/candy-machine-mint) candy machine frontend that works with Vue 3.

Personally I don't like react, so I've wanted an alternative for a while.

Unfortunately the react ui [components](https://www.npmjs.com/package/@solana/wallet-adapter-react-ui-starter) are not usable here, so I've only implemented Sollet and Phantom wallets as radio buttons, heavily inspired by [ravoken](https://github.com/2501babe/revoken).

**This has only been tested on Devnet**

## Environment variables

To run the project, first rename the .env.example file at the root directory to .env and update the following variables:

``VUE_APP_CANDY_MACHINE_CONFIG=__PLACEHOLDER__``

This is a Solana account address. You can get the value for this from the .cache/temp file. This file is created when you run the metaplex upload command in terminal.

``VUE_APP_CANDY_MACHINE_ID=__PLACEHOLDER__``

Same as above; this is a Solana account address. You can get the value for this from the ./cache/temp file. This file is created when you run the metaplex upload command in terminal.

``VUE_APP_TREASURY_ADDRESS=__PLACEHOLDER__``

This the Solana address that receives the funds gathered during the minting process. More docs coming as we can test this.

``VUE_APP_CANDY_START_DATE=__PLACEHOLDER__``

This is a unix time stamp that configures when your mint will be open.

``VUE_APP_SOLANA_NETWORK=devnet``

This identifies the Solana network you want to connect to. Options are devnet, testnet, and mainnet.

``REACT_APP_SOLANA_RPC_HOST=https://explorer-api.devnet.solana.com``

This identifies the RPC server your web app will access the Solana network through.

