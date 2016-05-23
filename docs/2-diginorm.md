================================
2. Running digital normalization
================================

.. shell start

.. note::

   Make sure you're running in screen!

Start with the QC'ed files from :doc:`1-quality` or copy them into a
working directory; 

Run a First Round of Digital Normalization
------------------------------------------

Normalize everything to a coverage of 20, starting with the (more valuable)
PE reads; keep pairs using '-p'

::

   cd /mnt/work
   normalize-by-median.py -p -k 20 -C 20 -N 4 -x 1e9 --savetable normC20k20.kh  *.pe.qc.fq.gz

and continuing into the (less valuable but maybe still useful) SE reads

::

   normalize-by-median.py -C 20 --loadtable normC20k20.kh --savetable normC20k20.kh *.se.qc.fq.gz

This produces a set of '.keep' files, as well as a normC20k20.kh
database file.

Error-trim Our Data
--------------------

Use 'filter-abund' to trim off any k-mers that are abundance-1 in
high-coverage reads.  The -V option is used to make this work better
for variable coverage data sets:

::

   filter-abund.py -V normC20k20.kh *.keep


This produces .abundfilt files containing the trimmed sequences.

The process of error trimming could have orphaned reads, so split the
PE file into still-interleaved and non-interleaved reads

::

    for i in *.pe.*.keep*
    do
       extract-paired-reads.py $i
    done

This leaves you with PE files (.pe.qc.fq.gz.keep.abundfilt.pe) and
two sets of SE files (.se.qc.fq.gz.keep.abundfilt and
.pe.qc.fq.gz.keep.abundfilt.se).  (Yes, the naming scheme does make
sense.  Trust me.)


Normalize Down to C=5
---------------------

Now that we've eliminated many more erroneous k-mers, let's ditch some more
high-coverage data.  First, normalize the paired-end reads 

::
    
   normalize-by-median.py -C 5 -k 20 -N 4 -x 1e9 --savetable normC5k20.kh -p *.pe.qc.fq.gz.keep.abundfilt.pe

and then do the remaining single-ended reads

::
    
   normalize-by-median.py -C 5 --savetable normC5k20.kh --loadtable normC5k20.kh *.pe.qc.fq.gz.keep.abundfilt.se *.se.qc.fq.gz.keep.abundfilt


Compress and Combine the Files
------------------------------

Now let's tidy things up.  Here are the paired files (kak =
keep/abundfilt/keep) 

::
   
    for pe in *.pe.qc.fq.gz.keep.abundfilt.pe.keep
    do 
    	se=${pe/pe.keep/se.keep}
	newfile=${pe/.pe.qc.fq.gz.keep.abundfilt.pe.keep/.pe.kak.qc.fq.gz}
	cat $pe $se |gzip -c > $newfile
    done

and for the single-ended files 

::

    for se in *.se.qc.fq.gz.keep.abundfilt.keep 
    do 
        newfile=${se/.se.qc.fq.gz.keep.abundfilt.keep/.se.kak.qc.fq.gz}
        gzip -c  $se >$newfile
    done

You can now remove all of these various files:: 

   *.pe.qc.fq.gz.keep
   *.pe.qc.fq.gz.keep.abundfilt
   *.pe.qc.fq.gz.keep.abundfilt.pe
   *.pe.qc.fq.gz.keep.abundfilt.pe.keep
   *.pe.qc.fq.gz.keep.abundfilt.se
   *.pe.qc.fq.gz.keep.abundfilt.se.keep

by typing
 
::

    rm *.keep *.abundfilt *.pe *.se


If you are *not* doing partitioning (see :doc:`3-partition`), you may
also want to remove the k-mer hash tables::

   rm *.kh

If you *are* running partitioning, you can remove the ``normC20k20.kh`` file::

   rm normC20k20.kh

but you will need the ``normC5k20.kh`` file.

Read Stats
----------

Try running

::

   /usr/local/share/khmer/sandbox/readstats.py *.kak.qc.fq.gz *.?e.qc.fq.gz

after a long wait, you'll see::

   ---------------
   861769600 bp / 8617696 seqs; 100.0 average length -- SRR606249.pe.qc.fq.gz
   79586148 bp / 802158 seqs; 99.2 average length -- SRR606249.se.qc.fq.gz
   531691400 bp / 5316914 seqs; 100.0 average length -- SRR606249.pe.qc.fq.gz
   89903689 bp / 904157 seqs; 99.4 average length -- SRR606249.se.qc.fq.gz

   173748898 bp / 1830478 seqs; 94.9 average length -- SRR606249.pe.kak.qc.fq.gz
   8825611 bp / 92997 seqs; 94.9 average length -- SRR606249.se.kak.qc.fq.gz
   52345833 bp / 550900 seqs; 95.0 average length -- SRR606249.pe.kak.qc.fq.gz
   10280721 bp / 105478 seqs; 97.5 average length -- SRR606249.se.kak.qc.fq.gz
   
   ---------------

This shows you how many sequences were in the original QC files, and
how many are left in the 'kak' files.  Not bad -- considerably more
than 80% of the reads were eliminated in the kak!

----

Next: :doc:`3-partition`
