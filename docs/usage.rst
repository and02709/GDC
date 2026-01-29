=====
Usage
=====

Genome-wide Data Cleaner (GDC) is designed to automate the quality control (QC) and cleaning of genomic data. It primarily acts as a streamlined interface for PLINK 1.9 and PLINK 2.0.

To use GDC in a Python project:

.. code-block:: python

    import GDC

Data Requirements
-----------------

Before running GDC, ensure your genomic data is in PLINK binary format. GDC expects the following files to be present in your input directory:

* ``.bed`` (Binary pedigree file)
* ``.bim`` (Extended MAP file)
* ``.fam`` (Family file)

Command Line Interface (CLI)
----------------------------

GDC is most commonly used via the command line. This allows for easy integration into SLURM scripts or bash pipelines.

Basic Syntax
~~~~~~~~~~~~

.. code-block:: console

    $ python -m GDC --bfile [input_prefix] --out [output_prefix] [options]

Arguments and Flags
~~~~~~~~~~~~~~~~~~~

The following parameters allow you to customize the cleaning threshold:

* **Input/Output**
    * ``--bfile``: Prefix of the input PLINK binary files.
    * ``--out``: Prefix for the generated cleaned files.

* **Quality Control Thresholds**
    * ``--mind``: Filter individuals with missing phenotypes (default: 0.1).
    * ``--geno``: Filter variants with missing call rates (default: 0.1).
    * ``--maf``: Filter variants with Minor Allele Frequency below a threshold (default: 0.01).
    * ``--hwe``: Filter variants failing Hardy-Weinberg Equilibrium test (default: 1e-6).

* **Advanced Processing**
    * ``--indep-pairwise``: Perform linkage disequilibrium (LD) pruning (e.g., ``50 5 0.2``).
    * ``--rem-multiallelic``: Remove multiallelic variants to ensure dataset consistency.

Usage Examples
--------------

Standard QC Pipeline
~~~~~~~~~~~~~~~~~~~~

To perform a standard clean with a 5% missingness threshold and 1% MAF:

.. code-block:: console

    $ python -m GDC --bfile raw_data --out clean_data --mind 0.05 --geno 0.05 --maf 0.01

LD Pruning and Multiallelic Removal
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

For datasets intended for PCA or relatedness testing, you may want to prune for LD and clean the variant types:

.. code-block:: console

    $ python -m GDC --bfile raw_data --out pruned_data --indep-pairwise 50 5 0.2 --rem-multiallelic

Integration with SLURM
----------------------

For large-scale university research projects, GDC can be included in a batch script. 

.. code-block:: bash

    #!/bin/bash
    #SBATCH --job-name=GDC_QC
    #SBATCH --output=GDC_QC_%j.log
    #SBATCH --mem=8gb
    #SBATCH --cpus-per-task=1

    # Load necessary modules (if applicable)
    # module load plink

    python -m GDC --bfile my_dataset --out my_dataset_cleaned --maf 0.05 --rem-multiallelic

Python API Usage
----------------

You can also call the GDC cleaning logic directly within Python scripts for more complex bioinformatics workflows:

.. code-block:: python

    from GDC import gdc_clean

    gdc_clean(
        bfile="my_study_data",
        out="cleaned_output",
        maf=0.01,
        geno=0.05,
        rem_multiallelic=True
    )
