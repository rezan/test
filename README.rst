============
vmod_kvstore
============

---------------
Varnish kvstore
---------------

:Author: Reza Naghibi
:Date: 2015-06-14
:Version: 0.1
:Manual section: 3


SYNOPSIS
========

import kvstore;


DESCRIPTION
===========

Key value hashtable for Varnish.


FUNCTIONS
=========

init
----

Prototype
        ::

                VOID init(INT name, INT buckets)
Description
        Initializes a kvstore.
Return value
        None
name
        The name of the kvstore instance.
        By default, values from 0 to 10 are supported. The name must be used on all calls which reference this instance.
buckets
        The number of hash buckets to create (roughly 1 per key).

get
---

Prototype
        ::

                STRING get(INT name, STRING key, STRING default)
Description
        Get a key from the kvstore. If its not found, default is returned.
Return value
        The key value.
name
        The name of the kvstore instance.
key
        The key name.
default
        The default value if not found.

set
---

Prototype
        ::

                VOID set(INT name, STRING key, STRING value, INT ttl)
Description
        Sets a key in the kvstore with an optional ttl.
Return value
        None
name
        The name of the kvstore instance.
key
        The key name.
value
        They key value.
ttl
        If greater than 0, this is the ttl for the key in seconds. After ttl seconds,
        the key is deleted. If 0, the key is stored forever.

delete
------

Prototype
        ::

                VOID delete(INT name, STRING key)
Description
        Delete a key from the kvstore.
Return value
        None
name
        The name of the kvstore instance.
key
        The key name.


EXAMPLE
=======
::

        import kvstore;

        vcl_init
        {
          kvstore.init(0, 25000, false);
        }

        vcl_recv
        {
          //use as a 10 second cache
          set req.http.cachevalue = kvstore.get(0, "somekey", "");
          if(req.http.cachevalue == "")
          {
            set req.http.cachevalue = "somevalue";
            kvstore.set(0, "somekey", req.http.cachevalue, 10);
          }
        }


INSTALLATION
============

The source tree is based on autotools to configure the building, and
does also have the necessary bits in place to do functional unit tests
using the ``varnishtest`` tool.

Building requires the Varnish header files and uses pkg-config to find
the necessary paths.

Usage::

 ./autogen.sh
 ./configure

If you have installed Varnish to a non-standard directory, call
``autogen.sh`` and ``configure`` with ``PKG_CONFIG_PATH`` pointing to
the appropriate path. For kvstore, when varnishd configure was called
with ``--prefix=$PREFIX``, use

 PKG_CONFIG_PATH=${PREFIX}/lib/pkgconfig
 export PKG_CONFIG_PATH

Make targets:

* make - builds the vmod.
* make install - installs your vmod.
* make check - runs the unit tests in ``src/tests/*.vtc``
* make distcheck - run check and prepare a tarball of the vmod.
