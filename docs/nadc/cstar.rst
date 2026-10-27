NADC CSTAR Queries
==================

``astroquery.nadc.cstar`` searches the CSTAR source catalog by sky position,
magnitude, and photometric uncertainty. Queries return Astropy tables.

Finding sources
---------------

Use ``query_sources`` to combine a cone search with magnitude and uncertainty
limits. Replace ``"your-nadc-token"`` with your Query Data access token before
running this example:

.. code-block:: python

   import astropy.units as u
   from astropy.coordinates import SkyCoord
   from astroquery.nadc.cstar import CstarClass

   client = CstarClass(token="your-nadc-token")
   center = SkyCoord(19.07925*u.deg, -87.35715556*u.deg)
   sources = client.query_sources(
       center, 5*u.arcsec,
       mag_range=(10, 13.5), magerr_max=0.05,
       columns=["id", "ra", "dec", "mag", "magerr"],
       max_rows=100,
   )
   print(sources[:5])

An empty table means no sources matched. ``max_rows=100`` limits the result to
100 rows; see :ref:`nadc-query-limits` for larger queries.
The remaining examples reuse ``client`` and ``center`` from this block.
For environment-variable tokens or timeouts, see :ref:`nadc-query-authentication`.

``mag_range`` selects an inclusive magnitude range; ``magerr_max`` sets the
maximum photometric uncertainty. Omit these filters for a position-only search,
or use ``query_region(center, radius=5*u.arcsec)``.

Inspecting available columns
----------------------------

Use catalog discovery before choosing additional fields or constructing a
custom query:

.. code-block:: python

   print(client.list_catalogs())
   print(client.list_tables())
   print(client.list_columns(table="catalog"))

``query_table`` accepts custom columns, filters, and sorting:

.. code-block:: python

   from astroquery.nadc import ColumnConstraint

   selected = client.query_table(
       table="catalog",
       coordinates=center, radius=5*u.arcsec,
       showcol=["id", "ra", "dec", "mag"],
       column_constraints=[ColumnConstraint.between("mag", 10, 13.5)],
       sort="mag", order="asc", max_rows=20,
   )
   print(selected[:5])

See :ref:`nadc-query-filters` for additional filter types and units.

For rectangular areas or a list of positions, use
``query_catalog_rectangle`` or ``query_catalog_proximity``. The default
coordinate group is ``"ra,dec"``; use ``pos_group`` to select another group
listed by the archive.

Downloading source products
---------------------------

Inspect ``client.get_catalog_metadata()`` for available product categories.
For a source returned by the query, download its FITS product:

.. code-block:: python

   if len(sources):
       path = client.download_file(
           str(sources["id"][0]), category="fits",
           out_path="cstar-source.fits", overwrite=False,
       )
       print(path)

Use ``download_products`` to request products for the returned source rows:

.. code-block:: python

   if len(sources):
       path = client.download_products(
           sources, fmt="targz", categories=["fits", "diurnal-fits"],
           out_path="cstar-products.tar.gz", overwrite=False,
       )
       print(path)

Both methods return a local path. See :ref:`nadc-query-downloads` for download
scope and file replacement behavior, or :ref:`nadc-query-errors` if a request
fails.

Reference/API
-------------

.. automodapi:: astroquery.nadc.cstar
    :no-inheritance-diagram:
    :inherited-members:
