# async-payments-ln
asynchronous Lightning Network payments

## Problem
Currently both parties have to be online at the same time to send/ receive money. This can be avoided by using something like the Lightning Rod Protocol by Breez (https://github.com/breez/LightningRod). However, funds have to be locked up longer than usual. We can do better than that. 

## Solution
The payer A pre-signs a transaction handing money over to their local channel partner S and sends it to the payee B in an end to end encrypted communication channel. Then B can sell the signature to S using pay-for-signature made possible by payment points and scalars.

We will be using eltoo because we don't have to worry about toxic channel states.

A and S are online, A and S have a channel 
1. A contacts S: they commit and exchange the R (= k*G) part of the Schnorr Signature for the update and settlement transaction 
2. A and S sign the settlement transaction increasing the balance between them by amt + fee in favor of S and exchange the signatures
3. A pre signs the corresponding update transaction but does not give it so S 
4. A sends the update transaction to B using an end to end encrypted asynchronous communication channel

A can go offline
B comes online

4. Decrypts the update transaction and sells it to S for amt (https://lists.linuxfoundation.org/pipermail/lightning-dev/2019-July/002077.html)  

### Issues

#### Privacy
S currently knows both the sender and the receiver of the payment. If we split the payment from S to B into two payments between S and public routing node P and P and B by still using the same scalar + payment point, S won't know the receiver anymore. We can split the payment multiple times but all nodes involved must support this feature.

#### Locked up capital
While B hasn't yet claimed its money, the funds in the channel between A and S are essentially locked up. However, A and S could simply overwrite the payment (new update + settlement transaction), then A could send multiple payments with the remaining balance and before going offline A would do the procedure again. If A has sufficient inbound capacity in other channels it can also re-balance its channel A-S so that its outbound capacity - (amt + fees) in this channel is zero and then do the procedure.

#### communication channel
Obviously the communication channel must be end to end encrypted otherwise this is highly insecure. Hopefully, we will have a sort of decentralized paid mail server system which is compatible across all LN wallets and part of BOLT.

## the other way around
We can also do in a way that A can instantly send B (who is offline) money but now A is in charge of enforcing the channel state if S cheats. Because it has more issues like who pays the transaction fees if S cheats and because I believe Lightning is generally not designed for people who are offline for a long time I prefer the first one. But here is the other one:

B and S are online, B and S have a channel 
 
1. B and S sign new settlement transaction increasing balance in favor of B by amt
2. B signs corresponding update transaction, encrypts it and sends it and the settlement transaction to A 

B can go offline
A comes online

3. A encrypts the transactions, A pays S to reveal the missing part of the update transaction
4. now A essentially acts like a watchtower while B is offline

## Conclusion

This enables truly asynchronous Lightning network payments and is yet another reason to move to payment points and scalars. In addition, this outsources the routing to S for the payment from A to B in the first scheme.
