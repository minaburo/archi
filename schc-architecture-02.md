---

stand_alone: true
ipr: trust200902
docname: draft-ietf-schc-architecture-02
cat: info
submissionType: IETF

pi:
  symrefs: 'yes'
  sortrefs: 'yes'
  strict: 'yes'
  compact: 'yes'
  toc: 'yes'

title: "Static Context Header Compression (SCHC) Architecture"
abbrev: SCHC Architecture
wg: SCHC Working Group
area: Internet
author:
- ins: A. Pelov
  name: Alexander Pelov
  org: IMT Atlantique
  street: rue de la Chataigneraie
  city: 35576 Cesson-Sevigne Cedex
  country: France
  email: alexander.pelov@imt-atlantique.fr
- ins: P. Thubert
  name: Pascal Thubert
  city:  06330 Roquefort les Pins
  country: France
  email: pascal.thubert@gmail.com

- ins: A. Minaburo
  name: Ana Minaburo
  org: Consultant
  city: 35510 Cesson-Sevigne Cedex
  country: France
  email: anaminaburo@gmail.com
normative:
  rfc8724: SCHC
  rfc8824: SCHC-CoAP
informative:
  rfc8376: LPWANs
  rfc8200: IPv6
  rfc7950: YANG
  rfc8376: Overview
  rfc9011: SCHCoLoRaWAN
  rfc9363: Model
  rfc9442: SCHCoSigFox
  rfc2119:
  rfc8174:
  I-D.ietf-schc-over-ppp: SCHCoPPP
  I-D.ietf-core-comi: COMI
  I-D.ietf-6lo-schc-15dot4: SCHCo15dot4
  I-D.ietf-intarea-schc-protocol-numbers: PN_and_Ethertype
  I-D.ietf-schc-access-control: SCHCAC


--- abstract

This document defines the SCHC architecture.

--- middle

# Introduction {#Introduction}

<!--- (compiled with:  "kdrfc schc-architecture.md" ) -->

The IETF LPWAN WG defined the necessary operations to enable IPv6 over
selected Low-Power Wide Area Networking (LPWAN) radio technologies.
{{-Overview}} presents an overview of those technologies.

The Static Context Header Compression (SCHC) {{rfc8724}} technology is the core
product of the IETF LPWAN working group and was the basis to form the SCHC
Working Group.
{{rfc8724}} defines a generic framework for header compression and fragmentation,
based on a static context that is pre-installed on the SCHC endpoints.

This document details the constitutive elements of a SCHC-based solution, and
how the solution can be deployed. It provides a general architecture for a SCHC
deployment, positioning the required specifications, describing the possible
deployment types, and indicating models whereby the rules can be distributed and
installed to enable reliable and scalable operations.


# Requirements Language

{::boilerplate bcp14}


# Terminology

* C/D. Compression and Decompression.
* Context. All the information related to the Rules for SCHC Header, Non-Compression, C/D and F/R and CORECONF_Management.
* FID. Field Identifiers, describing the name of the field in a protocol header.
* F/R. Fragmentation and Reassembly.
* Rule. A description of the header fields to performs compression/decompression, fragmentation/reassembly, SCHC Instances and CORECONF_Management.
* SCHC Entities. A host (Device, Application and Network Gateway) involved in the SCHC process.
* SCHC Instance. The different stages of SCHC in a host. Each instance will have its Set of Rules (SoR), based on the profile, the protocols, the device, the behaviour and a Set of Variables (SoV).
* SCHC Session. This layer provides the management of SCHC instances, the SoR of each instance and the dialog between hosts to keep the SCHC synchronization.
* SoR (Set of rules). Group of Rules used in a SCHC Instance. The set of rules contains Rules for different nature as compression, no compression, fragmentation, SCHC Instances and CORECONF management.
* SoV (Set of Variables). External information that needs to be known to identify the correct protocol, the session id, and the flow when there is one.
* Core SCHC. SCHC entity located upstream in the Network Gateway.
* Device SCHC. SCHC entity located downstream.

# Building Blocks
This section specifies the principal blocks defined for building and using the SCHC architecture in any network topology and protocol.

## SCHC layer
SCHC Layer is a building block composed at least of a SCHC Packet Instance as described in the {{rfc8724}}. A SCHC Header Instance may be added, it controls the SCHC Layer. The SCHC Header Instance is used with different SCHC Packets Instances if they are defined in the same SCHC Layer.

