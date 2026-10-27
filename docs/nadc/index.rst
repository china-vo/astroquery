NADC
====

The National Astronomical Data Center (NADC) modules provide access to source
catalogs, images, and spectra through ``astroquery.nadc``. Start with the module
for your data; each module page includes a complete first-query example.

.. list-table::
   :header-rows: 1
   :widths: 25 50 25

   * - Module
     - Data
     - Examples and API
   * - ``nadc.lamost``
     - LAMOST spectral catalogs and LRS/MRS spectra
     - :doc:`LAMOST <lamost>`
   * - ``nadc.cstar``
     - CSTAR source photometry
     - :doc:`CSTAR <cstar>`
   * - ``nadc.fashi``
     - FASHI HI sources and precomputed catalog counterparts
     - :doc:`FASHI <fashi>`
   * - ``nadc.legacyplate``
     - Photographic plate records and digitized images
     - :doc:`Legacy Plate <legacyplate>`
   * - ``nadc.sages``
     - SAGES photometry and stellar parameters
     - :doc:`SAGES <sages>`
   * - ``nadc.scuss``
     - SCUSS photometry, proper motions, and image metadata
     - :doc:`SCUSS <scuss>`

For additional configuration, custom filters, and query limits in CSTAR, FASHI,
Legacy Plate, SAGES, and SCUSS, consult :doc:`query_guide` as needed. LAMOST has
its own configuration and pagination rules, described on its module page.

.. toctree::
   :maxdepth: 1
   :hidden:

   LAMOST <lamost>
   CSTAR <cstar>
   FASHI <fashi>
   Legacy Plate <legacyplate>
   SAGES <sages>
   SCUSS <scuss>
   Configuration and advanced queries <query_guide>
