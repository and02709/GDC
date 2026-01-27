Usage
=====

Downloading Data
----------------
Use the included script to fetch genomic data from the GDC portal. 
You will need your ``gdc-token.txt`` in the root directory.

.. code-block:: console

   $ python src/download_data.py --project TCGA-BRCA --limit 10

Running the Pipeline
--------------------
To process the downloaded VCF files, run the main analysis script:

.. code-block:: python

   import gdc_tools
   
   # Initialize the analysis
   analysis = gdc_tools.Analysis(project="TCGA-BRCA")
   analysis.run_pipeline()

SLURM Batch Submission
----------------------
For large-scale genomic analysis on MSI, use the provided SLURM template:

.. code-block:: console

   $ sbatch src/run_gdc_agate.sh
