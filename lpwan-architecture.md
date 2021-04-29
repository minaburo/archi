---
stand_alone: true
ipr: trust200902
docname: draft-pelov-lpwan-architecture-02
cat: info
pi:
  symrefs: 'yes'
  sortrefs: 'yes'
  strict: 'yes'
  compact: 'yes'
  toc: 'yes'

title: "LPWAN Static Context Header Compression (SCHC) Architecture"
abbrev: LPWAN Architecture
wg: LPWAN Working Group
area: Internet
author:
- ins: A. Pelov
  name: Alexander Pelov
  org: Acklio
  street: 1137A avenue des Champs Blancs
  city: 35510 Cesson-Sevigne Cedex
  country: France
  email: a@ackl.io
- ins: P. Thubert
  name: Pascal Thubert
  org:  Cisco Systems
  street:  Building D
  street: 45 Allee des Ormes - BP1200
  city:  06254 Mougins - Sophia Antipolis
  country: France
  email: pthubert@cisco.com
- ins: A. Minaburo
  name: Ana Minaburo
  org: Acklio
  street: 1137A avenue des Champs Blancs
  city: 35510 Cesson-Sevigne Cedex
  country: France
  email: ana@ackl.io
normative:
  rfc8724: SCHC
informative:
  rfc8200: IPv6
  rfc8376: Overview
  I-D.ietf-lpwan-schc-yang-data-model: Model
  I-D.ietf-lpwan-coap-static-context-hc: SCHC-CoAP
  I-D.thubert-intarea-schc-over-ppp: SCHCoPPP
  I-D.ietf-core-comi: COMI



--- abstract

This document defines the LPWAN SCHC architecture.

--- middle

# Introduction {#Introduction}


The IETF LPWAN WG defined the necessary operations to enable IPv6 over
selected Low-Power Wide Area Networking (LPWAN) radio technologies.
{{-Overview}} presents an overview of those technologies.

The Static Context Header Compression (SCHC) {{rfc8724}} technology is the core
product of the IETF LPWAN working group. {{rfc8724}} defines a generic framework
for header compression and fragmentation, based on a static context that is pre-installed on the SCHC endpoints.

This document details the constitutive elements of a SCHC-based solution, and
how the solution can be deployed. It provides a general architecture for a SCHC
deployment, positioning the required specifications, describing the possible
deployment types, and indicating models whereby the rules can be distributed and
installed to enable reliable and scalable operations.

# LPWAN Technologies and Profiles

Because LPWAN technologies {{-Overview}} have strict yet distinct constraints,
e.g., in terms of maximum frame size, throughput, and/or directionality, a SCHC
instance must be profiled to adapt to the specific necessities of the technology
to which it is applied.

Appendix D. "SCHC Parameters" of {{rfc8724}} lists the information that an LPWAN
technology-specific document must provide to profile SCHC for that technology.

As an example, {{!rfc9011}} provides the SCHC profile for LoRaWAN networks.

# The Static Context Header Compression

