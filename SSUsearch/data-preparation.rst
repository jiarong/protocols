

Data preparation
~~~~~~~~~~~~~~~~~~~~

First go to a working dirtory::

    cd ~/Desktop/SSUsearch

Make a directory for data::

    mkdir -p ./data

Change directory into ./data::

    cd ./data


Download database files
~~~~~~~~~~~~~~~~~~~~~~~

Download the database files we need::

    curl -O http://athyra.oxli.org/~gjr/public2/misc/SSUsearch_db.tgz

Unzip the file::

    tar -xzvf SSUsearch_db.tgz

Output should look like the following:

.. parsed-literal::

    x SSUsearch_db/
    x SSUsearch_db/Gene_db.silva_108_rep_set.fasta
    x SSUsearch_db/Gene_tax.silva_taxa_family.tax
    x SSUsearch_db/Gene_model_org.16s_ecoli_J01695.fasta
    x SSUsearch_db/Gene_db_cc.greengene_97_otus.fasta
    x SSUsearch_db/Gene_tax_cc.greengene_97_otus.tax
    x SSUsearch_db/Copy_db.copyrighter.txt
    x SSUsearch_db/Ali_template.silva_ssu.fasta
    x SSUsearch_db/readme
    x SSUsearch_db/Ali_template.silva_lsu.fasta
    x SSUsearch_db/Ali_template.test.fasta
    x SSUsearch_db/Ali_template.test_lsu.fasta
    x SSUsearch_db/Gene_db.lsu_silva_rep.fasta
    x SSUsearch_db/Gene_db.ssu_rdp_rep.fasta
    x SSUsearch_db/Gene_tax.lsu_silva_rep.tax
    x SSUsearch_db/Gene_tax.ssu_rdp_rep.tax
    x SSUsearch_db/Hmm.lsu.hmm
    x SSUsearch_db/clean.sh
    x SSUsearch_db/Hmm.ssu.hmm


download a small test dataset
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**ATT: for real (larger) dataset, make sure there is enough disk space.**

Download a directory of files::

    wget http://athyra.oxli.org/~gjr/public2/misc/SSUsearch/test.tgz
    tar -xzvf test.tgz
    ls test/data/


**This tutorial assumes that you ready finished quality trimming, and also paired end merge, if you paired end reads overlap.**

For quality trimming, we recommend
`trimmomatic <http://www.usadellab.org/cms/?page=trimmomatic>`_ written
in java, or
`fastq-mcf <https://code.google.com/p/ea-utils/wiki/FastqMcf>`_ written
in C.

For paired end reads merging, we recommend
`pandseq <https://github.com/neufeld/pandaseq>`_ or
`flash <http://ccb.jhu.edu/software/FLASH/>`_

