# HOPR-PL-Substrate
A payment layer implementation of HOPR in Substrate

# Payment Layer
## Principles
- Optimistic fair exchange:

   optimize for the good case while sacrificing efficiency in the unhappy case

- Keep computation for consensus-related state changes as low as possible

   nodes do not need on-chain state changes for every relayed packet. On-chain state are only required once they stake funds or they initiate a payout for multiple relayed packets.
   On-chain state changes require only verification of hash values, signature verification, repeated application of the group operation, e.g. on elliptic curves

- Chain-agnostic

   payment layer and messaging layer do not depend on a specific chain. The only requirement is a general-purpose on-chain scripting programming language in combination with a sufficiently high instruction limit.

## Role of the blockchain resp. Polkadot
Each node in the network is able to alter its local state independently. It can then convince other nodes of the validity of the state change. Other nodes may then use the advertised state change as well as the global state upon which a consensus has been reached to check the plausibility. If a node found a state change appropriate, it will most likely perform the requested actions.

More precisely, in order to initiate and become part of the network, a HOPR node will lock a certain amount of funds and publish that state change to the network. Once it aims to send a message, it will create a local state change that moves a small amount of funds from its own account to the account of a node that will relay the message towards the desired recipient. That node checks now whether the sender has indeed locked enough funds for that value transfer. Furthermore, that node checks whether that state change is currently the most recent state that follows the previously known state. As HOPR nodes increment counters on every change, the first relayer will accept, when coming from state **n**, only a state change to state **n+1**. In particular, it will reject a second state change to state **n+1**.

Once in a while, nodes would like to merge their local state with the global state. Therefore, they publish their accumulated state changes and ask the blockchain network to verify that state change. The verifiers in the network will check whether the local state changes of that particular node have led to valid state. 

HOPR makes use of special behavior of distributed ledger systems that they can force senders to reveal pre-images of one-way functions like hash functions or elliptic curve operations into a transaction even if the result of the respective one-way function is necessary to verify the embedded signature.

## Techniques
### Secret-sharing
### Signature verification
### Verification of hash values
### Verification of group operations

# Message Layer
## Acknowledgements