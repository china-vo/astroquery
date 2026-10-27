NADC FASHI Queries
==================

``astroquery.nadc.fashi`` searches the FAST All Sky HI survey catalog and its
precomputed ALFALFA, SDSS, and SGA counterparts.

Finding HI sources
------------------

Use ``query_hi_sources`` to combine a cone search with velocity and S/N
selections. Replace ``"your-nadc-token"`` with your Query Data access token
before running this example:

.. code-block:: python

   import astropy.units as u
   from astropy.coordinates import SkyCoord
   from astroquery.nadc.fashi import FashiClass

   client = FashiClass(token="your-nadc-token")
   center = SkyCoord(125*u.deg, -6*u.deg)
   sources = client.query_hi_sources(
       center, 5*u.arcmin,
       cz_range=(1000, 8000), snr_min=8,
       columns=["id_fashi", "ra", "dec", "cz", "snr"],
       max_rows=100,
   )
   print(sources[:5])

The result is an `~astropy.table.Table`; an empty table means no sources matched.
``max_rows=100`` limits the result to 100 rows; see :ref:`nadc-query-limits`
for larger queries. The remaining examples reuse ``client`` and ``center``
from this block. For environment-variable tokens or timeouts, see
:ref:`nadc-query-authentication`.

Additional filters include
``z_range``, ``distance_range``, and ``log10mass_range``. Numeric filters use
the catalog's column units; inspect ``list_columns`` for field descriptions.

Finding catalog counterparts
----------------------------

Use the counterpart methods to retrieve existing associations in the archive:

.. list-table::
   :header-rows: 1
   :widths: 35 40 25

   * - Data
     - Method
     - Example filter
   * - ALFALFA matches
     - ``query_alfalfa_matches``
     - ``agcnr_alfalfa``
   * - SDSS photometric counterparts
     - ``query_sdss_phot_counterparts``
     - ``probability_min``
   * - SDSS spectroscopic counterparts
     - ``query_sdss_spec_counterparts``
     - ``z_sdss_range``
   * - SGA counterparts
     - ``query_sga_counterparts``
     - ``name_sga``

For example, select SDSS photometric counterparts around the same position:

.. code-block:: python

   counterparts = client.query_sdss_phot_counterparts(
       center, 5*u.arcmin, probability_min=0.8,
       columns=["id_fashi", "objid_sdss", "probability_sdss"],
       max_rows=100,
   )
   print(counterparts[:5])

These methods search precomputed matches; they do not perform a new cross-match
against the external surveys.

Choosing the spatial coordinates
--------------------------------

Counterpart tables contain coordinates for both FASHI and the external survey.
Spatial searches use the FASHI position by default. Pass ``pos_group`` to search
around the external-catalog position instead:

.. code-block:: python

   matches = client.query_alfalfa_matches(
       center, 5*u.arcmin,
       pos_group="ra_alfalfa,dec_alfalfa", max_rows=100,
   )
   print(matches[:5])

Use ``list_coordinate_groups`` to inspect the choices for a table:

.. code-block:: python

   print(client.list_coordinate_groups(
       catalog="alfalfa_crossmatch", table="alfalfa_crossmatch",
   ))

Custom queries
--------------

The main catalog and counterpart catalogs use the following table names:

.. list-table::
   :header-rows: 1
   :widths: 50 50

   * - Catalog
     - Table
   * - ``FASHI``
     - ``extragalactic_hi_source_catalog``
   * - ``alfalfa_crossmatch``
     - ``alfalfa_crossmatch``
   * - ``optical_counterparts_sdss_phot``
     - ``optical_counterparts_sdss_phot``
   * - ``optical_counterparts_sdss_spec``
     - ``optical_counterparts_sdss_spec``
   * - ``optical_counterparts_sga``
     - ``optical_counterparts_sga``

Use ``list_columns`` to inspect fields before choosing custom columns and
filters. ``query_table`` accepts explicit table selection and sorting:

.. code-block:: python

   from astroquery.nadc import ColumnConstraint

   print(client.list_columns(
       catalog="FASHI", table="extragalactic_hi_source_catalog",
   ))
   selected = client.query_table(
       catalog="FASHI", table="extragalactic_hi_source_catalog",
       coordinates=center, radius=5*u.arcmin, pos_group="ra,dec",
       showcol=["id_fashi", "ra", "dec", "snr"],
       column_constraints=[ColumnConstraint.greaterequal("snr", 10)],
       sort="snr", order="desc", max_rows=100,
   )
   print(selected[:5])

See :ref:`nadc-query-filters` for additional filter types,
:ref:`nadc-query-results` for saving results, or :ref:`nadc-query-errors` if a
request fails.

Reference/API
-------------

.. automodapi:: astroquery.nadc.fashi
    :no-inheritance-diagram:
    :inherited-members:
