# Network Policy Markup Language

A proposal of an XML markup language suitable to express host and subnet TCP/IP connectivity requirements accross multiple subnets of quite complex local area network, optionally connected to the Internet.

## The Problem

As a network administrator, you are continually facing a challenge how to make a computer network provide all the required services to its users and have it adequately well secured at the same time, which means you have to ensure all network hosts are allowed to access the required services only, but no more.

Each day in any organization, new requests for network connectivity adjustments are coming and firewall rules are slowly turning into a huge mess over time. If you are required to manage more than single network firewall at the same time, the problem is made even more challenging.

When I have been thinking about the firewall management problem mentioned above, at first I tried to search for an off-the-shelf solution, but I haven't found anything suitable for my situation. Some solutions were too complex, economically unaffordable and almost impossible to evaluate at all. Most solutions were operating at the firewall level of abstraction, which does not make the problem easier to manage at all.

After some more thinking about the situation, I decided to design my own solution, hopefully being more flexible and without too much overhead of the off-the-shelf products I have researched previously.

## Example Network Topology

For the purpose of demonstration, I have designed a simple but representative network topology, which, in the next sections, I will use to infer a network policy described with a markup language I propose in the project:

![Example Network Topology](/Diagrams/ExampleNetworkTopology.svg)

## Example Connectivity Requirements

Users in the example network have various, somewhat typical connectivity requirements. These requirements are summarized in a form of a list specified below:

* All hosts of the subnet **10.1.0.0/24** can access:
  * web services available at the **sr-primary** server
  * storage services available at the **sr-secondary** server
  * outbound services on the Internet via the **1.2.3.0/28** subnet
* All hosts of subnet **10.2.0.0/24** can access:
  * outbound services on the Internet via the **1.2.3.0/28** subnet
  * remote desktop services of the **10.1.0.0/24**
* All hosts on the Internet can access via the **1.2.3.0/28**:
  * web services available at the **sr-primary** server
* The **pc-one** workstation can access:
  * remote desktop services of all hosts at the **10.2.0.0/24** subnet
  * remote console service to the **rt-edge** router
* The **pc-two** workstation can access:
  * remote console service to the **rt-edge** router

To have a clearer understanding what particular services specified in the requirements do really mean in the sense of network protocols and ports, I have summarized them in a more comprehensive list:

* **web services**
  * HTTP (TCP port 80)
  * HTTPS (TCP port 443)
* **storage services**
  * FTP control (TCP port 21)
  * FTP passive (TCP ports 52100-52199)
  * SMB (TCP port 445)
* **remote desktop services**
  * RDP over TCP (TCP port 3389)
  * RDP over UDP (UDP port 3389)
* **remote console services**
  * SSH (TCP port 22)
  * Telnet (TCP port 23)
* **outbound services**
  * HTTP (TCP port 80)
  * HTTPS (TCP port 443)
  * DNS over UDP (UDP port 53)
  * DNS over TCP (TCP port 53)

There are three private and one public subnet explicitly mentioned in the requirements and/or in the network topology diagram:

* **net-clients** – for workstations
  * 10.1.0.0/24 (private)
* **net-services** – for servers
  * 10.2.0.0/24 (private)
* **net-backbone** – interconnecting all routers
  * 10.3.0.0/16 (private)
* **net-public** – connecting the Internet
  * 1.2.3.0/28 (public)

In the following sections I will try to demonstrate the syntax, the usage and the overall benefits that the proposed network policy markup language can bring to the hands of network administrators.