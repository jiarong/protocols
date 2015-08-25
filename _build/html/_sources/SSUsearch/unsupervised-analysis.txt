
Set another directory for unsupervised analysis::

    cd /usr/local/notebooks
    mkdir -p ./workdir/clust

Set parameters::

    Prefix='SS'    # name for the analysis run
    Script_dir='./SSUsearch/scripts'
    Wkdir='./workdir'
    Mcclust_jar='./external_tools/Clustering/dist/Clustering.jar'
    Java_xmx='10g'
    Java_gc_threads='2'
    Otu_dist_cutoff='0.05'
    Design='./data/test/SS.design'

    # get absolute path
    import os
    Script_dir=$(readlink -f ${Script_dir})
    Wkdir=$(readlink -f ${Wkdir})
    Mcclust_jar=$(readlink -f ${Mcclust_jar})
    Design=$(readlink -f ${Design})
    
Go the clust direcory::

    cd ./workdir/clust

Combine all the aligned sequences from each sample:

    cat $Wkdir/*.ssu.out/*.forclust > combined_seqs.afa

Make group file for mcclust and mothur. First part of the file basename will be the group label, e.g. file "aa.bb.cc" will have "aa" as group label::

    python $Script_dir/make-groupfile.py $Prefix.groups $Wkdir/*.ssu.out/*.forclust

Dereplication with mcclust::

    echo "*** Starting mcclust derep.."
    time java -Xmx$Java_xmx -XX:+UseParallelOldGC -XX:ParallelGCThreads=$Java_gc_threads -jar $Mcclust_jar derep -a -o derep.fasta temp.mcclust.names temp.txt combined_seqs.afa
    rm temp.txt

Precluster to collapse sequences with 1 base difference (most likely caused by sequencing errors)::

    #Convert mcclust names to mothur names
    python $Script_dir/mcclust2mothur_names_file.py temp.mcclust.names temp.mothur.names

    echo "***starting preclust.."
    # output: derep.precluster.fasta, derep.precluster.names
    mothur "#pre.cluster(fasta=derep.fasta, diffs=1, name=temp.mothur.names)"

    #Convert names back to mcclust
    python $Script_dir/mothur2mcclust_names_file.py derep.precluster.names $Prefix.names

Make distance matrix::

    time java -Xmx$Java_xmx -XX:+UseParallelOldGC -XX:ParallelGCThreads=$Java_gc_threads -jar $Mcclust_jar dmatrix -l 25 -o matrix.bin -i $Prefix.names -I derep.precluster.fasta

Clustering with mcclust::

    time java -Xmx$Java_xmx -XX:+UseParallelOldGC -XX:ParallelGCThreads=$Java_gc_threads -jar $Mcclust_jar cluster -m upgma -i $Prefix.names -s $Prefix.groups -o complete.clust -d matrix.bin
    
Convert clust results to mothur list::

    python $Script_dir/mcclust2mothur-list-cutoff.py complete.clust $Prefix.list $Otu_dist_cutoff

Remove ":" in sequence names (mothur automatically converts ":" to "_")::

    sed -i 's/:/_/g' $Prefix.names $Prefix.groups $Prefix.list
    echo "*** Replace ':' with '_' in seq names (original illumina name has ':' in them)"

Get representitives of each OTU at OTU distance cutoff defined at top of this page::

    java -jar $Mcclust_jar rep-seqs -c -l -s complete.clust $Otu_dist_cutoff combined_seqs.afa
    mv complete.clust_rep_seqs.fasta otu_rep_align.fa

Get OTU table::
    mothur "#make.shared(list=$Prefix.list, group=$Prefix.groups, label=$Otu_dist_cutoff);"

Get consensus taxonomy for each OTU::

    cat $Wkdir/*.ssu.out/*.silva.taxonomy > $Prefix.taxonomy
    mothur "#classify.otu(list=$Prefix.list, taxonomy=$Prefix.taxonomy, label=$Otu_dist_cutoff)"

Make biom file::

    mothur "#make.biom(shared=$Prefix.shared, constaxonomy=$Prefix.$Otu_dist_cutoff.cons.taxonomy)"
    mv $Prefix.$Otu_dist_cutoff.biom $Prefix.biom

    # clean up tempfiles
    !rm -f mothur.*.logfile *rabund complete* derep.fasta matrix.bin nonoverlapping.bin temp.*

**With SS.groups, SS.names and SS.list**, most diversity analysis can be done by mothur. You can look at `mothur wiki <http://www.mothur.org/wiki/454_SOP>`_ for details (Do not forgot to do even sampling before beta-diversity analysis).

**SS.biom file can used in most tools. (qiime, rdp, and phyloseq)**

Since the purpose of this tutorial is to show our new pipeline, we will skip details of community analysis with mothur. Following are some common commands in mothur::
    
    mothur "#make.shared(biom=$Prefix.biom); sub.sample(shared=$Prefix.shared); summary.single(calc=nseqs-coverage-sobs-chao-shannon-invsimpson); dist.shared(calc=braycurtis); pcoa(phylip=$Prefix.userLabel.subsample.braycurtis.userLabel.lt.dist); nmds(phylip=$Prefix.userLabel.subsample.braycurtis.userLabel.lt.dist); amova(phylip=$Prefix.userLabel.subsample.braycurtis.userLabel.lt.dist, design=$Design); tree.shared(calc=braycurtis); unifrac.weighted(tree=$Prefix.userLabel.subsample.braycurtis.userLabel.tre, group=$Design, random=T)"
    !rm -f mothur.*.logfile; 
    !rm -f *.rabund

Some simple visualization::

    # alpha diveristy index
    python $Script_dir/plot-diversity-index.py "userLabel" "chao,shannon,invsimpson" "c,d" "SS.userLabel.subsample.groups.summary" "test" "test.alpha" 

    # taxon distribution
    python $Script_dir/plot-taxa-count.py 2 test.taxa.dist ../*.ssu.out/*.silva.taxonomy.count

    # ordination
    python $Script_dir/plot-pcoa.py  SS.userLabel.subsample.braycurtis.userLabel.lt.pcoa.axes  SS.userLabel.subsample.braycurtis.userLabel.lt.pcoa.loadings  test.beta.pcoa
