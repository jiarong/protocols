
Install dependencies
====================

**If you are running this tutorial using Amazon EC2 instance loaded with
the recommended AMI, you can skip this part.**

This pipeline requires:

-  HMMER3.1
-  mothur
-  RDP mcclust
-  python pandas, numpy, scipy, matplotlib, and screed package.

Following steps should work for linux machines. If you are running this
tutorial using Amazon EC2 instance loaded with the recommended AMI, you
can skip this part.

Setup installation directory
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Go to the working directory::

    cd /usr/local/notebooks

Make a directory called external_tools and go into it::

    mkdir -p ./external_tools
    cd ./external_tools

Make a directory call bin for binaries::

    mkdir -p ./bin

Install HMMER
~~~~~~~~~~~~~

.. code-block:: bash

    wget -c http://selab.janelia.org/software/hmmer3/3.1b1/hmmer-3.1b1-linux-intel-x86_64.tar.gz -O hmmer-3.1b1-linux-intel-x86_64.tar.gz

    tar -xzvf hmmer-3.1b1-linux-intel-x86_64.tar.gz

    cp hmmer-3.1b1-linux-intel-x86_64/binaries/hmmsearch /usr/local/bin

    cp hmmer-3.1b1-linux-intel-x86_64/binaries/hmmsearch ./bin

Install mothur
~~~~~~~~~~~~~~

.. code:: bash

    wget http://www.mothur.org/w/images/8/88/Mothur.cen_64.zip -O mothur.zip

    unzip mothur.zip

    cp mothur/mothur /usr/local/bin

    cp mothur/mothur ./bin

Install RDP mcclust tool
~~~~~~~~~~~~~~~~~~~~~~~~

.. code:: bash

    wget http://lyorn.idyll.org/~gjr/public2/misc/Clustering.tar.gz

    tar -xzvf Clustering.tar.gz

Install python packages
~~~~~~~~~~~~~~~~~~~~~~~

Skip if you already have the packages installed

.. code:: bash

    source ~/.bashrc

    pip install -U pip

    pip install screed

    pip install brewer2mpl

    pip install biom-format


Install numpy, matplotlib, scipy, and pandas

.. code:: bash

    pip install numpy matplotlib scipy pandas

Alternatively, you can install **anaconda** that have most popular
python packages installed: https://store.continuum.io/cshop/anaconda/

The anaconda installation guide is `here <http://docs.continuum.io/anaconda/install#linux-install>`__.

check dependencies installed
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code:: bash

    make -f SSUsearch/Makefile tool_check Hmmsearch=hmmsearch Mothur=mothur Flash=flash Mcclust_jar=./Clustering/dist/Clustering.jar

