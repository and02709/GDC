Genomics Pipeline
===================

This document outlines the standard and optional procedures for the genomics pipeline, specifically designed for GRCh38 builds on high-performance computing clusters.

.. contents:: Table of Contents
   :depth: 2
   :local:

Standard Procedure
------------------

Modules should be executed in the following order.

Module 1: Crossmap (Optional)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Converts the genome build to GRCh38 (default: GRCh37 to GRCh38).

* **Flag:** ``--use_crossmap``
* **Default:** ``1`` (Active)
* **Manual Override:** Set to ``0`` if the build is already GRCh38.

Module 2: GenotypeHarmonizer (Optional)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Aligns samples to the GRCh38 reference genome using ``ALL.hgdp1kg.filtered.SNV_INDEL.38.phased.shapeit5.vcf``.

* **Flag:** ``--use_genome_harmonizer`` (Default: ``1``).

Module 3: Initial QC
~~~~~~~~~~~~~~~~~~~~
Initial Quality Control performed prior to relatedness checks.

* Exclude SNPs with >10% missingness (**Plink**)
* Exclude individuals with >10% missingness (**Plink**)
* Exclude SNPs with >2% missingness (**Plink**)
* Exclude individuals with >2% missingness (**Plink**)

Module 4: Relatedness
~~~~~~~~~~~~~~~~~~~~~
Uses **KING** to separate related and unrelated study samples. Ancestry estimation requires unrelated subjects. This module performs **PC-AiR** and **PC-Relate** to calculate kinship coefficients and IBD probabilities.

Module 5: Standard QC
~~~~~~~~~~~~~~~~~~~~~
Standard GWAS quality control for unrelated individuals.

* **Missingness:** 10% and 2% filters for SNPs/individuals.
* **Sex Check:** F-values < 0.2 assigned as female, > 0.8 as male.
* **Minor Allele Frequency:** Exclude SNPs with MAF < 0.01.
* **HWE:** Exclude SNPs with p-values < 1e-6 (controls) or < 1e-10 (cases).
* **Relatedness:** Exclude parent-offspring and pi_hat > 0.2 (**Primus**).
* **PCA:** Principal Component Analysis via **FRAPOSA**.

Module 6: Phasing
~~~~~~~~~~~~~~~~~
Phasing performed via **shapeit4.2** using reference map ``chr${CHR}.b38.gmap.gz``.

Module 7: rfmix
~~~~~~~~~~~~~~~
Infers sample ancestry using phased files, ``hg38_phased.vcf.gz``, and a genetic map. Assignment to global ancestry requires a posterior probability > 0.8.

Module 8: Ancestry Plots
~~~~~~~~~~~~~~~~~~~~~~~~
Visualization provided via:

* **GAP:** Ancestry proportions in individual samples.
* **LAP:** Posterior ancestry by chromosomal regions.

---

Technical Implementation
------------------------

Module 1: Crossmap
~~~~~~~~~~~~~~~~~~
.. code-block:: console

   $ python CrossMap.py GRCh37_to_GRCh38.chain.gz prep.bed study_lifted.bed

Module 2: GenotypeHarmonizer
~~~~~~~~~~~~~~~~~~~~~~~~~~~~
.. code-block:: console

   $ java -Xmx8g -jar GenotypeHarmonizer.jar --input lifted.chr${CHR} --inputType PLINK_BED --ref ALL.hgdp1kg.filtered.SNV_INDEL.38.phased.shapeit5 --refType VCF --keep --output lifted.chr${CHR}.aligned

Module 3: Initial QC
~~~~~~~~~~~~~~~~~~~~
.. code-block:: console

   $ plink --bfile file_stem --geno 0.02 --make-bed QC1
   $ plink --bfile QC1 --mind 0.02 --make-bed --out QC2

Module 4: Relatedness
~~~~~~~~~~~~~~~~~~~~~
.. code-block:: bash

   # Kinship test
   ./king -b QC_Initial.bed --kinship --prefix kinships
   
   # PCAIR and PCRelate
   Rscript src/pca_ir_pipeline.R $WORK_DIRECTORY $FILE_NAME

Module 6: Phasing
~~~~~~~~~~~~~~~~~
.. code-block:: console

   $ bcftools index -f study.chr${CHR}.vcf.gz
   $ shapeit4.2 --input study.chr${CHR}.vcf.gz --map chr${CHR}.b38.gmap.gz --region ${CHR} --output study.chr${CHR}.phased.vcf.gz --thread

Module 7: rfmix
~~~~~~~~~~~~~~~
.. code-block:: console

   $ rfmix -f study.chr${CHR}.phased.vcf.gz -r hg38_phased.vcf.gz -m super_population_map_file.txt -g genetic_map_hg38.txt -o ancestry_chr${CHR} --chromosome=$CHR
