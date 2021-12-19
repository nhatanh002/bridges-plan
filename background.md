Notes on chainbridges implementation approaches and design space
================================================================

* Approaches to bridge designs can be categorized by 3 axes:
  * Scope: 
    * application-specific:
      - only transfer crosschains values for a specific application
    * assets-specific:
      - only transfer certain assets among certain chains
    * chains(pair)-specific:
      - lock&mint/burn&release tokens between specific pair of chains. Quick to develop and deploy, but hard to scale
    * generalized:
      - full-fledged protocol designed for transferring information accross chains. Huge upfront cost, most flexible, most complex.
  * Cross-chain validators network:
    * "Mailbox"-validators:
      - On the source chain, there's a "mailbox" account monitored by a federation of
        validators/signers, When a transaction to the mailbox is acknowledged on the
        source chain, the validators perform actions accordingly on the destination chain,
        usually by minting an equivalent of the asset on the dest chain, example:
        [ChainSafe](https://github.com/ChainSafe/ChainBridge)
    * Light clients-relays:
      - Actors (not necessarily validators) must generate cryptographic proof about past
        events on the source chain, then forward, along with block headers, to dest
        chain's contracts, so the dest chain contracts can verify events on the source
        chain. Safer but more resource extensive. Actors can be contracts themselves,
        sending information to other chain's contracts by particular means, usually a
        relayers network. Example: [Gravity](https://github.com/cosmos/gravity-bridge),
        [Rainbow Bridge](https://github.com/aurora-is-near/rainbow-bridge)
    * Liquidity network:
      - Each node (router) in the liquidity network holds an inventory of assets of both
        source and dest chain, and provide value transferring by locking and exchanging:
        the routers provide native assets on each chain rather than derivatives.
        A prime example is [Connext](https://docs.connext.network/Integration/SystemOverview/howitworks).
    * Hybrid: Uses a combinations of features from different categories, or uses different
      models on different direction of crosschains communication.
  * Security model:
    * Trustless: no weaker than the underlying blockchains. Ideal but not realistic.
    * Insured: operating actors are required to post collateral to make sure cheating is not
      profitable. User's fund will be reimbursed through slashed collateral.
    * Bonded: malicious actors' collaterals are burned, user does not recover funds.
    * Trusted: dependent on the reputation of the bridge operator. Operating nodes are
      most likely closed and private.

Key management approaches
=========================

The problem: you are in charge of securely storing your users' private keys and use these to sign
transactions, and somehow prevent possible malicious actors inside of your own operation
to compromise the private keys. 

Of course, to be securely stored the private keys should be encrypted, so the problem
boils down to how to securely store the "master key" used to decrypt the private keys.
A stricter requirement would be to securely store the master key in a threshold scheme
such that it's only possible to reconstruct the secret master key with every share of the
secret master key available, i.e. the users' keys are secure as long as the whole
organization itself is not completely compromised, which is a moot point in a custodial
wallet scheme in the first place. References:
[Secret sharing](https://en.wikipedia.org/wiki/Secret_sharing), [Shamir's secret sharing](https://en.wikipedia.org/wiki/Shamir%27s_Secret_Sharing)

A less contrived and probably the usual approach is to store the master key in a medium
trusted to be secure, managed by a trusted party, and that trusted party supplies it to
the process doing the transaction signing at startup so it could decrypt the users'
private keys.
