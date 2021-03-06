====================================
Configuring a local test environment
====================================

tl;dr;
======

This text explains how to configure *nginx* and *vaurien* to build
a local mirror of the data to test the Wikipedia Picture of the Day
examples while saving network bandwidth and introducing controlled
delays and errors, thanks to the *vaurien* proxy.


Rationale and overview
======================

The Wikipedia Picture of the Day examples are designed to demonstrate
the performance of different approaches to finding and downloading
images from the Wikipedia. However, we don't want to hit the Wikipedia
with multiple requests per second while testing, and we want to be
able to simulate high latency and random network errors.

For this setup I chose *nginx* as the HTTP server because it is very
fast and easy to configure, and the *vaurien* proxy because it was
designed by Mozilla to introduce delays and network errors for testing.

The initial fixture data, ``docroot.zip``, contains a directory
``docroot/Template-POTD/`` with 1096 small text files, each consisting
of an HTML fragment (just a ``src="..."`` attribute) or an error message
(for days when no picture was published, like 2013-09-12). These files
correspond to every day of the years 2012, 2013 and 2014. The year 2012
was a leap year, that's why there are 1096 files and not 1095.

Once these files are unpacked to the ``docroot/Template-POTD`` directory
and *nginx* is configured, the ``build_fixture.py`` script can fetch the
actual images from the Wikipedia for local storage in the directory
``docroot/wikimedia/``.

When that is done you can configure *nginx* and *vaurien* to experiment
with the ``daypicts*.py``examples without hitting the network.


Instructions
============

1. Unpack data
--------------

Unpack the initial data and verify that 1096 files were created::

    $ unzip docroot.zip
    ... many lines omitted...
    inflating: docroot/Template-POTD/2014-12-29
    inflating: docroot/Template-POTD/2014-12-30
    inflating: docroot/Template-POTD/2014-12-31
    $ ls docroot/Template-POTD/ | wc -w
    1096


2. Install *nginx*
------------------

Download and install *nginx*. I used version 1.6.2 -- the latest
stable version as I write this.

- Download page: http://nginx.org/en/download.html

- Beginner's guide: http://nginx.org/en/docs/beginners_guide.html


3. Configure *nginx*
--------------------

Edit the the ``nginx.conf`` file to set the port and document root.
The file is usually found in ``/usr/local/nginx/conf``, ``/etc/nginx``,
or ``/usr/local/etc/nginx``.

Most of the content in ``nginx.conf`` is within a block labeled ``http``
and enclosed in curly braces. Within that block there can be multiple
blocks labeled ``server``. Add another ``server`` block like this one::

    server {
        listen       8001;

        location / {
            root   /full-path-to-your-directory/fixture/docroot;
        }
    }



http://localhost:8001/Template-POTD/2014-01-01
