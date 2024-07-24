---
title: "Generic Encapsulation of Messages for Bundle Protocol"
abbrev: "bp-gem"
category: std

docname: draft-taylor-dtn-bp-gem-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: "Internet"
workgroup: "Delay/Disruption Tolerant Networking"
keyword:
 - DTN
 - ipn
 - BPv7
venue:
  group: "Delay/Disruption Tolerant Networking"
  type: "Working Group"
  mail: "dtn@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/dtn/"
  github: "ricktaylor/bp-gem"
  latest: "https://ricktaylor.github.io/bp-gem/draft-taylor-dtn-bp-gem.html"

author:
 -
    fullname: Rick Taylor
    organization: High Frontier Ltd
    email: rick.taylor@highfrontier.co.uk

normative:
informative:

--- abstract

As the availability of Bundle Protocol version 7 [!RFC9171] becomes more pervasive, there is increasing need to port existing network services designed for the terrestrial Internet to function correctly and efficiently over Bundle Protocol.  This document introduces 2 modes:  Tunnel mode, whereby opaque messages are encapsulated and forwarded from an ingress gateway, and decapsulated at an egress gateway, across a Bundle Protocol segment of the end to end path; and Transport mode, whereby application messages are delivered from host to host using the Bundle Protocol as the end-to-end message transport protocol.

--- middle

# Introduction

Bundle Protocol version 7 is fundamentally a message passing protocol.  Individual messages are encapsulated as the opaque payload of bundles, and the bundles are transported across the network using just the information in the bundle envelope.  This encapsulation is sufficiently general purpose that porting existing message-based applications to use the bundle protocol is reasonably simple, and there is sufficient commonality of required behaviour that general rules can be recommended.

The encapsulation mechanism described address the following requirements:

1. The delivery of application messages across a network that features Bundle Protocol as part or all of the end-to-end path.

1. The delivery of messages that may have a relative sequential ordering.  I.e. message A must be delivered before message B, even if re-ordering occurs within the network.

1. The reliable delivery of some or all messages, such that an application designed to use a reliable transport layer can continue to hold the reliability assumption when operating over Bundle Protocol.

This document provides mechanisms that are applicable to applications and services that exchange framed messages.  The mechanisms described are not suitable for byte-streaming applications.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Generic encapsulation



## Abstract information fields

In order to deliver a particular application specific message from source to destination, the following information fields must be known:

* MESSAGE_ID:
  The unique identifier of the message.  If two messages have the same identifier then the message content must be logically identical, such that either message may be discarded without breaking a higher-level message protocol.

* SOURCE_ID:
  The identity of the source of the message.

* DESTINATION_ID:
  The identity of the destination of the message.

* SEQUENCE_ID (optional):
  If messages have a temporal relationship, such that one message must be processed before another, the identity of the sequence.

* SEQUENCE_NUMBER (optional):
  If messages have a temporal relationship, such that one message must be processed before another, the order of the message within the sequence.

* ACK_REQUIRED (optional):
  If an explicit acknowledgement of the successful delivery of a message is required from the receiver.

It may well be the case that some or all of these required information fields may be encoded in the application message format, or map directly to a field in the Bundle Protocol itself, but they are defined here in abstract terms in order to support the protocol specifications that follow.

# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
