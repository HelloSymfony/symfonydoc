.. index::
    single: Cache; Varnish

How to use Varnish to speed up my Website
如何使用Varnish来加速我的网页
=========================================

Because Symfony2's cache uses the standard HTTP cache headers, the
:ref:`symfony-gateway-cache` can easily be replaced with any other reverse
proxy. Varnish is a powerful, open-source, HTTP accelerator capable of serving
cached content quickly and including support for :ref:`Edge Side
Includes<edge-side-includes>`.
因为symfony2的缓存使用的是标准的HTTP缓存头文件，所以:ref:`symfony-gateway-cache`能够
便捷地与其他reverse proxy调换。Varnish是一个非常强大、开源的、能迅速缓存内容的
HTTP加速器，而且它支持:ref:`Edge Side
Includes<edge-side-includes>`。

.. index::
    single: Varnish; configuration

Configuration
配置
-------------

As seen previously, Symfony2 is smart enough to detect whether it talks to a
reverse proxy that understands ESI or not. It works out of the box when you
use the Symfony2 reverse proxy, but you need a special configuration to make
it work with Varnish. Thankfully, Symfony2 relies on yet another standard
written by Akamaï (`Edge Architecture`_), so the configuration tips in this
chapter can be useful even if you don't use Symfony2.
原来讲过，symfony2会检测它的reverse proxy是否支持ESI。你完全可以使用symfony2 reverse proxy，
但是要使它能够与Varnish一起工作，你还要配置一下。symfony2依赖于另一个Akamaï所规定的标准(`Edge Architecture`_)，
所以即使你不使用symfony2，这章讲的配置信息也对你有用。

.. note::

    Varnish only supports the ``src`` attribute for ESI tags (``onerror`` and
    ``alt`` attributes are ignored).
    varnish只支持ESI标签中的src属性（onerror和alt属性被忽略）。

First, configure Varnish so that it advertises its ESI support by adding a
``Surrogate-Capability`` header to requests forwarded to the backend
application:
首先，配置varnish，使它能支持ESI。将``Surrogate-Capability``头文件添加到要转发到后台的请求：

.. code-block:: text

    sub vcl_recv {
        set req.http.Surrogate-Capability = "abc=ESI/1.0";
    }

Then, optimize Varnish so that it only parses the Response contents when there
is at least one ESI tag by checking the ``Surrogate-Control`` header that
Symfony2 adds automatically:
然后，假如检测到symfony2自动添加的``Surrogate-Capability``头文件有至少一个ESI标签的话，
就通过优化varnish来使它只解析响应内容：

.. code-block:: text

    sub vcl_fetch {
        if (beresp.http.Surrogate-Control ~ "ESI/1.0") {
            unset beresp.http.Surrogate-Control;

            // for Varnish >= 3.0
            set beresp.do_esi = true;
            // for Varnish < 3.0
            // esi;
        }
    }

.. caution::

    Compression with ESI was not supported in Varnish until version 3.0
    (read `GZIP and Varnish`_). If you're not using Varnish 3.0, put a web
    server in front of Varnish to perform the compression.
    varnish知道3.0才支持ESI压缩（参阅`GZIP and Varnish`_）。如果你没有使用varnish3.0，
    就在varnish前方添加一个web server，从而实现压缩。

.. index::
    single: Varnish; Invalidation

Cache Invalidation
cache取消验证
------------------

You should never need to invalidate cached data because invalidation is already
taken into account natively in the HTTP cache models (see :ref:`http-cache-invalidation`).
由于HTTP缓存模型已经考虑到取消验证（invalidation），你永远也不需考虑取消验证缓存数据。（参见:ref:`http-cache-invalidation`）

Still, Varnish can be configured to accept a special HTTP ``PURGE`` method
that will invalidate the cache for a given resource:
但是，varnish仍可以被配置，从而使它能够接受HTTP PURGE方法，这个方法可以取消某个源的缓存验证：

.. code-block:: text

    sub vcl_hit {
        if (req.request == "PURGE") {
            set obj.ttl = 0s;
            error 200 "Purged";
        }
    }

    sub vcl_miss {
        if (req.request == "PURGE") {
            error 404 "Not purged";
        }
    }

.. caution::

    You must protect the ``PURGE`` HTTP method somehow to avoid random people
    purging your cached data.
    你必须保护PURGE方法，以免被人滥用。

.. _`Edge Architecture`: http://www.w3.org/TR/edge-arch
.. _`GZIP and Varnish`: https://www.varnish-cache.org/docs/3.0/phk/gzip.html