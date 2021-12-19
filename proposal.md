Chainbridge solution
====================

For the time being, a reasonably sufficent design choice for the bridge would be the
Mailbox-Monitor approach (see *background.md* for more details), with contracts on both chains implementing
lock&mint/burn&release operations in both direction. The "mailbox" account on each chain
would also be the corresponding contract's account.

The ETH-to-Welups flow would be like this:
  1. User A chooses to transfer x ETHs to Welups chain, as x W_ETHs
     minted tokens representing x ETHs.
  2. A `send()` transaction to transfer x ETH from the user's wallet to the contract
     ("mailbox") account is initiated on Ethereum.
  3. The contract upon receiving x ETHs locks them and emits `Deposited` event
  4. The monitor upon hearing `Deposited` calls the contract on Welups to mint x W_ETHs tokens
    4.1. If succeeded, transfers those to user A's balance, goto 5.
    4.2. If failed, emits `MintFailure` event, the monitor then calls the contract on
         Ethereum to release A's x ETHs and sends them back
         to A, notifies A of the failure, end.
  5. A then uses y W_ETHs to make transactions on Welups, x - y = z W_ETHs remaining in
     A's possession
  6. A redeems z W_ETHs, the contract on Welups is called to burn z W_ETHs, then emits
     `Withdrawal` event
  7. Upon hearing `Withdrawal` event, the monitor calls the contract on Ethereum end to release z
     ETHs and sends them back to A's wallet
     7.1. If succeeded, end
     7.2. If failed, locks z ETHs again, emits `ReleaseFailure` event, the monitor then calls
          the contract on Welups to mint z W_ETHs and transfers back to A's Welups wallet,
          notifies A of the failure, end.
  
The other Welups-to-Eth direction should be the more or less the same.

(Chart coming soon...)

Key management solution proposal
================================

To quote the relevant section, for more details check out *background.md*:

>A less contrived and probably the usual approach is to store the master key in a medium
>trusted to be secure, managed by a trusted party, and that trusted party supplies it to
>the process doing the transaction signing at startup so it could decrypt the users'
>private keys.
