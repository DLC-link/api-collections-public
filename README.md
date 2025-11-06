# CBTC User Flow

This is a collection of API calls that demonstrate a typical user flow for Minting, Burning, and Transferring CBTC on the Canton network using calls to the JSON-API of a participant node, and to various other APIs to collect necessary information.

There are two ways to use this collection:

1. Opening it in Yaak (https://yaak.app/), which allows you to run the requests directly from the app for testing and exploration.
2. Using it as a reference to implement the calls in your own client application.

### Steps to interact with the collection:

- clone this repo
- download and open Yaak: https://yaak.app/
- open the repository folder in Yaak as a workspace
- select the relevant environment from the dropdown in the top left corner
  - note that many fields will be encrypted, as they are 1. sensitive, 2. specific to your setup. Update these to your own values.
  - if you are a Bitsafe team member, you can find the encryption key in our secrets manager.

### Prerequisites

- a Canton participant node with JSON-API enabled
- a Canton user with appropriate OAuth setups (you can configure your OAuth settings in the Yaak environments)
- a Canton party that that user can readAs and actAs
- Bitcoin wallet and balance if you want to mint. Note that for Canton Devnet Bitsafe uses an internal regtest Bitcoin environment, which is not publicly accessible. We can provide you with access if needed. Testnet uses Testnet3 Bitcoin.

## Endpoints

### Setup & Configuration

**LedgerEnd**
`GET /v2/state/ledger-end`
Retrieves the current ledger end offset, used as a reference point for querying active contracts.

**Get Account Rules Contracts**
`POST /app/get-account-contract-rules`
Fetches necessary Rules (factory) contracts from Attestors for account creation on a specific Canton network chain.

**Get Token Standard Contracts**
`POST /app/get-token-standard-contracts`
Retrieves token standard contracts including burn/mint factory and instrument configuration for CBTC operations.

**Get Full ACS**
`POST /v2/state/active-contracts`
Queries all active contracts for the authenticated party with full details including created event blobs.

### Mint Flow

**Create Deposit Account**
`POST /v2/commands/submit-and-wait-for-transaction-tree`
Creates a new CBTC deposit account by exercising the `CBTCDepositAccountRules_CreateDepositAccount` choice. This requires disclosed Rules contracts from the Attestors. See above.

**Get Deposit Accounts**
`POST /v2/state/active-contracts`
Retrieves all CBTCDepositAccount contracts for the authenticated user.

**Get Bitcoin Deposit Address**
`POST /app/get-bitcoin-address`
Obtains the Bitcoin deposit address associated with a specific deposit account ID. The user sends Bitcoin to this address to mint CBTC.

**Send Bitcoin** (example only for demo purposes)
`POST {bitcoin_rpc_url}`
Sends Bitcoin to a specified address using Bitcoin RPC `sendtoaddress` method.

**Get All Holdings**
`POST /v2/state/active-contracts`
Queries all token holdings by filtering for contracts implementing the `Splice.Api.Token.HoldingV1:Holding` interface. Once the mint is successful, the user's holdings will reflect the newly minted CBTC.

### Transfer Flow

**Transfer**
`POST /v2/commands/submit-and-wait-for-transaction-tree`
Initiates a CBTC transfer by exercising the `TransferFactory_Transfer` choice, creating a transfer instruction from sender to receiver.

**Get TransferOffers**
`POST /v2/state/active-contracts`
Retrieves all pending TransferOffer contracts for the authenticated party.

**Get Accept ChoiceContexts**
`POST /api/token-standard/v0/registrars/{registrar}/registry/transfer-instruction/v1/{contractId}/choice-contexts/accept`
Fetches the necessary choice contexts and disclosed contracts required to accept a specific transfer instruction.

**Accept TransferOffer**
`POST /v2/commands/submit-and-wait-for-transaction-tree`
Accepts a pending transfer offer by exercising the `TransferInstruction_Accept` choice with the appropriate context and disclosed contracts.
