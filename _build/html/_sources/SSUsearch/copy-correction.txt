
Copy corrections is based on `copyrighter <http://www.ncbi.nlm.nih.gov/pubmed/24708850>`_. One copy database for each Greengene taxon at each level is provided by the tool. We will use that database for correcting our Greengene taxonomy abundance and OTU abundance.
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Set up a directory::

    cd /usr/local/notebooks
    ### set up directory
    !mkdir -p ./workdir/copy_correction

Set up parameters::

    Prefix='SS'    # name for the analysis run
    Script_dir='./SSUsearch/scripts'
    Wkdir='./workdir'
    Design='./data/test/SS.design'
    Otu_dist_cutoff='0.05'
    Copy_db='./data/SSUsearch_db/Copy_db.copyrighter.txt'

    Script_dir=$(readlink -f $Script_dir)
    Wkdir=$(readlink -f $Wkdir)
    Design=$(readlink -f $Design)
    Copy_db=$(readlink -f $Copy_db)
    
Go to the directory::

    cd ./workdir/copy_correction

Get input files from clustering directory::

    ln -sf $Wkdir/clust/$Prefix.biom
    ln -sf $Wkdir/clust/$Prefix.list

Get Greengene taxonomy::

    cat $Wkdir/*.ssu.out/*.gg.taxonomy > $Prefix.taxonomy
    mothur "#classify.otu(list=$Prefix.list, taxonomy=$Prefix.taxonomy, label=$Otu_dist_cutoff)"
    mv SS.0.03.cons.taxonomy SS.cons.taxonomy
    mv SS.0.03.cons.tax.summary SS.cons.tax.summary

Get original OTU table::

    mothur "#make.shared(biom=$Prefix.biom)"
    
Do copy correction and even sampling::

    python $Script_dir/copyrighter-otutable.py $Copy_db $Prefix.cons.taxonomy $Prefix.shared $Prefix.cc.shared
    mv $Prefix.cc.shared $Prefix.shared
    mothur "#make.biom(shared=$Prefix.shared, constaxonomy=$Prefix.cons.taxonomy);"
    mv $Prefix.userLabel.biom $Prefix.biom
    rm -f mothur.*.logfile

**SS.biom** can be further used for diversity analysis, important but not focus of this tutorial (details see `mothur wiki <http://www.mothur.org/wiki/454_SOP>`_)::

    mothur "#make.shared(biom=$Prefix.biom); sub.sample(shared=$Prefix.shared); summary.single(calc=nseqs-coverage-sobs-chao-shannon-invsimpson); dist.shared(calc=braycurtis); pcoa(phylip=$Prefix.userLabel.subsample.braycurtis.userLabel.lt.dist); nmds(phylip=$Prefix.userLabel.subsample.braycurtis.userLabel.lt.dist); amova(phylip=$Prefix.userLabel.subsample.braycurtis.userLabel.lt.dist, design=$Design); tree.shared(calc=braycurtis); unifrac.weighted(tree=$Prefix.userLabel.subsample.braycurtis.userLabel.tre, group=$Design, random=T)"
    rm -f mothur.*.logfile; 
    rm -f *.rabund
