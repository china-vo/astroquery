NADC SAGES Queries
==================

``astroquery.nadc.sages`` searches SAGES photometric catalogs and stellar
parameters hosted by NADC. The query methods select these catalogs and tables:

.. list-table::
   :header-rows: 1
   :widths: 30 35 35

   * - Data
     - Method
     - Catalog / table
   * - UV photometry
     - ``query_uv_sources``
     - ``SAGES-DR1`` / ``dr1_uv``
   * - gri photometry
     - ``query_gri_sources``
     - ``SAGES-DR1`` / ``dr1s_gri``
   * - Stellar parameters
     - ``query_stellar_parameters``
     - ``SAGES-StellarParameters`` / ``sages_param``

Finding photometric sources
---------------------------

Use ``query_uv_sources`` for magnitude, uncertainty, and flag selections in the
UV catalog. Replace ``"your-nadc-token"`` with your Query Data access token
before running this example:

.. code-block:: python

   import astropy.units as u
   from astropy.coordinates import SkyCoord
   from astroquery.nadc.sages import SagesClass

   client = SagesClass(token="your-nadc-token")
   center = SkyCoord(180*u.deg, 30*u.deg)
   uv_sources = client.query_uv_sources(
       center, 5*u.arcsec,
       mag_u_range=(14, 21), err_u_max=0.1, flag_u=0,
       max_rows=100,
   )
   print(uv_sources[:5])

The result is an `~astropy.table.Table`; an empty table means no sources matched.
``max_rows=100`` limits the result to 100 rows; see :ref:`nadc-query-limits`
for larger queries. The remaining examples reuse ``client`` and ``center``
from this block. For environment-variable tokens or timeouts, see
:ref:`nadc-query-authentication`.

Use ``query_gri_sources`` for the gri catalog, with separate filters for each
band:

.. code-block:: python

   gri_sources = client.query_gri_sources(
       center, 5*u.arcsec,
       mag_g_range=(14, 21), err_g_max=0.1, flag_g=0,
       max_rows=100,
   )
   print(gri_sources[:5])

Both methods return an `~astropy.table.Table`. Use ``columns`` to select output
fields after inspecting their names with ``list_columns``.

Selecting stellar parameters
----------------------------

Use ``query_stellar_parameters`` to select temperature and metallicity ranges
with uncertainty or RUWE limits:

.. code-block:: python

   stars = client.query_stellar_parameters(
       center, 5*u.arcsec,
       teff_range=(4500, 6500), feh_range=(-1.0, 0.5),
       err_teff_max=150, ruwe_max=1.4,
       max_rows=100,
   )
   print(stars[:5])

The position is optional. Omitting both ``center`` and ``radius`` selects
across the catalog, so choose column filters and a row limit for large searches.

Inspecting tables and custom queries
------------------------------------

Select the catalog explicitly when inspecting its tables and columns:

.. code-block:: python

   print(client.list_tables(catalog="SAGES-StellarParameters"))
   print(client.list_columns(
       catalog="SAGES-StellarParameters", table="sages_param",
   ))

For custom selections, use ``query_table`` with the table names above and
``ColumnConstraint`` filters. For example:

.. code-block:: python

   from astroquery.nadc import ColumnConstraint

   selected = client.query_table(
       catalog="SAGES-DR1", table="dr1_uv",
       coordinates=center, radius=5*u.arcsec,
       showcol=["ra", "dec"],
       column_constraints=[ColumnConstraint.greaterequal("ra", 120)],
       max_rows=100,
   )
   print(selected[:5])

See :ref:`nadc-query-filters` for additional filter types,
:ref:`nadc-query-results` for saving results, or :ref:`nadc-query-errors` if a
request fails.

Reference/API
-------------

.. automodapi:: astroquery.nadc.sages
    :no-inheritance-diagram:
    :inherited-members:
