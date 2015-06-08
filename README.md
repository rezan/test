============
vmod_kvstore
============


DESCRIPTION
===========

kvstore is a key value hashtable vmod. It will be optimized for speed and
concurrency by reusing data structures and ideas from varnish.


FUNCTIONS
=========


init
----

    VOID init(INT name, INT buckets, BOOLEAN readonly)

* Description

    Initializes a kvstore. All kvstores must be initialized before they can be used.

* name

    The name of the kvstore instance. By default, values from 0 to 10 are
    supported. The name must be used on all calls which reference this instance.

* buckets

    The number of hash buckets to create (roughly 1 per key).

* readonly

    If true, get operations will not lock and set/delete operations will not be allowed.


load_from_file
--------------

    INT load_from_file(INT name, STRING path, STRING delimiter)

* Description

    Loads a kvstore from file. The format is key + delimiter + value. The previous instance
    of the kvstore will be deleted and a new instance will be created from the file path. While
    loading the new instance, get and set operations will be permited on the previous instance.
    When completed, the previous instance will be safely destroyed.

* Return Value

    The number of keys loaded.

* name

    The name of the kvstore instance.

* path

    The path of the file.

* delimiter

    The delimiter for parsing the key and value.


get
---

    STRING get(INT name, STRING key, STRING default)

* Description

    Get a key from the kvstore. If its not found, default is returned.

* Return Value

    The key value.

* name

    The name of the kvstore instance.

* key

    The key name.

* default

    The default value if not found.


set
---

    VOID set(INT name, STRING key, STRING value, INT ttl)

* Description

    Sets a key in the kvstore with an optional ttl.

* name

    The name of the kvstore instance.

* key

    The key name.

* value

    The key value.

* ttl

    If greater than 0, this is the ttl for the key in seconds. After ttl seconds,
    the key is deleted. If 0, the key is stored forever.


delete
------

    VOID delete(INT name, STRING key)

* Description

    Delete a key from the kvstore.

* name

    The name of the kvstore instance.

* key

    The key name.


EXAMPLE
=======

```vcl

import kvstore;

vcl_init
{
  kvstore.init(0, 1000000, true);
  kvstore.init(1, 25000, true);
  kvstore.init(2, 25000, false);

  kvstore.load_from_file(0, "/some/path/hostmappings", "|");
  kvstore.load_from_file(1, "/some/path/urldata", "|");
}

vcl_recv
{
  if(req.url == "/reload")
  {
    kvstore.load_from_file(0, "/some/path/hostmappings", "|");
    kvstore.load_from_file(1, "/some/path/urldata", "|");
    return(synth(200, "reloaded"));
  }

  set req.http.mapping = kvstore.get(0, req.http.Host, "error");
  set req.http.data = kvstore.get(1, req.url, "error");

  //use as a 10 second cache
  set req.http.cachevalue = kvstore.get(2, "somekey", "");
  if(req.http.cachevalue == "")
  {
    set req.http.cachevalue = "somevalue";
    kvstore.set(2, "somekey", req.http.cachevalue, 10);
  }
}


```

