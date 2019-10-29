# async-payments-ln
asynchronous payments over the Lightning Network

## Problem
Currently both parties have to be online at the same time to send/ receive money. This can be avoided by using something like the Lightning Rod Protocol by Breez (https://github.com/breez/LightningRod). However, funds have to be locked up longer than usual. We can do better than that. 

## Solution
We can pre sign transaction 
The payer A pre-signs a transaction handing money over to their local channel partner S and sends it to the payee B in an end to end encrypted communication channel. Then B can sell the signature without trust to S using pay-for-signature made possible by payment points and scalars.

We will be using eltoo because we don't have to worry about toxic channel states.

A and S are online 
1. A contacts S: they commit and exchange R part of the Schnorr Signature (2 communication rounds)
2. A pre signs transaction increasing the balance between them by amt + fee in favor of S  
3. send it to B using an end to end encrypted asynchronous communication channel

A can go offline
B comes online

4. Sell signature to S  
