# HOPR-PL-Substrate

A payment layer implementation of HOPR in Substrate

# Objectives

dApps, pillars, missing hole

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

<table>
    <tbody>
        <tr>
            <td>Application(s)</td>
            <td>CLI/UI</td>
        </tr>
        <tr>
            <td rowspan=3><b>HOPR</b></td>
            <td><b>API</b><td>
        </tr>
        <tr>
            <td><b>Messaging layer</b></td>
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

# Payment Layer

## Principles

- Optimistic fair exchange:

  optimize for the happy case while sacrificing efficiency in the unhappy case

- Keep computation for consensus-related state changes as low as possible

  nodes do not need on-chain state changes for every relayed packet. On-chain state are only required once they stake funds or they initiate a payout for multiple relayed packets.
  On-chain state changes require only verification of hash values, signature verification, repeated application of the group operation, e.g. on elliptic curves

- Chain-agnostic

  payment layer and messaging layer do not depend on a specific chain. The only requirement is a general-purpose on-chain scripting programming language in combination with a sufficiently high instruction limit.

## Role of the blockchain resp. Polkadot

Each node in the network is able to alter its local state independently. It can then convince other nodes of the validity of the state change. Other nodes may then use the advertised state change as well as the global state upon which a consensus has been reached to check the plausibility. If a node found a state change appropriate, it will most likely perform the requested actions.

More precisely, in order to initiate and become part of the network, a HOPR node will lock a certain amount of funds and publish that state change to the network. Once it aims to send a message, it will create a local state change that moves a small amount of funds from its own account to the account of a node that will relay the message towards the desired recipient. That node checks now whether the sender has indeed locked enough funds for that value transfer. Furthermore, that node checks whether that state change is currently the most recent state that follows the previously known state. As HOPR nodes increment counters on every change, the first relayer will accept, when coming from state **n**, only a state change to state **n+1**. In particular, it will reject a second state change to state **n+1**.

Once in a while, nodes would like to merge their local state with the global state. Therefore, they publish their accumulated state changes and ask the blockchain network to verify that state change. The verifiers in the network will check whether the local state changes of that particular node have led to valid state.

HOPR makes use of special behavior of distributed ledger systems that they can force senders of transactions to reveal pre-images of one-way functions like hash functions or elliptic curve operations into a transaction even if only the result of the respective one-way function is necessary to verify the embedded signature.

## Techniques

Each hop possesses a key pair which is also used as an address of that node. Once a hop receives a packet, it multiplies the embedded curve point with its own private key and derives the intermediate keying material (IKM). The nodes will use that keying material to multiple derive keys that are necessary to process the packet.

### Secret-sharing

The IKM is used to derive a key share s_a per incoming packet and another key share s_b per outgoing packet. Outgoing key share s_b and incoming key share s_a of the next downstream node yield a key that will be used to the pre-image of the key that is necessary to redeem the money.

### Payment channel update transactions

concept payment channel nonce, time to present better transaction

### Signature verification

Every time a node sends a packet to the next downstream node, it creates an update transaction that updates the state of the payment channel. The transaction contains not only the amount that is transferrred but also an elliptic curve point. The signature is computed over the transaction data as well as the curve point, so once a node receives that transaction, it can verify whether the signature is correct without requiring any additional helping values. However, the on-chain application logic accepts that update transaction only if the sender of the transaction is able to present either a value that solves the discrete logarithm problem for that point or renounce a fraction of the received money.

The reason for this is that the payment between a node and the next downstream node relies on the acknowledgments that this node receives from those nodes to which it forwards the messages. More precisely, the settlement and therefore the payout depends on the behavior of third parties. 

% TODO: more details

### Verification of hash values

# Message Layer

## Acknowledgements
