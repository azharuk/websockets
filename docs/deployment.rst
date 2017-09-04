Deployment
==========

Application server
------------------

The author of ``websockets`` isn't aware of best practices for deploying
network services based on :mod:`asyncio`, let alone application servers.

You can run a script similar to the :ref:`server example <server-example>`,
inside a supervisor if you deem that useful.

You can also add a wrapper to daemonize the process. Third-party libraries
provide solutions for that.

If you can share knowledge on this topic, please file an issue_. Thanks!

.. _issue: https://github.com/aaugustin/websockets/issues/new

Graceful shutdown
-----------------

You may want to close connections gracefully when shutting down the server,
perhaps after executing some cleanup logic. There are two ways to achieve this
with the object returned by :func:`~websockets.server.serve`:

- using it as a asynchronous context manager, or
- calling its ``close()`` method, then waiting for its ``wait_closed()``
  method to complete.

Tasks that handle connections will be cancelled, in the sense that
:meth:`~websockets.protocol.WebSocketCommonProtocol.recv` raises
:exc:`~asyncio.CancelledError`.

On Unix systems, shutdown is usually triggered by sending a signal.

Here's a full example (Unix-only):

.. literalinclude:: ../example/shutdown.py

``async``, ``await``, and asynchronous context managers aren't available in
Python < 3.5. Here's the equivalent for older Python versions:

.. literalinclude:: ../example/oldshutdown.py

It's more difficult to achieve the same effect on Windows. Some third-party
projects try to help with this problem.

If your server doesn't run in the main thread, look at
:func:`~asyncio.AbstractEventLoop.call_soon_threadsafe`.

Port sharing
------------

The WebSocket protocol is an extension of HTTP/1.1. It can be tempting to
serve both HTTP and WebSocket on the same port.

The author of ``websockets`` doesn't think that's a good idea, due to the
widely different operational characteristics of HTTP and WebSocket.

If you need to respond to requests with a protocol other than WebSocket, for
example TCP or HTTP health checks, run a server for that protocol on another
port, within the same Python process, with :func:`~asyncio.start_server`.
