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

Every network policy document starts with a `networkPolicy` root element, which contains several child sections. Each child section stores hierarchy of elements for particular object type:

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

Any of defined protocols can be easily specified inside definitions of `services`, `subnets`, `hosts` or `targets` sections, as you will see in the following sections (no pun intended).

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

As you can see, the concept of hierarchical grouping by use of `group` elements is applied in a similar manner as in the `services` section.

Inside a `subnet` element, you can see several elements of `access`, `expose`, `publish` and `masquerade` type. These elements refer to the name of particular `target` elements, which we will discuss later in their own separate section.

If the all hosts in the subnet need to access a particular `target`, you can express that by use of `access` element. You are allowed to place it inside or outside of the `subnet` element. If you place it outside, it will be inherited by one or more child `subnet` elements. This allows you to easily assign an `access` to more than one `subnet` at once. The same is true for remaining three elements as well.

Similarly, if you want to apply SNAT onto outbound public traffic originating from all hosts of particular `subnet`, you can specify that by use of `masquerade` element.

To be aware of a public subnet interconnecting our local area network with the Internet, the "net-public" `subnet` has the `role` attribute by which we can easily recognize its special treatment it desires for generation of final firewall rules.

In the "net-public" `subnet` element, there is a `publish` element specifying that some internal services defined inside a referred `target` will be available to any host on the Internet.

## Defining Hosts

According to our example connectivity requirements, there are some specific hosts for which we need to declare particular access or specify that there are some services these hosts are providing to the network:

```
<hosts>
  <group name="workstations">

    <host name="pc-one">
      <address value="10.1.0.11" type="ipv4"/>
      <address value="00:01:00:11:11:11" type="mac"/>
      <access target="management-access"/>
    </host>

    <host name="pc-two">
      <address value="10.1.0.12" type="ipv4"/>
      <address value="00:01:00:12:12:12" type="mac"/>
      <access target="router-access"/>
    </host>
  </group>

  <group name="servers">
    <host name="sr-primary">
      <address value="10.2.0.11" type="ipv4"/>
      <address value="00:02:00:11:11:11" type="mac"/>
      <expose target="web-services"/>
    </host>

    <host name="sr-secondary">
      <address value="10.2.0.12" type="ipv4"/>
      <address value="00:02:00:12:12:12" type="mac"/>
      <expose target="storage-services"/>
    </host>
  </group>

  <group name="routers">

    <host name="rt-edge">
      <interface name="wan">
        <address value="1.2.3.4" type="ipv4" name="wan1"/>
        <address value="1.2.3.5" type="ipv4" name="wan2"/>
      </interface>
      <interface name="lan-backbone">
        <address value="10.3.0.1" type="ipv4" name="lan-backbone"/>
        <address value="00:03:00:AA:AA:AA" type="mac"/>
      </interface>
      <expose target="router-access"/>
    </host>

    <host name="rt-clients">
      <inteface name="lan-clients">
        <address value="10.1.0.1" type="ipv4" name="lan-clients"/>
        <address value="00:01:00:AA:AA:AA" type="mac"/>
      </inteface>
      <interface name="lan-backone">
        <address value="10.3.1.1" type="ipv4" name="lan-backbone"/>
        <address value="00:03:00:11:11:11" interface="lan-backbone" type="mac"/>
      </interface>
      <expose target="router-access"/>
    </host>

    <host name="rt-services">
      <interface name="lan-services">
        <address value="10.2.0.1" type="ipv4" name="lan-services"/>
        <address value="00:02:00:AA:AA:AA" type="mac"/>
      </interface>
      <interface name="lan-backbone">
        <address value="10.3.2.1" type="ipv4" name="lan-backbone"/>
        <address value="00:03:00:22:22:22" type="mac"/>
      </interface>
      <expose target="router-access"/>
    </host>

  </group>
</hosts>
```

## Defining Targets

An finally, `target` elements are the last piece of the network policy puzzle. Their purpose is to bind several protocols or services together with one or more `subnet` or `host` elements and assign a meaningful name to that combination:

```
<targets>
  <group name="services">

    <target name="outbound-services">
      <service name="http+https"/>
      <subnet name="net-public"/>
    </target>

    <target name="web-services">
      <service name="http+https"/>
      <host name="sr-primary"/>
    </target>

    <target name="storage-services">
      <protocol name="smb"/>
      <host name="sr-secondary"/>
    </target>

    <target name="rdp-to-workstations">
      <service name="rdp"/>
      <subnet name="net-clients"/>
    </target>

    <group name="management-access">

      <target name="rdp-to-services">
        <service name="rdp"/>
        <subnet name="net-services"/>
      </target>

      <target name="router-access">
        <service name="remote-console-services"/>
        <host name="rt-edge"/>
      </target>

    </group>
  </group>

  <group name="publishing">

    <target name="published-web-services" type="publish">
      <service name="http+https"/>
      <host name="rt-edge" address="wan1" role="public"/>
      <host name="sr-primary" role="private"/>
    </target>

  </group>

  <group name="masquerade">

    <target name="clients-to-public" type="masquerade">
      <service name="http+https"/>
      <subnet name="net-public"/>
      <host name="rt-edge" address="wan1" role="public"/>
    </target>

    <target name="services-to-public" type="masquerade">
      <service name="http+https"/>
      <subnet name="net-public"/>
      <host name="rt-edge" address="wan2" role="public"/>
    </target>

  </group>
</targets>
```

Again, even for `target` element, you can use the `group` elements to better organize them into hierarchical, meaningful structures. Sometimes, it can also be beneficial to refer to several `target` elements by use of the name of the parent `group` element.

An important (and rather peculiar) feature of `target` elements is that you can nest them into each other to gain more flexibility for hierarchical specification of `host` and `subnet` elements. Unfortunately, our example network is so simple that it does not allow to demonstrate it easily. Let me explain it in a completely independent example:

```
<targets>
  <target name="rdp-from-all">
    <service name="rdp"/>

    <host name="pc-one"/>

    <target name="rdp-from-servers">
      <host name="pc-two"/>
    </target>

  </target>
</targets>
```

If you specify the "rdp-from-all" `target` by use of `access` element in all private `subnet` elements, then only the "pc-one `host` will be accessible by remote desktop client. Next, if you specify the "rdp-from-servers" `target` by use of `access` element in the "net-services" `subnet` element, then both "pc-one" and "pc-two" will be accessible from the "net-services" `subnet` hosts, without need to repeat `host` and `service` declaration of "pc-one" inside the "rdp-from-servers" `target` element.