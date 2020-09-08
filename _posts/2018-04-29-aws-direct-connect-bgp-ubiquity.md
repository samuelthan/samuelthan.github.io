---
title: 'AWS Direct Connect via BGP with Ubiquiti EdgeRouter'
date: 2018-04-29 00:00:00
featured_image: '/images/post/AWS_BGP_Direct_Connect.png'
excerpt: Documentation on configuring Ubiquiti EdgeRouter for BGP and AWS Direct Connect
---

The Ubiquiti EdgeRouter is a remarkable product for it's affortable price, the following example showcases an approach to establish a BGP connection for AWS Direct Connect service.

**Goal**: Establish connection between Ubiquiti EdgeRouter X with AWS Direct Connect via BGP


__Example Environment__
```
VLAN: 100
Upstream ASN: 7000
Carriers IP: 10.0.105.1/30 “Neighbor”
Customer IP: 10.0.105.2/30
Customer ASN: 65105
Customer network to announce to the upstream: 10.0.10.0/24
```

Firstly setup the relevant access control list (acl), these will be governing what the connection’s ingress and egress can or cannot communicate.

Upstream means Amazon network in our context.

Allow all traffic from Upstream into Customer’s network
```
set policy prefix-list IMPORT-AS65105 rule 10 action permit
set policy prefix-list IMPORT-AS65105 rule 10 description ALLOW-DEFAULT-ROUTE
set policy prefix-list IMPORT-AS65105 rule 10 prefix 0.0.0.0/0
set policy prefix-list IMPORT-AS65105 rule 10 le 32
commit
```

Allow network from customer into Upstream’s network

```
set policy prefix-list EXPORT-AS65105 rule 10 action permit
set policy prefix-list EXPORT-AS65105 rule 10 description “Announce 10.0.10.0/24”
set policy prefix-list EXPORT-AS65105 rule 10 prefix 10.0.10.0/24
commit
```

Setup the actual configuration using the previously created acl

```
set protocols bgp 65105 parameters router-id 10.0.105.1
set protocols bgp 65105 neighbor 10.0.105.2 remote-as 7000
set protocols bgp 65105 neighbor 10.0.105.2 password XXXXX
set protocols bgp 65105 neighbor 10.0.105.2 soft-reconfiguration inbound
set protocols bgp 65105 parameters log-neighbor-changes
set protocols bgp 65105 neighbor 10.0.105.2 prefix-list export EXPORT-AS65105
set protocols bgp 65105 neighbor 10.0.105.2 prefix-list import IMPORT-AS65105
set protocols bgp 65105 neighbor 10.0.105.2 update-source 10.0.105.1
set protocols bgp 65105 network 10.0.10.0/24
commit
```

```
set protocols static route 10.0.10.0/24 blackhole
commit
```