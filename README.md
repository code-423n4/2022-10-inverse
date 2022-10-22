---

# Inverse Finance contest details
- $47,500 USDC main award pot
- $2,500 USDC gas optimization award pot
- Join [C4 Discord](https://discord.gg/code4rena) to register
- Submit findings [using the C4 form](https://code4rena.com/contests/2022-10-inverse-finance-contest/submit)
- [Read our guidelines for more details](https://docs.code4rena.com/roles/wardens)
- Starts October 25, 2022 20:00 UTC
- Ends October 30, 2022 20:00 UTC


## Fixed Rates Market Protocol Overview
FiRM is an over collateralized money market protocol for borrowing the DOLA stablecoin at a fixed price, over an arbitrary period of time. This is accomplished with the *Dola Borrowing Rights*token  (**DBR**) .
One DBR token gives the right to borrow one DOLA for one year. As time progresses, DBR will be burnt from the borrower's wallet at a rate that depends on their debt. A borrower may repay their loan at any time, and sell their DBR at a potential profit, if interest rates have gone up. DBR are minted and offered to the market by Inverse Finance. DBR liquidity and market making is outside the scope of this contest.

The DOLA stablecoin is Inverse Finance's own stablecoin, which has it's peg actively managed by a system of Fed contracts, that enact expansionary or contractionary monetary policy to keep the peg of the stable coin near 1$. The Fed attached to FiRM protocol markets lend DOLA to the markets by expanding supply, and contracts supply by recalling dola from the markets. DOLA is the only borrowable asset in the FiRM protocol.
## Architecture
Simplified overview of the FiRM architecture:

<img width="400" alt="SimplifiedArchitecture" src="https://user-images.githubusercontent.com/85371239/197343588-ddc1925a-94e2-4b9e-a91f-79310e00c4fe.png">

## Contracts

**Market.sol (SLOCs: 377)**
The market contract is the central contract of the FiRM protocol and contains most logic pertaining to borrowing and liquidations. A DOLA Fed mints DOLA to a market, which is then available to borrow for users holding DBR, using the Borrow function.

If a borrower's credit limit falls below the value of their outstanding debt, a percentage of their collateral may be liquidated on behalf of the protocol. The liquidation carries an additional fee, which will be paid out to the liquidator, and may benefit protocol governance as well.

Markets do not hold any collateral, but instead deposits and withdraws collateral from Escrows unique to each user and market.

External Contracts Called:
- ERC20 DOLA
- ERC20 Collateral

**DBR.sol (SLOCs: 228)**
The DBR token differs from standard ERC20 tokens in a few ways. It will actively be burnt from user's wallets, as they hold debt within the broader FiRM system, which can not just go to 0, but can drop into a deficit.

Since the burn rate is deterministic depending on the user's debt, it's only necessary to update the accrued debt whenever the borrower increase or decrease their debt position.

The borrower can be forced to replenish their DBR balance through a *forced replenishment*. Force replenishments will mint fresh DBR tokens to a user, at a price high enough that it's unnecessary to  query an oracle about the market value of DBR tokens. To pay for the forced replenishment, additional DOLA debt is accrued to the borrower. Forced replenishments can be initiated by anyone and will immediately pay out  a percentage of the debt accrued by the action to the caller.

External Contracts Called: None

**BorrowController.sol (SLOCs: 19)**
A borrow controller contract is connected to the market, which may add additional logic to who may borrow. In this implementation it's a simple contract allow list.

External Contracts Called: None

**Oracle.sol (SLOCs: 85)**
The Oracle is a pessimistic oracle, logging the lowest price of the day, whenever it's getPrice() function sees a new low price.
The Pessimistic Oracle introduces collateral factor into the pricing formula. It ensures that any given oracle price is dampened to prevent borrowers from borrowing more than the lowest recorded value of their collateral over the past 2 days.
This has the advantage of making price manipulation attacks more difficult, as an attacker needs to log artificially high lows.
It has the disadvantage of reducing borrow power of borrowers to a 2-day minimum value of their collateral, where the value must have been seen by the oracle.
External Contracts Called:
- ChalinkFeed feed

**Fed.sol (SLOCs: 83)**
Feds are a class of contracts in the Inverse Finance ecosystem responsible for minting DOLA in a way that preserves the peg and can't be easily abused. In the FiRM protocol, the role of the Fed is to supply and remove DOLA to and from markets. The Fed contract is essentially the only lender in the protocol.

External Contracts Called:
- ERC20 DOLA


#### Escrow Contracts
User's collateral are not held in the market contract, but are instead held in individual escrows. Every user has a unique escrow for every market. This allows for unique collateral interactions, like individually delegating votes for governance tokens. These are deployed as [minimal proxies](https://eips.ethereum.org/EIPS/eip-1167) to save gas. 

**SimpleERC20Escrow.sol (SLOCs: 22)**
Basic escrow implementation.
 External Contracts Called:
- ERC20 token
 
**INVEscrow.sol (SLOCs: 56)**
Token escrow that deposits INV into xINV tokens, and allows the depositor to delegate voting power in the Inverse DAO.
External Contracts Called:
- ERC20 token
- xINV xINV

**GovTokenEscrow.sol (SLOCs: 31)**
An example token escrow implementation that highlights the ability of escrow contracts to allow individual delegation of governance tokens used as collateral.
External Contracts Called:
- ERC20 token

### Special areas of concern for Wardens
We would like wardens to pay special attention to any issues that may:
1. Allow an attacker to borrow or transfer DOLA without fulfilling the necessary collateral requirements.
2. Allow an attacker to manipulate the price feed of the oracle, leading to undue liquidations or allowing them to borrow more than they should.
3. Allow an attacker to withdraw funds from other users, or lock user funds in escrows.
4. Allow an attacker to avoid paying their DBR deficit, beyond negligible dust amounts.
5. Cause accounting errors that may cause the contracts to automatically revert, either unintentionally or maliciously provoked.
6. Allow an attacker to bypass the price dampening funcitonality of the pessimistic oracle.

### Setup
FiRM is built using the Foundry framework. An installation guide can be found [here](https://book.getfoundry.sh/getting-started/installation). There are no other dependencies.
### Tests
The test suite can be run with a simple "`forge test`". "`forge test --gas-report`" can be run for a detailed gas report.  Additional test options can be found [here](https://book.getfoundry.sh/reference/forge/forge-test).