Note that a SCHC Layer is different from an ISO layer {{Fig-SCHCSESSION}}.


## SCHC Header Instance
The SCHC Header Instance manages the SCHC Headers and provides the information and the selection of a SCHC Packet Instance.
SCHC Header is mandatory and can be free of charge for very constrained networks such as LPWAN. It allows the recognition of SCHC as the next header and can give the protocol that SCHC has compressed.
{{Fig-SCHCSESSION}} shows the SCHC layer that needs to be introduced in the Architecture when SCHC is used to compress different protocols together or independently as {{rfc8824}} has described for CoAP. Notice that the parenthesis in the figure indicates a SCHC compression.

A discriminator identifies the SCHC Instances, which can be:

* A source and destination addresses of the packet carrying SCHC packets
* A source and destination port number of the transport layer carrying SCHC packets
* An MPLS label
* A TLS Association
* Any other kind of connection id.


~~~~

+---------------+---------------+---------------+  
| SCHC Packet   | SCHC Packet   | SCHC Packet   | S
| Instance ___  | Instance ___  | Instance ___  | C
|         [SoR] |         [SoR] |         [SoR] | H
|         [___] |         [___] |         [___] | C
|               |               |               |  
|               |               |               | L
+----inst_id1---+----inst_id2---+----inst_id3---+ A
.            SCHC Header Instance         ___   . Y
.                                        [SoR]  . E
.                                        [___]  . R
+...............................................+
               _____________^        
              /                    
            /
           +-- Discriminator: (SCHC HEADER)(SCHC PACKET)    

Each Packet Instance contains its own Set of Rules,
but share the same SCHC Header.  

