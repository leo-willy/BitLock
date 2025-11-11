# **BitLock Protocol**

### *Transform idle Bitcoin into productive capital through trustless collateralization.*

---

## **Overview**

**BitLock Protocol** enables Bitcoin holders to unlock liquidity without sacrificing long-term price exposure. By leveraging **Stacks** smart contracts, BitLock offers a **trustless, overcollateralized lending system** that allows users to lock BTC, mint stablecoins, and maintain full control of their underlying assets.

The protocol enforces **algorithmic risk management** through automated health checks, dynamic collateral ratios, and liquidation mechanisms — ensuring protocol solvency and user protection.

---

## **System Overview**

BitLock operates as a **non-custodial collateralized lending platform** where Bitcoin (and optionally STX) serves as collateral for stablecoin loans.
Users interact with the protocol through smart contract calls to:

1. **Deposit BTC as collateral**
2. **Request stablecoin loans** based on collateral ratios
3. **Repay loans** with accrued interest
4. **Trigger liquidation events** when positions fall below risk thresholds

A **price oracle feed** provides real-time BTC and STX pricing data to calculate collateralization levels and assess liquidation risks.

---

## **Core Components**

| Component                 | Description                                                                                |
| ------------------------- | ------------------------------------------------------------------------------------------ |
| **Collateral Management** | Handles BTC deposits and tracks total locked collateral.                                   |
| **Loan Engine**           | Manages creation, tracking, repayment, and status updates of active loans.                 |
| **Oracle Module**         | Feeds up-to-date BTC/STX prices to maintain accurate collateral ratios.                    |
| **Liquidation Mechanism** | Automatically closes undercollateralized positions to protect protocol solvency.           |
| **Admin Controls**        | Allows the contract owner to initialize, configure, and maintain core protocol parameters. |

---

## **Contract Architecture**

### **1. Platform Configuration**

Stores key operational parameters:

* `minimum-collateral-ratio` → Minimum ratio required for loan approval (e.g., 150%)
* `liquidation-threshold` → Ratio below which liquidation is triggered (e.g., 120%)
* `platform-fee-rate` → Protocol-level fee percentage

All configuration values are set by the **contract owner** upon initialization.

---

### **2. Loan Management**

Each loan is uniquely identified by a `loan-id` and tracked in the `loans` map:

| Field               | Type        | Description                                    |
| ------------------- | ----------- | ---------------------------------------------- |
| `borrower`          | `principal` | Loan owner                                     |
| `collateral-amount` | `uint`      | Locked BTC collateral                          |
| `loan-amount`       | `uint`      | Stablecoin amount issued                       |
| `interest-rate`     | `uint`      | Fixed interest per loan                        |
| `status`            | `string`    | Loan status (`active`, `repaid`, `liquidated`) |

Users can maintain multiple active loans (up to 10) via the `user-loans` index map.

---

### **3. Oracle Integration**

The `collateral-prices` map provides pricing data for supported assets:

```clarity
(define-map collateral-prices
  { asset: (string-ascii 3) }
  { price: uint })
```

Only the **platform administrator** can update oracle values using `update-price-feed`, ensuring reliable and verified data inputs.

---

### **4. Risk & Liquidation**

A loan’s health is continuously monitored using its **collateral ratio**:

```
collateral-ratio = (collateral-value / loan-amount) * 100
```

If this ratio falls below the `liquidation-threshold`, the protocol:

* Automatically triggers liquidation
* Marks the loan as `liquidated`
* Removes it from the borrower’s active loan list
* Adjusts the total BTC locked metric

---

## **Data Flow (Simplified)**

```
┌──────────────────────────┐
│   User deposits BTC      │
└──────────┬───────────────┘
           │
           ▼
┌──────────────────────────┐
│  Loan request submitted  │
│  - Collateral checked    │
│  - Oracle price fetched  │
└──────────┬───────────────┘
           │
           ▼
┌──────────────────────────┐
│   Loan record created    │
│   Funds issued to user   │
└──────────┬───────────────┘
           │
           ▼
┌──────────────────────────┐
│  Loan monitored for risk │
│  - Collateral ratio calc │
│  - Auto-liquidation if   │
│    below threshold       │
└──────────┬───────────────┘
           │
           ▼
┌──────────────────────────┐
│  User repays loan        │
│  Collateral released     │
└──────────────────────────┘
```

---

## **Public Functions**

| Function                                      | Description                                            |
| --------------------------------------------- | ------------------------------------------------------ |
| `initialize-platform`                         | Initializes protocol for the first time.               |
| `deposit-collateral(amount)`                  | Deposits BTC into the protocol.                        |
| `request-loan(collateral, loan-amount)`       | Opens a new loan position.                             |
| `repay-loan(loan-id, amount)`                 | Repays principal and interest, marking loan as repaid. |
| `update-price-feed(asset, price)`             | Updates oracle feed (admin-only).                      |
| `update-collateral-ratio(new-ratio)`          | Updates minimum collateral ratio (admin-only).         |
| `update-liquidation-threshold(new-threshold)` | Updates liquidation ratio (admin-only).                |

---

## **Security Considerations**

* **Authorization Control**: Only the contract owner can perform configuration or oracle updates.
* **Collateral Verification**: Loans are approved only if the collateral-to-loan ratio exceeds the minimum threshold.
* **Automated Liquidation**: Protects protocol liquidity during market volatility.
* **Immutable Loan Tracking**: Each loan is uniquely indexed and verifiable on-chain.

---

## **Future Enhancements**

* Integration with **decentralized oracle networks** (e.g., Chainlink, Stacks Oracle)
* Support for **multi-collateral types** (BTC, STX, sBTC)
* Implementation of **dynamic interest rates**
* **Reward mechanisms** for liquidators and protocol participants

---

## **License**

This project is open-sourced under the **MIT License**.
Use and modify freely with attribution.
