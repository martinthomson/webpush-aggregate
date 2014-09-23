---
title: Web Push to Multiple Devices
abbrev: Push Multiples
docname: draft-thomson-webpush-multi-latest
date: 2014-09-22
category: std

ipr: trust200902
area: General
workgroup: WebPush Working Group
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
  RFC3986:

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
directes pushed messages to multiple devices.


# Terminology

In cases where normative language needs to be emphasized, this document back on
established shorthands for expressing interoperability requirements on
implementations: the capitalized words "MUST", "MUST NOT", "SHOULD" and "MAY".
The meaning of these is described in {{RFC2119}}.


# List Registration Service

A new link relation, `....:push:multi`, is provided in response to a registration
request.  This link relation identifies a service that can be used to create an aggregated push channel that

Push servers provide a small number of different values for the `...:multi` link
relation.  Applications that with to send notifications to a large number of
users establish a list of

# Amending List Registrations


# Security Considerations

This protocol provides an application a way to use a relatively small message to
cause a large amount of data to be sent.  This adds considerably to the denial
of service risks the protocol poses to devices.  The basic mitigations in
{{I-D.thomson-webpush-http2}} apply, with a heightened urgency.
