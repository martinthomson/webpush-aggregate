---
title: Web Push Channel Aggregation
abbrev: Push Aggregation
docname: draft-thomson-webpush-aggregation-latest
date: 2014-09-22
category: std

ipr: trust200902
area: RAI
workgroup: WebPush
keyword:
 - Internet-Draft
 - Push
 - Web Push
 - Notification
 - Broadcast
 - Multicast

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
 -
    ins: M. Thomson
    name: Martin Thomson
    org: Mozilla
    street: 331 E Evelyn Street
    city: Mountain View
    code: 94041
    country: United States
    email: martin.thomson@gmail.com

normative:
  RFC2119:
  I-D.thomson-webpush-http2:
  RFC5988:
  RFC7159:
  I-D.ietf-appsawg-json-merge-patch:
  RFC3339:

informative:
  JWE:
    target: TODO
    title: JWE
    author:
      name: Michael Jones
      ins: M. Jones
      org: Microsoft
    date: 2014-09-22


--- abstract

The Web Push protocol provides a means of ensuring constant network availability
of devices that would otherwise have limited availability.  This document
describes extensions to that protocol that enable the efficient delivery of
messages to multiple devices.  This allows an application to request that a web
push server deliver the same message to a potentially large set of devices.

--- middle


# Introduction {#intro}

The delivery of the same message to large numbers of devices is a common feature
of push notification services.  This document describes a mechanism based on the
Web Push protocol {{I-D.thomson-webpush-http2}}.

A new link relation is added to the Web Push registration response.  This
identifies a service that can be used to create a push channel endpoint that
aggregates multiple individual push channels.

Applications can use the aggregated channel to deliver the same push message on
all of the aggregated channels with a single request.  This makes the
large-scale delivery of identical messages more efficient.


# Terminology

In cases where normative language needs to be emphasized, this document back on
established shorthands for expressing interoperability requirements on
implementations: the capitalized words "MUST", "MUST NOT", "SHOULD" and "MAY".
The meaning of these is described in {{RFC2119}}.


# List Registration Service

A new link relation {{RFC5988}}, `....:push:aggregate`, is provided in response
to a push registration or channel creation request.  This link relation
identifies an aggregation service that can be used to create a new aggregated
push channel.

If the link relation is provided in response to a push registration creation
request, it applies to all channels created on that registration; if the link
relation is provided in response to a channel creation request, it applies to
just that channel.

Applications that send notifications to a large number of users first establish
a list of devices that have the same aggregation service URI.  Push servers
provide a small number of different values for the aggregate link relation.


Note:

: Though the use of different push servers will ensure that applications will
need to support multiple aggregation services, a large number of endpoints
diminishes the value of having messages distributed by the push server.

{:br }

Absence of the `...:aggregate` link relation indicates that the push server does
not support channel aggregation.


## Creating an Aggregated Channel

A new aggregated channel is created by sending an HTTP POST request to the
aggregation service URI.  The request contains

The response is identical to the response to the `channel` resource, as
described in Section 5 of {{I-D.thomson-webpush-http2}}.  The 201 (Created)
response contains the identity of the aggregated channel in the Location header
field.

Messages pushed to the aggregated channel URI (see Section 3 of
{{I-D.thomson-webpush-http2}}) are forwarded to all of the channels that are
included in the provided list.


## Aggregation Channel Request Format

The content of this request is a JSON {{RFC7159}} object.  The keys in the object
are the URIs of the channels being aggregated.  The corresponding value is an
object containing the following keys:

expires:

: A date and time in {{RFC3339}} format that identifies when the provided
  channel becomes invalid.  The push server MUST remove the channel from the
  aggregation set when this time expires.  This field is optional, in which case
  the channel does not expire.

pubkey:

: The public key to be used for encrypting messages on ths channel. This field
  is optional.  [[TBD: This - primarily the corresponding CPU load - is probably
  the largest problem with this security architecture.]]

{:br }

This format is identified using a MIME media type of
"application/push-aggregation+json" {{IANA}}.

Push aggregation services MUST support gzip Content-Encoding for this format.


## Determining Aggregation Set Status

Editors note: This might needs to live on a different URI to avoid confusion
about what is being PUT there (for pushing) and all this stuff.

A GET request to the aggregated channel URI does not provide the last message
sent.  Instead, it produces the current set of channels that are included in
"application/push-aggregation+json" format.


## Modifying the Aggregation Set

A PATCH request to the aggregated channel URI can be used to update the set of
channels that are included in the set.  This uses an request body containing a
JSON Merge {{I-D.ietf-appsawg-json-merge-patch}} document.


# Security Considerations

This protocol provides an application a way to use a relatively small message to
cause a large amount of data to be sent.  This adds considerably to the denial
of service risks the protocol poses to devices.  The basic mitigations in
{{I-D.thomson-webpush-http2}} apply, though these are significantly more
important.

Of particular concern is access control to the aggregated channel URI.  The
aggregate channel URI is only used by the entity that requests its creation;
therefore, this can be ensured by making the URI difficult to guess.  That is,
the same entropy requirements apply to aggregated channel URIs as for other
channel URIs.

Messages sent over aggregated push channels do not have confidentiality and
integrity protection, unless applications provide a mechanism within the message
payload.  Since the information is pushed to multiple recipients, these channels
are unsuitable for confidential information.


# IANA Considerations {#IANA}

TODO: expand with details


## Registration of Link Relation Type

A link relation for the link aggregation resource is registered accordinging to
the rules in {{RFC5988}}.


## Registration of MIME Media Type

A new MIME media type, "application/push-aggregation+json" is registered
according to the rules in TODO.
