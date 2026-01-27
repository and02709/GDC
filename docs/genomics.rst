Genomics
========

This section outlines the standard procedures for the genomic data processing pipeline.
Checking to See if it works

.. contents:: Table of Contents
   :depth: 2
   :local:

Standard Procedure
------------------

Module 1: Crossmap (optional)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
CrossMap converts genome coordinates and annotation files between different reference assemblies. It supports a wide array of used file formats, including BAM, CRAM, SAM, VCF, Wiggle, BigWig, BED, GFF, and GTF. For the purposes of our pipeline we will make use of PLINK Binary format and convert the genome build to GRCh38 (default: GRCh37 to GRCh38). 

* **Flag:** ``--use_crossmap`` (Default: ``1``).
* **Override:** Set to ``0`` if the build is already GRCh38.

Module 2: GenotypeHarmonizer (optional)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Aligns samples to the GRCh38 reference genome using ``ALL.hgdp1kg.filtered.SNV_INDEL.38.phased.shapeit5.vcf``.

Module 3: Initial QC
~~~~~~~~~~~~~~~~~~~~
Performs Quality Control prior to relatedness checks.

* Exclude SNPs and individuals with >10% missingness (**Plink**).
* Exclude SNPs and individuals with >2% missingness (**Plink**).

Module 4: Relatedness
~~~~~~~~~~~~~~~~~~~~~
Uses **KING** to separate related and unrelated study samples. This module also performs **PC-AiR** and **PC-Relate** for kinship estimation.

Module 5: Standard QC
~~~~~~~~~~~~~~~~~~~~~
Standard GWAS quality control measures on unrelated individuals.

* **MAF:** Exclude SNPs with Minor Allele Frequency < 0.01.
* **HWE:** Exclude SNPs with p-values < 1e-6 (controls) or < 1e-10 (cases).
* **Sex Check:** F-values < 0.2 assigned as female, > 0.8 as male.

Module 6: Phasing
~~~~~~~~~~~~~~~~~
Phasing performed via **shapeit4.2** with reference map ``chr${CHR}.b38.gmap.gz``.

Module 7: rfmix
~~~~~~~~~~~~~~~
Infers ancestry using phased files and ``hg38_phased.vcf.gz``. Global ancestry requires a posterior probability > 0.8.

Technical Implementation
------------------------


Module 1: Crossmap
~~~~~~~~~~~~~~~~~~
.. code-block:: console

   $ python CrossMap.py GRCh37_to_GRCh38.chain.gz prep.bed study_lifted.bed

Module 3: Initial QC
~~~~~~~~~~~~~~~~~~~~
.. code-block:: console

   $ plink --bfile file_stem --geno 0.02 --make-bed QC1
   $ plink --bfile QC1 --mind 0.02 --make-bed --out QC2

Module 5: Standard QC (Sex Check)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
.. code-block:: console

   $ plink --bfile QC4 --check-sex
   $ grep 'PROBLEM' plink.sexcheck | awk '{print $1, $2}' > sex_discrepancy.txt

Module 7: rfmix Execution
~~~~~~~~~~~~~~~~~~~~~~~~~
.. code-block:: console

   $ rfmix -f study.phased.vcf.gz -r hg38_phased.vcf.gz -m super_pop.txt -g genetic_map.txt -o ancestry
