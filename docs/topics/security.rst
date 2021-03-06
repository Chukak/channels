Security
========

This covers basic security for protocols you're serving via Channels and
helpers that we provide.


WebSockets
----------

WebSockets start out life as a HTTP request, including all the cookies
and headers, and so you can use the standard :doc:`/topics/authentication`
code in order to grab current sessions and check user IDs.

There is also a risk of cross-site request forgery (CSRF) with WebSockets though,
as they can be initiated from any site on the internet to your domain, and will
still have the user's cookies and session from your site. If you serve private
data down the socket, you should restrict the sites which are allowed to open
sockets to you.

This is done via the ``channels.security.websocket`` package, and the two
ASGI middlewares it contains, ``OriginValidator`` and
``AllowedHostsOriginValidator``.

``OriginValidator`` lets you restrict the valid options for the ``Origin``
header that is sent with every WebSocket to say where it comes from. Just wrap
it around your WebSocket application code like this, and pass it a list of
valid domains as the second argument::

    from channels.security.websocket import OriginValidator

    application = ProtocolTypeRouter({

        "websocket": OriginValidator(
            AuthMiddlewareStack(
                URLRouter([
                    ...
                ])
            ),
            [".goodsite.com", "*"],
        ),
    })

Note: If you want to resolve any domains, just add ``["*"]`` as second argument.

``OriginValidator`` also has a ``full`` option. If this option is enabled,
``OriginValidator`` checks the domain schema and port of each ``Origin`` header.
To do this, pass the list of valid domains as the second argument:
``scheme://domain[:port]``. The port is an optional parameter, but recommended.
And just pass ``full=True`` as the third argument::

    application = ProtocolTypeRouter({

        "websocket": OriginValidator(
            AuthMiddlewareStack(
                URLRouter([
                    ...
                ])
            ),
            ["http://.goodsite.com", "https://domain.goodsite.com:443",
            "http://.goodsite.com:80"],
            full=True,
        ),
    })

Note: If you want to resolve any domains, just add ``["*"]`` as second argument.

``OriginValidator`` can accept the ``check_cert`` as fourth argument.
Works only when the ``full`` option is enabled. If the argument is specified,
then for ``https`` verifies the certificate. Just pass
``check_cert=True`` as fourth argument.

Often, the set of domains you want to restrict to is the same as the Django
``ALLOWED_HOSTS`` setting, which performs a similar security check for the
``Host`` header, and so ``AllowedHostsOriginValidator`` lets you use this
setting without having to re-declare the list::

    from channels.security.websocket import AllowedHostsOriginValidator

    application = ProtocolTypeRouter({

        "websocket": AllowedHostsOriginValidator(
            AuthMiddlewareStack(
                URLRouter([
                    ...
                ])
            ),
        ),
    })

``AllowedHostsOriginValidator`` will also automatically allow local connections
through if the site is in ``DEBUG`` mode, much like Django's host validation.