[SCHC](#rfc8724) specifies an extreme compression capability based on a state
that must match on the compressor and decompressor side.
This state comprises a set of Compression/Decompression (C/D) rules.

The SCHC Parser analyzes incoming packets and creates a list of fields that it
matches against the compression rules.
The rule that matches best is used to compress the packet, and the rule
identifier (RuleID) is transmitted together with the compression residue to the decompressor.
Based on the RuleID and the residue, the decompressor can rebuild the original packet and forward it in its uncompressed form over the Internet.

{{rfc8724}} also provides a Fragmentation/Reassembly (F/R) capability to cope
with the maximum frame size of a Link, which is extremely constrained in the
case of an LPWAN network.

If a SCHC-compressed packet is too large to be sent in a single Link-Layer PDU,
the SCHC fragmentation can be applied on the compressed packet.
The process of SCHC fragmentation is similar to that of compression;
the fragmentation rules that are programmed for this device are checked to find
the most appropriate one, regarding the SCHC packet size, the link error rate,
and the reliability level required by the application.

The nature of a ruleID allows to determine if it is a compression or
fragmentation rule.


# SCHC Endpoints

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
information with LPWAN Application Servers (Apps) through a central Network
Gateway (NGW), which can be powered and is typically a lot less constrained than
the Devices.
Because devices embed built-in applications, the traffic flows to be compressed
are known in advance and the location of the C/D and F/R functions (e.g., at the Dev and NGW), and the associated rules, can be pre provisionned in the network .

Then again, SCHC is very generic and its applicability is not limited to
star-oriented deployments and/or to use cases where applications are very static
and the state can provisionned in advance.
{{-SCHCoPPP}} describes an alternate deployment where
the C/D and/or F/R operations are performed between peers of equal capabilities
over a PPP {{?rfc2516}} connection. SCHC over PPP  illustrates that with SCHC,
the protocols that are compressed can be discovered dynamically and the
rules can be fetched on-demand by both parties from the same Uniform Resource
Name (URN) {{?rfc8141}}, ensuring that the peers use the exact same set of rules.

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


# SCHC Instances

The rule database contains a set of rules that are specific per device.
There is thus a SCHC instance per pair of endpoints.
{{rfc8724}} states that a SCHC instance obtains the rules to process
C/D and F/R before the session starts, and that rules cannot be modified during
the session.

{{rfc8724}} was defined to compress [IPv6](#rfc8200) and UDP;
but SCHC really is a generic compression and fragmentation technology.
As such, SCHC is agnostic to which
protocol it compresses and at which layer it is operated. The C/D peers may be
hosted by different entities for different layers, and the F/R operation may
also be performed between different parties, or different sub-layers in the same
stack, and/or managed by different organizations.

If a protocol or a layer requires additional capabilities, it is always possible
to document more specifically how to use SCHC in that context, or to specify
additional behaviours.
For instance, {{-SCHC-CoAP}} extends the compression to CoAP {{?RFC7252}} and
OSCORE {{?RFC8613}}.

As represented figure {{Fig-SCHCCOAP2}}, the fragmentation and the compression
of the IP and UDP headers may be operated by a network SCHC instance whereas the
end-to-end compression of the application payload happens between the device and
the application. The compression of the application payload may be split in two
instances to deal with the encrypted portion of the application PDU.

~~~~

         (device)            (NGW)                              (App)

         +--------+                                           +--------+
  A S    |  CoAP  |                                           |  CoAP  |
  p C    |  inner |                                           |  inner |
  p H    +--------+                                           +--------+
  . C    |  SCHC  |                                           |  SCHC  |
         |  inner |   cryptographical boundary                |  inner |
 -._.-._.-._.-._.-._.-._.-._.-._.-._.-._.-._.-._.-._.-._.-._.-._.-._.-._
  A S    |  CoAP  |                                           |  CoAP  |
  p C    |  outer |                                           |  outer |
  p H    +--------+                                           +--------+
  . C    |  SCHC  |                                           |  SCHC  |
         |  outer |   layer / functional boundary             |  outer |
 -._.-._.-._.-._.-._.-._.-._.-._.-._.-._.-._.-._.-._.-._.-._.-._.-._.-._
  N      .  UDP   .                                           .  UDP   .
  e      ..........     ..................                    ..........
  t      .  IPv6  .     .      IPv6      .                    .  IPv6  .
  w S    ..........     ..................                    ..........
  o C    .  SCHC  .     .  SCHC  .       .                    .        .
  r H    ..........     ..........       .                    .        .
  k C    .  LPWAN .     . LPWAN  .       .                    .        .
         ..........     ..................                    ..........
             ((((LPWAN))))             ------   Internet  ------
~~~~
{: #Fig-SCHCCOAP2 title='Different SCHC instances in a global system'}


This document defines a generic architecture for SCHC that can be used at any of these levels.
The goal of the architectural document is to orchestrate the different protocols and data model
defined by the LPWAN woeking group to design an operational and interoperable
framework for allowing IP application over contrained networks.


# SCHC Data Model

A SCHC instance, summarized in the {{Fig-Glob-Arch1}}, implies C/D and/or F/R present in both end and that both ends are provisionned with the same set of rules.

~~~~
       (-------)                                (-------)
       ( Rules )                                ( Rules )
       (-------)                                (-------)
        . read                                   . read
        .                                        .
       +-------+                                +-------+
   <===| R & D |<===                        <===| C & F |<===
   ===>| C & F |===>                        ===>| R & D |===>
       +-------+                                +-------+
       +-------+
~~~~
{: #Fig-Glob-Arch1 title='Summarized SCHC elements'}


To be able to provision end-points from different vendors, a common rule representation is needed that expresses the SCHC rules in an interoperable
fashion. To that effect, {{-Model}} defines a rule representation using the
[YANG](RFC7950) formalism.

{{I-D.ietf-lpwan-schc-yang-data-model}} defines an YANG data model to represent the rules. This enables the use of several protocols for rule management, such as NETCONF{{?RFC6241}}, RESTCONF{{?RFC8040}}, and CORECONF{{-COMI}}. NETCONF uses SSH, RESTCONF uses HTTPS, and CORECONF uses CoAP(s) as their respective transport layer protocols. The data is represented in XML under NETCONF, in JSON{{?RFC8259}} under RESTCONF and in CBOR{{?RFC8949}} under CORECONF.

~~~~
                  create
       (-------)  read   +=======+ *
       ( rules )<------->|Rule   |<--|-------->
       (-------)  update |Manager|   NETCONF, RESTCONF or CORECONF
          . read  delete +=======+   request
          .
       +-------+
   <===| R & D |<===
   ===>| C & F |===>
       +-------+
~~~~
{: #Fig-RM title='Summerized SCHC elements'}

The Rule Manager (RM) is in charge of handling data derived from the YANG Data
Model and apply changes to the rules database {{Fig-RM}}.

The RM is a application using the Internet to exchange information, therefore:

* for the network-level SCHC, the communication does not require routing. Each of the end-points having an RM and both RMs can be viewed on the same link, therefore wellknown Link Local addresses can be used to identify the device and the core RM. L2 security MAY be deemed as sufficient, if it provides the necessary level of protection.

* for application-level SCHC, routing is involved and global IP addresses SHOULD be used. End-to-end encryption is RECOMMENDED.

Management messages can also be carried in the negotiation protocol as proposed in {{-SCHCoPPP}}.
The RM traffic may be itself compressed by SCHC, especially if CORECONF is used, {{-SCHC-CoAP}} can be used.




# Security Considerations

SCHC is sensitive to the rules that could be abused to form arbitrary long
messages or as a form of attack against the C/D and/or F/R functions, say to
generate a buffer overflow and either modify the device or crash it. It is
thus critical to ensure that the rules are distributed in a fashion that is
protected against tempering, e.g., encrypted and signed.


# IANA Consideration

This document has no request to IANA

# Acknowledgements

The authors would like to thank (in alphabetic order):

--- back



