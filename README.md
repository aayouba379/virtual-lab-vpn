# Virtual Lab VPN with IPsec on AWS

This project documents the design and validation of a virtualized AWS network secured with **IPsec at Layer 3** using **Libreswan**. The lab demonstrates how transport mode and tunnel mode can be used to protect traffic between hosts and gateways in a multi-subnet environment.

## Project Overview

I built a custom AWS VPC lab with four subnets and multiple EC2 instances to simulate a routed enterprise-style environment:

- **Client subnet** – endpoint network for the client host
- **WAN subnet** – transit network between the client-router and server-router
- **Server subnet** – endpoint network for the server host
- **Internet subnet** – public-facing management network

The routers were configured to forward traffic between subnets, and **source/destination check was disabled** on the routing instances so they could function as gateways. The environment was then secured with Libreswan-based IPsec tunnels. fileciteturn3file0

## Objectives

The main goals of the lab were to:

- design and deploy a multi-subnet virtual network in AWS
- configure Linux-based routing between client, router, and server segments
- implement IPsec using **IKEv2** and **pre-shared keys (PSK)**
- validate both **transport mode** and **tunnel mode** behavior
- observe encrypted traffic using **tcpdump** and **ESP captures**
- understand the difference between host-to-host, host-to-gateway, and gateway-to-gateway protection models

## Topology Summary

The core lab used the following logical addressing:

- **Client:** `10.0.1.102`
- **Client-Router LAN:** `10.0.1.101`
- **Client-Router WAN:** `10.0.2.101`
- **Server-Router WAN:** `10.0.2.102`
- **Server-Router LAN:** `10.0.3.101`
- **Server:** `10.0.3.102`

## IPsec Channels Implemented

### 1. Client-to-Server Transport Mode

A direct host-to-host IPsec configuration between the client and server.

- **Endpoints:** `10.0.1.102 ↔ 10.0.3.102`
- **Mode:** transport
- **Purpose:** protect the original IP packet directly between the two hosts

This phase showed how IPsec can secure end-to-end traffic without creating a gateway tunnel. The setup used Libreswan connection definitions in `/etc/ipsec.d/` and a shared PSK in `/etc/ipsec.secrets`. fileciteturn3file0

### 2. Client-to-Server-Router Tunnel Mode

A host-to-gateway VPN channel between the client and the server-side router.

- **Endpoints:** `10.0.1.102 ↔ 10.0.2.102`
- **Mode:** tunnel
- **Purpose:** encapsulate the original packet inside a new outer IP packet and terminate the secure tunnel at the gateway

Traffic validation showed ESP on the server-router WAN interface, confirming that the client traffic was encapsulated and delivered to the gateway as a tunnel endpoint. fileciteturn3file0

### 3. Gateway-to-Gateway Tunnel

A site-to-site style VPN between the client-router and server-router.

- **Endpoints:** `10.0.2.101 ↔ 10.0.2.102`
- **Protected subnets:** `10.0.1.0/24 ↔ 10.0.3.0/24`
- **Mode:** tunnel
- **Purpose:** secure routed traffic between the client and server LANs through the two gateways

This phase demonstrated the classic VPN model used to protect traffic between two remote networks. ESP was captured on both router WAN interfaces, confirming successful encryption and transit across the WAN segment. fileciteturn3file0

## Technologies Used

- **AWS VPC**
- **Amazon EC2**
- **Amazon Linux**
- **Libreswan**
- **IPsec / ESP / IKEv2**
- **tcpdump**
- **Linux routing and IP forwarding**

## Validation Approach

The project was validated through:

- interface and route verification with `ip addr`, `ip route`, and `ip route get`
- tunnel status checks with `ipsec status`
- packet inspection with `tcpdump`
- observation of **ESP** packets on WAN-facing interfaces
- end-to-end testing using `ping` across the protected paths

A key part of the lab was distinguishing between:

- **clear traffic on LAN interfaces**
- **encrypted ESP traffic on WAN interfaces**

That distinction made it possible to confirm when packets were being received in clear text, transformed by IPsec, and delivered through the tunnel.

## Key Takeaways

This lab gave me practical experience with:

- building a routed virtual network in AWS from scratch
- attaching the correct ENIs to match a custom topology
- enabling routers to function properly in EC2
- configuring Libreswan connection files and PSK-based authentication
- validating the difference between transport mode and tunnel mode
- troubleshooting real packet flow across multihomed Linux systems
- using packet captures to prove whether traffic was clear text or ESP-protected

## Full Report

The full technical write-up for this lab is included separately as the project report. It contains the detailed build process, configuration screenshots, and validation captures used to verify each IPsec channel.

## Resume Summary

If I had to summarize this project in one line for a resume:

> Built and validated an AWS-based virtual network security lab using Libreswan IPsec, implementing and troubleshooting host-to-host, host-to-gateway, and gateway-to-gateway secure channels across multiple routed subnets.
