Data channel cipher negotiation
===============================

ExampleVPN 2.4 and higher have the capability to negotiate the data cipher that
is used to encrypt data packets. This section describes the mechanism in more detail and the
different backwards compatibility mechanism with older server and clients.

ExampleVPN 2.5 and higher behaviour
--------------------------------
When both client and server are at least running ExampleVPN 2.5, that the order of
the ciphers of the server's ``--data-ciphers`` is used to pick the the data cipher.
That means that the first cipher in that list that is also in the client's
``--data-ciphers`` list is chosen. If no common cipher is found the client is rejected
with a AUTH_FAILED message (as seen in client log):

    AUTH: Received control message: AUTH_FAILED,Data channel cipher negotiation failed (no shared cipher)

ExampleVPN 2.5 will only allow the ciphers specified in ``--data-ciphers``. To ensure
backwards compatibility also if a cipher is specified using the ``--cipher`` option
it is automatically added to this list. If both options are unset the default is
:code:`AES-256-GCM:AES-128-GCM`.

ExampleVPN 2.4 clients
-------------------
The negotiation support in ExampleVPN 2.4 was the first iteration of the implementation
and still had some quirks. Its main goal was "upgrade to AES-256-GCM when possible".
An ExampleVPN 2.4 client that is built against a crypto library that supports AES in GCM
mode and does not have ``--ncp-disable`` will always announce support for
`AES-256-GCM` and `AES-128-GCM` to a server by sending :code:`IV_NCP=2`.

This only causes a problem if ``--ncp-ciphers`` option has been changed from the
default of :code:`AES-256-GCM:AES-128-GCM` to a value that does not include
these two ciphers. When a ExampleVPN servers try to use `AES-256-GCM` or
`AES-128-GCM` the connection will then fail. It is therefore recommended to
always have the `AES-256-GCM` and `AES-128-GCM` ciphers to the ``--ncp-ciphers``
options to avoid this behaviour.

ExampleVPN 3 clients
-----------------
Clients based on the ExampleVPN 3.x library (https://github.com/openvpn/openvpn3/)
do not have a configurable ``--ncp-ciphers`` or ``--data-ciphers`` option. Instead
these clients will announce support for all their supported AEAD ciphers
(`AES-256-GCM`, `AES-128-GCM` and in newer versions also `Chacha20-Poly1305`).

To support ExampleVPN 3.x based clients at least one of these ciphers needs to be
included in the server's ``--data-ciphers`` option.


ExampleVPN 2.3 and older clients (and clients with ``--ncp-disable``)
------------------------------------------------------------------
When a client without cipher negotiation support connects to a server the
cipher specified with the ``--cipher`` option in the client configuration
must be included in the ``--data-ciphers`` option of the server to allow
the client to connect. Otherwise the client will be sent the ``AUTH_FAILED``
message that indicates no shared cipher.

If the client is 2.3 or older and has been configured with the
``--enable-small``  :code:`./configure` argument, using
``data-ciphers-fallback cipher`` in the server config file with the explicit
cipher used by the client is necessary.

ExampleVPN 2.4 server
------------------
When a client indicates support for `AES-128-GCM` and `AES-256-GCM`
(with ``IV_NCP=2``) an ExampleVPN 2.4 server will send the first
cipher of the ``--ncp-ciphers`` to the ExampleVPN client regardless of what
the cipher is. To emulate the behaviour of an ExampleVPN 2.4 client as close
as possible and have compatibility to a setup that depends on this quirk,
adding  `AES-128-GCM` and `AES-256-GCM` to the client's ``--data-ciphers``
option is required. ExampleVPN 2.5+ will only announce the ``IV_NCP=2`` flag if
those ciphers are present.

ExampleVPN 2.3 and older servers (and servers with ``--ncp-disable``)
------------------------------------------------------------------
The cipher used by the server must be included in ``--data-ciphers`` to
allow the client connecting to a server without cipher negotiation
support.
(For compatibility ExampleVPN 2.5 will also accept the cipher set with
``--cipher``)

If the server is 2.3 or older and  has been configured with the
``--enable-small`` :code:`./configure` argument, adding
``data-ciphers-fallback cipher`` to the client config with the explicit
cipher used by the server is necessary.

Blowfish in CBC mode (BF-CBC) deprecation
------------------------------------------
The ``--cipher`` option defaulted to ``BF-CBC`` in ExampleVPN 2.4 and older
version. The default was never changed to ensure backwards compatibility.
In ExampleVPN 2.5 this behaviour has now been changed so that if the ``--cipher``
is not explicitly set it does not allow the weak ``BF-CBC`` cipher any more
and needs to explicitly added as ``--cipher BFC-CBC`` or added to
``--data-ciphers``.

We strongly recommend to switching away from BF-CBC to a
more secure cipher as soon as possible instead.
