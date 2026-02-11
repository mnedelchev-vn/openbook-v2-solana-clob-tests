# OpenBook V2 _( On-chain CLOB on Solana )_

A central-limit order-book program based on [Mango V4](https://github.com/blockworks-foundation/mango-v4) and the [previous OpenBook program](https://github.com/openbook-dex/program) (which was a fork of [Serum](https://github.com/project-serum/serum-dex)).

## Deployed versions
| network | program ID                                  |
| ------- | ------------------------------------------- |
| Devnet  | [4euFSfDacDh7XauCX1mQ6dxyS48WFoykMTLVsWfjSncz](https://solscan.io/account/4euFSfDacDh7XauCX1mQ6dxyS48WFoykMTLVsWfjSncz?cluster=devnet) |

## Testing
### Orderbook state changes
1. `ts-node tests/createMarket.ts` - Deploying 2 new SPLTokens and creating a market between in the OpenBook V2 program. After execution is finished now place the `market`, `quoteMint` and `baseMint` addresses at ***test/config.ts***
2. `ts-node tests/createOpenOrders.ts` - creation of Open Orders Account for a user on a specific market. After command is finished place the `openOrders` address at ***test/config.ts.*** Only the maker needs such account, the taker doesn’t have to create such account if he is only taking orders *( he will need such account if he decides to become maker as well )*.
3. `ts-node tests/placeOrder.ts` - A maker places *Limit* **Ask** order inside the orderbook.
4. `ts-node tests/cancelOrder.ts` - cancels order from the order book by order ID. Currently it always cancels the first active order in the list of user’s orders. The order ID is the one used by the time of creation of order - `PlaceOrderArgs.clientOrderId`.
5. `ts-node tests/cancelAllOrder.ts` - canceling all open orders inside the orderbook for a specific market for a specific user.
6. `ts-node tests/placeTakeOrder.ts` - A taker places *Market* **Bid** order inside the orderbook which fills the order created from `ts-node tests/placeOrder.ts`.
7. `ts-node tests/consumeEvents_settleFunds.ts` - consumes recent events recorded in the memory heap through `consumeEventsIx` instruction and withdraws all pending free funds from the openOrders account to the user’s balances through instruction `settleFundsIx`
8. `ts-node tests/depositToOpenOrdersAccount.ts` - depositing base and/ or quote token amounts to open position *( `openOrdersAccount` )*
### Orderbook getters
1. `ts-node tests/getters/getMarkets.ts` - fetching all of available markets. *( free Solana devnet RPC overflow requests limit )*
2. `ts-node tests/getters/getMarketTotalAmounts.ts` - returning the total bids and asks values for a specific market.
3. `ts-node tests/getters/getUserAccountFreeBalance.ts` - returning the base and quote balances for a open orders account - these balances do not participate in the orders yet, but are inside the position account.
4. `ts-node tests/getters/getUserOpenOrders.ts` - returning the list of all orders for a open orders account.

## Deploying
1. `yarn install`
2. `yarn build`
3. `solana config set --url https://api.devnet.solana.com`
4. Get yourself devnet SOLs _( if you already have Devnet SOLs skip this step )_:
    ```jsx
    curl https://api.devnet.solana.com -X POST -H "Content-Type: application/json" --data '{"jsonrpc":"2.0", "id":1, "method":"requestAirdrop", "params": ["8HzCjhBNP3rs7SydUrZAiQGEoqXHNtpNPE475zzHmzba", 100000000000]}'
    ```
5. `anchor build -- --features enable-gpl` *( adding `-- --features enable-gpl` is necessary, because the OpenBook V2 program has GPL licensing requirements )*
6. `solana address -k target/deploy/openbook_v2-keypair.json` and place the address to:
    1.  ***Anchor.toml***
    2. ***programs/openbook-b2/src/lib.rs***
    3. ***test/config.ts***
7. Re-build again with updated address `anchor build -- --features enable-gpl`
8. `anchor deploy --provider.cluster https://api.devnet.solana.com`
9. `solana program show {PROGRAM_ID} --url https://api.devnet.solana.com` - validating that program deploy was successful
10. ***[Optional]*** Verify the IDL in the Solana explorer - `anchor idl init -f target/idl/openbook_v2.json {PROGRAM_ID} --provider.cluster https://api.devnet.solana.com`

### Pre-requisites
Before you can build the program, you will first need to install the following:
- [Rust](https://www.rust-lang.org/tools/install)
- [Solana](https://docs.solana.com/cli/install-solana-cli-tools)
- [Anchor](https://www.anchor-lang.com/docs/installation) (v0.28.0)
- [Just](https://github.com/casey/just#installation)