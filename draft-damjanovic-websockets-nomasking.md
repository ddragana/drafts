---
title: "WebSocket Extension to disable masking"
docname: draft-damjanovic-websockets-nomasking
category: exp

ipr: trust200902
area: ART
workgroup: HTTP
keyword: Internet-Draft

author:
 -
    name: Dragana Damjanovic
    organization: Microsoft
    email: dragana.damjano@gmail.com

normative:

informative:


--- abstract

The WebSocket protocol specifies that all frames sent from the client to the
server must be masked. This was introduced as a protection against a possible
attack on the infrastructure. With careful consideration, the masking could be
omitted when intermediaries do not have access to the unencrypted traffic.

This specification introduces a WebSocket extension that disables the mandatory
masking of frames sent from the client to the server. The extension is allowed
only under special circumstances where masking is known to be unnecessary.

--- middle

# Introduction

The WebSocket protocol specifies that all frames sent from the client to the
server must be masked {{!RFC6455}}. This was introduced as a protection against
a possible attack on the infrastructure described in {{Section 10.3 of
!RFC6455}}. The attack can be performed on intermediaries, such as proxies and
it could cause cache poisoning. Using end-to-end encryption, the attack can be
mitigated without the use of masking. This is because every intermediary will
be denied access to the unencrypted traffic, which prevents the caching attack.
The masking has been made mandatory for the connection using TLS to protect the
infrastructure that is behind TLS terminating proxies.

This specification introduces a WebSocket extension that disables the masking
of frames sent from the client to the server. The support for the extension
will be advertised by the client (see {{Section 9 of !RFC6455}}). The server
may accept the extension only after careful consideration discussed in Section
{{serv}}. The extension may only be advertised if secure transport is used.


# Conventions and Definitions

{::boilerplate bcp14-tagged}

# The "no-masking" extension

The "no-masking" extension is negotiated using the WebSocket extension
mechanism described in {{Section 9 of !RFC6455}}. The client advertises support
for the extension by sending "no-masking" in the list of supported extensions
sent in the "Sec-WebSocket-Extensions" header field. The server accepts the
extension, by sending "no-masking" in the "Sec-WebSocket-Extensions" header
value.

The client MUST NOT send the extension if a non-secure connection is not used
on the connection. The server MUST reject the upgrade request if the
"no-masking" extension is advertised on a non-secure connection.

If the "no-masking" extension is negotiated the client and the server behavior
are:

* The client MUST send data to the server without masking. The client sets the
field "frame-masked" to 0 on all frames. As defined in {{!RFC6455}}, the field
"frame-masking-key" will not be present.

* The server must only accept frames with the field "frame-masked" set to 0.
If the server receives a frame with the field "frame-masked" set to 1, it MUST
close the connection with the status code 1002 define in {{Section 7.4.1 of
!RFC6455}}.


## Server behavior considerations {#serv}


If a WebSocket connection is end-to-end encrypted the server can accept the
"no-masking" extension.

In case the connection is not end-to-end encrypted MUST NOT accept the
"no-masking" extension.

Intermediaries that terminate TLS connection should remove the extension from
the "Sec-WebSocket-Extensions" header field.

# Security Considerations

TBA

# IANA Considerations

IANA has registered the following WebSocket extension name in the "WebSocket
Extension Name Registry" defined in {{!RFC6455}}.

Extension Identifier: no-masking

Extension Common Name: Disable the WebSocket client-to-server masking

Extension Definition: This document.

Known Incompatible Extensions: None

The "no-masking" extension name is used in the "Sec-WebSocket-Extensions"
header in the WebSocket opening handshake to negotiate disabling of the
client-to-server masking.

--- back

# Acknowledgments
{:numbered="false"}

