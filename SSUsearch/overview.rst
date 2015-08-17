
Overview
~~~~~~~~

SSUsearch is pipeline for identify SSU rRNA gene and use them for
diversity analysis. THe pipeline requires HMMER3.1, mothur, RDP mcclust,
and python numpy, pandas, scipy, matplotlib, and screed package.
Following are step by step tutorial for this pipeline:

1. `Install dependencies <./pipeline-dependency-installation.rst>`_.
   If running amazon EC2 with ami "**ami-d87571b0**\ ", skip this step.

2. `Database and dataset preparation <./data-preparation.rst>`_

3. `SSU rRNA gene fragment search and
   classification <./ssu-search.rst>`_. ssu-search-Copy1.ipynb,
   ssu-search-Copy2.ipynb, ssu-search-Copy3.ipynb, and
   ssu-search-Copy4.ipynb are the ones to run for test data

4. `Unsupervised analysis <./unsupervised-analysis.rst>`_

5. `Copy correction <./copy-correction.rst>`_

.. code:: python

    # update the notebooks
    !rm -rf SSUsearch
    !git clone https://github.com/jiarong/SSUsearch.git
    !cp SSUsearch/notebooks/*.ipynb .
