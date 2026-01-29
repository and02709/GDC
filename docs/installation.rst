.. highlight:: shell

============
Installation
============


Stable release
--------------

To install Genome-wide Data Cleaner, run this command in your terminal:

.. code-block:: console

    $ pip install GDC

This is the preferred method to install Genome-wide Data Cleaner, as it will always install the most recent stable release.

If you don't have `pip`_ installed, this `Python installation guide`_ can guide
you through the process.

.. _pip: https://pip.pypa.io
.. _Python installation guide: http://docs.python-guide.org/en/latest/starting/installation/


From sources
------------

The sources for Genome-wide Data Cleaner can be downloaded from the `Github repo`_.

You can either clone the public repository:

.. code-block:: console

    $ git clone git://github.com/and02709/GDC

Or download the `tarball`_:

.. code-block:: console

    $ curl -OJL https://github.com/and02709/GDC/tarball/master

Once you have a copy of the source, you can install it with:

.. code-block:: console

    $ python setup.py install


External Dependencies
---------------------

GDC functions as a wrapper and pipeline for several genomic toolsets. For full functionality, ensure the following are installed and accessible in your system PATH:

1. **PLINK 1.9 & PLINK 2.0**: 
   Required for most data cleaning and conversion tasks. 
   Download from: `cog-genomics.org <https://www.cog-genomics.org/plink/>`_

2. **Python Dependencies**:
   The following will be installed automatically if using pip, but are required for manual setups:
    * ``pandas``
    * ``numpy``
    * ``scipy``

.. _Github repo: https://github.com/and02709/GDC
.. _tarball: https://github.com/and02709/GDC/tarball/master
