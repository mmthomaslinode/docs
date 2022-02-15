---
title: VLANs
linkTitle: VLANs
description: "Linode VLANs (Virtual Local Area Network) is a feature that enable secure and private communications across Linodes within the same Data Center region."
tab_group_main:
    is_root: true
    title: Overview
    weight: 10
cascade:
    date: 2020-10-22
    product_description: "Linode VLANs (Virtual Local Area Network) is a feature that enable secure and private communications across Linodes within the same Data Center region."
---

## Availability

Atlanta, GA, USA; Mumbai, India; Sydney, Australia; Toronto, Canada;

## Features

### Secure and Private Communication

Create a private L2 network and attach your Linodes to it for secure and private communication in the cloud. Two or more Linodes connected via the Private VLAN can see each other as if they were directly connected to the same physical Ethernet network. Devices outside of the network cannot see any traffic within the private network.

### Simple Configuration

Use the Cloud Manager to create a VLAN and attach Linode's to it in just a few clicks. Create up to 10 VLANs per data center, and apply up to 3 VLANs to a single Linode. For more fine-grained control, use the Linode APIv4 or Linode CLI.

### Reduce Network Transfer Costs

Private network transfer is free. Any communication between Linodes over the private VLAN does not count against your [monthly network transfer usage](/docs/guides/network-transfer/).

## Pricing

VLANs are free to use. Communication across your private network does not count against your [monthly network transfer usage](/docs/guides/network-transfer/).


{{< content "vlans-limitations-shortguide" >}}