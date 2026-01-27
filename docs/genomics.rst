Genomics Pipeline
============================

.. contents:: Table of Contents
   :depth: 2

Standard Procedure
------------------

The following modules are executed in order to process genomic data from raw formats to ancestry-stratified datasets.

Module 1: Crossmap (Optional)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
This module converts the genome build to GRCh38. The default setting converts from GRCh37 to GRCh38 using the ``--use_crossmap`` flag. 

* **Default:** ``1`` (active)
* **Manual Override:** Set to ``0`` if the build is already GRCh38.

Module 2: GenotypeHarmonizer (Optional)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Aligns samples to the GRCh38 reference genome using ``ALL.hgdp1kg.filtered.SNV_INDEL.38.phased.shapeit5.vcf``.
* **Control Flag:** ``--use_genome_harmonizer`` (Default: ``1``).

Module 3: Initial QC
~~~~~~~~~~~~~~~~~~~~
Initial Quality Control prior to relatedness checks:
* Exclude SNPs and individuals with >10% missingness (**Plink**).
* Exclude SNPs and individuals with >2% missingness (**Plink**).

Module 4: Relatedness
~~~~~~~~~~~~~~~~~~~~~
Uses **KING** to separate the dataset into related and unrelated study samples. This is critical as subsequent ancestry estimation requires unrelated subjects. We perform **PC-AiR** and **PC-Relate** for accurate kinship coefficients and IBD probabilities.

Module 5: Standard QC
~~~~~~~~~~~~~~~~~~~~~
Standard GWAS quality control measures applied to unrelated individuals:
* **Missingness:** Filters for SNPs and individuals at 10% and 2% thresholds.
* **Sex Check:** Compares input data with imputed X chromosome coefficients (F-values < 0.2 assigned as female; > 0.8 as male).
* **Allele Frequency:** Exclude SNPs with MAF < 0.01.
* **Hardy-Weinberg:** Exclude SNPs with HWE p-values < 1e-6 (controls) or < 1e-10 (cases).
* **Relatedness:** Exclude parent-offspring relationships and pi_hat threshold > 0.2 (**Primus**).
* **Structure:** PCA performed via **FRAPOSA**.

Module 6: Phasing
~~~~~~~~~~~~~~~~~
Performed using **shapeit4.2** with reference map ``chr${CHR}.b38.gmap.gz``. Data is recoded into VCF format separated by chromosome.

Module 7: RFMIX
~~~~~~~~~~~~~~~
Infers local and global ancestry. 
* **Requirements:** Phased study files, ``hg38_phased.vcf.gz``, and a genetic map.
* **Classification:** Assignment to global ancestry requires a posterior probability > 0.8. Subjects falling below this are classified as "Other."

Module 8: Ancestry Plots
~~~~~~~~~~~~~~~~~~~~~~~~
* **GAP:** Visualizes ancestry proportions in individual samples.
* **LAP:** Shows most probable posterior ancestry by chromosomal regions.

Module 9: PCA
~~~~~~~~~~~~~
Constructs principal components for use in future analysis, specifically for ancestry correction.

Module 10 & 11: Stratification
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Separates samples into individual Plink files based on RFMIX results and provides subpopulation-specific QC.

Technical Implementation
------------------------

Module 1: Crossmap Execution
~~~~~~~~~~~~~~~~~~~~~~~~~~~~
.. code-block:: console

   $ plink --bfile prep1 --recode --output 'MT' --out prep2
   $ awk '{print $1, $4-1, $4, $2}' prep2.map > prep.bed
   $ python CrossMap.py GRCh37_to_GRCh38.chain.gz prep.bed study_lifted.bed

Module 3: Initial QC Commands
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
.. code-block:: console

   $ plink --bfile file_stem --geno 0.02 --make-bed QC1
   $ plink --bfile QC1 --mind 0.02 --make-bed --out QC2

Module 7: RFMIX Command
~~~~~~~~~~~~~~~~~~~~~~~
.. code-block:: console

   $ rfmix -f study.phased.vcf.gz -r hg38_phased.vcf.gz -m pop_map.txt -g genetic_map.txt -o ancestry


