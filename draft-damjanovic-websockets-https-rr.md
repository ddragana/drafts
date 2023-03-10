---
title: "Advertising the WebSockets support in the HTTPS resource record"
abbrev: "Advertising WebSockets support in HTTPSRR"
docname: draft-damjanovic-websockets-https-rr
category: info

ipr: trust200902
area: ART
workgroup: HTTP
keyword: Internet-Draft
stand_alone: yes
smart_quotes: no
pi: [toc, sortrefs, symrefs]

author:
 -
    name: Dragana Damjanovic
    organization: Microsoft
    email: dragana.damjano@gmail.com

normative:
  HTTPSRR: I-D.draft-ietf-dnsop-svcb-https

informative:


--- abstract

This specification introduces a mechanism to advertise the support for WebSockets
over different HTTP versions using HTTPS resource records {{HTTPSRR}}. This
mechanism allows clients to avoid delays in establishing WebSocket connections
using HTTP-based advertisement for WebSocket support.


--- middle

# Introduction

The mechanisms for running the WebSocket Protocol over a single stream of an
HTTP/2 and HTTP/3 connection are defined in {{!RFC8441}} and {{!RFC9220}}.
For bootstrapping WebSockets from HTTP/2 and HTTP/3 the extended CONNECT is used.
The support for the extended CONNECT is advertised using HTTP/2 and HTTP/3 settings
(see {{!RFC7540}} and {{!RFC9114}}). A client needs to establish an HTTP/2 or
HTTP/3 connection and wait for the setting frames to be exchanged to discover
whether it can try to use WebSockets over HTTP/2 or HTTP/3. The request still may
be rejected because the settings advertise the support for the extended CONNECT
but not explicitly the support for the WebSockets Protocol. The clients may choose
to attempt HTTP/2 or HTTP/3 first and fall back to HTTP/1.1 or HTTP/2 if the
WebSocket Protocol is not supported. This may add a delay. The other option is to
try to use WebSockets over HTTP/2 or HTTP/3 only on connections that are already
established and where it is known the extended CONNECT is supported. This approach
leads to WebSockets over HTTP/2 or HTTP/3 being used less frequently.

This specification adds a way to advertise the support for WebSockets over HTTP
versions using HTTPS resource record {{HTTPSRR}}. The client may choose to try
using an HTTP/2 or HTTP/3 connection only if the support for the protocol is
advertised. This will eliminate the delay in most cases and increase usage of
WebSockets over HTTP/2 and HTTP/3.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Extending HTTPS DNS resource record

This specification introduces the "wss" SvcParamKey (see {{HTTPSRR}}) that
indicates a set of HTTP versions that support the WebSocket Protocol on the
particular service endpoint. The HTTP versions are identified using alpn-id
specified in {{HTTPSRR}}.

The presentation value SHALL be a comma-separated list of one or more alpn-ids.
The wire format values for the "wss" SvcParamKey consists of at least one
alpn-id prefixed by its length as a single octet, and these length-value pairs
are concatenated to form the SvcParamValue. These pairs MUST exactly fill the
SvcParamValue; otherwise, the SvcParamValue is malformed.

All alpn-ids listed in the "wss" MUST also be present in the "alpn" key.

    example.net              IN HTTPS 1 . alpn=h2,h3 wss=h2,h3

# The Client Behavior


Upon receiving an HTTPS RR, a client should use the "wss" SvcParamKey as an
indication of whether a particular service endpoint supports the WebSocket
Protocol over HTTP /2 or HTTP/3.

If the key is present, that is a strong indication that the service endpoint
supports WebSockets over HTTP/2 or HTTP/3 protocol and the client can
attempt using WebSockets over HTTP/2 or HTTP/3 protocol. Due to difficulties
of deployments, the client may discover that the feature, although
advertised, is not supported and in this case, the client should fall back
to using HTTP/1.1.

If the "no-default-alpn" key is present, the WebSocket Protocol over HTTP/1.1
is not supported by the endpoint. Otherwise, it might be supported whether
the "wss" key is present or not.

If the "wss" key is not present, the client should not try using WebSockets over
HTTP/2 and HTTP/3, and should directly use HTTP/1.1.




# Security Considerations

This specification only adds a new SvcParamKey that is a hint of whether
the WebSockets over HTTP/2 and HTTP/3 are supported. Therefore, it does not
introduce additional security considerations beyond one described in
{{HTTPSRR}}, {{!RFC8441}} and {{!RFC9220}}.

# IANA Considerations

This specification adds the following entry to the Service Parameter Keys (SvcParamKeys) registry:

| --------- | ----------------- | ---------------- | --------------- |
| Number    | Name              | Meaning          | Format Reference |
| --------- | ----------------- | ---------------- | --------------- |
| XX        | wss               | Support for WebSockets over HTTP/2 and HTTP/3 | (This document) Section 3|
| --------- | ----------------- | ---------------- | --------------- |

--- back

# Acknowledgments
{:numbered="false"}

