TRUP is a pipeline designed for analyzing RNA-seq data from Tumor samples.


Learn More
---
Cancer cells express many rearranged transcripts posing increased complexity to transcriptome analysis. As an unified pipeline, TRUP is designed to sensitively and accurately dissect the complexity of the cancer transcriptome by analyzing RNA-seq data obtained from tumour tissues. The current functionalities of TRUP include: 1) identification of fusion transcripts; 3) RNA-seq quality assesment; 2) Gene-read counting. The fusion detection module in TRUP combines split-read/read-pair mapping with regional de-novo assembly to achieve a balance between sensitivity and precision.


Dependencies
---
+   Samtools
+   Bowtie2
+   BLAT
+   GSNAP (version > 2012-07-20, recommended for fusion detection) or STAR (version 2.4.0)
+   Velvet (https://github.com/dzerbino/velvet).
+   Oases (https://github.com/dzerbino/oases).
+   R libraries: feilds, KernSmooth, lattice, RColorBrewer and R2HTML.
+   Bioconductor packages: DESeq2, edgeR.

Annotation files
---
The hg19 and mm10 annoatation is downloadable via the link below: 

**hg19**

	https://drive.google.com/file/d/0B8NcFh4TSjvOQVN1anhrSVdLVjA/view?usp=sharing
	https://drive.google.com/file/d/0B8NcFh4TSjvOWVlvOVVFWjRxZDQ/view?usp=sharing

**mm10**

	https://drive.google.com/file/d/0B8NcFh4TSjvOaFNCM2FmLVU1eFE/view?usp=sharing
	https://drive.google.com/file/d/0B8NcFh4TSjvOOFNXWDRKVUtUVzg/view?usp=sharing


All the annotation files and directories should be put under an central annotation directory whose name will be used for argument "--anno" when running TRUP. The structure of the annotation directory and sub-directories are illustrated as the following example:

Example:

         TRUP_ANNOTATION  /   hg19   /    hg19.SelfChain_UCSC.txt
         TRUP_ANNOTATION  /   hg19   /    hg19.bowtie2_index   /  *
               |               |                  |
         central_directory / version  / annotation files and directories

If users choose to prepare annotation directories and files by themselves, it can be achieved in the following way (using hg19 as an example):
I am devising a script to generate the annotations automatically and also make them more consistent in terms of naming.

TRUP_ANNOTATION/hg19/

+ hg19.genome_UCSC.2bit (for running BLAT: runlevel 4)
  How to get: Use faToTwoBit to generate a 2.bit file from a genome fasta file (http://genome.ucsc.edu/goldenPath/help/blatSpec.html#faToTwoBitUsage)

+ hg19.transcriptome_length.txt (for generating html report: runlevel 2)
  How to get: Sum up the length of all exons for each chromosome. Each line is formatted as "chromosome	sun_of_exons_length" (tab delimited).

+ hg19.genes_Ensembl.bed12 (for gene read counting: runlevel 2)
+ hg19.gencode/gencode.v14.annotation.gene.bed12 (not available for organism other than hg19)
  How to get: These 12 column bed files can be generated by my script "exon_union_gtf2bed.pl" under the TRUP directory src/ by using normal .gtf files downloadable from Ensembl ftp site for a specific organism.

+ hg19.transcripts_Ensembl.gtf (for tophat2 mapping and cufflinks: runlevel 2 and 5)
  How to get: This file can be downloaded from Ensembl ftp. use this to generate hg19.genes_Ensembl.bed12 mentioned above.

+ hg19.gencode/gencode.v14.annotation.gtf (not available for organism other than hg19)
  How to get: downloadable from UCSC table browser, use this to generate hg19.gencode/gencode.v14.annotation.gene.bed12 mentioned above.

+ hg19.gencode/gencode.v14.genemap (for runlevel 2. not available for organism other than hg19)
  How to get: use the script "gencode_annotation.map.pl" under TRUP/src and the gencode gtf file to generate this gene map file.

+ hg19.genes_RefSeq.bed12 (for runlevel 2. may not be used in the new version of TRUP)
+ hg19.SelfChain_UCSC.txt (for fusion detection: runlevel 4)
+ hg19.repeats_UCSC.gff (for fusion detection: runlevel 4)
  How to get: Downloadable from UCSC table browser

+ hg19.transcripts_Ensembl.gff
  How to get: turn hg19.transcripts_Ensembl.gtf to generate the corresponding gff file by using "gtf2gff3" (http://search.cpan.org/~lds/GBrowse-2.54/bin/gtf2gff3.pl)

+ hg19.biomart.txt
  How to get: use R package biomaRt or web converter at (http://www.ensembl.org/biomart/martview/), choose appropriate databse, feed in filter names (all ensembl gene id), and select attributes for "Associated Gene Name", "Gene Biotype", "Description", "EntrezGene ID", "WikiGene Name", "WikiGene Description".

+ hg19.gmap_index/hg19/hg19.*
  How to get: download from (http://research-pub.gene.com/gmap/) or follow instructions provided by gmap/gsnap

+ hg19.bowtie2_index/hg19/hg19.*
+ hg19.bowtie2_index/hg19_trans/hg19_known_ensemble_trans.*
  How to get: download from (http://bowtie-bio.sourceforge.net/bowtie2/index.shtml) or follow instructions provided by bowtie2. For annotation files under hg19_trans/, run tophat2 once with --transcriptome-index hg19.bowtie2_index/hg19_trans/hg19_known_ensemble_trans and --GTF hg19.transcripts_Ensembl.gtf to get tophat index for transcriptome (tophat2 can be stopped as long as the indexes are generated, thus user can use any fastq file as input for running tophat2, just to get the indexes). These transcriptome indexes can be used for later tophat2 calls.


Installation
---
No installation is required if pre-compiled binaries in bin/ are compatible with your system.

If pre-compiled binaries are incompatible with your system, you will need to rebuild them from source.

Make sure you have installed Bamtools (https://github.com/pezmaster31/bamtools). Then write down the bamtools_directory where ./lib/ and ./include/ sub-directories are located. Currently bamtools 2.3.0 has been tested. Also make sure you have zlib installed. Then write down the zlib_directory where ./lib/ and ./include/ sub-directories are located

run make in following way to install

	$ make BAMTOOLS_ROOT=/bamtools_directory/ ZLIB_ROOT=/zlib_directory/

The binaries will be built at bin/. Any existing files will be overwritten.


Usage
---

TRUP can be run with UNIX command-line interface.

Assuming that in the directory "RP" there are two gzipped fastq-files are called as SAMPLE_R?1(_XXX)?.fq.gz and SAMPLE_R?2(_XXX)?.fq.gz (where "\_R?1" and "\_R?2" means mate 1 and mate 2 reads, XXX means a run ID, R and _XXX are optional that can be missing), the path to the directory of the pipeline is called "PD" and the path to tha annotation files are called "AD", one can run the pipeline in the following way:

**Run-level 1** (Quality check, mapping of spiked-in read pairs and give a rough estimate of the insert size etc.):

	$ perl RTrace.pl --runlevel 1 --sampleName SAMPLE --seqType p --readpool RP --root PD --threads TH --anno AD 2>>run.log 

where "TH" is the number of computing threads. If one wants to stop after the quality check, add ``--QC`` in the command call. ``--seqType`` indicates sequencing type, either ``s``(single-end) or ``p``(paired-end). If your fastq files have names with a run ID (e.g., _001, _002, etc), usually produced by CASAVA, set ``runID`` to 001, 002, etc. --Rbinary`` can be used to set binary name for R (if you have an alternative binary name for R)

**Run-level 2** (mapping with gsnap [default] or STAR, generating reports):

	$ perl RTrace.pl --runlevel 2 --sampleName SAMPLE --seqType p --readpool RP --root PD --anno AD --threads TH --WIG --patient ID --tissue type --threads TH --gf pdf 2>>run.log

where ``--WIG`` is set to generate bigWiggle file for visualization purpose. ``--patient`` and ``--tissue`` can be set if user need to run edgeR and cuffdiff after processing all the samples. If a X11 is not always available, set ``--gf`` to be ``pdf`` to generate the report figures in pdf format instead of png. However, if X11 is available, do not set the --gf option.

**Run-level 3** (collect regions containning potential breakpoints from the mapping of gsnap, perform regional assembly in the candidate regions):

	$ perl RTrace.pl --runlevel 3 --sampleName SAMPLE --seqType p --readpool RP --root PD --anno AD --threads TH --RA 1 2>>run.log

where ``--RA`` is set to 1 to indicate an independent regional assembly around each breakpoint.

**Run-level 4** (Dectecting fusion events using the assembled transcripts):

	$ perl RTrace.pl --runlevel 4 --sampleName SAMPLE --seqType p --readpool RP --root PD --anno AD --threads TH 2>>run.log

User can use GMAP instead of BLAT for mapping the assembled contigs back to the reference, in which case, ``--GMAP`` should be set to use GMAP in this step. ``--misPen`` can be adjusted to integer numbers higher than 2 (default) to suppress low quality blat results. ``--uniqueBase`` can be set to integers more than 100 (default, which is exactly just one match, suggest for 300 maximum, which allows three times of matches to the reference) to allow multiple mapping segments to be granted as "unique" to increase sensitivity. 

**Run-level 5** (run cufflinks for gene/isoform quantification):

	$ perl RTrace.pl --runlevel 5 --sampleName SAMPLE --seqType p --readpool RP --root PD --anno AD --threads TH --gtf-guide --known-trans refseq 2>>run.log

where ``--known-trans`` can be set to 'ensembl' or 'refseq' to indicate the annotation to be used for cufflinks.

**Run-level 6** (run cuffdiff for diffrential gene/isoform expression analysis):

	$ perl RTrace.pl --runlevel 6 --root PD --anno AD --threads TH 2>>run.log

**Run-level 7** (run edgeR for diffrential gene expression analysis):

	$ perl RTrace.pl --runlevel 7 --root PD --anno AD --threads TH --priordf 5 --spaired 1 2>>run.log

where ``--priordf`` sets the prior.df parameter in edgeR. ``--spaired`` should be set to 1 if the samples are paired (paired normal-diese samples from the sample patient).

Note: do not set ``--lanename`` for runlevel 6 and 7, as these two runlevels are dealing with multiple samples. 

The Run-levels metioned above can also be combined in the same command line, as following,

	$ perl RTrace.pl --runlevel 1-4 --sampleName SAMPLE --seqType p --readpool RP --root PD --anno AD --threads TH --WIG --patient ID --tissue type --gf pdf --RA 1 2>>run.log

One can also call combinations of different Run-levels, such as ``--$runlevel 1,2`` or ``--runlevel 2,3,4``. But in these combined ways, one should specify all necessary options in the command line.

Runlevel dependencies (->): 4->3->2->1, 6->5->2->1, 7->2->1

Options
---
check the options by running the program without options or with ``--help``.


Contact
---
Sun Ruping

If you are willing to receive updates and notice, send an email to regularhand@gmail.com.

Current Affiliation:
Califano Lab, Department of Systems Biology, Columbia University, New York, NY, USA

Previous Affiliation:
Dept. Vingron (Computational Molecular Biology)
Max Planck Institute for Molecular Genetics. Ihnestr. 63-73, D-14195 Berlin, Germany

Email: regularhand@gmail.com

Project Website: https://github.com/ruping/TRUP
