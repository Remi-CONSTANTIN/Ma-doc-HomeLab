# Introduction
Lorem ipsum

# Linux Bridge VS OVS
## Linux Bridge
Lorem ipsum
## OVS
Lorem ipsum

# Bond

## Les différents types

- `Round-robin (balance-rr)`: Transmit network packets in sequential order from the first available network interface (NIC) slave through the last. This mode provides load balancing and fault tolerance.
- `Active-backup (active-backup)`: Only one NIC slave in the bond is active. A different slave becomes active if, and only if, the active slave fails. The single logical bonded interface’s MAC address is externally visible on only one NIC (port) to avoid distortion in the network switch. This mode provides fault tolerance.
- `XOR (balance-xor)`: Transmit network packets based on [(source MAC address XOR’d with destination MAC address) modulo NIC slave count]. This selects the same NIC slave for each destination MAC address. This mode provides load balancing and fault tolerance.
- `Broadcast (broadcast)`: Transmit network packets on all slave network interfaces. This mode provides fault tolerance.
- `IEEE 802.3ad Dynamic link aggregation (802.3ad)(LACP)`: Creates aggregation groups that share the same speed and duplex settings. Utilizes all slave network interfaces in the active aggregator group according to the 802.3ad specification.
- `Adaptive transmit load balancing (balance-tlb)`: Linux bonding driver mode that does not require any special network-switch support. The outgoing network packet traffic is distributed according to the current load (computed relative to the speed) on each network interface slave. Incoming traffic is received by one currently designated slave network interface. If this receiving slave fails, another slave takes over the MAC address of the failed receiving slave.
- `Adaptive load balancing (balance-alb)`: Includes balance-tlb plus receive load balancing (rlb) for IPV4 traffic, and does not require any special network switch support. The receive load balancing is achieved by ARP negotiation. The bonding driver intercepts the ARP Replies sent by the local system on their way out and overwrites the source hardware address with the unique hardware address of one of the NIC slaves in the single logical bonded interface such that different network-peers use different MAC addresses for their network packet traffic.


## Comment le mettre en place
1. Ajout des cartes réseaux
2. Création du bond
