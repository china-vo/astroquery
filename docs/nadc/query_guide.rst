Configuration and advanced queries
==================================

This optional guide covers configuration and advanced queries for
:doc:`CSTAR <cstar>`, :doc:`FASHI <fashi>`, :doc:`Legacy Plate <legacyplate>`,
:doc:`SAGES <sages>`, and :doc:`SCUSS <scuss>`. Start with the individual module
pages for basic examples. LAMOST has its own configuration and pagination
rules; see the :doc:`LAMOST documentation <lamost>`.

The examples below use CSTAR. Use the table and column names documented for
your module when adapting them to another catalog.

.. _nadc-query-authentication:

Authentication and configuration
--------------------------------

The public Query Data service can require a token for catalog discovery,
queries, and downloads. Set ``ASTROQUERY_NADC_TOKEN`` in your environment before
creating a client, or pass your Query Data access token directly:

.. doctest::

  >>> import astropy.units as u
  >>> from astropy.coordinates import SkyCoord
  >>> from astroquery.nadc.cstar import CstarClass, conf
  >>> with conf.set_temp("timeout", 120):
  ...     client = CstarClass(token="your-nadc-token")
  >>> center = SkyCoord(19.07925*u.deg, -87.35715556*u.deg)

Replace ``"your-nadc-token"`` with your access token before executing requests.
All subsequent examples use this same ``client``. When a token is configured
through the environment or ``conf.token``, omit the constructor's token argument.

A constructor token takes precedence over ``conf.token``, the shared
``astroquery.nadc.conf.token``, module-specific environment variables, and the
shared environment token, in that order. Module-specific variables include
``ASTROQUERY_CSTAR_TOKEN``, ``ASTROQUERY_FASHI_TOKEN``,
``ASTROQUERY_LEGACYPLATE_TOKEN``, ``ASTROQUERY_SAGES_TOKEN``, and
``ASTROQUERY_SCUSS_TOKEN``. Pass ``token=""`` to request anonymous access.

The default authentication method is ``"bearer"``. Pass
``auth_method="query"`` for a service that requires a token query parameter.
The server URL, timeout, and token are read when a client is created; create a
new instance after changing those settings. For another deployment,
``conf.server`` must contain the complete API root, including
``/data/api/v1.0``.

.. _nadc-query-filters:

Custom columns and filters
--------------------------

Use ``list_tables`` and ``list_columns`` to inspect a catalog. Then use
``query_table`` for a named table, or ``query_catalog`` for a catalog-wide
selection. ``ColumnConstraint`` provides named constructors for filters:

.. code-block:: python

   from astroquery.nadc import ColumnConstraint

   print(client.list_tables())
   print(client.list_columns(table="catalog"))
   selected = client.query_table(
       table="catalog",
       coordinates=center, radius=5*u.arcsec,
       showcol=["id", "ra", "dec", "mag"],
       column_constraints=[ColumnConstraint.between("mag", 10, 13.5)],
       sort="mag", order="asc", max_rows=100,
   )
   print(selected[:5])

Use column names from the selected catalog and supply range bounds in lower,
upper order. ``ColumnConstraint`` constructs filters; it does not check whether
their values are appropriate for a catalog. See :ref:`nadc-filter-reference`
for the available comparisons.

Use angular quantities or strings such as ``"5 arcsec"`` for spatial radii.
Bare radii mean arcseconds for these five modules; bare SIAP sizes and rectangle
coordinates mean degrees. Numeric column filters use the archive's column
units. Inspect ``list_columns`` for names, descriptions, and units before
choosing custom filters.

For an offline request preview, add ``get_query_payload=True``:

.. doctest::

  >>> preview = client.query_sources(center, "5 arcsec", max_rows=10,
  ...                                get_query_payload=True)
  >>> isinstance(preview, dict)
  True

This returns request descriptions without executing the query. Previews can
contain authentication credentials; remove those before sharing them.

.. _nadc-query-results:

Working with results
--------------------

Catalog queries return an `~astropy.table.Table`. An empty table means that
no rows matched. Inspect or save the result of the custom query above:

.. code-block:: python

   print(selected.colnames)
   if len(selected):
       print(selected["id", "mag"][:5])
   selected.write("cstar-sources.ecsv", format="ascii.ecsv", overwrite=False)

Choose a new filename if it already exists, or use ``overwrite=True`` when you
intend to replace it.

Catalog queries preserve identifiers declared as text, including leading zeros.
When calling ``get_results`` directly, CSV and tab-separated identifiers may
instead be inferred as numbers. Prefer JSON or VOTable with declared types when
the service offers them, and check identifier columns before joining catalogs.

.. _nadc-query-limits:

Row limits and long queries
---------------------------

``max_rows`` limits the rows retrieved by a query. The default is the module's
``conf.row_limit`` (10,000); ``max_rows=-1`` retrieves all pages. ``page_size``
controls rows requested per page. A row limit reduces data transfer, but may
not reduce the server's work for a broad selection. Prefer spatial and column
filters for large catalogs.

Requests run synchronously. Increasing ``conf.timeout`` before creating a
client allows a longer wait per request, but does not guarantee that a broad
query will finish. For submitted jobs, ``submit_query`` and
``submit_table_query`` return a ``sqlid`` that can be passed to ``get_results``
or ``get_count``. The client does not poll a separate job-status endpoint.

.. _nadc-query-downloads:

Downloading products
--------------------

File downloads are available only for catalogs that declare them in
``get_catalog_metadata``. The module pages show supported products and examples.
Use ``download_products`` with query results to download the returned rows.
A batch download using ``sqlid`` can instead cover all matches in the submitted
query, independently of ``max_rows``.

Downloads return a local path. Specify ``out_path`` to choose the destination
and ``overwrite=False`` to keep an existing file. Failed downloads leave an
existing destination intact. Choose ``targz`` to download an archive, or
``urllist`` to save download links. ``samba`` returns a submission response;
the client cannot check whether that transfer has finished.

.. _nadc-query-errors:

Handling failures
-----------------

``LoginError`` means that authentication is required or needs attention. Check
the token and permissions, then create a new client. ``InvalidQueryError``
indicates an invalid parameter or unsupported catalog operation.
``requests.HTTPError`` and ``RemoteServiceError`` report service failures;
``TableParseError`` means a response could not be read as the requested data.
Use ``cache=False`` to request fresh query or metadata responses when needed.

.. _nadc-filter-reference:

Filter reference
----------------

.. autoclass:: astroquery.nadc.ColumnConstraint
   :members: equal, notequal, less, lessequal, greater, greaterequal, between, contains, in_
