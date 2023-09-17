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
