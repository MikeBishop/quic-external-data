---
title: EXTERNAL_DATA Frame for HTTP/3
abbrev: HTTP/3 EXTERNAL_DATA
docname: draft-bishop-quic-external-data-latest
date: {DATE}
category: std
updates: 8336

ipr: trust200902
area: Transport
workgroup: QUIC
keyword: Internet-Draft

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
  -
    ins: M. Bishop
    name: Mike Bishop
    organization: Akamai
    email: mbishop@evequefou.be

normative:

informative:

--- abstract

In certain applications, it is useful to be able to process data as it arrives
out of order in an HTTP message.  This document describes an HTTP/3 extension
that facilitates partial generation and out-of-order consumption of HTTP/3
message bodies.

--- middle

# Introduction

This is important.

# Negotiating Support

Define a setting here.

# Using the EXTERNAL_DATA frame

Define a frame type and unidirectional stream type here.

# Security Considerations {#security}

What if the stream is delayed?  What if a stream is unreferenced?

# IANA Considerations {#iana}

This document has actions for IANA.  Define them.

--- back

