Search SSU rRNA gene
~~~~~~~~~~~~~~~~~~~~~

Set up working directory::

    cd ~/Desktop/SSUsearch
    mkdir -p ./workdir

Check seqfile files to process in data directory (make sure you still remember the data directory)::

    ls ./data/test/data

README
~~~~~~

**This part of pipeline search for the SSU rRNA gene fragments, classify them, and extract reads aligned specific region. It is also heavy lifting part of the whole pipeline (more cpu will help).**

**This part works with one seqfile a time. You just need to change the "Seqfile" and maybe other parameters in the two cells bellow.**

If your computer has **many processors**, there are two ways to make use of the resource:

1. Set "Cpu" to a higher number.

2. Run steps in this page on multiple terminal windows at the same time.
   One for each sample.

(Again we assume the "Seqfile" is quality trimmed.)

**Again, we will process one file at a time; set the "Seqfile" variable to the seqfile name to be be processed.**

**First part of seqfile basename (separated by ".") will be the label of this sample, so named it properly (unique).**

e.g. for "~/Desktop/SSUsearch/data/test/data/1c.fa", "1c" will the
label of this sample.

**Set the Seqfile variable to the sequence file to process**::

    Seqfile='./data/test/data/1c.fa'
    #Seqfile='./data/test/data/1d.fa'
    #Seqfile='./data/test/data/2c.fa'
    #Seqfile='./data/test/data/2d.fa'

Other parameters to set::

    Cpu='2'   # number of maxixum threads for search and alignment
    Hmm='./data/SSUsearch_db/Hmm.ssu.hmm'   # hmm model for ssu
    Gene='ssu'
    Script_dir='./scripts'
    Gene_model_org='./data/SSUsearch_db/Gene_model_org.16s_ecoli_J01695.fasta'
    Ali_template='./data/SSUsearch_db/Ali_template.silva_ssu.fasta'
    
    # pick start and end of a region in V4 for de novo clustering
    # the default numbers are for 150bp reads
    # rule of thumb is pick a region with more reads with larger overlap
    # change to Start=577, End=657, Len_cutoff=75 for 100bp reads
    Start='577'
    End='727'
    Len_cutoff='100' # min length for reads picked for the region
    
    Gene_tax='./data/SSUsearch_db/Gene_tax.silva_taxa_family.tax' # silva 108 ref
    Gene_db='./data/SSUsearch_db/Gene_db.silva_108_rep_set.fasta'
    
    Gene_tax_cc='./data/SSUsearch_db/Gene_tax_cc.greengene_97_otus.tax' # greengene 2012.10 ref for copy correction
    Gene_db_cc='./data/SSUsearch_db/Gene_db_cc.greengene_97_otus.fasta'

    ###
    ### do not need to change lines below
    ###

    ### first part of file basename will the label of this sample
    Filename=$(basename(${Seqfile}))
    Tag=${Filename%%.*}

    ### make absolute path
    export Hmm=$(readlink -f ${Hmm})
    export Seqfile=$(readlink -f ${Seqfile})
    export Script_dir=$(readlink -f ${Script_dir})
    export Gene_model_org=$(readlink -f ${Gene_model_org})
    export Ali_template=$(readlink -f ${Ali_template})
    export Gene_tax=$(readlink -f ${Gene_tax})
    export Gene_db=$(readlink -f ${Gene_db})
    export Gene_tax_cc=$(readlink -f ${Gene_tax_cc})
    export Gene_db_cc=$(readlink -f ${Gene_db_cc})
    
Double check key variables::

    echo "*** make sure: parameters are right"
    echo "Seqfile: $Seqfile\nCpu: $Cpu\nFilename: $Filename\nTag: $Tag"


Do the hmmsearch (**heavy weight lifting part**)::

    cd workdir
    mkdir -p $Tag.ssu.out

    ### start hmmsearch
    echo "*** hmmsearch starting"
    time hmmsearch --incE 10 --incdomE 10 --cpu $Cpu \
      --domtblout $Tag.ssu.out/$Tag.qc.$Gene.hmmdomtblout \
      -o /dev/null -A $Tag.ssu.out/$Tag.qc.$Gene.sto \
      $Hmm $Seqfile
    echo "*** hmmsearch finished"


Parsing the output::

    python $Script_dir/get-seq-from-hmmout.py \
        $Tag.ssu.out/$Tag.qc.$Gene.hmmdomtblout \
        $Tag.ssu.out/$Tag.qc.$Gene.sto \
        $Tag.ssu.out/$Tag.qc.$Gene

Pass hits to mothur aligner::

    echo "*** Starting mothur align"
    cat  $Gene_model_org $Tag.ssu.out/$Tag.qc.$Gene > $Tag.ssu.out/$Tag.qc.$Gene.RFadded
    
    # mothur does not allow tab between its flags, thus no indents here
    time mothur "#align.seqs(candidate=$Tag.ssu.out/$Tag.qc.$Gene.RFadded, template=$Ali_template, threshold=0.5, flip=t, processors=$Cpu)"
    
    rm -f mothur.*.logfile


