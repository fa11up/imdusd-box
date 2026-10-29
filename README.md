# Box

Pay for an IdentityMD swarm request in imdUSD.

A user pays for an order that an off-chain evaluator quoted. In one call to `pay`:

- the cost of the order's first request goes to the **float** at once (an operator wallet that pays the swarm in IMD),
- a small buffer goes to the imdUSD **Treasury**,
- anything else (the later steps of a chained order) is held in the contract as **escrow**.

`Paid` carries `commitment = keccak256(request body)`, so the body later sent to the swarm can be checked against it.

Escrow leaves the contract only by:

- **`settle` / `settleMany`**: an EIP-712 `Settlement{key, toTreasury, toFloat, toPayer}` signed by `SIGNER`, which must
  add up to the order's escrow exactly. Anyone may execute one. The signer can only split one order's escrow among the
  Treasury, the float and that order's payer.
- **`reclaim`**: the payer takes the escrow back if no settlement was executed within 60 days of payment.

Orders are stored under `keyOf(payer, orderId)`, so another wallet cannot take an order id from the mempool.
There is no owner, no pause, no upgrade. The contract checks no prices: the operator sends only orders whose `Paid`
matches its own quote.

Constants on mainnet: imdUSD v1 `0x61aAF8a992A3143e2B0C9cB0030b9fE702854a25` (plain ERC-20, 18 decimals, no permit),
Treasury `0x6Ac8fF96Fb8DCF5d79A8eF1E59D9AC23f2BD27Bc`.


Tests:

    forge test --no-match-contract BoxForkTest                       # unit
    MAINNET_RPC_URL=<rpc> forge test --match-contract BoxForkTest    # same suite against live imdUSD
