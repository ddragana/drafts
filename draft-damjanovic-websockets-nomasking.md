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
only if the client uses an encrypted connection.

--- middle

# Introduction

The WebSocket protocol specifies that all frames sent from the client to the
server must be masked {{!RFC6455}}. This was introduced as a protection
against a possible attack on the infrastructure described in {{Section 10.3
of !RFC6455}}. The attack can be performed on intermediaries, such as caching
proxies. The affected intermediaries interpret data sent after the WebSocket
upgrade handshake as a new request and cache it according to the hostname
only. The attacker would send specially crafted WebSocket messages that will
be interpreted as a new request by the intermediaries and cached according to
the hostname chosen by the attacker. This leads to cache poisoning.

If end-to-end encryption is used the messages would be changed on the wire and
intermediaries will be denied access to the unencrypted traffic, but this does
not protect against the attack because the attacker has access to TLS keys and
can craft WebSocket messages that after encryption look like a cleartext HTTP
request. For the attack to succeed when an encrypted connection is used the
following is needed:

* the caching proxies described above would cache content from the encrypted
connection, on port 443, as plain text which is highly unlikely, and

* clients would need to access otherwise HTTPS capable site using insecure HTTP.
Considering existing techniques, e.g. HSTS {{!RFC6797}} this is less likely to
affect many users.

Considering that the upgrade mechanism is widely use now, therefore many
intermediaries have adopted it and that it is highly unlikely that the attack
would succeed if an encrypted connection is used, we propose to disable
masking if end-to-end encryption is used. Removing masking will remove
unnecessary processing.

This specification introduces a WebSocket extension that disables the masking
of frames sent from the client to the server. The support for the extension
will be advertised by the client (see {{Section 9 of !RFC6455}}). The server
may accept or decline the extension. The client can advertise the extension
only if an encrypted connection.


# Conventions and Definitions

{::boilerplate bcp14-tagged}

# The "no-masking" extension

The "no-masking" extension is negotiated using the WebSocket extension
mechanism described in {{Section 9 of !RFC6455}}. The client advertises support
for the extension by sending "no-masking" in the list of supported extensions
sent in the "Sec-WebSocket-Extensions" header field. The server accepts the
extension, by sending "no-masking" in the "Sec-WebSocket-Extensions" header
value.

The client MUST NOT send the extension if a non-secure connection is used.

If the "no-masking" extension is negotiated the client and the server behavior
are:

* The client MUST send data to the server without masking. The client sets the
field "frame-masked" to 0 on all frames. As defined in {{!RFC6455}}, the field
"frame-masking-key" will not be present.

* The server must only accept frames with the field "frame-masked" set to 0.
If the server receives a frame with the field "frame-masked" set to 1, it MUST
close the connection with the status code 1002 define in {{Section 7.4.1 of
!RFC6455}}.

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

