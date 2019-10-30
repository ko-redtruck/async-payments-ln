# async-payments-ln
asynchronous Lightning Network payments

## Problem
Currently both parties have to be online at the same time to send/ receive money. This can be avoided by using something like the Lightning Rod Protocol by Breez (https://github.com/breez/LightningRod). However, funds have to be locked up longer than usual. We can do better than that. 

## Solution
The payer A pre-signs a transaction handing over money to their local channel partner S and sends the transaction to the payee B in an end to end encrypted communication channel. B can then sell the signature for the transaction to S using pay-for-signature made possible by payment points and scalars. (https://lists.linuxfoundation.org/pipermail/lightning-dev/2019-July/002077.html)

We will be using eltoo because we don't have to worry about toxic channel states.

A and S are online, A and S have a channel 
1. A contacts S: they commit and exchange the R (= k*G) part of the Schnorr Signature for the update and settlement transaction 
2. A and S sign the settlement transaction increasing the balance between them by amt + fee in favor of S and exchange the signatures
3. A signs the corresponding update transaction but does not give it so S 
4. A sends the update transaction to B using an end to end encrypted asynchronous communication channel

A can go offline
B comes online

4. Decrypts the update transaction and sells the signature s to S for amt   

When A comes back online S gives A the invoice signed (with the payment point s*G) by B, the corresponding scalar s (the signature for the update transaction) and its signature for the update transaction. They can now procede as normal.  

### Potential Issues

#### Privacy
S currently knows both the sender and the receiver of the payment. If we split the payment from S to B into two payments between S and public routing node P and P and B by still using the same scalar + payment point, S only knows the sender and P only knows the receiver. To further increase privavcy we can split the payment multiple times but all nodes involved must support this feature.

#### Locked up capital
While B hasn't yet claimed its money, the funds in the channel between A and S are essentially locked up. However, A and S could simply overwrite the payment (new update + settlement transaction), then A could send multiple payments with the remaining balance and before going offline A would do the procedure again. If A has sufficient inbound capacity in other channels it can also re-balance its channel A-S so that the outbound capacity - (amt + fees) in this channel is zero and then do the procedure.

#### Communication channel
Obviously the communication channel must be end to end encrypted otherwise this is highly insecure. Hopefully, we will have a sort of decentralized paid mail server system which is compatible across all LN wallets and part of BOLT.

#### Proof of payment 
The invoice by B with the payment point s*G and s is not sufficient as a PoP because S can simply give A the invoice and A already knows s.  

## the other way around
We can also do in a way that A can instantly send B (who is offline) money but now A is in charge of enforcing the channel state if S cheats. Because it has more issues like who pays the transaction fees if S cheats and because I believe Lightning is generally not designed for people who are offline for a long time I prefer the first one. But here is the other one:

B and S are online, B and S have a channel 
 
1. B and S sign a new settlement transaction increasing balance in favor of B by amt
2. B signs the corresponding update transaction, encrypts it and sends it together with the settlement transaction to A 

B can go offline
A comes online

3. A encrypts the transactions, A pays S to sign the update transaction which makes this new channel state valid/ enforceable
4. now A essentially acts like a watchtower while B is offline

## Conclusion

This enables truly asynchronous Lightning network payments and is yet another reason to move to payment points and scalars. In addition, this outsources the routing to S for the payment from A to B in the first scheme.


## Public Discussion on the mailing list
- https://lists.linuxfoundation.org/pipermail/lightning-dev/2019-October/002259.html
- https://lists.linuxfoundation.org/pipermail/lightning-dev/2019-October/002260.html
- https://lists.linuxfoundation.org/pipermail/lightning-dev/2019-October/002263.html
- https://lists.linuxfoundation.org/pipermail/lightning-dev/2019-October/002268.html
