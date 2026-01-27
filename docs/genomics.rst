Genomics
============

Requirements
------------
* Python 3.10+
* Access to the UMN MSI Agate or Mangrove clusters (optional for local use)
* A GDC Authentication Token (for controlled data)

Local Setup
-----------
To install the GDC pipeline tools locally, clone the repository and install the 
requirements:

.. code-block:: console

   $ git clone https://github.com/and02709/GDC.git
   $ cd GDC
   $ pip install -r requirements.txt

MSI Cluster Setup
-----------------
On the MSI clusters, we recommend using a Conda environment:

.. code-block:: console

   $ module load conda
   $ conda create -n gdc_env python=3.11
   $ conda activate gdc_env
