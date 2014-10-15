---
title: Web Push Channel Aggregation
abbrev: Push Aggregation
docname: draft-thomson-webpush-aggregate-latest
date: 2014-10-08
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
  RFC4648:
  RFC5988:
  RFC7159:
  I-D.ietf-appsawg-json-merge-patch:
  RFC3339:
  I-D.ietf-jose-json-web-encryption:
  RFC7231:

informative:


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
aggregation service URI.  The request contains a description of the set of
channels that are aggregated (see {{format}}).

The response is identical to the response to the `channel` resource, as
described in Section 5 of {{I-D.thomson-webpush-http2}}.  The 201 (Created)
response contains the URI of the aggregated channel in the Location header
field.  The response can also include information on when the aggregated channel
expires.

Messages pushed to the aggregated channel URI (see Section 3 of
{{I-D.thomson-webpush-http2}}) are forwarded to all of the channels that are
included in the provided list.  See {{push}} for details.


## Aggregated Channel Description Format {#format}

The content of a request to create an aggregated channel is a JSON {{RFC7159}}
object.  The keys in the object are the URIs of the channels being aggregated.
The corresponding value is an object containing the following keys:

expires:

: A date and time in {{RFC3339}} format that identifies when the provided
  channel becomes invalid.  The push server MUST remove the channel from the
  aggregated set when this time expires.  This field is optional, in which case
  the channel does not expire from the set.

encrypted_key:

: An encrypted version of the key that will be used to encrypt messages on the
  aggregated channel, encrypted for the recipient.  The value of this field is a
  JSON Web Encryption (JWE) Encrypted Key {{I-D.ietf-jose-json-web-encryption}}
  encoded using the Base 64 URL and Filename Safe alphabet {{RFC4648}}.

{:br }

Additional fields can be added to this format by updating this document.

This format is identified using a MIME media type of
"application/push-aggregation+json" (see {{iana}}).

Push aggregation services MUST support gzip Content-Encoding {{RFC7231}} for
this format.


## Determining Aggregated Channel Set Status

Editors note: This might needs to live on a different URI to avoid confusion
about what is being PUT there (for pushing) and all this gunk.  This design
overloads the uses of the resource, which is a bit stinky.

A GET request to the aggregated channel URI does not provide the last message
sent.  Instead, it produces the current set of channels that are included in
"application/push-aggregation+json" format.


## Modifying the Aggregated Channel Set

A PATCH request to the aggregated channel URI can be used to update the set of
channels that are included in the set.  This uses an request body containing a
JSON Merge {{I-D.ietf-appsawg-json-merge-patch}} document.


# Distributing Messages {#push}

Upon receipt of a PUT request, the aggregated channel resource sends a PUT
request to each unexpired channel in the set of resources it monitors.

This message is modified for each channel to include the JWE Encrypted Key that
is associated with the channel.

An aggregated channel resource MUST support the JWE JSON Serialization
{{I-D.ietf-jose-json-web-encryption}}.  The pushed message is modified to
include a `recipients` array containing a single object.  That object includes
an `encrypted_key` field that is taken from the `encrypted_key` field associated
with the channel.  Any existing `recipients` array MUST be removed.

An equivalent transform MAY be performed for the JWE Compact Representation
{{I-D.ietf-jose-json-web-encryption}} if that is supported.


# Security Considerations

The security considerations of {{I-D.thomson-webpush-http2}} apply, with
additional concerns around delivery of push messages to multiple devices, and
increased denial of service exposure.


## Message Encryption

Messages sent over aggregated push channels are encrypted.  Encrypted data are
available to all entities that are able to decrypt any JWE Encrypted Key.
Assuming that each device distributes its keying material to the application
server only, that means that push message contents can be decrypted by all
devices that have been in the aggregated channel set since the last time keys
were changed, plus the application server.

Removal of a channel from the aggregated set - or expiration of a channel - only
means that messages are not delivered to the corresponding device.  A device
that has been removed from the aggregated set can still decrypt any messages it
is able to acquire.

In order to ensure that removed devices are unable to decrypt messages, an
application server can generate a new content encryption key.  A change to the
content encryption key requires that all entries in the aggregated channel are
updated, which can be computationally expensive.  Applications can mitigate the
impact of removal by creating multiple aggregated channel sets of smaller sizes.
Alternatively, if push messages do not require confidentiality from the removed
devices, the same keys can be used.


## Packet Amplification

This protocol provides an application a way to use a relatively small message to
cause a large amount of data to be sent.  This adds considerably to the denial
of service risks the protocol poses to devices.

Of particular concern is access control for the aggregated channel URI.  The
aggregate channel URI is only used by the entity that requests its creation;
therefore, this can be ensured by making the URI difficult to guess.  In
particular, the same entropy requirements apply to aggregated channel URIs as
for other channel URIs (see {{I-D.thomson-webpush-http2}}).


# IANA Considerations {#iana}

This document relies on two new codepoint registrations for the aggregation
service link relation and the MIME media type of the document that describes
resources.

## Registration of Link Relation Type

A link relation for the channel aggregation resource is registered accordinging to
the rules in {{RFC5988}}.


## Registration of MIME Media Type

A new MIME media type, "application/push-aggregation+json" is registered
according to the rules in TODO.
