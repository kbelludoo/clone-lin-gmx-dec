# clone-lin-gmx-dec

Experimental i64-scale LIN clone of [gmx-io/gmx-contracts](https://github.com/gmx-io/gmx-contracts) Vault **adjustForDecimals** / **tokenToUsdMin** (`amount * 10^decMul / 10^decDiv`, USDG uses `USDG_DECIMALS=18` not `PRICE_PRECISION=30`).

- **LIN clone:** [`src/lin_gmx_dec.lin`](src/lin_gmx_dec.lin)
- **C11 oracle:** [`test/oracles/gmx_dec_c11.c`](test/oracles/gmx_dec_c11.c)
- **Proofs (results only) live in lin-open:** https://github.com/kbelludoo/lin-open
- **Event receipt:** `docs/events/EVENT_GMX_DEC_CLONE_LIN.rulel` in lin-open

This repository is the standalone LIN copy. It is **not** a GMX Vault, **not** GLP, and does **not** claim Solidity uint256 / mainnet / 30-decimal price parity. The kernel is the GMX V1 **decimals-normalize muldiv**. USDC 6-dec scaled to USDG 18-dec is `1e18`, not naive `1e6`. Treating USDC as 18-dec floors `tokenToUsdMin` to `0`. `PRICE_PRECISION=30` fail-closes (`pow10>18`). It is **not** GMX `validateLiquidation`, **not** GMX `getFundingFee`, **not** GMX `getNextFundingRate`, **not** GMX `getFeeBasisPoints` USDG tax/rebate, **not** Uniswap V2 `xy`.

Upstream is MIT. This LIN kernel is an original uint64-scale implementation of the published `adjustForDecimals` / `tokenToUsdMin` formula, not a verbatim Solidity dump. `adjustForDecimals` / `tokenToUsdMin` / `tokenDecimals` / `USDG_DECIMALS` are rejected by the Solidity transpiler (`REJ_SOLIDITY_GMX_DEC`).

Pinned upstream: commit `fe55c4ba8911ae2378b50bfb9bece5ee50cf2af8`, `contracts/core/Vault.sol` sha256 `0b49e265fc4c04d3eea74b07d698748050671a2df259f1faa8d05cabf2970f92` blob `4aef3c7dcf40b4657e7dfcef582958dc2d418126`.

Canonical goldens (uint64-scale, 128-bit muldiv):

- USDC `1e6` 6-dec to 18-dec: **1e18** vs naive ignore-decimals **1e6** (1e12x)
- WBTC `1e8` 8-dec to 18-dec: **1e18**
- WETH 18-dec to USDC 6-dec: **1e6**
- wrap trap `5e9 * 10^10 / 10^6`: 128-bit **5e13** vs naive i64 wrap
- `tokenToUsdMin` USDC `1e6` price `1e8` 6-dec: **1e8** vs treat-as-18 **0**
- `tokenToUsdMin` WBTC `1e8` price `40000e8` 8-dec: **40000e8**
- USDG_DECIMALS 18: **1e18**; PRICE_PRECISION 30: fail-closed **0**
