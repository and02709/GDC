=====
Usage
=====

Overview of Workflow
--------------------

GDC is designed to streamline the cleaning of genomic datasets. A typical workflow involves:

1.  **Data Ingestion**: Loading PLINK binary files (.bed, .bim, .fam).
2.  **Quality Control (QC)**: Filtering based on call rates, MAF, and HWE.
3.  **Population Stratification**: Handling ancestry and relatedness.
4.  **Export**: Generating cleaned datasets for downstream analysis.



Basic Command Line Usage
------------------------

If you are using GDC as a standalone tool, you can initiate a standard cleaning pipeline directly from the terminal. 

.. code-block:: console

    $ GDC --input my_data --output cleaned_data --maf 0.05 --geno 0.02

Python API Example
------------------

For more complex research workflows, such as integration into a larger Python script or Jupyter Notebook, use the API:

.. code-block:: python

    from GDC import Cleaner

    # Initialize the cleaner with your PLINK prefix
    cleaner = Cleaner(prefix="study_data")

    # Run a standard QC battery
    cleaner.filter_missingness(threshold=0.02)
    cleaner.filter_maf(min_freq=0.01)
    
    # Run a sex check (requires PLINK in PATH)
    cleaner.check_sex()

    # Execute and save
    cleaner.run_pipeline(out_prefix="study_data_cleaned")

Advanced Options
----------------

Handling Relatedness
~~~~~~~~~~~~~~~~~~~~

GDC can interface with tools like KING or PLINK's identity-by-descent (IBD) functions to handle related individuals:

.. code-block:: python

    cleaner.remove_related(cutoff=0.125)

Working on HPC (SLURM)
~~~~~~~~~~~~~~~~~~~~~~

When using GDC on a cluster, ensure your script requests sufficient memory for PLINK operations. A typical SLURM header for a GDC task might look like this:

.. code-block:: bash

    #!/bin/bash
    #SBATCH --job-name=GDC_Clean
    #SBATCH --mem=16gb
    #SBATCH --cpus-per-task=4
    #SBATCH --time=02:00:00

    python my_gdc_script.py