Get aligned seqs that have > 50% matched to references::

    python $Script_dir/mothur-align-report-parser-cutoff.py \
        $Tag.ssu.out/$Tag.qc.$Gene.align.report \
        $Tag.ssu.out/$Tag.qc.$Gene.align \
        $Tag.ssu.out/$Tag.qc.$Gene.align.filter \
        0.5
        

Get the unaligned fasta file::

    python $Script_dir/remove-gap.py $Tag.ssu.out/$Tag.qc.$Gene.align.filter $Tag.ssu.out/$Tag.qc.$Gene.align.filter.fa

**Search is done here (the computational intensive part). Hooray! There are two useful output files:**

- $Tag.ssu.out/$Tag.qc.$Gene.align.filter:
  aligned SSU rRNA gene fragments

- $Tag.ssu.out/$Tag.qc.$Gene.align.filter.fa:
  unaligned SSU rRNA gene fragments

Extract the reads mapped 150bp region in V4 (577-727 in \*E.coli\* SSU rRNA gene position) for unsupervised clustering::

    python $Script_dir/region-cut.py $Tag.ssu.out/$Tag.qc.$Gene.align.filter $Start $End $Len_cutoff
    
    mv $Tag.ssu.out/$Tag.qc.$Gene.align.filter."$Start"to"$End".cut.lenscreen $Tag.ssu.out/$Tag.forclust

Classify SSU rRNA gene seqs using SILVA::

    rm -f $Tag.ssu.out/$Tag.qc.$Gene.align.filter.*.wang.taxonomy
    mothur "#classify.seqs(fasta=$Tag.ssu.out/$Tag.qc.$Gene.align.filter.fa, template=$Gene_db, taxonomy=$Gene_tax, cutoff=50, processors=$Cpu)"
    mv $Tag.ssu.out/$Tag.qc.$Gene.align.filter.*.wang.taxonomy \
        $Tag.ssu.out/$Tag.qc.$Gene.align.filter.wang.silva.taxonomy

Get the \*.taxonomy file has taxon for each SSU rRNA fragment sequence id. We can get the count for each taxon::

    python $Script_dir/count-taxon.py \
        $Tag.ssu.out/$Tag.qc.$Gene.align.filter.wang.silva.taxonomy \
        $Tag.ssu.out/$Tag.qc.$Gene.align.filter.wang.silva.taxonomy.count
    rm -f mothur.*.logfile

Classify SSU rRNA gene seqs with Greengene for copy correction later::

    rm -f $Tag.ssu.out/$Tag.qc.$Gene.align.filter.*.wang.taxonomy
    mothur "#classify.seqs(fasta=$Tag.ssu.out/$Tag.qc.$Gene.align.filter.fa, template=$Gene_db_cc, taxonomy=$Gene_tax_cc, cutoff=50, processors=$Cpu)"
    mv $Tag.ssu.out/$Tag.qc.$Gene.align.filter.*.wang.taxonomy \
        $Tag.ssu.out/$Tag.qc.$Gene.align.filter.wang.gg.taxonomy

Count the taxon::

    python $Script_dir/count-taxon.py \
        $Tag.ssu.out/$Tag.qc.$Gene.align.filter.wang.gg.taxonomy \
        $Tag.ssu.out/$Tag.qc.$Gene.align.filter.wang.gg.taxonomy.count
    rm -f mothur.*.logfile

Check the output directory::

    ls $Tag.ssu.out

Here is the a list of output files:

.. parsed-literal::

    1c.577to727
    1c.cut
    1c.forclust
    1c.qc.ssu
    1c.qc.ssu.align
    1c.qc.ssu.align.filter
    1c.qc.ssu.align.filter.577to727.cut
    1c.qc.ssu.align.filter.577to727.cut.lenscreen.fa
    1c.qc.ssu.align.filter.fa
    1c.qc.ssu.align.filter.greengene_97_otus.wang.tax.summary
    1c.qc.ssu.align.filter.silva_taxa_family.wang.tax.summary
    1c.qc.ssu.align.filter.wang.gg.taxonomy
    1c.qc.ssu.align.filter.wang.gg.taxonomy.count
    1c.qc.ssu.align.filter.wang.silva.taxonomy
    1c.qc.ssu.align.filter.wang.silva.taxonomy.count
    1c.qc.ssu.align.report
    1c.qc.ssu.hmmdomtblout
    1c.qc.ssu.hmmdomtblout.parsedToDictWithScore.pickle
    1c.qc.ssu.hmmtblout
    1c.qc.ssu.RFadded
    1c.qc.ssu.sto


**This part of pipeline (working with one sequence file) finishes here. Next we will combine samples for community analysis (see unsupervised analysis).**

**Following are files useful for community analysis**:

- 1c.forclust:
  aligned fasta file of seqs mapped to target region for de novo clustering

- 1c.qc.ssu.align.filter:
  aligned fasta file of all SSU rRNA gene fragments

- 1c.qc.ssu.align.filter.wang.gg.taxonomy:
  Greengene taxonomy (for copy correction)

- 1c.qc.ssu.align.filter.wang.silva.taxonomy:
  SILVA taxonomy
