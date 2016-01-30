
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

This tutorial is made on a **64 bit** linux (ubuntu) machine. If you have 32 bit machine, the installer file links needs to be changed for HMMER3.1 and mothur.

**HMMER3.0 or lower does not work due to change in HMM format (.hmm).**

**Mothur 1.34.2 is used here**

Setup installation directory
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Clone the SSUsearch repository (**Skip if you have already done so**)::

    cd ~/Desktop
    git clone https://github.com/jiarong/SSUsearch

Go to the working directory::

    cd ~/Desktop/SSUsearch/

Make a directory called external_tools and go into it::

    mkdir -p ./external_tools
    cd ./external_tools

Install HMMER
~~~~~~~~~~~~~

.. code-block:: bash

    wget -c http://selab.janelia.org/software/hmmer3/3.1b1/hmmer-3.1b1-linux-intel-x86_64.tar.gz -O hmmer-3.1b1-linux-intel-x86_64.tar.gz

    tar -xzvf hmmer-3.1b1-linux-intel-x86_64.tar.gz

Copy the binary to a global PATH, you will need administrater privilege and password:

.. code:: bash

    sudo cp hmmer-3.1b1-linux-intel-x86_64/binaries/hmmsearch /usr/local/bin


Install mothur
~~~~~~~~~~~~~~

.. code:: bash

    wget http://www.mothur.org/w/images/8/88/Mothur.cen_64.zip -O mothur.zip

    unzip mothur.zip

Copy the binary to a global PATH, you will need administrater privilege and password:

.. code:: bash

    sudo cp mothur/mothur /usr/local/bin

Install RDP mcclust tool
~~~~~~~~~~~~~~~~~~~~~~~~

.. code:: bash

    wget https://s3.amazonaws.com/ssusearchdb/Clustering.tgz

    tar -xzvf Clustering.tgz

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

