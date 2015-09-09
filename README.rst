============
vmod_kvstore
============

---------------
Varnish kvstore
---------------

:Author: Reza Naghibi
:Date: 2015-09-03
:Version: 0.3-SNAPSHOT
:Manual section: 3


SYNOPSIS
========

import kvstore;


DESCRIPTION
===========

High performance key value storage with optional TTLs for Varnish.


FUNCTIONS
=========

init
----

Prototype
        ::

                VOID init(INT name, INT buckets)
Description
        Initializes a kvstore. Can be used to re-initialize an active kvstore.
        The resulting kvstore is empty.
Return value
        None
name
        The name of the kvstore instance.
        By default, values from 0 to 9 are supported.
        The name must be used on all calls which reference this instance.
        Referencing an uninitialized instance in a kvstore call will result in a Varnish assertion error.
        Any previous instance for this name will be safely freed.
buckets
        The number of hash buckets to create (roughly 1 per key).

init_file
---------

Prototype
        ::

                 INT init_file(INT name, INT buckets, STRING path, STRING delimiter)
Description
        Equivalent to init(), but initializes from a file. Can be used to rebuild an active kvstore.
        Modifications to the kvstore are not synced back to the file.
Return value
        The numbers of keys loaded. -1 if the file cannot be read.
name
        The name of the kvstore instance.
        By default, values from 0 to 9 are supported.
        The name must be used on all calls which reference this instance.
        Referencing an uninitialized instance in a kvstore call will result in a Varnish assertion error.
        Any previous instance for this name will be safely freed.
buckets
        The number of hash buckets to create (roughly 1 per key).
path
        The path to the file with the key value data. 1 key and value per line.
delimiter
        The delimiter string used to seperate the key and the value. Only the first occurrence of the
        delimiter is used as the key value seperator. If the delimeter is not found or is empty, the whole line
        is treated as a key with an empty value.

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


size
----

Prototype
        ::

                INT size(INT name)
Description
        Get the current size of the kvstore.
Return value
        The size.
name
        The name of the kvstore instance.


compact
-------

Prototype
        ::

                INT compact(INT name)
Description
        Removed expired keys from the kvstore.
Return value
        The number of keys removed.
name
        The name of the kvstore instance.


RELOADING FROM FILE
===================

init_file() can be used to regularly sync data from file into a kvstore.
This function, and init(), can be safely used while threads are accessing the kvstore
being initialized. In the case of init_file(), threads will not block during this process
and will have access to the previous instance of the kvstore while the new kvstore is being
built from file. When ready, threads will be directed to the new kvstore and the previous
instance will be safely cleaned up.

Note that data from the previous instance will not carry over to the new instance.


MEMORY AND PERFORMANCE
======================

kvstore has a memory overhead of 100 bytes per bucket and 100 bytes per
key plus the actual key and value size. So for 1 million keys with a average 1KB
key and value size stored in 100,000 buckets, the memory usage is approximately:

::

    100000 buckets * 100 bytes = 10MB
    1000000 keys * 100 bytes = 100MB
    1000000 keys * 1KB size = 1GB

    TOTAL = 1.11GB

On a 4 core Intel Core i7 with an SSD, the above kvstore takes 1.25s to load
from file and has an average key read time of 600ns.

On the same system, a kvstore with 1 million keys, 100k buckets, 70% readers, and 30%
writers has a throughput of 3 millions operations per second.

Key length, value length, and overall kvstore size is only bounded by available memory.

When using TTLs, expired keys are only removed when they are landed on during a get() or set() operation
or when explicitly deleted. If you have a highly dynamic key set, expired keys
may never be landed on and your kvstore size can grow unchecked. This is due to the way keys
are hashed and the fact that each bucket stores keys in a red black tree. If this is the case,
its recommended to periodically call compact() to remove expired keys. You can use the size()
function to gauge kvstore growth.

EXAMPLE
=======
::

        import kvstore;

        vcl_init
        {
          kvstore.init(0, 25000);
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


HISTORY
=======

0.3

Init from file

0.2

Release version

0.1

Initial release
