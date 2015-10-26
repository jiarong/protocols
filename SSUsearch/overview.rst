
Overview
~~~~~~~~

SSUsearch is pipeline for identify SSU rRNA gene and use them for
diversity analysis. THe pipeline requires HMMER3.1, mothur, RDP mcclust,
and python numpy, pandas, scipy, matplotlib, and screed package.
Following are step by step tutorial for this pipeline:

1. `Install dependencies <./pipeline-dependency-installation.html>`_.
   If running amazon EC2 with ami "**ami-d87571b0**", skip this step.

2. `Database and dataset preparation <./data-preparation.html>`_

3. `SSU rRNA gene fragment search and classification <./ssu-search.html>`_. Apply steps here to all samples (1c.fa, 1d.fa, 2c.fa, 2d.fa in the test data set) one by one. Only variable called **"Seqfile"** needs to be changed to sequence file path of each sample.

4. `Unsupervised analysis <./unsupervised-analysis.html>`_

5. `Copy correction <./copy-correction.html>`_
