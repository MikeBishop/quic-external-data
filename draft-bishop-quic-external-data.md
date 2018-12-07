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
out of order in an HTTP message or to generate message body incrementally in
small chunks.  This document describes an HTTP/3 extension that facilitates
partial generation and out-of-order consumption of HTTP/3 message bodies.

--- middle

# Introduction

{{!HTTP3}} defines a mapping of HTTP semantics to the QUIC transport protocol
{{?QUIC=I-D.ietf-quic-transport}}. This mapping assumes a fully reliable
transport and is most easily used where the payload body is of a size known from
the beginning of the response.  A fully reliable transport of HTTP data is most
useful where the payload will only be useful when fully present or when consumed
in a streaming fashion.

Some HTTP message bodies are incrementally generated and have an indeterminate
size. {{!HTTP3}} requires the use of either multiple length-prefixed DATA frames
(increasing overhead) or a DATA frame which is the final frame of the stream
(preventing any other frames on the stream).

Other HTTP message bodies have a known internal structure, such that fragments
received out of order can be usefully consumed based on the offset or other
indicators within received data.  While {{?QUIC}} permits implementations to
expose out-of-order delivery capabilities, the design of HTTP/3 limits their
usefulness in HTTP/3 responses.

Extensions have been proposed which enable partial reliability in QUIC.
However, as HTTP/3 relies on reliable delivery of the frames which make up the
request stream, it becomes complex to indicate that loss of only certain pieces
of the stream can be tolerated.  For many video streaming applications,
particularly live content, partial reliability is attractive since content
delayed by loss repair is no longer relevant to the recipient, but consumes
bandwidth which could instead be used to send newer content.

This document permits the payload of a DATA frame to be sent unframed on a
separate unidirectional stream.  Following the required type byte, this stream
can safely be made partially reliable if the QUIC implementations support such a
state.  The content can be consumed out-of-order if the receiving QUIC
implementation exposes this capability. Since there is no requirement to
length-prefix the payload, the frame can be used for incrementally-generated
responses without losing the ability to send additional frames on the request
stream if the need arises.

# Negotiating Support

This extension MUST NOT be used unless the peer has declared support for it in
their SETTINGS frame (Section 4.2.5 of {{!HTTP3=I-D.ietf-quic-http}}).

This extension defines a new SETTINGS parameter:

SETTINGS_EXTERNAL_DATA_SUPPORTED (0x9):
: The default value is zero.  This extension is supported if the value is
  non-zero.

# Using the EXTERNAL_DATA frame

The EXTERNAL_DATA frame indicates the point in a request stream where a certain
extent of payload body would be transmitted, but transfers the actual payload
body on a separate unidirectional stream.  This frame MAY be used instead of the
DATA frame.

For HTTP messages where the payload has intervening frames (for example,
PUSH_PROMISE or DUPLICATE_PUSH frames), multiple EXTERNAL_DATA frames and
multiple unidirectional streams can be used just as multiple DATA frames would
be in {{!HTTP3}}.

## The EXTERNAL_DATA Frame {#frame-type}

The EXTERNAL_DATA frame (type=0xF) indicates a unidirectional stream which will
convey an arbitrary, variable-length sequences of bytes associated with an HTTP
request or response payload.

EXTERNAL_DATA frames MUST be associated with an HTTP request or response. If an
EXTERNAL_DATA frame is received on either control stream, the recipient MUST
respond with a connection error of type HTTP_WRONG_STREAM.

~~~ drawing
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                        Stream ID (i)                        ...
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~
{: #fig-frame title="EXTERNAL_DATA frame payload"}

The payload of an EXTERNAL_DATA frame is a variable-length integer containing a
QUIC stream ID.  This ID MUST identify a bidirectional stream initiated by the
sender of the EXTERNAL_DATA frame.  Receipt of an EXTERNAL_DATA frame with a
Stream ID of any other type MUST be treated as a stream error of type
HTTP_MALFORMED_FRAME.

Upon receipt, if the type of the indicated stream is not External Data (`0x44`),
this MUST be treated as a stream error of type `HTTP_UNKNOWN_STREAM_TYPE` on the
request stream.  The indicated unidirectional stream MAY still be consumed
according to its declared type, if understood.

An EXTERNAL_DATA frame is interpreted as if it were a DATA frame whose payload
were the full contents of the indicated unidirectional stream.  Multiple
EXTERNAL_DATA frames MUST NOT indicate the same unidirectional stream; if a
subsequent EXTERNAL_DATA frame indicates the same unidirectional stream ID, this
MUST be treated as a stream error of type HTTP_WRONG_STREAM_COUNT.

## The External Data Stream Type {#stream-type}

An external data stream is indicated by a stream type of `0x44` (ASCII 'D').
There is no additional framing on this unidirectional stream; the entire stream
payload is interpreted as a fragment of the HTTP message body.

# Security Considerations {#security}

If there is a long delay between receipt of an External Data stream and the
corresponding EXTERNAL_DATA frame, regardless of the order, this places
additional stress on the receiver. The receiver SHOULD NOT consume payload from
an External Data stream before receiving the EXTERNAL_DATA frame which indicates
to which HTTP exchange the body belongs.  This permits flow control to limit the
amount of data which cannot be reliably interpreted.

If an EXTERNAL_DATA frame is received which indicates a unidirectional stream
which does not arrive in a timely manner, the same mitigations should be
employed as if a DATA frame's header arrived but the payload were delayed.  A
reasonable timeout should be used to ensure that the request and response are
transferred in a timely manner.

# IANA Considerations {#iana}

## Frame Type {#iana-frame}

This document registers a new entry in the "HTTP/3 Frame Type" registry defined
in {{!HTTP3}}.

Frame Type:
: EXTERNAL_DATA

Code:
: 0x0F

Specification:
: This document, {{frame-type}}

## Stream Type {#iana-stream}

This document registers a new entry in the "HTTP/3 Stream Type" registry defined
in {{!HTTP3}}.

Stream Type:
: External Data

Code:
: 0x44

Specification:
: This document, {{stream-type}}

--- back

