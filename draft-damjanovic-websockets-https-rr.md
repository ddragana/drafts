---
title: "Advertising WebSocket support in the HTTPS resource record"
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
    organization: Mozilla
    email: dragana.damjano@gmail.com

normative:
  WSHTTP3: I-D.draft-ietf-httpbis-h3-websockets
  HTTPSRR: I-D.draft-ietf-dnsop-svcb-https
  HTTP3: I-D.draft-ietf-quic-http

informative:


--- abstract

This specification introduces a way to advertise the support for the extended CONNECT feature that
is used for running the WebSocket protocol over a single stream of an HTTP/2 and HTTP/3
connection using HTTPS resource records {{HTTPSRR}}.


--- note_Note_to_Readers

*RFC EDITOR: please remove this section before publication*

Discussion of this draft takes place on the HTTP working group mailing list (ietf-http-wg@w3.org), which is archived at <https://lists.w3.org/Archives/Public/ietf-http-wg/>.

Working Group information can be found at <https://httpwg.org/>; source code and issues list for this draft can be found at <https://github.com/httpwg/http-extensions/labels/h3-websockets>.

--- middle

# Introduction

The mechanisms for running the WebSocket protocol over a single stream of an HTTP/2 and HTTP/3
connection are defined in {{!RFC8441}} and {{WSHTTP3}}.  For bootstrapping WebSockets from
HTTP/2 and HTTP/3  extended CONNECT is used.  Support for the extended CONNECT is  advertised
using HTTP/2 and HTTP/3 settings (see {{!RFC7540}} and {{HTTP3}}).  Therefore, clients need
to establish a HTTP/2 or HTTP/3 connection and wait for the setting frames to be exchanged
to discover whether the connection can be used  for the WebSocket requests. This specification
adds a way to advertise the support for the extended CONNECT using HTTPS resource record {{HTTPSRR}}
which will eliminate the need for a HTTP/2 or HTTP/3 connection and introduce delays. The
clients can skip trying a HTTP/2 or HTTP/3 connection if the support for the extended CONNECT is
not advertised.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Extending HTTPS DNS resource record

This specification introduces the "extended-connect" SvcParamKey (see {{HTTPSRR}}) that  indicates that a
particular service endpoint supports the extended CONNECT feature defined in  {{!RFC8441}}
and {{WSHTTP3}}. The presentation and wire format values for the "extended-connect"
SvcParamKey MUST be empty.  If the "extended-connect" key is present, "alpn" MUST also be
specified with at least the h2 or h3 value.

# The Client Behavior

Upon receiving a HTTPS RR, a client should use the "extended-connect" SvcParamKey as an indication
whether a particular service endpoint supports the extended CONNECT feature.  If the key is not
present the client should not try using QUIC or TCP with the alpn set to h2 for requests that
need the feature, e.g.  WebSocket, and should directly use HTTP/1.1.

If the key is present, that is a strong indication that the service endpoint supports the
feature and the client should attempt using HTTP/2 or HTTP/3 protocol.  Due to difficulties of
deployments the client may discover that the feature, although advertised, is not supported and
in this case the client should fallback to using HTTP/1.1.


# Security Considerations

This specification only adds a hint of whether a feature is supported or not and therefore,
it does not introduce additional security considerations beyond one described in {{!RFC8441}}
and {{WSHTTP3}}.

# IANA Considerations

This specification adds the following entry to the Service Parameter Keys (SvcParamKeys) registry:

| --------- | ----------------- | ---------------- | --------------- |
| Number    | Name              | Meaning          | Format Reference |
| --------- | ----------------- | ---------------- | --------------- |
| 7         | extended-connect  | Support of the extended CONNECT feature | (This document) Section 3|
| --------- | ----------------- | ---------------- | --------------- |

--- back

# Acknowledgments
{:numbered="false"}

