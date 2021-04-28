---
stand_alone: true
ipr: trust200902
docname: draft-pelov-lpwan-architecture-01
cat: std
pi:
  symrefs: 'yes'
  sortrefs: 'yes'
  strict: 'yes'
  compact: 'yes'
  toc: 'yes'

title: LPWAN Static Context Header Compression (SCHC) Architecture
abbrev: LPWAN Architecture
wg: lpwan Working Group
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
  rfc8724:
  I-D.ietf-lpwan-schc-yang-data-model:
  I-D.ietf-lpwan-coap-static-context-hc:
  I-D.thubert-lpwan-schc-over-ppp:



--- abstract

This document defines the SCHC architecture.

--- middle

# Introduction {#Introduction}

The base operation and the definition of the SCHC compression & Fragmentation are now described in several documents published by the LPWAN working group.

Among them:

* The {{rfc8724}} defines the generic compression and fragmentation mechanisms for SCHC and applies it to IPv6 and UDP.
* The {{I-D.ietf-lpwan-coap-static-context-hc}} extend the compression to CoAP and OSCORE.
* The {{I-D.ietf-lpwan-schc-yang-data-model}} defines a rule representation using the YANG formalism.




As {{I-D.ietf-lpwan-coap-static-context-hc}} states, the SCHC compression and fragmentation mechanism can be implemented at different levels and/or managed by different organizations.
For instance, as represented figure {{Fig-SCHCCOAP2}}, IP compression and fragmentation may be managed by the network SCHC instance and end-to-end  compression between the device and the application. The former can itself be split in two instances since encryption
hides the field structure.


~~~~

         (device)            (NGW)                              (App)

         +--------+                                           +--------+
  A S    |  CoAP  |                                           |  CoAP  |
  p C    |  inner |                                           |  inner |
  p H    +--------+                                           +--------+
  . V    |  SCHC  |                                           |  SCHC  |
         |  inner |   cryptographical boundary                |  inner |
 ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  A S    |  CoAP  |                                           |  CoAP  |
  p C    |  outer |                                           |  outer |
  p H    +--------+                                           +--------+
  . C    |  SCHC  |                                           |  SCHC  |
         |  outer |   functional boundary                     |  outer |
 ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  N      .  udp   .                                           .  udp   .
  e      ..........     ..................                    ..........
  t      .  ipv6  .     .      ipv6      .                    .  ipv6  .
  w C    ..........     ..................                    ..........
  o S    .  schc  .     .  schc  .       .                    .        .
  r H    ..........     ..........       .                    .        .
  k C    .  lpwan .     . lpwan  .       .                    .        .
         ..........     ..................                    ..........
             ((((LPWAN))))             ------   Internet  ------

                Figure 3: OSCORE compression/decompression.

~~~~
{: #Fig-SCHCCOAP2 title='Different SCHC instances in a global system'}


This document defines a generic architecture for SCHC that can be used at any of these levels.
The goal of the architectural document is to orchestrate the different protocols and data model
defined by the LPWAN woeking group to design an operational and interoperable
framework for allowing IP application over contrained networks.

# Definitions

# Global architecture

As described in  {{rfc8724}} a SCHC service is composed of a Parser, analyzing
packets and creating a list of fields what will be used to match against the compression
rules. If a packet matches rules, compression can be applied following rules instructions.

If SCHC compressed packet is too large to be send in a single L2 frame, fragmentation
will apply. The process is similar, device rules are checked to find the most appropriate
fragmentation rule, regarding the SCHC packet size, the link error rate, the reliability
required by the application, ...

On the other direction, when a packet SCHC arrives, the ruleID is used to find the rule.
Its nature allows to select if it is a compression or fragmentation rule.

The rule database contains a set of rules specific to a single device. The {{rfc8724}}
indicates that the SCHC instance reads the rules to process C/D and F/R, rules are not
modified during these actions.


A SCHC instance, summarized in the {{Fig-Glob-Arch1}}, implies C/D and F/R present in both end.
The device connected to a constrained network is in one end and the other end
is either located in the core network or at the application.

In any cases, the rules must be the same in both ends to perform C/D and F/R.


~~~~
    (device)                                 (core|app)

     (---)                                     (---)
     ( r )                                     ( r )
     (---)                                     (---)
        . read                                   . read
        .                                        .
     +-----+                                  +-----+
 <===| R&D |<=..............................<=| C&F |<===
 ===>| C&F |=>..............................=>| R&D |===>
     +-----+                                  +-----+


~~~~
{: #Fig-Glob-Arch1 title='Summarized SCHC elements'}

To enable rule synchronization between both ends, a common rule representation must be defined.

# Data management

{{I-D.ietf-lpwan-schc-yang-data-model}} defines an YANG data model to represent the rules. This enables the use of several protocols for rule management, such as NETCONF, RESTCONF and CORECONF. NETCONF uses SSH, RESTCONF uses HTTPS, and CORECONF uses CoAP(s) as their respective transport layer protocols. The data is represented in XML under NETCONF, in JSON under RESTCONF and in CBOR under CORECONF.


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

Rule Manager (RM) is in charge of handling data derived from the YANG Data Model and apply changes to the rules database {{Fig-RM}}.

The RM is a application using the Internet to exchange information, therefore:

* for the network-level SCHC, the communication does not require routing. Each of the end-points having an RM and both RMs can be viewed on the same link, therefore wellknown Link Local addresses can be used to identify the device and the core RM. L2 security MAY be deemed as sufficient, if it provides the necessary level of protection.

* for application-level SCHC, routing is involved and global IP addresses SHOULD be used. End-to-end encryption is recommended.

Management messages can also be carried in the negotiation protocol as proposed in {{I-D.thubert-lpwan-schc-over-ppp}}

The RM traffic may be itself compressed by SCHC, especially if CORECONF is used, {{I-D.ietf-lpwan-coap-static-context-hc}} can be used.



# Acknowledgements

The authors would like to thank (in alphabetic order):




--- back



