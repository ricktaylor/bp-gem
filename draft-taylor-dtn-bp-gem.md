---
title: "Generic Encapsulation of Messages for Bundle Protocol"
abbrev: "bp-gem"
category: info

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

As the availability of Bundle Protocol version 7 {!RFC9171} becomes more pervasive, there is increasing need to port existing network services designed for the terrestrial Internet to function correctly and efficiently over Bundle Protocol.  This document describes a generic mechanism, in terms of abstract data fields and behaviours, to transport a message-based application protocol over the Bundle Protocol.

--- middle

# Introduction

Bundle Protocol version 7 is fundamentally a message passing protocol.  Individual messages are encapsulated as the opaque payload of bundles, and the bundles are transported across the network using just the information in the bundle envelope.  This encapsulation is sufficiently general purpose that porting existing message-based applications to use the bundle protocol is reasonably simple, and there is sufficient commonality of required behaviour that general rules can be recommended.  To reduce the complexity of performing such porting, this document details generic information model, rules and recommendations for the use of authors when defining an application protocol specific bundle encapsulation protocol.  The mechanisms are applicable to applications and services that exchange framed messages.  The mechanisms described are not suitable for byte-streaming applications.

The encapsulation mechanisms described address the following requirements:

1. The delivery of application messages across a network that features Bundle Protocol as part or all of the end-to-end path.

1. The delivery of messages that may have a relative sequential ordering.  I.e. message A must be delivered before message B, even if re-ordering occurs within the network.

1. The reliable delivery of some or all messages, such that an application designed to use a reliable transport layer can continue to hold the reliability assumption when operating over Bundle Protocol.


# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Generic encapsulation

The mechanisms to provide the functionality above are defined in terms of operations on generic information fields below.

## Abstract information fields

In order to deliver a particular application specific message from source to destination, the following information fields must be known:

* SOURCE_ID:
  The identity of the source of the message.

* MESSAGE_ID:
  The identifier of the message, unique within the scope of SOURCE_ID.  If two messages have the same identifier then the message content must be logically identical, such that either message may be discarded without breaking a higher-level message protocol.

* SEQUENCE_ID (optional):
  If messages have a temporal relationship, such that one message must be processed before another, the identity of the sequence.

* SEQUENCE_NUMBER (optional) (Numeric):
  If messages have a temporal relationship, such that one message must be processed before another, the order of the message within the sequence SEQUENCE_ID.

* IS_RELIABLE (optional) (Flag):
  If an explicit acknowledgement of the successful delivery of a message is required by the sender.

It may well be the case that some or all of these required information fields may be encoded in the application message format, or map directly to a field in the Bundle Protocol itself, but they are defined here in abstract terms in order to support the protocol specifications that follow.

### Identifiers {#ids}

The particular wire representation of an identifier is not specified, only the requirement that it MUST be able of uniquely representing a particular entity.  This may, for example, be through the concatenation of a textual representation, a numeric hash, or structured data.

The uniqueness of an identifier has a scope in which it MUST be unique.  SEQUENCE_ID and MESSAGE_ID identifiers MUST be unique for a given SOURCE_ID.  SOURCE_IDs MUST be unique within the scope of all messages that may be received by a protocol agent.

## Generic Sequencing

It may be the case that the higher-level application protocol requires that certain messages be processed in a particular order.  As Bundle Protocol does not enforce the ordering of messages, although individual Convergence Layer Adaptors may, a mechanism is defined to ensure that particular sequences of messages are delivered in order.  It should be noted that this mechanism is sufficiently flexible to allow multiple independent sequences of messages multiplexed with messages with no sequential requirements.

To enforce an ordering between messages, every message that requires strict ordering MUST include a SEQUENCE_ID and SEQUENCE_NUMBER.  The SEQUENCE_ID identifies the particular sequence, and the SEQUENCE_NUMBER defines the order of the message within the sequence.  Although there is no strict requirement to combine sequencing with reliability, implementations MUST separate reliable sequences of messages from unreliable sequences by using different SEQUENCE_IDs.

For each message within a given sequence, the sender assigns the SEQUENCE_ID, and attaches the SEQUENCE_NUMBER of each message as an unsigned integral number, starting at zero (0), and incrementing by one (1) with each subsequent message.   SEQUENCE_NUMBERs MUST NOT roll-over the implementation specific maximum integer value, and any protocol specification based on this document MUST specify the correct behaviour when an integer overflow would occur.

For each message received with a SEQUENCE_ID and SEQUENCE_NUMBER, a receiver must queue up the message until the next the expected message in the particular sequence arrives.  If the next expected SEQUENCE_NUMBER is the SEQUENCE_NUMBER of the new message, then the message is immediately delivered to the application, followed by any pending messages in sequence that are pending delivery.

If the message sof the sequence do not have the IS_RELIABLE flag, there MUST be one or more mechanisms defined in the corresponding application protocol specification that describes how to make progress when a message in an ordered sequence fails to arrive at the receiver, such as a timeout.

## Generic Reliability

Many message-based application protocols developed for the public Internet rely on the reliable delivery properties of the TCP/IP protocol.  When porting such applications, it may be too disruptive to update the application protocol to provide an acknowledgement mechanism in-band, and hence an acknowledgement mechanism is described that can be added at the encapsulation layer to provide similar reliability behaviour without requiring updates the the application protocol.

Reliability in this case means providing the sender sufficient feedback such that, after a reasonable period of time, it can guarantee that a message was either received by the destination, or will never be received by the destination.

In order to request such reliability, the sender MUST add the IS_RELIABLE flag to the message.

Because the reliability mechanism relies on the retransmission of messages that are assumed to have been lost by the network, a destination may receive multiple copies of messages with the the IS_RELIABLE flag.  In this case, only one instance of the message is delivered to the application layer, and duplicates silently dropped.  However, because there is no reliability mechanism for acknowledgement messages, even if a destination has already delivered a particular message, an acknowledgement MUST be sent, as the previous acknowledgement might have been dropped.

### Acknowledgements

The reliability mechanism requires the sending of acknowledgements from the destination back to the source.  These content of these acknowledgements depends on whether the acknowledgement concerns a message that is part of an ordered sequence or not.

Acknowledgements MAY be sent immediately, but because an acknowledgement message is in general very small, it is RECOMMENDED that individual acknowledgements be accumulated and send in the same bundle as an outgoing protocol specific message if possible, reducing the number of small bundles in the network.  Of course, if no outgoing messages are sent in a timely manner, accumulated acknowledgements MUST be sent promptly anyway in order to ensure progress.  The exact details of how much time is to be spent accumulating acknowledgements MUST be defined in a particular application protocol.

Although acknowledgements may piggyback reliable messages, they are not part of the message to be reliably delivered, and hence must be assumed unreliable and resent.

### Unsequenced Acknowledgement

If a message is received with the IS_RELIABLE flag, but without a SEQUENCE_ID, then the acknowledgement MUST include the MESSAGE_ID of the received message.

### Sequenced Acknowledgement

If a message is received with the IS_RELIABLE flag and a SEQUENCE_ID and SEQUENCE_NUMBER, then the acknowledgement MUST include the SEQUENCE_ID and SEQUENCE_NUMBER of the message.

It is important to note that the acknowledgement of the message MUST be sent, even if the message itself is queued up awaiting the arrival of a message with a prior SEQUENCE_NUMBER.  This behaviour enforces selective acknowledgement, whereby the sender receives notification that not only has a message been lost, but that later messages have arrived.  This mechanism allows senders to pipeline many message in an ordered sequence, but such a mechanism MUST be defined in a particular application protocol.

# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
