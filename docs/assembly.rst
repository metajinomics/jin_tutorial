어셈블리 튜토리얼
=================
이 튜토리얼은 아래 링크의 튜토리얼을 한글화 한 것이다. https://khmer-protocols.readthedocs.io/en/v0.8.4/index.html 리눅스 환경에서 작동한다. 


1. 퀄리티 트림, 시퀀스 필터
===========================

프로그램 설치
-------------
screed 설치
::
   pip install screed

.. clean up previous installs if we're re-running this...

.. ::

   echo Removing previous installs, if any.
   rm -fr /usr/local/share/khmer
   rm -fr /root/Trimmomatic-*
   rm -f /root/libgtextutils-*.bz2
   rm -f /root/fastx_toolkit-*.bz2

Install `khmer <http://khmer.readthedocs.org/>`__:

::

    pip install khmer


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


각각의 경우에서 필요한 프로그램을 다운받았다. 프로그램의 자세한 내용이 알고 싶다면 검색을 통해 쉽게 알 수 있으며 이곳에서 추가 설명을 하지 않겠다. 다운로드 받은 프로그램은 압축을 풀고 필요에 따라서 컴파일 하기도 하였다. 그리고 설치하였다. 


작업 폴더 만들기   
---------------------------------------------
작업을 할 폴더를 만들자 

::
 
    cd /mnt
    mkdir assembly
    cd assembly
 
데이터 링크하기
---------------
이 튜토리얼에서 사용한 데이터는 Sharon et al. 2013 에서 가져왔다. 유아 대변 샘플 두개이다. 

::

   ln -fs /data/SRR49206?_?.fastq.gz .
이 과정이 데이터를 /mnt/assembly 폴더로 링크를 만들어 준다. 
   
데이터를 트림하기 
---------------
첫번째 데이터 트림하기 (20분 가량 소요)
::

   mkdir trim
   cd trim
   
   java -jar /usr/local/bin/trimmomatic-0.30.jar PE ../SRR492065_?.fastq.gz s1_pe s1_se s2_pe s2_se ILLUMINACLIP:/usr/local/share/adapters/TruSeq3-PE.fa:2:30:10
   
   /usr/local/share/khmer/scripts/interleave-reads.py s?_pe > combined.fq
   
   fastq_quality_filter -Q33 -q 30 -p 50 -i combined.fq > combined-trim.fq 
   fastq_quality_filter -Q33 -q 30 -p 50 -i s1_se > s1_se.trim
   fastq_quality_filter -Q33 -q 30 -p 50 -i s2_se > s2_se.trim
   /usr/local/share/khmer/scripts/extract-paired-reads.py combined-trim.fq
   
   gzip -9c combined-trim.fq.pe > ../SRR492065.pe.qc.fq.gz
   gzip -9c combined-trim.fq.se s1_se.trim s2_se.trim > ../SRR492065.se.qc.fq.gz

   cd ../
   rm -fr trim

두번째 데이터 (20분 가량 소요)
::

   mkdir trim
   cd trim
   
   java -jar /usr/local/bin/trimmomatic-0.30.jar PE ../SRR492066_?.fastq.gz s1_pe s1_se s2_pe s2_se ILLUMINACLIP:/usr/local/share/adapters/TruSeq3-PE.fa:2:30:10

   /usr/local/share/khmer/scripts/interleave-reads.py s?_pe > combined.fq
   
   fastq_quality_filter -Q33 -q 30 -p 50 -i combined.fq > combined-trim.fq
   fastq_quality_filter -Q33 -q 30 -p 50 -i s1_se > s1_se.trim
   fastq_quality_filter -Q33 -q 30 -p 50 -i s2_se > s2_se.trim
   /usr/local/share/khmer/scripts/extract-paired-reads.py combined-trim.fq
   
   gzip -9c combined-trim.fq.pe > ../SRR492066.pe.qc.fq.gz
   gzip -9c combined-trim.fq.se s1_se.trim s2_se.trim > ../SRR492066.se.qc.fq.gz
   
   cd ../
   rm -fr trim

이제 다음 4개의 파일을 볼 수 있을 것이다. (SRR492065.pe.qc.fq.gz, SRR492065.se.qc.fq.gz, SRR492066.pe.qc.fq.gz, and SRR492066.se.qc.fq.gz) 확장자 .pe 파일은 interleave 된 paired-end 파일이다. 아래와 같이 하면 파일의 첫 부분을 볼 수 있다. 
::

   gunzip -c SRR492065.pe.qc.fq.gz | head
   
다른 두개의 파일은 single-end파일이다. 트림과정에서 짝을 잃어버린 것들이다. 모든 파일은 FASTQ 포멧이다.  

----

Next: :doc:`2-diginorm`
