NADC Legacy Plate Queries
=========================

``astroquery.nadc.legacyplate`` searches photographic plate records and
digitized image products hosted by NADC. Select plates by position, observation
year, observatory, telescope, or target name.

Finding plate records
---------------------

Use ``query_plates`` to search the ``legacyplateedr`` catalog. Replace
``"your-nadc-token"`` with your Query Data access token before running this
example:

.. code-block:: python

   from astroquery.nadc.legacyplate import LegacyplateClass

   client = LegacyplateClass(token="your-nadc-token")
   plates = client.query_plates(
       year_range=(1950, 1970), observatory="Shao",
       columns=["id", "year", "observat", "object"],
       sort="year", order="asc", max_rows=100,
   )
   print(plates[:5])

The result is an `~astropy.table.Table`; an empty table means no plates matched.
``max_rows=100`` limits the result to 100 rows; see :ref:`nadc-query-limits`
for larger queries. The remaining examples reuse ``client`` from this block.
For environment-variable tokens or timeouts, see :ref:`nadc-query-authentication`.

``year_range`` is inclusive. Observatory codes include ``Naoc``, ``Pmo``,
``Qdo``, ``Shao``, and ``Ynao``. Add ``telescope`` for an exact match in the
archive's telescope field or ``object_name`` to search target-name substrings.
For a spatial selection, also pass ``coordinates`` and an angular ``radius``
as in the image example below.

Finding digitized images
------------------------

Use ``query_plate_images`` with the same filters to search image metadata in
``legacyplate-image``:

.. code-block:: python

   import astropy.units as u
   from astropy.coordinates import SkyCoord

   center = SkyCoord(1*u.deg, 2*u.deg)
   images = client.query_plate_images(
       center, 5*u.arcmin, observatory="Naoc",
       columns=["filename", "year", "observat"], max_rows=20,
   )
   print(images[:5])

Both query methods return an `~astropy.table.Table`. Image results identify the
file in ``image_filename``; keep this column when selecting products to download.

Downloading images
------------------

Inspect ``client.get_catalog_metadata(catalog="legacyplate-image")`` for
available product categories. Download one FITS product from the image results:

.. code-block:: python

   if len(images):
       path = client.download_file(
           str(images["image_filename"][0]),
           catalog="legacyplate-image", category="fits",
           out_path="plate.fits", overwrite=False,
       )
       print(path)

To save links for all the returned images, use ``download_products``:

.. code-block:: python

   if len(images):
       path = client.download_products(
           images, catalog="legacyplate-image", id_column="image_filename",
           fmt="urllist", categories=["fits"],
           out_path="plate-links.txt", overwrite=False,
       )
       print(path)

Use ``fmt="targz"`` and a ``.tar.gz`` output filename to download an archive
instead of a link list. These methods return a local path. When ``out_path``
is omitted, files are saved under the module's astroquery cache directory.
See :ref:`nadc-query-downloads` for download scope and file replacement behavior,
or :ref:`nadc-query-errors` if a request fails.

Custom filters and column selection
-----------------------------------

Use ``list_tables`` and ``list_columns`` to inspect a catalog before choosing
custom fields. The image catalog can also be queried directly:

.. code-block:: python

   from astroquery.nadc import ColumnConstraint

   print(client.list_tables(catalog="legacyplate-image"))
   print(client.list_columns(catalog="legacyplate-image", table="image"))
   selected = client.query_table(
       catalog="legacyplate-image", table="image",
       showcol=["filename", "year", "observat"],
       column_constraints=[
           ColumnConstraint.between("year", 1950, 1970),
           ColumnConstraint.equal("observat", "Naoc"),
       ],
       sort="year", order="desc", max_rows=20,
   )
   print(selected[:5])

See :ref:`nadc-query-filters` for additional filter types and units.

Use ``query_catalog_rectangle`` or ``query_catalog_proximity`` for other spatial
selections. ``query_region`` provides a position-only cone search; use
``query_plates`` or ``query_plate_images`` when also filtering archive fields.

Image access and coordinate groups
----------------------------------

For an image-oriented search, ``query_siap`` uses the Simple Image Access
Protocol:

.. code-block:: python

   image_records = client.query_siap(
       center, 0.5*u.deg, catalog="legacyplate-image",
   )
   print(image_records[:5])

Use explicit angular units: bare SIAP sizes mean degrees, while bare cone
radii mean arcseconds. For tables with multiple coordinate groups, inspect
``list_coordinate_groups`` and select a ``coordinate_group`` with
``query_table_cone`` or a ``pos_group`` with ``query_table``.

Use ``list_docs`` to find the archive's catalog documentation and ``get_doc``
to retrieve a document by its identifier. ``list_catalogs`` normally includes
only queryable catalogs; pass ``queryable_only=False`` to include catalog
entries that provide documentation but cannot be queried.

Reference/API
-------------

.. automodapi:: astroquery.nadc.legacyplate
    :no-inheritance-diagram:
    :inherited-members:
