=========
Bookmarks
=========

.. contents::

.. _bookmark:

Bookmark model
==============

A bookmark has at least the following properties

.. object:: Bookmark

   :param int id: The bookmarks unique id
   :param string url: The Uniform Resource Locator that this bookmark represents
   :param string title: A short humanly readable label for the bookmark
   :param string description: A longer description or note on the bookmark
   :param int added: The UNIX timestamp when this bookmark was created
   :param string userId: The user ID of the nextcloud account that owns this bookmark
   :param array tags: A list of tags this bookmark is tagged with
   :param array folders: The folders this bookmark has been added to
   :param int clickcount: The number of times this bookmark was opened
   :param bool available: A boolean indicating whether the app could reach the bookmarked URL

      .. versionadded:: 3.4.0

   :param string|null htmlContent: The html content of the bookmarked web page

      .. versionadded:: 4.2.0

   :param string|null textContent: The html content of the bookmarked web page

      .. versionadded:: 4.2.0

   :param int|null archivedFile: The nextcloud file id, if this bookmark links to a non-HTML, non-plaintext file

      .. versionadded:: 3.4.0


Query bookmarks
===============

.. get:: /public/rest/v2/bookmark

   :synopsis: Filter and query all bookmarks by the authenticated user.

   .. versionadded:: 0.11.0

   :query tags[]: An array of tags that bookmarks returned by the endpoint should have
   :query page: if this is non-negative, results will be paginated by ``limit`` bookmarks a page. Default: ``0``.
   :query limit: Results will be paginated by this amount of bookmarks per page. Default: ``10``.
   :query sortby: The column to sort the results by; one of ``url``, ``title``, ``description``, ``public``, ``lastmodified``, ``clickcount``. Default: ``lastmodified``.
   :query search[]: An array of words to search for in the following columns ``url``, ``title``, ``description``, ``tags``
   :query conjunction: Set to ``and`` to require all search terms to be present, ``or`` if one should suffice. Default: ``or``
   :query folder: Only return bookmarks that are direct children of the folder with the passed ID. The root folder has id ``-1``.
   :query url: Only return bookmarks with this URL. With this parameter you can test whether a URL exists in the user's bookmarks.

      .. versionadded:: 1.0.0

   :query unavailable: Only return bookmarks that are dead links, i.e. return 404 status codes or similar.

      .. versionadded:: 3.4.0

   :query archived: Only return bookmarks that whose contents have been archived.

      .. versionadded:: 3.4.0

   :query untagged: Only return bookmarks that have no tags set

      .. versionadded:: 0.12.0

   :query duplicated: Only return bookmarks that are in multiple folders

      .. versionadded:: 10.2.0

   :>json string status: ``success`` or ``error``
   :>json array data: The list of resulting bookmarks

   Note that before v3.4.0 You couldn't mix ``folder``, ``url``, ``unavailable`` and ``archive``.

   **Example:**

   .. sourcecode:: http

      GET /index.php/apps/bookmarks/public/rest/v2/bookmark?tags[]=firsttag&tags[]=secondtag&page=-1 HTTP/1.1
      Host: example.com
      Accept: application/json

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json

      {
        "status": "success",
        "data": [{ "id": 7, "title": "Google", "tags": ["firsttag"] }]
      }

Create a bookmark
=================

