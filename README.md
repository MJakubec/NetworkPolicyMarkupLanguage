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

## Basic Structure of Network Policy Definition

Basic document structure of each network policy definition looks like this:

```
<?xml version="1.0" encoding="utf-8"?>
<networkPolicy xmlns="urn:asbest-network-policy-1.0">
  <protocols>...</protocols>
  <services>...</services>
  <subnets>...</subnets>
  <hosts>...</hosts>
  <targets>...</targets>
</networkPolicy>
```

Every network policy document starts with a root element "networkPolicy", which contains several child sections. Each child section stores hierarchy of elements for particular object type:

* `protocols` – defines fundamental combinations of protocols and their appropriate port numbers
* `services` – defines combinations of protocols comprising services
* `subnets` – defines individual subnets used in the network
* `hosts` – defines individual hosts having specific connectivity needs
* `targets` – defines combinations of services and hosts or subnets for access or exposition

## Defining Protocols

Our example network requirements specify several fundamental protocol and port combinations. To ease subsequent declarations of subnets, hosts and targets, it will be convenient to define them upfront:

```
<protocols>
    <protocol name="ssh"
              transport="tcp"
              port="22"
              category="management"
              comment="Secure Shell Protocol"
    />
    <protocol name="telnet"
              transport="tcp"
              port="23"
              category="management"
              comment="TELNET Protocol"
    />
    <protocol name="dns/tcp"
              transport="tcp"
              port="53"
              category="naming"
              comment="Domain Name System over TCP"
    />
    <protocol name="dns/udp"
              transport="udp"
              port="53"
              category="naming"
              comment="Domain Name System over UDP"
    />
    <protocol name="http"
              transport="tcp"
              port="80"
              category="media"
              comment="HyperText Transfer Protocol"
    />
    <protocol name="https"
              transport="tcp"
              port="443"
              category="media"
              comment="HyperText Transfer Protocol over SSL/TLS"
    />
    <protocol name="smb"
              transport="tcp"
              port="445"
              category="storage"
              comment="Server Message Block"
    />
    <protocol name="rdp/tcp"
              transport="tcp"
              port="3389"
              category="management"
              comment="Remote Desktop Protocol over TCP"
    />
    <protocol name="rdp/udp"
              transport="udp"
              port="3389"
              category="management"
              comment="Remote Desktop Protocol over UDP"
    />
</protocols>
```

Any of the defined protocol can be easily specified inside definitions of `services`, `subnets`, `hosts` or `targets` sections, as you will see in the following sections (no pun intended).

## Defining Services

Now it will be beneficial to express all distinct services contained inside our requirements. To do this, we use the `services` section of the network policy and refer previously defined fundamental protocols by name:

```
<services>
    <group name="bundles">

      <service name="dns">
        <protocol name="dns/tcp"/>
        <protocol name="dns/udp"/>
      </service>

      <service name="rdp">
        <protocol name="rdp/tcp"/>
        <protocol name="rdp/udp"/>
      </service>

      <service name="http+https">
        <protocol name="http"/>
        <protocol name="https"/>
      </service>

    </group>

    <group name="composed-services">

      <service name="remote-console-services">
        <protocol name="ssh"/>
        <protocol name="telnet"/>
      </service>

      <service name="outbound-services">
        <include name="http+https"/>
        <include name="dns"/>
      </service>

    </group>

</services>
```

As you can see, it is possible to group several `service` definitions together by use of `group` element. Its name can be used as a referral when you need to refer several services by single name or just informally to better organize specific `service` declarations if their number is growing over time.

## Defining Subnets

To have a complete set of information about the network topology, each subnet of our example network has to be specified in the `subnets` section. With all subnets specified, we can then easily express connectivity requirements for all hosts inside a particular subnet. According to the requirements, our `subnets` section would look like this:

```
  <subnets>
    <group name="net-private">

      <subnet name="net-clients">
        <address value="10.1.0.0" length="24" type="ipv4"/>
        <gateway value="10.1.0.1"  type="ipv4"/>
        <expose target="rdp-to-workstations"/>
        <access target="web-services"/>
        <access target="storage-services"/>
        <access target="outbound-services"/>
        <masquerade target="clients-to-public"/>
      </subnet>

      <subnet name="net-services">
        <address value="10.2.0.0" length="24" type="ipv4"/>
        <gateway value="10.2.0.1" type="ipv4"/>
        <access target="rdp-to-workstations"/>
        <access target="outbound-services"/>
        <expose target="rdp-to-services"/>
        <masquerade target="services-to-public"/>
      </subnet>

      <subnet name="net-backbone">
        <address value="10.3.0.0" length="16" type="ipv4"/>
        <gateway value="10.3.0.1" type="ipv4"/>
      </subnet>

    </group>

    <subnet name="net-public" role="public">
      <address value="1.2.3.0" length="29" type="ipv4"/>
      <gateway value="1.2.3.4" type="ipv4"/>
      <expose target="outbound-services"/>
      <publish target="published-web-services"/>
      <access target="web-services"/>
    </subnet>

  </subnets>
```

Inside a `subnet` element, you can see several elements of `access`, `expose`, `publish` and `masquerade` type. These elements refer to the name of particular `target` elements, which we will discuss later in their own separate section.

If the all hosts in the subnet need to access a particular `target`, you can express that by use of `access` element. You are allowed to place it inside or outside of the `subnet` element. If you place it outside, it will be inherited by one or more child `subnet` elements. This allows you to easily assign an `access` to more than one `subnet` at once. The same is true for remaining three elements as well.

As you can see, the concept of hierarchical grouping by use of `group` elements is applied in a similar manner as in the `services` section.
