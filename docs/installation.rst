Installation
============

Process
-------

Each of the following steps needs to be configured for the Debug Toolbar to be
fully functional.

1. Install the Package
^^^^^^^^^^^^^^^^^^^^^^

The recommended way to install the Debug Toolbar is via pip_::

    $ python -m pip install django-debug-toolbar

If you aren't familiar with pip, you may also obtain a copy of the
``debug_toolbar`` directory and add it to your Python path.

.. _pip: https://pip.pypa.io/

To test an upcoming release, you can install the in-development version
instead with the following command::

     $ python -m pip install -e git+https://github.com/jazzband/django-debug-toolbar.git#egg=django-debug-toolbar

If you're upgrading from a previous version, you should review the
:doc:`change log <changes>` and look for specific upgrade instructions.

2. Check for Prerequisites
^^^^^^^^^^^^^^^^^^^^^^^^^^

The Debug Toolbar requires two things from core Django. These are already
configured in Django’s default ``startproject`` template, so in most cases you
will already have these set up.

First, ensure that ``'django.contrib.staticfiles'`` is in your
``INSTALLED_APPS`` setting, and `configured properly
<https://docs.djangoproject.com/en/stable/howto/static-files/>`_:

.. code-block:: python

    INSTALLED_APPS = [
        # ...
        "django.contrib.staticfiles",
        # ...
    ]

    STATIC_URL = "static/"

Second, ensure that your ``TEMPLATES`` setting contains a
``DjangoTemplates`` backend whose ``APP_DIRS`` options is set to ``True``:

.. code-block:: python

    TEMPLATES = [
        {
            "BACKEND": "django.template.backends.django.DjangoTemplates",
            "APP_DIRS": True,
            # ...
        }
    ]

3. Install the App
^^^^^^^^^^^^^^^^^^

Add ``"debug_toolbar"`` to your ``INSTALLED_APPS`` setting::

    INSTALLED_APPS = [
        # ...
        "debug_toolbar",
        # ...
    ]

4. Add the URLs
^^^^^^^^^^^^^^^

Add django-debug-toolbar's URLs to your project's URLconf:

.. code-block:: python

    from django.urls import include, path

    urlpatterns = [
        # ...
        path('__debug__/', include('debug_toolbar.urls')),
    ]

This example uses the ``__debug__`` prefix, but you can use any prefix that
doesn't clash with your application's URLs.

5. Add the Middleware
^^^^^^^^^^^^^^^^^^^^^

The Debug Toolbar is mostly implemented in a middleware. Add it to your
``MIDDLEWARE`` setting:

.. code-block:: python

    MIDDLEWARE = [
        # ...
        "debug_toolbar.middleware.DebugToolbarMiddleware",
        # ...
    ]

.. warning::

    The order of ``MIDDLEWARE`` is important. You should include the Debug
    Toolbar middleware as early as possible in the list. However, it must come
    after any other middleware that encodes the response's content, such as
    :class:`~django.middleware.gzip.GZipMiddleware`.

.. _internal-ips:

6. Configure Internal IPs
^^^^^^^^^^^^^^^^^^^^^^^^^

The Debug Toolbar is shown only if your IP address is listed in Django’s
:setting:`INTERNAL_IPS` setting.  This means that for local
development, you *must* add ``"127.0.0.1"`` to :setting:`INTERNAL_IPS`.
You'll need to create this setting if it doesn't already exist in your
settings module:

.. code-block:: python

   INTERNAL_IPS = [
       # ...
       "127.0.0.1",
       # ...
   ]

You can change the logic of determining whether or not the Debug Toolbar
should be shown with the :ref:`SHOW_TOOLBAR_CALLBACK <SHOW_TOOLBAR_CALLBACK>`
option.

.. warning::

    If using Docker the following will set your ``INTERNAL_IPS`` correctly in Debug mode::

        if DEBUG:
            import os  # only if you haven't already imported this
            import socket  # only if you haven't already imported this
            hostname, _, ips = socket.gethostbyname_ex(socket.gethostname())
            INTERNAL_IPS = [ip[:-1] + '1' for ip in ips] + ['127.0.0.1', '10.0.2.2']

Troubleshooting
---------------

On some platforms, the Django ``runserver`` command may use incorrect content
types for static assets. To guess content types, Django relies on the
:mod:`mimetypes` module from the Python standard library, which itself relies
on the underlying platform's map files. If you find improper content types for
certain files, it is most likely that the platform's map files are incorrect or
need to be updated. This can be achieved, for example, by installing or
updating the ``mailcap`` package on a Red Hat distribution, ``mime-support`` on
a Debian distribution, or by editing the keys under ``HKEY_CLASSES_ROOT`` in
the Windows registry.

Cross-Origin Request Blocked
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The Debug Toolbar loads a `JavaScript module`_. Typical local development using
Django ``runserver`` is not impacted. However, if your application server and
static files server are at different origins, you may see `CORS errors`_ in
your browser's development console:

.. code-block:: text

    Cross-Origin Request Blocked: The Same Origin Policy disallows reading the remote resource at http://localhost/static/debug_toolbar/js/toolbar.js. (Reason: CORS header ‘Access-Control-Allow-Origin’ missing).

Or

.. code-block:: text

    Access to script at 'http://localhost/static/debug_toolbar/js/toolbar.js' from origin 'http://localhost:8000' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.

To resolve, configure your static files server to add the
`Access-Control-Allow-Origin header`_ with the origin of the application
server. For example, if your application server is at ``http://example.com``,
and your static files are served by NGINX, add:

.. code-block:: nginx

    add_header Access-Control-Allow-Origin http://example.com;

And for Apache:

.. code-block:: apache

    Header add Access-Control-Allow-Origin http://example.com

.. _JavaScript module: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules
.. _CORS errors: https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS/Errors/CORSMissingAllowOrigin
.. _Access-Control-Allow-Origin header: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Origin

Django Channels & Async
^^^^^^^^^^^^^^^^^^^^^^^

The Debug Toolbar currently doesn't support Django Channels or async projects.
If you are using Django channels are having issues getting panels to load,
please review the documentation for the configuration option
:ref:`RENDER_PANELS <RENDER_PANELS>`.
