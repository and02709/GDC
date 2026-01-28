Genomics
========

This section outlines the standard procedures for the genomic data processing pipeline.

.. contents:: Table of Contents
   :depth: 2
   :local:

Standard Procedure
------------------

Module 1: Crossmap (optional)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
...

Module 4: Relatedness
~~~~~~~~~~~~~~~~~~~~~
This module uses **KING** to separate related and unrelated study samples. The module also performs **PC-AiR** and **PC-Relate** for kinship estimation.

**KING** is a toolset that makes use of SNP data to identify how closely two individuals are related based on their DNA. This inference is based on the **kinship coefficient** (:math:`\phi`).

The fundamental equation KING uses to estimate the kinship coefficient :math:`\phi` between individuals :math:`i` and :math:`j` is based on the counts of **Heterozygote-Heterozygote (Het-Het)** and **Heterozygote-Homozygote (Het-Hom)** mismatches:

.. math::

   \phi_{ij} = \frac{N_{Aa,Aa} - 2N_{AA,aa}}{N_{Aa,i} + N_{Aa,j}}


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
