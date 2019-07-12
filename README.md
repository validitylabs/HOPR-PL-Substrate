# HOPR-PL-Substrate

A payment layer implementation of HOPR in Substrate.

# Objectives

We see the Web3 as an ecosystem of decentralised asset systems like Polkadot and Ethereum, decentralised storage solutions like Filecoin or NuCypher as well as decentralised computation providers like Golem or Enigma. In this new ecosystem, there will be multiple decentralised applications (dApps) which will interact with these systems.

In order to deliver messages that are sent by dApps, most of them make at the moment use of Whisper which was developed by Ethereum community and suffers from certain shortcomings like missing scalability and unclear delivery behavior.

<table>
    <tbody>
        <tr>
            <td colspan=2 align="middle" width=33%><b>Assets</b>
            <br>Bitcoin, Ethereum, Polkadot, Cosmos</td>
            <td colspan=2 align="middle" width=33%><b>Storage</b>
            <br>Filecoin / IPFS</td>
            <td colspan=2 align="middle" width=33%><b>Computation</b>
            <br>Golem, Nucypher</td>
        </tr>
        <tr>
            <td colspan=3 align="middle"><b>dApp #1</b></td>
            <td colspan=3 align="middle"><b>dApp #2</b></td>
        </tr>
        <tr>
            <td colspan=6 align="middle"><b>Messaging</b><br>HOPR</td>
        </tr>
    </tbody>
</table>

We also see that messages exchanged by dApps will be of value and that its perception will reveal sensible information to potential attackers. For that reason, we are designing a messaging system that adds privacy on the network level and allows dApps to communicate with each other privately.

HOPR will fill the missing hole between p2p networks and dApps that exchange sensible information.

<table>
    <thead>
        <tr>
            <td width=20%><b>Layer</b></td> 
            <td width=60%><b>Purpose</b></td>
            <td width=20%><b>Examples</b></td>
        </tr>
    <tbody>
        <tr>
            <td>Application</td>
            <td>Application logic</td>
            <td>Chat app</td>
        </tr>
        <tr>
            <td>Storage / Sync</td>
            <td>Synchronisation of data, version management, medium-term message caching</td>
            <td><a href="https://matrix.org">Matrix</a></td>
        </tr>
        <tr>
            <td><b>Privacy</b></td>
            <td>Scalable, decentralised metadata protection. Incentivisations for packet relayers and short-term packet caching.</td>
            <td><b>HOPR</b></td>
        </tr>
        <tr>
            <td>P2P</td>
            <td>Overlay routing, NAT traversal</td>
            <td><a href="https://libp2p.io">libp2p</a>, <a href="https://webrtc.org">WebRTC</a></td>
        </td>
        <tr>
            <td>Network</td>
            <td>Underlay routing, congestion control</td>
            <td><a href="https://en.wikipedia.org/wiki/Internet_protocol_suite">TCP / IP</a>, <a href="https://en.wikipedia.org/wiki/QUIC">QUIC</a></td>
        </tr>
    </tbody>
</table>

# Architecture

HOPR is structured into multiple layers: an API facing applications that will be built with HOPR, a message layer which performs the cryptographic operations that ensure privacy of the network participants, as well as a payment layer that assembles HOPR with a distributed ledger system. HOPR will have an internal API that is used to build modules that connect HOPR nodes to DLT like Polkadot / Substrate and Ethereum.

<table>
    <tbody>
        <tr>
            <td>Application(s)</td>
            <td>CLI/UI</td>
        </tr>
        <tr>
            <td rowspan=3><b>HOPR</b></td>
            <td><b>API</b></td>
        </tr>
        <tr>
            <td><b>Message layer</b></td>
        </tr>
        <tr>
            <td><b>Payment layer</b></td>
        </tr>
        <tr>
            <td>Polkadot</td>
            <td>Distributed ledger</td>
        </tr>
    </tbody>
</table>