~~~~
{: #Fig-SCHCSESSION title='SCHC Session Layer'}

### SCHC Header

SCHC Header carries information to allow the SCHC layer to work correctly. For example, it selects the correct Instance and checks the validity of the datagram.
There IS NOT always a RuleID if there is only one Rule for the SCHC Header, whose length is 0. The SCHC Header format is not fixed, and the SoR MUST have one or more Rules describing the formats. SCHC Header contains different fields.
For Instance, when the SCHC header may identify the next protocol in the stack, the format of the SCHC header takes the format as {{Fig-SCHCHDR}} shows.

~~~~

Non-compressed SCHC Header Format:
+- - - - - - +- - - - - - -+- - -+
| Session ID | Protocol ID | CRC |
+- - - - - - +- - - - - - -+- - -+

SCHC Header Compressed:
+- - - - -+- - - - - - - - - - +
| Rule ID | Compressed Residue |
+- - - - -+- - - - - - - - - - +

Rule uses to compressed the SCHC Header:
RuleID
+------------+--+---+--+-----+------+-----------+
|     FID    |FL|POS|DI| TV  |  MO  |     CDA   |
+------------+--+---+--+-----+------+-----------+
| SCHC.sesid |10| 1 |Bi|0x00 |MSB(7)| LSB       |
| SCHC.proto | 8| 1 |Bi|value|equal | not-sent  |
| SCHC.CRC   | 8| 1 |Bi|     |ignore| value-sent|
+------------+--+---+--+-----+------+-----------+


~~~~
{: #Fig-SCHCHDR title='Example of SCHC Header Format and the corresponding Rule'}

In this example the Rule defines:

* A SessionID is 10 bits length and it is used to identify the SoR used for this instance of SCHC.
* A Protocol ID in 1-byte length giving the value send in the layer below the SCHC packet to identify the uncompressed protocol stack.
* And A CRC. The CRC field is 8 bits length and covers the SCHC header and the SCHC packet from error. When it is elided by the compression, the layer-4 checksum MUST be replaced by another validation sequence.


## SCHC Packet Instance {#Instances}
SCHC Packet Instance is characterized by a particular SoR common with the corresponding distant entity.
The {{rfc8724}} defines a protocol operation between a pair of peers.
In a SCHC layer, several SCHC Instances may contain different SoR.

When the SCHC Device is a highly constrained unit, there is typically only one
Instance for that Device, and all the traffic from and to the device is
exchanged with the same Network Gateway. All the traffic can thus be implicitly
associated with the single Instance that the device supports, and the Device
does not need to manipulate the concept. For that reason, SCHC avoids to signal
explicitly the Instance identification in its data packets.

The Network Gateway, on the other hand, maintains multiple Instances, one per
SCHC Device. The Instance is derived from the lower layer, typically the source
of an incoming SCHC packet as a discriminator in the {{Fig-SCHCSESSION}}.
The Instance is used in particular to select the set of rules that apply to the SCHC Device, and the
current state of their exchange, e.g., timers and previous fragments.


### SCHC Packet
The SCHC Packet is composed of a RuleID follows by the content described in the Rule.
The content may be a C/D packet, a F/R packet, a CORECONF_Management or a Non Compressed packet.
As defined in the {{rfc8724}}, the SCHC packet for C/D is composed of the Compressed Header followed by the payload from the original packet.
{{Fig-SCHCPacket}} shows the compressed header format that is composed of the RuledID and a Compressed Residue, which is the output of compressing a packet header with a Rule.

~~~~
C/D Compressed Packet:

+------------+----------------------+
|   RuleID   | Compressed Residue   |
+------------+----------------------+

F/R Compressed Packet:

+------------+----------------------+-----
|   RuleID   | Fragmentation Header | Tiles
+------------+----------------------+-----

CORECONF_Management Compressed Packet:

+------------+----------------------+
|   RuleID   | Compressed Residue   |
+------------+----------------------+

~~~~
{: #Fig-SCHCPacket title='SCHC Packet'}

## SCHC Profiles
A SCHC profile is the specification to adapt the use of SCHC with the necessities of the technology to which it is applied.
In the case of star topologies and because LPWAN technologies {{-Overview}} have strict yet distinct constraints, e.g., in terms of maximum frame size, throughput, and directionality, also a SCHC instance and the fragmentation model with the parameters' values for its use.

Appendix D. "SCHC Parameters" of {{rfc8724}} lists the information that an LPWAN
technology-specific document must provide to profile SCHC fragmentation for that technology.

As an example, {{rfc9011}} provides the SCHC fragmentation profile for LoRaWAN networks.



## SCHC Operation
The SCHC operation requires a shared sense of which SCHC Device is Uplink
(Dev to App) and which is Downlink (App to Dev), see {{rfc8376}}.
In a star deployment, the hub is always considered Uplink and the spokes are
Downlink. The expectation is that the hub and spoke derive knowledge of their
role from the network configuration and SCHC does not need to signal which is
hub thus Uplink vs. which is spoke thus Downlink. In other words, the link
direction is determined from extrinsic properties, and is not advertised in the
protocol.

Nevertheless, SCHC is very generic and its applicability is not limited to
star-oriented deployments and/or to use cases where applications are very static
and the state provisioned in advance.
In particular, a peer-to-peer (P2P) SCHC Instance (see {{Instances}}) may be set
up between peers of equivalent capabilities, and the link direction cannot be
inferred, either from the network topology nor from the device capability.

In that case, by convention, the device that initiates the connection that
sustains the SCHC Instance is considered as being Downlink, i.e. it plays the
role of the Dev in {{rfc8724}}.

This convention can be reversed, e.g., by configuration,
but for proper SCHC operation, it is required that the method used ensures that
both ends are aware of their role, and then again this determination is based
on extrinsic properties.

### SCHC Rules
SCHC Rules are a description of the header protocols fields, into a list of Field Descriptors. The {{rfc8724}} gives the format of the Rule description for C/D, F/R and non-compression. In the same manner the SCHC Header and SCHc CORECONF_Management will use the {{rfc8724}} field descriptors to compress the format information.

Each type of Rule is identified with a RuleID. There are different types of Rules: C/D, F/R, SCHC Header, CORECONF_Management and No Compression. Notice that each Rule type used an independent range of RuleID to identify its rules.   

A Rule does not describe how the compressor parses a packet header. Rules only describe the behavior for each header field.

SCHC Action. ToDo

### SoR identification
ToDo


## SCHC Management
RFC9363 writes that only the management can be done by the two entities of the instance, and other SoR cannot be manipulated.

Management rules are explicitly define in the SoR, see {{Fig-SCHCManagement}}. They are compression Rules for CORECONF messages to get or modify the SoR of the instance. The management can be limited with the {{-SCHCAC}} access definition.

~~~~
+-----------------+                 +-----------------+
|       ^         |                 |       ^         |
|  C/D  !  M ___  |                 |       !  M ___  |
|       +-->[SoR] |       ...       |       +-->[SoR] |
|       !   [___] |                 |       !   [___] |
|       !         |                 |       !         |
|      F/R        |                 |      F/R        |
+------ins_id1----+-----ins_idi-----+------ins_idn----+         
.                   C/D  !                       ___  .
.                        +--------------------->[SoR] .    
.                       F/R               M     [___] .
+.................. Discriminator ....................+



~~~~
{: #Fig-SCHCManagement title='Inband Management'}

### SCHC Data Model
A SCHC instance, summarized in the {{Fig-Glob-Arch1}}, implies C/D and/or F/R and CORECONF_Management and SCHC Instances Rules present in both end and that both ends are provisioned with the same SoR.

~~~~
        -----                                  -----
       [ SoR ]                                [ SoR ]
        -----                                  -----
          .                                      .
          .                                      .
          .                                      .
       +- M ---+                              +- M ---+
   <===| R & D |<===                      <===| C & F |<===
   ===>| C & F |===>                      ===>| R & D |===>
       +-------+                              +-------+


~~~~
{: #Fig-Glob-Arch1 title='Summarized SCHC elements'}


A common rule representation that expresses the SCHC rules in an interoperable
fashion is needed to be able to provision end-points from different vendors
to that effect, {{-Model}} defines a rule representation using the
[YANG](#rfc7950) formalism.

{{rfc9363}} defines a YANG data model to represent the rules. This enables the use of several protocols for rule management, such as NETCONF{{?RFC6241}}, RESTCONF{{?RFC8040}}, and CORECONF{{-COMI}}. NETCONF uses SSH, RESTCONF uses HTTPS, and CORECONF uses CoAP(s) as their respective transport layer protocols. The data is represented in XML under NETCONF, in JSON{{?RFC8259}} under RESTCONF and in CBOR{{?RFC8949}} under CORECONF.

~~~~
               create
        -----  read    +=======+   
       [ SoR ]<------->|Rule   |<-----+ NETCONF,
        -----  update  |Manager|      | RESTCONF or
                delete +=======+      | CORECONF
           +--------------------------+ request
           |
           v M
       +---+---+
   <===| R & D |<===
   ===>| C & F |===>
       +-------+
~~~~
{: #Fig-RM title='Summerized SCHC elements'}

The Rule Manager (RM) is in charge of handling data derived from the YANG Data
Model and apply changes to the context and SoR of each SCHC Instance {{Fig-RM}}.

The RM is an Application using the Internet to exchange information, therefore:

* for the network-level SCHC, the communication does not require routing. Each of the end-points having an RM and both RMs can be viewed on the same link, therefore wellknown Link Local addresses can be used to identify the Device and the core RM. L2 security MAY be deemed as sufficient, if it provides the necessary level of protection.

* for application-level SCHC, routing is involved and global IP addresses SHOULD be used. End-to-end encryption is RECOMMENDED.

Management messages can also be carried in the negotiation protocol, for instace, the {{-SCHCoPPP}} proposes a solution.
The RM traffic may be itself compressed by SCHC: if CORECONF protocol is used, {{-SCHC-CoAP}} can be applied.


# SCHC Architecture

As described in {{rfc8824}}, SCHC feasibility enables combining several SCHC instances.
The {{rfc8724}} states that a SCHC instance needs the rules to process C/D and F/R before the session starts and that the SoR of the instance control layer cannot be modified. However, the rules may be updated in certain instances to improve the performance of C/D, F/R, or CORECONF_Management. The {{-SCHCAC}} defines the possible modifications and who can modify, update, create and delete Rules or part of them in the instances' SoR.


As represented in {{Fig-SCHCCOAP2}}, the compression
of the IP and UDP headers may be operated by a network SCHC instance whereas the
end-to-end compression of the application payload happens between the Device and
the application. The compression of the application payload may be split in two
instances to deal with the encrypted portion of the application PDU. Fragmentation
applies before LPWAN transportation layer.

~~~~

         (Device)            (NGW)                           (App)

         +--------+                                        +--------+
  A S    |  CoAP  |                                        |  CoAP  |
  p C    |  inner |                                        |  inner |
  p H    +--------+                                        +--------+
  . C    |  SCHC  |                                        |  SCHC  |
         |  inner |   cryptographical boundary             |  inner |
 -._.-._.-._.-._.-._.-._.-._.-._.-._.-._.-._.-._.-.._.-._.-._.-._.-._
  A S    |  CoAP  |                                        |  CoAP  |
  p C    |  outer |                                        |  outer |
  p H    +--------+                                        +--------+
  . C    |  SCHC  |                                        |  SCHC  |
         |  outer |   layer / functional boundary          |  outer |
 -._.-._.-._.-._.-._.-._.-._.-._.-._.-._.-._.-._.-._.--._.-._.-._.-._
  N      .  UDP   .                                        .  UDP   .
  e      ..........     ..................                 ..........
  t      .  IPv6  .     .      IPv6      .                 .  IPv6  .
  w S    ..........     ..................                 ..........
  o C    .SCHC/L3 .     . SCHC/L3.       .                 .        .
  r H    ..........     ..........       .                 .        .
  k C    .  LPWAN .     . LPWAN  .       .                 .        .
         ..........     ..................                 ..........
             ((((LPWAN))))             ------   Internet  ------
~~~~
{: #Fig-SCHCCOAP2 title='Different SCHC instances in a global system'}


This document defines a generic architecture for SCHC that can be used at any of these levels.
The goal of the architectural document is to orchestrate the different protocols and data model
defined by the LPWAN and SCHC working groups to design an operational and interoperable
framework for allowing IP application over constrained networks.

The {{Fig-SCHCArchi}} shows the protocol stack and the corresponding SCHC layers enabling the compression of the different protocol headers.
The SCHC header eases the introduction of intermediary host in the end-to-end communication transparently.
All the SCHC headers are compressed and in some cases are elided, for example for LPWAN networks. The layers using encryption does not have a SCHC header in the middle because they are the same entity.
{{Fig-SCHCArchiEx}} shows an example of an IP/UDP/CoAP in an LPWAN network.


~~~~

DEV                                 NGW              APP

{[(Encrypted Application Layer)]} . . . . . . . . {[(EAL)]}
(Application Layer Protocol) . . . . . . . . . . .({[ALP]})
(SCHC) . . . . . . . . . . . . . . . . . . . . . ({[SCHC]})
{[(Encrypted Security Layer)]} . . . . . . . . . .{[(ESL)]}
{(Security Layer Protocol)}. . . . . . . . . . . . .{(SLP)}
{(SCHC)} . . . . . . . . . . . . . . . . . . . . . {(SCHC)}
(Transport Layer Protocol). . . (TLP) TLP . . . . . .TLP
{(SCHC)} . . . . . . . . . . {(SCHC)}
(Internet Layer Protocol) . . . (IP)  IP . . . . . . IP
(SCHC). . . . . . . . . . . . .(SCHC)  
Network Layer Protocol . . . . . . . . . . . . . . . NLP

Where: {} Optional; [] Encrypted; () Compressed.

~~~~
{: #Fig-SCHCArchi title='SCHC Architecture'}

In {{Fig-SCHCArchi}}, each line represents a layer, parentheses surround a compressed header, and if it is optional, it has curly brackets.
All the SCHC layers are compressed.
Square brackets represent the encrypted data; if the encryption is optional, curly brackets precede the square brackets.

Intermediary Routers or gateways may know only one layer's SoR and forward the rest of the compressed packet to the next hub.

~~~~

Example for an LPWAN network receiving a IP/UDP/CoAP using OSCORE:

   +--------------CoAP OSCORE & PAYLOAD----------------------+
A  .                                                         .
P  |----------------------(OSCORE)---------------------------|
P  |                                                         |    
   |                                                         |      
   +-----------------------CoAP(OSCORE)----------------------+
   | +-----------------+                 +-----------------+ |
   | |       ^         |                 |       ^         | |
   | |  C/D  !  M ___  |                 |       !  M ___  | |
S  | |       +-->[SoR] |       ...       |       +-->[SoR] | |
C  | |       !   [___] |                 |       !   [___] | |
H  | |       !         |                 |       !         | |
C  | |      F/R        |                 |      F/R        | |
   | +------ins_id1----+-----ins_idi-----+------ins_idn----+ |         
   | |                   C/D  !                            | |
   | |             (RuleID)(CoAP)(OSCORE)             ___  | |
   | |                        +--------------------->[SoR] | |    
   | |                       F/R               M     [___] | |
   +--IP:A->B/UDP:Dest=SCHC(SCHC HDR)(RuleID)(CoAP)(OSCORE)--+
U  |         ^                                               |         
D  |         |                                               |
P  |         |                                               |
   +---------------------------------------------------------+
I  |         |                                               |
P  |         |                                               |
   +--IP:A->B/UDP:Dest=SCHC;(SCHC HDR)(RuleID)(CoAP)(OSCORE)-+
   | +-----------------+                 +-----------------+ |
   | |       ^         |                 |       ^         | |
   | |  C/D  !  M ___  |                 |       !  M ___  | |
S  | |       +-->[SoR] |       ...       |       +-->[SoR] | |
C  | |       !   [___] |                 |       !   [___] | |
H  | | RuleID(IP/UDP)  |                 |       !         | |
C  | |   (SCHC HEADER) |                 |       !         | |
   | |  (RuleID)(CoAP))|                 |       !         | |
   | |       ! (OSCORE)|                 |       !         | |
   | |      F/R        |                 |      F/R        | |
   | +------ins_id1----+-----ins_idi-----+------ins_idn----+ |         
   | |                   C/D  !                            | |
   | |                  (SCHC Packet)                      | |
   | |                        |                       ___  | |
   | |                        +--------------------->[SoR] | |    
   | |                       F/R               M     [___] | |
   +-+-----------Discriminator Implicit--------------------+-+
N      ______________^  
E     /
T    | ()(RuleID(IP/UDP)(SCHC_HEADER)RuleID(COAP)(OSCORE))PAYLOAD
W    | ^ ^---------+       
     | |            \
     +-(SCHC Header)(SCHC Packet)


~~~~
{: #Fig-SCHCArchiEx title='SCHC Architecture Example'}




# The Static Context Header Compression

[SCHC](#rfc8724) specifies an extreme compression capability based on a description
that must match on the compressor and decompressor side.
This description comprises a set of Compression/Decompression (C/D) rules.

The SCHC Parser analyzes incoming packets and creates a list of fields that it
matches against the compression rules.
The rule that matches is used to compress the packet, and the rule
identifier (RuleID) is transmitted together with the compression residue to the decompressor.
Based on the RuleID and the residue, the decompressor can rebuild the original packet and forward it in its uncompressed form over the Internet.
When no Rule matches the header, the No Compression Rule is used.
When several Rules match the header the implementation must choose one.
How it is done or based on which parameters is out of the scope of this document. SCHC compresses datagrams and there is no notion of flows.

{{rfc8724}} also provides a Fragmentation/Reassembly (F/R) capability to cope
with the maximum and/or variable frame size of a Link, which is extremely constrained in the
case of an LPWAN network.

If a SCHC-compressed packet is too large to be sent in a single Link-Layer PDU,
the SCHC fragmentation can be applied on the compressed packet.
The process of SCHC fragmentation is similar to that of compression;
the fragmentation rules that are programmed for this Device are checked to find
the most appropriate one, regarding the SCHC packet size, the link error rate,
and the reliability level required by the application.

The ruleID allows to determine if it is a compression or
fragmentation rule or any other type of Rule.



## SCHC over Network Technologies

SCHC can be used in multiple environments and multiple protocols.
It was designed by default to work on native MAC frames with LPWAN technologies such as LoRaWAN{{rfc9011}}, IEEE std 802.15.4 {{-SCHCo15dot4}}, and SigFox{{rfc9442}}.

To operate SCHC over Ethernet, IPv6, and UDP, the definition of, respectively, an Ethertype, an IP Protocol Number, and a UDP Port Number are necessary, more in {{-PN_and_Ethertype}}. In either case, there's a need for a SCHC header that is sufficient to identify the SCHC peers (endpoints) and their role (device vs. app), as well as the session between those peers that the packet pertains to.

In either of the above cases, the expectation is that the SCHC header is transferred in a compressed form. This implies that the rules to uncompress the header are well known and separate from the rules that are used to uncompress the SCHC payload. The expectation is that for each layer, the format of the SCHC header and the compression rules are well known, with enough information to identify the session at that layer, but there is no expectation that they are the same across layers.


### SCHC over PPP

The LPWAN architecture ({{Fig-LPWANnetarch}}) generalizes the model to any kind of peers.
In the case of more capable devices, a SCHC Device may maintain more than one Instance with the same peer, or a set of different peers.
Since SCHC does not signal the Instance in its packets, the information must be
derived from a lower layer point to point information.
For instance, the SCHC instance control can be associated one-to-one with a tunnel, a TLS session, or a TCP or a PPP connection.

For instance, {{-SCHCoPPP}} describes a type of deployment where
the C/D and/or F/R operations are performed between peers of equal capabilities
over a PPP {{?rfc2516}} connection. SCHC over PPP illustrates that with SCHC,
the protocols that are compressed can be discovered dynamically and the
rules can be fetched on-demand using CORECONF messages Rules, ensuring that the peers use the exact same set of rules.

~~~~
    +----------+  Wi-Fi /   +----------+                ....
    |    IP    |  Ethernet  |    IP    |            ..          )
    |   Host   +-----/------+  Router  +----------(   Internet   )
    | SCHC C/D |  Serial    | SCHC C/D |            (         )
    +----------+            +----------+               ...
                <-- SCHC -->
                  over PPP
~~~~
{: #Fig-PPPnetarch title='PPP-based SCHC Deployment'}

In that case, the SCHC Instance is derived from the PPP connection. This
means that there can be only one Instance per PPP connection, and that all the
flow and only the flow of that Instance is exchanged within the PPP connection.
As discussed in {{EndPoints}}, the Uplink direction is from the node that
initiated the PPP connection to the node that accepted it.


### SCHC over Ethernet
Before the SCHC compression takes place, the SCHC header showed in the {{Fig-SCHC_hdr}}, is virtually inserted before the real protocol header and data that are compressed in the session, e.g. a IPv6 in this figure.

~~~~
                                       |---- SCHC PACKET ----|
 +------------------+------------------+---------+-----------+
 | IEEE 802 Header  | SCHC Header      | Rule ID | Compressed|
 | Ethertype = SCHC | Ethertype = IPv6 |         | Residue   |
 +------------------+------------------+---------+-----------+
                     <-
                       SCHC overhead
                                     ->
~~~~
{: #Fig-SCHC_hdr title='SCHC over Ethernet'}


### SCHC over IPv6
In the case of IPv6, the expectation is that the Upper Layer Protocol (ULP) checksum can be elided in the SCHC compression of the ULP,
because the SCHC header may have its own checksum that protects both the SCHC header and the whole ULP, header and payload.

The SCHC Header between IPv6 and the ULP is not needed because of the Next Header field on the IPv6 header format.
<!--
[//]: IP has Next Hdr and does not need SCHC header. My question is do we C/D this 2 layer at the same time or not?  
-->

~~~~
                             |---- SCHC Packet -----|
 +-------------+-------------+---------+------------+
 | IPv6 Header | SCHC Header | Rule ID | Compressed |
 |  NH=SCHC    | NH = ULP    |         | Residue    |
 +-------------+-------------+---------+------------+
                <-
                SCHC overhead
                           ->
~~~~
{: #Fig-SCHC_hdr1 title='SCHC over IPv6'}

<!--
[//]: Update this figure after discussion  
-->

In the air, both the SCHC header and the ULP are compressed.
The session endpoints are typically identified by the source and destination IP addresses. If the roles are well-known, then the endpoint information can be elided and deduced from the IP header.
If there is only one session, it can be elided as well, otherwise a rule and residue are needed to extract the session ID.


### SCHC over UDP
When SCHC operates over the Internet, middleboxes may block packets with a next header that is SCHC. To avoid that issue, it would be desirable to prepend a UDP header before the SCHC header as shown in figure {{Fig-SCHC_hdr2}}.

~~~~
                                           |---- SCHC Packet -----|
 +-------------+-------------+-------------+---------+------------+
 | IPv6 Header | UDP Header  | SCHC Header | Rule ID | Compressed |
 |  NH=UDP     | Port = SCHC | NH = ULP    |         | Residue    |
 +-------------+-------------+-------------+---------+------------+
                <-
                       SCHC overhead
                                          ->
~
~~~~
{: #Fig-SCHC_hdr2 title='SCHC over UDP'}

In that case, the destination port can indicate SCHC as in an header chain, and the source port can indicate the SCHC session in which case it can be elided in the compressed form of the SCHC header.
The UDP checksum protects both the SCHC header and the whole ULP, so the SCHC and the ULP checksums can both be elided.
In other words, in the SCHC over UDP case, the SCHC header can be fully elided, but the packet must carry the overhead of a full UDP header.
<!--
[//]: Update this part after discussion. I'm not sure we have discussed in this way.   
-->

# SCHC Endpoints for LPWAN Networks {#EndPoints}

<!--
[//]: # (to Eric's point, how do we ensure that both ends have the same SoR)
-->
Section 3 of {{rfc8724}} depicts a typical network architecture for
an LPWAN network, simplified from that shown in {{rfc8376}} and reproduced in
{{Fig-LPWANnetarch}}.

~~~~
 ()   ()   ()       |
  ()  () () ()     / \       +---------+
() () () () () () /   \======|    ^    |             +-----------+
 ()  ()   ()     |           | <--|--> |             |Application|
()  ()  ()  ()  / \==========|    v    |=============|   Server  |
  ()  ()  ()   /   \         +---------+             +-----------+
 Dev            RGWs             NGW                      App
~~~~
{: #Fig-LPWANnetarch title='Typical LPWAN Network Architecture'}

Typically, an LPWAN network topology is star-oriented, which means that all
packets between the same source-destination pair follow the same path from/to a
central point. In that model, highly constrained Devices (Dev) exchange
information with LPWAN Application Servers (App) through a central Network
Gateway (NGW), which can be powered and is typically a lot less constrained than
the Devices.
Because Devices embed built-in applications, the traffic flows to be compressed
are known in advance and the location of the C/D and F/R functions
(e.g., at the Dev and NGW), and the associated rules, can be pre provisioned in the system before use.


## SCHC Device Lifecycle
In the context of LPWANs, the expectation is that SCHC rules are associated with a
physical device that is deployed in a network. This section describes the actions
taken to enable an automatic commissioning of the device in the network.

### Device Development

The expectation for the development cycle is that message formats are documented as a data model that is used to generate rules. Several models are possible:

1. In the application model, an interface definition language and binary communication protocol such as Apache Thrift is used, and the parser code includes the SCHC operation.
This model imposes that both ends are compiled with the generated structures and linked with generated code that represents the rule operation.

2. In the device model, the rules are generated separately. Only the device-side code is linked with generated code. The Rules are published separately to be used by a generic SCHC engine that operates in a middle box such as a SCHC gateway.

3. In the protocol model, both endpoint generate a packet format that is imposed by a protocol.
In that case, the protocol itself is the source to generate the Rules. Both ends of the SCHC compression are operated in middle boxes, and special attention must be taken to ensure that they operate on the compatible SoR, basically the same major version of the same SoR.

Depending on the deployment, the tools that generate the Rules should provide knobs to optimize the SoR, e.g., more rules vs. larger residue.

### Rules Publication

In the device model and in the protocol model, at least one of the endpoints must obtain the SoR dynamically. The expectation is that the SoR are published to a reachable repository and versionned (minor, major). Each SoR should have its own Uniform Resource Names (URN) {{!RFC8141}} and a version.


The SoR should be authenticated to ensure that it is genuine, or obtained from a trusted app store.
A corrupted SoR may be used for multiple forms of attacks, more in {{Security}}.


### SCHC Device Deployment
<!--
[//]: # (how to provision the GW with the security and the sortrefs for the new device?)
-->

The device and the network should mutually authenticate themselves. The autonomic approach {{?RFC8993}} provides a model to achieve this at scale with zero touch, in networks where enough bandwidth and compute are available.
In highly constrained networks, one touch is usually necessary to program keys in the devices.

The initial handshake between the SCHC endpoints should comprise a capability exchange whereby URN and the version of the SoR are obtained or compared.
SCHC may not be used if both ends can not agree on an URN and a major version.  
Manufacturer Usage Descriptions (MUD) {{?RFC8520}} may be used for that purpose in the device model.

Upon the handshake, both ends can agree on a SoR, their role when the rules are asymmetrical, and fetch the SoR if necessary. Optionally, a node that fetched a SoR may inform the other end that it is reacy from transmission.


### SCHC Device Maintenance

URN update without device update (bug fix)
FUOTA => new URN => reprovisioning

### SCHC Device Decommissionning

Signal from device/vendor/network admin


# Security Considerations {#Security}


SCHC is sensitive to the rules that could be abused to form arbitrary long
messages or as a form of attack against the C/D and/or F/R functions, say to
generate a buffer overflow and either modify the Device or crash it. It is
thus critical to ensure that the rules are distributed in a fashion that is
protected against tempering, e.g., encrypted and signed.

<!--
[//]: # (Ben Kaduk comment on SCHC CoAP; compression may leak information ???)
[//]: # (Add text to say that this is not effective because not dictionary based)
-->

# IANA Consideration

This document has no request to IANA

# Acknowledgements

The authors would like to thank (in alphabetic order): Laurent Toutain

--- back
