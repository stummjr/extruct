=======
extruct
=======

.. image:: https://img.shields.io/travis/scrapinghub/extruct.svg
    :target: https://travis-ci.org/scrapinghub/extruct

*extruct* is a library for extracting embedded metadata from HTML markup.

It also has a built-in HTTP server to test its output as JSON.

Currently, *extruct* only supports `W3C's HTML Microdata`_
and `embedded JSON-LD`_.

.. _W3C's HTML Microdata: http://www.w3.org/TR/microdata/
.. _embedded JSON-LD: http://www.w3.org/TR/json-ld/#embedding-json-ld-in-html-documents

The microdata algorithm is a revisit of `this Scrapinghub blog post`_ showing how to use EXSLT extensions.

.. _this Scrapinghub blog post: http://blog.scrapinghub.com/2014/06/18/extracting-schema-org-microdata-using-scrapy-selectors-and-xpath/

Roadmap
-------

- support for `RDFa Lite`_ (e.g. Facebook `Open Graph protocol metadata`_)

.. _RDFa Lite: http://www.w3.org/TR/rdfa-lite/
.. _Open Graph protocol metadata: http://ogp.me/#metadata


Installation
------------

::

    pip install extruct


Usage
-----

::

    >>> from pprint import pprint
    >>>
    >>> from extruct.w3cmicrodata import MicrodataExtractor
    >>>
    >>> # example from http://www.w3.org/TR/microdata/#associating-names-with-items
    >>> html = """<!DOCTYPE HTML>
    ... <html>
    ...  <head>
    ...   <title>Photo gallery</title>
    ...  </head>
    ...  <body>
    ...   <h1>My photos</h1>
    ...   <figure itemscope itemtype="http://n.whatwg.org/work" itemref="licenses">
    ...    <img itemprop="work" src="images/house.jpeg" alt="A white house, boarded up, sits in a forest.">
    ...    <figcaption itemprop="title">The house I found.</figcaption>
    ...   </figure>
    ...   <figure itemscope itemtype="http://n.whatwg.org/work" itemref="licenses">
    ...    <img itemprop="work" src="images/mailbox.jpeg" alt="Outside the house is a mailbox. It has a leaflet inside.">
    ...    <figcaption itemprop="title">The mailbox.</figcaption>
    ...   </figure>
    ...   <footer>
    ...    <p id="licenses">All images licensed under the <a itemprop="license"
    ...    href="http://www.opensource.org/licenses/mit-license.php">MIT
    ...    license</a>.</p>
    ...   </footer>
    ...  </body>
    ... </html>"""
    >>>
    >>> mde = MicrodataExtractor()
    >>> data = mde.extract(html)
    >>> pprint(data)
    {'items': [{'properties': {'license': 'http://www.opensource.org/licenses/mit-license.php',
                               'title': u'The house I found.',
                               'work': 'http://www.example.com/images/house.jpeg'},
                'type': 'http://n.whatwg.org/work'},
               {'properties': {'license': 'http://www.opensource.org/licenses/mit-license.php',
                               'title': u'The mailbox.',
                               'work': 'http://www.example.com/images/mailbox.jpeg'},
                'type': 'http://n.whatwg.org/work'}]}


REST API service
----------------

*extruct* also ships with a REST API service to test its output from URLs.

Dependencies
++++++++++++

* bottle_ (Web framework)
* gevent_ (Aysnc framework)
* requests_

.. _bottle: https://pypi.python.org/pypi/bottle
.. _gevent: http://www.gevent.org/
.. _requests: http://docs.python-requests.org/

Usage
+++++

::

    python -m extruct.service

launches an HTTP server listening on port 10005.

Methods supported
+++++++++++++++++

::

    /extruct/<URL>
    method = GET


    /extruct/batch
    method = POST
    params:
        urls - a list of URLs separted by newlines
        urlsfile - a file with one URL per line

E.g. http://localhost:10005/extruct/http://www.sarenza.com/i-love-shoes-susket-s767163-p0000119412

will output something like this:

::

    {
       "url":"http://www.sarenza.com/i-love-shoes-susket-s767163-p0000119412",
       "status":"ok",
       "microdata":{
          "items":[
             {
                "type":"http://schema.org/Product",
                "properties":{
                   "name":"Susket",
                   "color":[
                      "http://www.sarenza.com/i-love-shoes-susket-s767163-p0000119412",
                      "http://www.sarenza.com/i-love-shoes-susket-s767163-p0000119412"
                   ],
                   "brand":"http://www.sarenza.com/i-love-shoes",
                   "aggregateRating":{
                      "type":"http://schema.org/AggregateRating",
                      "properties":{
                         "description":"Soyez le premier \u00e0 donner votre avis"
                      }
                   },
                   "offers":{
                      "type":"http://schema.org/AggregateOffer",
                      "properties":{
                         "lowPrice":"59,00 \u20ac",
                         "price":"A partir de\r\n                  59,00 \u20ac",
                         "priceCurrency":"EUR",
                         "highPrice":"59,00 \u20ac",
                         "availability":"http://schema.org/InStock"
                      }
                   },
                   "size":[
                      "36 - Epuis\u00e9 - \u00catre alert\u00e9",
                      "37 - Epuis\u00e9 - \u00catre alert\u00e9",
                      "38 - Epuis\u00e9 - \u00catre alert\u00e9",
                      "39 - Derni\u00e8re paire !",
                      "40",
                      "41",
                      "42 - Derni\u00e8re paire !"
                   ],
                   "image":[
                      "http://cdn2.sarenza.net/static/_img/productsV4/0000119412/MD_0000119412_223992_09.jpg?201509221045",
                      "http://cdn1.sarenza.net/static/_img/productsV4/0000119412/MD_0000119412_223992_03.jpg?201509221045",
                      "http://cdn3.sarenza.net/static/_img/productsV4/0000119412/MD_0000119412_223992_04.jpg?201509221045",
                      "http://cdn2.sarenza.net/static/_img/productsV4/0000119412/MD_0000119412_223992_05.jpg?201509221045",
                      "http://cdn1.sarenza.net/static/_img/productsV4/0000119412/MD_0000119412_223992_06.jpg?201509221045",
                      "http://cdn1.sarenza.net/static/_img/productsV4/0000119412/MD_0000119412_223992_07.jpg?201509221045",
                      "http://cdn1.sarenza.net/static/_img/productsV4/0000119412/MD_0000119412_223992_08.jpg?201509221045",
                      "http://cdn2.sarenza.net/static/_img/productsV4/0000119412/MD_0000119412_223992_02.jpg?201509291747"
                   ],
                   "description":""
                }
             }
          ]
       }
    }


Development version
-------------------

::

    mkvirtualenv extruct
    pip install -r requirements-dev.txt


Tests
-----

Run tests in current environment::

    py.test tests


Use tox_ to run tests with different Python versions::

    tox


.. _tox: https://testrun.org/tox/latest/


Versioning
----------

Use bumpversion_ to conveniently change project version::

    bumpversion patch  # 0.0.0 -> 0.0.1
    bumpversion minor  # 0.0.1 -> 0.1.0
    bumpversion major  # 0.1.0 -> 1.0.0

.. _bumpversion: https://pypi.python.org/pypi/bumpversion