For details concerning the messaging layer and especially the packet format, we refer to the [HOPR repository](https://github.com/validitylabs/hopr).

# Payment Layer

Privacy comes from the service of others. We therefore allow parties to have serious business model in providing that kind of service to individuals.

The main architectural guidelines behind HOPR are:

- **simplicity**: The interactions with the blockchain should be as simple as possible to prevent mistakes in applications and to decrease the effort that is necessary to formally proof security properties of the system.
- **efficiency**: As on-chain computation is costly, we prevent expensive operations like NIZK proof verification whereever we can.

We also made a few design choices:
- optimisation for the **happy case**: We sacrifice some efficiency in the unhappy case for an extra small overhead in the happy case.
- **near-real-time** instead of real-time communication: Message delivery in the HOPR network will have a slightly higher latency than direct communication as this is necessary to ensure privacy but it will be strictly lower than *1 second*.

## Role of the blockchain resp. Polkadot for HOPR

Each node in the network is able to alter its local state independently. It can then convince other nodes of the validity of the state change. Other nodes may then use the advertised state change as well as the global state upon which a consensus has been reached to check the plausibility. If a node found a state change appropriate, it will most likely perform the requested actions.

More precisely, in order to initiate and become part of the network, a HOPR node will lock a certain amount of funds and publish that state change to the network. Once it aims to send a message, it will create a local state change that moves a small amount of funds from its own account to the account of a node that will relay the message towards the desired recipient. That node checks now whether the sender has indeed locked enough funds for that value transfer. Furthermore, that node checks whether that state change is currently the most recent state that follows the previously known state. As HOPR nodes increment counters on every change, the first relayer will accept, when coming from state *n*, only a state change to state *n+1*. In particular, it will reject a second state change attempt to state *n+1*.

Once in a while, nodes would like to merge their local state with the global state. Therefore, they publish their accumulated state changes and ask the blockchain network to verify that state change. The verifiers in the network will check whether the local state changes of that particular node have led to valid state.

HOPR makes use of special behavior of distributed ledger systems that they can force senders of transactions to reveal pre-images of one-way functions like hash functions or elliptic curve operations into a transaction even if only the result of the respective one-way function is necessary to verify the embedded signature.

## Techniques

Each hop possesses a key pair which is also used as an address of that node. Once a hop receives a packet, it multiplies the embedded curve point with its own private key and derives the intermediate keying material (IKM). The nodes will use that keying material to multiple derive keys that are necessary to process the packet.

### Secret-sharing

The IKM is used to derive a key share s_a per incoming packet and another key share s_b per outgoing packet. Outgoing key share s_b and incoming key share s_a of the next downstream node yield a key that will be used to the pre-image of the key that is necessary to redeem the money.

### Payment channel update transactions

HOPR uses payment channels between adjacent nodes in the network and routes payments along these edges. During the initialisation phase, each node will crawl the network in order to find others who are also willing to speak the HOPR protocol. They will then select from the set of received a subset of nodes with whom they establish a payment channel.

Initialising a payment channel means that two participants agree on a certain amount of assets from each of them and lock these assets until both of them agree on how to distribute them. In case they do not find a consensus, they always have the chance to restore the original distribution.

Once they agree on a new distribution, they sign a message that encodes the new state and store this transaction until either one of them initiates a payout or they agree on another distribution.

State changes are incrementally numbered such that an honest node will always be able to convince the distributed ledger from rightful most recent state of the payment channel. In order to give nodes that opportunity, each node will listen the payment channel closing event and are allowed within a predetermined amount of time to present a more recent transaction.

### Signature verification

Every time a node sends a packet to the next downstream node, it creates an *update transaction* that alters the state of the payment channel. The transaction contains not only the amount that is transferrred but also an elliptic curve point. The signature is computed over the transaction data as well as the curve point, so once a node receives that transaction, it can verify whether the signature is correct without requiring any additional helping values. However, the on-chain application logic accepts that update transaction only if the sender of the transaction is able to present either a value that solves the discrete logarithm problem for that point or renounce a fraction of the received money.

The reason for this mechanism is that the payment channel between a node and the next downstream node relies on the acknowledgments that this node receives from those nodes to which it forwards the messages. More precisely, the settlement and therefore the payout depends on the behavior of third parties. As this contradicts the principle of a payment channel between exactly two parties, both nodes need to be able to settle their payment channel even when others do not acknowledge the reception of packet in time.

# Message Layer

The goal of the payment layer is to incentivise operations in the message layer. In order to do that, we specify those actions we want to pay nodes for:
- correct transformation of packets such that the next downstream nodes is able to perform their transformations on the received packets
- delivery of the packet to the next node and caching the packet for a small amount of time, e. g. 2 hours, until the next downstream node is able to receive the packet.

The only party who can prove this is the next downstream node by acknowledging the reception and the validity of the packet. For that reason, the sender prepares while creating the whole packet several secrets that are derivable by the nodes along the path. These secrets are then used to create a two-out-of-two secret sharing between every two adjacent nodes along the path. The sender then applies a one-way function on the second key share and embeds that value in the part of the packet that is visible to the first node. This allows the first node to check whether the derived value and the value which it is going to receive from the next downstream node are sufficient to reconstruct the secret that is embedded in the secret sharing. It also allows the first node to challenge the second node for sending back the desired secret share. And in case the second node answers with an invalid acknowledgement, it gives the first node an evidence to prove towards the distributed ledger that the acknowledgement was invalid.