.. post:: /public/rest/v2/bookmark

   :synopsis: Create a bookmark

   .. versionadded:: 0.11.0

   :param url: the url of the new bookmark
   :param array tags: Array of tags for this bookmark (these needn't exist and are created on-the-fly) (optional)
   :param string title: the title of the bookmark. (optional; If absent the title of the html site referenced by `url` is used)
   :param string description: A description for this bookmark (optional)
   :param array folders: An array of IDs of the folders this bookmark should reside in. (optional; if absent the new bookmark will be put in the root folder)

   :>json string status: ``success`` or ``error``
   :>json object item: The created bookmark

   **Example:**

   .. sourcecode:: http

      POST /index.php/apps/bookmarks/public/rest/v2/bookmark?tags[]=firsttag&tags[]=secondtag&page=-1 HTTP/1.1
      Host: example.com
      Accept: application/json

      {
        "url": "http://google.com",
        "title": "Google",
        "description":"in case i forget",
        "tags": ["search-engines", "uselessbookmark"]
      }

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json

      {
        "status": "success",
        "item": {
          "id": 7,
          "url": "http://google.com",
          "title": "Google",
          "description":"in case i forget",
          "tags": ["search-engines", "uselessbookmark"],
          "folders": [-1]
        }
      }

Get a bookmark
==============

.. get:: /public/rest/v2/bookmark/(int:id)

   :synopsis: Retrieve a bookmark

   .. versionadded:: 0.11.0

   :>json string status: ``success`` or ``error``
   :>json object item: The retrieved bookmark

   **Example:**

   .. sourcecode:: http

      GET /index.php/apps/bookmarks/public/rest/v2/bookmark/7 HTTP/1.1
      Host: example.com
      Accept: application/json


   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json

      {
        "status": "success",
        "item": {
          "id": 7,
          "url": "http://google.com",
          "title": "Google",
          "description":"in case i forget",
          "tags": ["search-engines", "uselessbookmark"],
          "folders": [-1]
        }
      }

Edit a bookmark
===============

.. put:: /public/rest/v2/bookmark/(int:id)

   :synopsis: Edit a bookmark

   .. versionadded:: 0.11.0

   :param url: the url of the new bookmark (optional; if absent will not be changed)
   :param array tags: Array of tags for this bookmark (these needn't exist and are created on-the-fly). (optional; if absent, will not be changed)
   :param string title: the title of the bookmark. (optional; if absent, will not be changed)
   :param string description: A description for this bookmark (optional; if absent, will not be changed)
   :param array folders: An array of IDs of the folders this bookmark should reside in, the bookmark will be removed from all other folders it may have resided in (optional; if absent, will not be changed)

   :>json string status: ``success`` or ``error``
   :>json object item: The new bookmark after editing

   **Example:**

   .. sourcecode:: http

      PUT /index.php/apps/bookmarks/public/rest/v2/bookmark/7 HTTP/1.1
      Host: example.com
      Accept: application/json

      { "title": "Boogle" }


   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json

      {
        "status": "success",
        "item": {
          "id": 7,
          "url": "http://google.com",
          "title": "Boogle",
          "description":"in case i forget",
          "tags": ["search-engines", "uselessbookmark"],
          "folders": [-1]
        }
      }

Delete a bookmark
=================

.. delete:: /public/rest/v2/bookmark/(int:id)

   :synopsis: Delete a bookmark. Note: Often you only want to remove a bookmark from a folder, not delete it from all folders. There is a different endpoint for the former.

   .. versionadded:: 0.11.0

   :>json string status: ``success`` or ``error``

   **Example:**

   .. sourcecode:: http

      DELETE /index.php/apps/bookmarks/public/rest/v2/bookmark/7 HTTP/1.1
      Host: example.com
      Accept: application/json


   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json

      {
        "status": "success"
      }

Get a preview image
===================

.. get:: /public/rest/v2/bookmark/(int:id)/image

   :synopsis: Retrieve the preview image of a bookmark

   .. versionadded:: 1.0.0

   **Example:**

   .. sourcecode:: http

      GET /index.php/apps/bookmarks/public/rest/v2/bookmark/7/image HTTP/1.1
      Host: example.com


   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: image/png

      ... binary data ...

Get a favicon
=============

.. get:: /public/rest/v2/bookmark/(int:id)/favicon

   :synopsis: Retrieve the favicon of a bookmark

   .. versionadded:: 1.0.0

   **Example:**

   .. sourcecode:: http

      GET /index.php/apps/bookmarks/public/rest/v2/bookmark/7/favicon HTTP/1.1
      Host: example.com


   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: image/png

      ... binary data ...

Export all bookmarks
====================

.. get:: /public/rest/v2/bookmark/export

   :synopsis: Export all bookmarks of the current user in a HTML file.

   .. versionadded:: 0.11.0

   **Example:**

   .. sourcecode:: http

      GET /index.php/apps/bookmarks/public/rest/v2/bookmark/export HTTP/1.1
      Host: example.com


   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: text/html

      <html>
      ...

Register that a bookmark was opened
==================================

.. post:: /public/rest/v2/bookmark/click

   :synopsis: Delete a bookmark. Note: Often you only want to remove a bookmark from a folder, not delete it from all folders. There is a different endpoint for the former.

   :query string url: The URL of the bookmark

   **Example:**

   .. sourcecode:: http

      POST /index.php/apps/bookmarks/public/rest/v2/bookmark/click?url=https://nextcloud.com/ HTTP/1.1
      Host: example.com
      Accept: application/json


   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json

      {
        "status": "success"
      }