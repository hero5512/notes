Here is your original article organized and summarized as English notes:

---

## **Privacy-Pools and the Concept of "Proof-of-Innocence"**

---

### **Background**

**TornadoCash** is a well-known crypto mixer that uses **Zero-Knowledge Proofs (ZKPs)** to hide the source of funds. While it enhances privacy, the U.S. government argues that it facilitates money laundering and sanctioned it in August 2022. Despite the frontend and GitHub being taken down, the smart contracts remained live on-chain. Eventually, the GitHub repo was restored after efforts by the EFF.

This leads to a key question:
**Can we design a system that provides privacy while preventing illicit activity?**

A possible answer comes from **Privacy-Pools**, proposed by **ameen.eth**, an early TornadoCash developer.

---

### **TornadoCash Mechanism (Recap)**

* TornadoCash uses **commitments**, which are a hash of a `secret` and a `nullifier`.
* Commitments are stored in a **Merkle Tree**, enabling users to prove their deposits without revealing the specific deposit.
* **Zero-Knowledge Proofs** ensure anonymity.
* **Nullifiers** prevent double-spending.

A **TornadoCash receipt** serves two purposes:

1. Proves that the sender made a deposit.
2. Ensures each note can be withdrawn only once.

![tornado_cash.png](images%2Ftornado_cash.png)!

---

### **The Problem**

Withdrawals are anonymous—no one can tell which specific deposit the funds came from. This allows bad actors to launder money undetected.

---

### **Introducing: "Proof-of-Innocence"**

A new proof can be added to **prove that the withdrawal is *not* associated with blacklisted (illicit) deposits**.

There are two approaches:

1. **Allowlist-based**: Prove funds come from a known set of legal deposits.
2. **Denylist-based**: Prove funds do *not* come from a known set of illicit deposits.

Privacy-Pools adopts the **allowlist** method.

![privacy_pool.png](images%2Fprivacy_pool.png)

---

### **Privacy-Pools Design**

In addition to TornadoCash’s original mechanisms, Privacy-Pools adds a **third component** to receipts:

> **Proves that the withdrawal comes from an allowed deposit.**

This is done using an **Allow Merkle Tree**, which:

* Has the same structure and index positions as the Deposit Merkle Tree.
* Contains `allowed` or `blocked` flags per leaf.
* Uses a custom `subsetRoot` (Allow Merkle Root) provided by the user.

---

### **How It Works**

#### 1. **Allowlist Withdrawal**

* The user proves their deposit exists in the Deposit Merkle Tree.
* They also provide a ZKP proving their deposit is marked as `allowed` in the Allow Merkle Tree (subset tree).
* If the proof passes, the withdrawal is allowed.

![allowlist.png](images%2Fallowlist.png)

#### 2. **Denylist Scenario**

* If a deposit is marked `blocked`, it cannot be used to produce a valid proof for the default Allow Merkle Tree (e.g., maintained by U.S. regulators).
* The user must use a different `subsetRoot` to withdraw funds—possibly one they control.

**Result**: Regulators can infer whether the withdrawn funds come from allowed or blocked sources based on the `subsetRoot` used.

![deny_list.png](images%2Fdeny_list.png)

---

### **Key Tradeoff: Decentralization vs. Enforcement**

* Since **subsetRoots are user-provided**, Privacy-Pools **does not prevent** illicit users from withdrawing.
* However, it ensures that **such withdrawals are traceable and flagged** based on the origin of the subsetRoot.
* This preserves **decentralization**, as no single party controls withdrawals.

---

### **Common Questions**

**Q1: Can a user forge a fake allowlist root to withdraw?**

**Yes**. But such withdrawals are traceable and likely suspicious. This preserves decentralization while allowing regulators to analyze on-chain behavior.

**Q2: Who determines whether a deposit is "blocked"?**

Each regulatory body or third-party organization may maintain their own allowlist. Multiple allow trees can coexist.

---

### **Privacy-Pools Circuit Code Overview**

* Written in Circom: `withdraw_from_subset.circom`
* Inputs:

    * `root`: deposit Merkle root
    * `subsetRoot`: allow Merkle root
    * `nullifier`: to prevent double spend
    * ZKP metadata and paths for both trees
* Includes double Merkle proof logic to validate both:

    * Deposit Merkle Tree
    * Allow Merkle Tree (with allowed leaf)

```circom
doubleTree.leaf <== hasher.commitment;
doubleTree.path <== path;
doubleTree.mainProof[i] <== mainProof[i];
doubleTree.subsetProof[i] <== subsetProof[i];

root === doubleTree.root;
subsetRoot === doubleTree.subsetRoot;
```

---

### **TL;DR**

* **Proof-of-Innocence** proves that funds come from *allowed* deposits.
* **Privacy-Pools** augments TornadoCash by enforcing this through a second Merkle Tree (Allowlist).
* The system enables **regulatory traceability** without compromising **decentralization**.
* ZKPs are used not only for hiding info, but also to **prove compliance**.
* The openness of subsetRoots means **users are free to withdraw**, but withdrawals from unknown or suspicious roots can be flagged.
