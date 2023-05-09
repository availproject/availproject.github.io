---
id: avail-verify-da
title: Verify data availability on Ethereum
sidebar_label: Verify DA on Ethereum
description: How to verify data availability on Ethereum
keywords:
- docs
- avail
- data availability
- da
image: https://availproject.github.io/img/avail/AvailDocs.png
slug: avail-verify-da
---
import useBaseUrl from '@docusaurus/useBaseUrl';

## Validium

Validiums are scaling solutions that use off-chain data and computation to improve throughput of the layer 1 chain.
It uses validity proof that enforces integrity of the transactions but does not store transactions on the main layer 1 chain. 
Proof of the off-chain computation is published to the layer 1 chain which can be used to validate off-chain transactions. 
This prevents invalid state transitions and increases security.

### What is Validium?

Validiums are scaling solutions that are using off-chain computation and validity proofs, but data is not stored on Ethereum chain which significantly increases transactions throughput.
Validity proof can come in from of zero knowledge proofs like _ZK-SNARK_ or _ZK-STARK_ in which one party can prove 
to another party that the given statement is true while the prover avoids disclosure of additional information apart from the fact that the statement is indeed true.
Validity of all transactions is enforced using validity proofs while data availability is kept off chain.
User can withdraw funds by providing a Merkle proof which can prove inclusion of the users withdrawal transaction and allow 
the on-chain contract to process withdrawal. Validium interact with the Ethereum with a collection of contracts 
including main _attestation_ contract that stores state commitments (Merkle data roots) submitted by the block produce and
_verification_ contract which verifies the validity proof when making state transitions.

## Verify data availability on Ethereum

In order to verify data availability on the Ethereum it is necessary to submit data transaction to the Avail network.
Submitting data availability transaction to Avail can be done using `Polkadot-JS` which is a collection of tools for communication with Substrate based chains. 
Example:
 ```
    async function submitData(availApi, data, account) {
        let submit = await availApi.tx.dataAvailability.submitData(data);
        return await sendTx(availApi, account, submit);
    }
   ```
Function `submitData` receives `availApi` api instance, `data` that will be submitted and the `account` which is sending the transaction.
In order to create account it is necessary to create keyring pair for the account that wants to send the data. 
This can be done with `keyring.addFromUri(secret)` which creates keyring pair via suri (the secret can be a hex string, mnemonic phrase or a string).
After creating keyring pair, it is possible to send data availability transaction `availApi.tx.dataAvailability.submitData(data);` to the Avail.
Once transaction is included in the block it is possible to dispatch data root by creating transaction
`availApi.tx.daBridge.tryDispatchDataRoot(destinationDomain, bridgeRouterEthAddress, header);` with the parameters:

`destinationDomain` 1000

`bridgeRouterEthAddress` Address of the main data availability router contract

`header` provided from the block when data is submitted

```
   async function dispatchDataRoot(availApi, blockHash, account) {
    const header = await availApi.rpc.chain.getHeader(blockHash);
    let tx = await availApi.tx.daBridge.tryDispatchDataRoot(process.env.DESTINATION_DOMAIN, process.env.DA_BRIDGE_ADDRESS, header);
    return await sendTx(availApi, account, tx);
   }
   ```
Dispatching data root will trigger the Nomad Bridge that will process data root. Since Nomad bridge is optimistic
bridge, it is necessary to wait for 30 minutes before the data root is available on the Ethereum network.
Complete example of submitting data to Avail can be found here: [todo link].

After successfully bridging data root to the main data availability attestation contract,
it is possible to prove that data is available on Avail by submitting a Merkle proof to the verification contract.
Fetching proof from Avail is possible with RPC call `kate_queryDataProof` for example `availApi.rpc.kate.queryDataProof(dataIndex, hashBlock);`
where `dataIndex` is index of the data (leaf) in the Merkle tree and `hashBlock` which is a hash of the block in which the data is included. 
This RPC endpoint returns `DataProof` object that can be used to prove data availability on Avail.
Example: 
```
async function getProof(availApi, hashBlock, dataIndex) {
    const dataProof = await availApi.rpc.kate.queryDataProof(dataIndex, hashBlock);
    return dataProof;
}
```
Returned data:
```
DataProof: {
   root: 'H256',
   proof: 'Vec<H256>',
   numberOfLeaves: 'Compact<u32>',
   leaf_index: 'Compact<u32>',
   leaf: 'H256'
}
```

`root` Root hash of generated merkle tree.

`proof` Merkle proof items (does not contain the leaf hash, nor the root).

`numberOfLeaves`  Number of leaves in the original tree.

`leaf_index` Index of the leaf the proof is for (starts from 0).

`leaf` Leaf for which is the proof.

By submitting proof to the verification contract [todo link to contract implementation] it is possible to verify that data is available on Avail.
Example of submitting a proof to the verification contract deployed on Ethereum, that can be queried by calling 
data root membership function `validiumContract.checkDataRootMembership(blockNumber, leafHash, dataIndex, proof);` where

`blockNumber` Number of the block for which data is checked

`leafHash` Hash of the data leaf that membership is checked

`dataIndex` Data index of the leaf of where the proof is

`proof` Array of the merkle proofs to construct root

This contract call will return `true` or `false` depending if the proof is valid or not.

