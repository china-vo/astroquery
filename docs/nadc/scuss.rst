NADC SCUSS Queries
==================

``astroquery.nadc.scuss`` searches SCUSS source photometry, SDSS matches,
proper motions, and image metadata hosted by NADC.

Finding sources
---------------

Use ``query_sources`` for a cone search with magnitude and uncertainty limits.
Replace ``"your-nadc-token"`` with your Query Data access token before running
this example:

.. code-block:: python

   import astropy.units as u
   from astropy.coordinates import SkyCoord
   from astroquery.nadc.scuss import ScussClass

   client = ScussClass(token="your-nadc-token")
   center = SkyCoord(180*u.deg, 30*u.deg)
   sources = client.query_sources(
       center, 5*u.arcmin,
       mag_range=(14, 22), magerr_max=0.1,
       columns=["id", "ra", "dec", "psfmag", "psferr"],
       max_rows=100,
   )
   print(sources[:5])

The result is an `~astropy.table.Table`; an empty table means no sources matched.
``max_rows=100`` limits the result to 100 rows; see :ref:`nadc-query-limits`
for larger queries. The remaining examples reuse ``client`` and ``center``
from this block. For environment-variable tokens or timeouts, see
:ref:`nadc-query-authentication`.

The magnitude filters apply to
``psfmag`` and ``psferr``. Inspect ``list_columns`` for other available fields.

SDSS matches and proper motions
-------------------------------

Use ``query_sdss_matches`` for the archive's SCUSS/SDSS DR10 match table:

.. code-block:: python

   matches = client.query_sdss_matches(
       center, 5*u.arcmin, match_error_max=1.0,
       columns=["sdssobjid", "match_err", "psfmag_u", "psfmag_g"],
       max_rows=100,
   )
   print(matches[:5])

Use ``query_proper_motions`` to select motion ranges and the minimum number of
observations used in a measurement:

.. code-block:: python

   motions = client.query_proper_motions(
       center, 10*u.arcmin,
       pmra_range=(-50, 50), pmdec_range=(-50, 50),
       obsused_min=3, max_rows=100,
   )
   print(motions[:5])

Numeric ranges use the archive's column units. Check column metadata before
combining these measurements with another catalog.

Finding and downloading images
------------------------------

``query_images`` returns image metadata, including filenames and image-center
coordinates. Its default output format is JSON because some image VOTable
responses cannot be parsed reliably by Astropy:

.. code-block:: python

   images = client.query_images(
       center, 0.2*u.deg, seeing_max=2.0,
       columns=["id", "filename", "cra", "cdec", "seeing"],
       max_rows=20,
   )
   print(images[:5])

Inspect ``client.get_catalog_metadata(catalog="scuss-image")`` for available
file categories. For a returned image, request its PNG product:

.. code-block:: python

   if len(images):
       path = client.download_file(
           str(images["id"][0]), catalog="scuss-image", category="png",
           out_path="scuss-image.png", overwrite=False,
       )
       print(path)

The client selects the download parameters for the category. The returned
value is the local filename; no file is replaced when ``overwrite=False``.
See :ref:`nadc-query-downloads` for batch downloads and file replacement behavior.

Custom table queries
--------------------

The common methods select the following catalogs and tables:

.. list-table::
   :header-rows: 1
   :widths: 30 35 35

   * - Data
     - Catalog
     - Table
   * - Source photometry
     - ``scuss``
     - ``catalogue``
   * - SDSS matches
     - ``scuss``
     - ``sdss10``
   * - Image metadata
     - ``scuss-image``
     - ``image``
   * - Proper motions
     - ``scuss-proper-motion``
     - ``proper_motion``

Use ``list_tables`` and ``list_columns`` before choosing custom fields, and
pass a table name explicitly for the multi-table ``scuss`` catalog:

.. code-block:: python

   from astroquery.nadc import ColumnConstraint

   print(client.list_columns(catalog="scuss", table="catalogue"))
   selected = client.query_table(
       catalog="scuss", table="catalogue",
       coordinates=center, radius=5*u.arcmin,
       showcol=["id", "ra", "dec"],
       column_constraints=[ColumnConstraint.greaterequal("ra", 120)],
       max_rows=100,
   )
   print(selected[:5])

For ``scuss-cat``, check ``get_catalog_metadata`` for query support. This catalog
does not expose table metadata through the discovery API; use catalog-level
methods only when its metadata declares catalog-query support.
See :ref:`nadc-query-filters` for additional filter types, or
:ref:`nadc-query-errors` if a request fails.

Reference/API
-------------

.. automodapi:: astroquery.nadc.scuss
    :no-inheritance-diagram:
    :inherited-members:
