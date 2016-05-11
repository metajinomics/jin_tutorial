어셈블리 튜토리얼
=================
이 튜토리얼은 아래 링크의 튜토리얼을 한글화 한 것이다. https://khmer-protocols.readthedocs.io/en/v0.8.4/index.html 리눅스 환경에서 작동한다. 


1. 퀄리티 트림, 시퀀스 필터
===========================

프로그램 설치
-------------
.. clean up previous installs if we're re-running this...

.. ::

   echo Removing previous installs, if any.
   rm -fr /usr/local/share/khmer
   rm -fr /root/Trimmomatic-*
   rm -f /root/libgtextutils-*.bz2
   rm -f /root/fastx_toolkit-*.bz2

Install `khmer <http://khmer.readthedocs.org/>`__:

::

    cd /usr/local/share
    git clone https://github.com/ged-lab/khmer.git
    cd khmer
    git checkout v1.1
    make install


`Trimmomatic <http://www.usadellab.org/cms/?page=trimmomatic>`__ 설치 :
=======
::

    cd /root
    curl -O http://www.usadellab.org/cms/uploads/supplementary/Trimmomatic/Trimmomatic-0.30.zip
    unzip Trimmomatic-0.30.zip
    cd Trimmomatic-0.30/
    cp trimmomatic-0.30.jar /usr/local/bin
    cp -r adapters /usr/local/share/adapters


`libgtextutils and fastx <http://hannonlab.cshl.edu/fastx_toolkit/>`__ 설치 :
::

    cd /root
    curl -O http://hannonlab.cshl.edu/fastx_toolkit/libgtextutils-0.6.1.tar.bz2
    tar xjf libgtextutils-0.6.1.tar.bz2
    cd libgtextutils-0.6.1/
    ./configure && make && make install

::

    cd /root
    curl -O http://hannonlab.cshl.edu/fastx_toolkit/fastx_toolkit-0.0.13.2.tar.bz2
    tar xjf fastx_toolkit-0.0.13.2.tar.bz2
    cd fastx_toolkit-0.0.13.2/
    ./configure && make && make install


In each of these cases, we're downloading the software -- you can use
google to figure out what each package is and does if we don't discuss
it below.  We're then unpacking it, sometimes compiling it (which we
can discuss later), and then installing it for general use.


폴더 생성 및 데이터 가져오기  
---------------------------------------------
Put your data in /mnt/data.

::
 
    cd /mnt
    mkdir work 
    cd work
    ln -fs /mnt/data/*.fastq.gz .
 
Trim Your Data
---------------

::
 
    cd /mnt/work
    python /usr/local/share/khmer/sandbox/write-trimmomatic.py > trim.sh 
    more trim.sh

If it looks like it contains the right commands, you can run it by doing 

::

    bash trim.sh

.. note::  This is a prime example of scripting to make your life much easier and less error prone. Take a look at this file sometime – ‘more /usr/local/share/khmer/sandbox/write-trimmomatic.py’ – to get some idea of how this works.

Quality Trim Each Pair of Files
--------------------------------

After you run this, you should have a bunch of ‘.pe.fq.gz’ files and a bunch of ‘.se.fq.gz’ files. The former are files that contain paired, interleaved sequences; the latter contain single-ended, non-interleaved sequences.

Next, for each of these files, run::

gunzip -c <filename> | fastq_quality_filter -Q33 -q 30 -p 50 | gzip -9c > <filename>.qc.fq.gz 

This uncompresses each file, removes poor-quality sequences, and then recompresses it. Note that (following Short-read quality evaluation) you can also trim to a specific length by putting in a ‘fastx_trimmer -Q33 -l 70 |‘ into the mix.

If fastq_quality_filter complains about invalid quality scores, try removing the -Q33 in the command; Illumina has blessed us with multiple quality score encodings.

위의 과정을 자동화 하기 
---------------------

This step can be automated with a ‘for’ loop at the shell prompt. Try:

::

    for i in *.pe.fq.gz *.se.fq.gz
    do
        echo working with $i
        newfile="$(basename $i .fq.gz)"
        gunzip -c $i | fastq_quality_filter -Q33 -q 30 -p 50 | gzip -9c > "${newfile}.qc.fq.gz"
    done
What this loop does is:

* for every file ending in pe.fq.gz and se.fq.gz,
* print out a message with the filename,
* construct a name ‘newfile’ that omits the trailing .fq.gz
* uncompresses the original file, passes it through fastq, recompresses it, and saves it as ‘newfile’.qc.fq.gz

Extracting Paired Ends From The Interleaved Files
--------------------------------------------------

The fastx utilities that we’re using to do quality trimming aren’t paired-end aware; they’re removing individual sequences. Because the pe files are interleaved, this means that there may now be some orphaned sequences in there. Downstream, we will want to pay special attention to the remaining paired sequences, so we want to separate out the pe and se files. How do we go about that? Another script, of course!

The khmer script ‘extract-paired-reads.py’ does exactly that. You run it on an interleaved file that may have some orphans, and it produces .pe and .se files afterwards, containing pairs and orphans respectively.

To run it on all of the pe qc files, do:

::

    for i in *.pe.qc.fq.gz
    do
        extract-paired-reads.py $i
    done

파일 이름 바꾸기 
---------------
I’m a fan of keeping the files named somewhat sensibly, and keeping them compressed. Let’s do some mass renaming:

::
    
    for i in *.pe.qc.fq.gz.pe 
    do
        echo working on PE file $i
        newfile="$(basename $i .pe.qc.fq.gz.pe).pe.qc.fq"
        rm $(basename $i .pe)
        mv $i $newfile
        gzip $newfile
    done

and also some mass combining:

::

    for i in *.pe.qc.fq.gz.se
    do
        echo working on SE file $i
        otherfile="$(basename $i .pe.qc.fq.gz.se).se.qc.fq.gz"
        gunzip -c $otherfile > combine
        cat $i >> combine
        rm -f $otherfile
        gzip -c combine > $otherfile
        rm $i combine
    done

then make it hard to delete the files you just created

::

    chmod u-w *.qc.fq.gz

Done!  Now you have two files: SRR606249-extract.pe.qc.fq.gz, SRR606249-extract.se.qc.fq.gz.

The '.pe' file are interleaved paired-end; you can take a look at them like so 


The '.se' files is a single-ended file, where the reads have been
orphaned because we discarded stuff.

All TWO files are in FASTQ format.

----

Next: :doc:`2-diginorm`
