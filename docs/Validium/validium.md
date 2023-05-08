## Validium

Validiums are scaling solutions that use off-chain data and computation to improve throughput of the layer 1 chain.
It uses validity proof that enforces integrity of the transactions but does not store transactions on the main layer 1 chain. 
Proof of the off-chain computation is published to the layer 1 chain which can be used to validate off-chain transactions. 
This prevents invalid state transitions and increase security.

### What is Validium

Validium uses validity proofs but data is not stored on Ethereum chain which significantly increases transactions throughput.
Validity proof can come in from of "zero knowledge proofs" like _ZK-SNARK_ or _ZK-STARK_ in which one party can prove 
to another party that the given statement is true while the prover avoids disclosure of additional information apart from the fact that the statement is indeed true.
Validity of all transactions is enforced using validity proofs while data availability is kept off chain.
User can withdraw funds by providing a Merkle proof which can prove inclusion of the users withdrawal transaction and allow 
the on-chain contract to process withdrawal. Validium interact with the Ethereum with a collection of multiple contracts 
including _verification_ contract which verifies the validity proof when making state transitions and main _attestation_ contract 
that stores state commitments (Merkle data roots) submitted by the block producer.


## How to verify data availability on Ethereum

In order to verify data availability on the Ethereum it is necessary first to submit data transaction to Avail.
Submitting data availability transaction to Avail can be done using `polkadot.js`. 
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

`bridgeRouterEthAddress` deployed main data availability router contract

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

After successfully bridging data root to the main data availability attestation contract,
it is possible to prove that data is available on Avail by submitting a Merkle proof to the verification contract.
Fetching proof from Avail is possible with RPC call `kate_queryDataProof` for example `availApi.rpc.kate.queryDataProof(dataIndex, hashBlock);`
where `dataIndex` is index of the data (leaf) in the Merkle tree and `hashBlock` hash of the block. This RPC endpoint
returns `DataProof` object that can be used to prove data availability on Avail.
Example: 
```
async function getProof(availApi, hashBlock, dataIndex) {
    const dataProof = await availApi.rpc.kate.queryDataProof(dataIndex, hashBlock);
    console.log(`Fetched proof from Avail for txn index ${dataIndex} inside block ${hashBlock}`);
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

[TODO]
By submitting proof to the verification contract it is possible to verify that data is available on Avail.
Example of submitting a proof to the verification contract deployed on Ethereum and can be queried by calling 
data root membership function `validiumContract.checkDataRootMembership(blockNum, leafHash, dataIndex, proof);` where

`blockNum` number of the block for which data is checked

`leafHash` hash of the data leaf that membership is checked

`dataIndex` data index of the leaf of where the proof is

`proof` array of the merkle proofs to construct root

