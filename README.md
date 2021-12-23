# NAL RNA-Seq Annotation Pipeline (Under Development)

[![Build Status](https://travis-ci.org/NAL-i5K/NAL_RNA_seq_annotation_pipeline.svg?branch=master)](https://travis-ci.org/NAL-i5K/NAL_RNA_seq_annotation_pipeline)

A RNA-Seq annotation pipeline based on [SRA Toolkit](https://github.com/ncbi/sra-tools), [fastQC](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/), [Trimmomatic](http://www.usadellab.org/cms/?page=trimmomatic), [HISAT2](https://github.com/infphilo/hisat2), [BBMap](https://sourceforge.net/projects/bbmap/), [picard](https://broadinstitute.github.io/picard/), [GATK3](https://github.com/broadgsa/gatk-protected), and [samtools](https://github.com/samtools/samtools). It's distributed as a python package.

There are two parts in this pipeline. 

**Rnannot part -** doing sequence alignment and converting output to bigwig format. The outputs of this part should contains one bam file, one indexed bam file, one bigwig file, one bed file and one Source.txt file. If temp files are kept in this part(option -t), it will also generate one unsorted bam file, one bam and one sam file generated by SINGLE layout SRA files, one bam and one sam file generated by PAIRED layout SRA files.

**Add_trackList part -** adding bam, bigwig file and junction reads to trackList.json file on apollo-stage server. This part should be run on apollo-stage server. It will transfer output files of anannot to apollo-stage and node1 server and update the trackList.json file. 

## Prerequisite
For rnannot
- At least Python 3.5
- Java
- [SRA Toolkit](https://github.com/ncbi/sra-tools)
- [samtools](https://github.com/samtools/samtools)
- [bam_to_bigwig(python3-version)](https://github.com/NAL-i5K/bam_to_bigwig.git)
- [rsem](https://github.com/deweylab/RSEM/releases) (prerequisite of bam_to_bigwig)
- [wigToBigWig](http://hgdownload.cse.ucsc.edu/admin/exe/) (prerequisite of bam_to_bigwig)
- [regtools](https://regtools.readthedocs.io/en/latest/)
- [edirect](https://dataguide.nlm.nih.gov/edirect/install.html#edirect-installation)


For add_trackList, the following JBrowse processing scripts are needed 
- add-bam-track.pl
- add-bw-track.pl
- flatfile-to-json.pl

## Installation by yourself

After you set up all of the prerequisites, run setup.py file for installing. 
- `python setup.py install`. It will install a copy of FastQC, Trimmomatic, HISAT2, GATK3, and picard in this python package. You may need to add `--user` in arguments.

## Uninstallation

- `pip uninstall rnannot` 

## Usage

``` shell

download_sra_metadata.py [-t TAXID] [-o [OUTPUT]]

Use pipeline to download the sra metadata,the output file will be used for the input file of RNAseq_annotate.py.

optional arguments:
  -t TAXID, --taxid TAXID    find all RNA SRA files for a given taxid
  -o [OUTPUT], --output [OUTOUT] directory and name of output folder at, if not specified, use current folder


download_sra_metadata_by_accessions.py [-a ACCESSION] [-o [OUTPUT]]

Download the sra metadata of specified accessions, the output file will be the input file of RNAseq_annotate.py. 
If processing multiple accessions, use spaces between each accession and output file directory.

optional arguments:
  -a ACCESSION, --accession ACCESSION    find the RNA SRA files for the given accessions
  -o [OUTPUT], --output [OUTOUT]    output directory and output file name
                                    if not specified, use current folder and file name "ACCESSION.tsv"


RNAseq_annotate.py [-h] [-i INPUT] [-g GENOME] [-n [NAME]]
                               [-o [OUTDIR]] [-a ASSEMBLY] [-t]
                               [-m MAXIMUMSRA]

Easy to use pipeline built for large-scale RNA-seq mapping with a genome
assembly

optional arguments:
  -h, --help            show this help message and exit
  -i INPUT, --input INPUT
                        A tsv file with a list of SRA runs' information.
  -g GENOME, --genome GENOME
                        A fasta file to align with.
  -n [NAME], --name [NAME]
                        name of the output folder, if not specified, use the
                        time of start
  -o [OUTDIR], --outdir [OUTDIR]
                        directory of output folder - it must already exit. if not specified, use
                        current folder
  -a ASSEMBLY, --assembly ASSEMBLY
                        The assembly name is used for naming output file
  -t, --tempFile        
                        if specified, intermediate output bam files will be kept
  -m MAXIMUMSRA, --MaximumSRA MAXIMUMSRA
                        The maximum amout of SRA files downloaded from NCBI. The default is 10
  
add_trackList.py [-h] [-a INPUT_ACCOUNT] [-p INPUT_PATH]
                        [-bam INPUT_BAM] [-bigwig INPUT_BIGWIG]
                        [-bai INPUT_BAI] [-bed INPUT_BED] [-track INPUT_TRACK]
                        [-s SOURCE]
                        
optional arguments:
  -h, --help            show this help message and exit
  -a INPUT_ACCOUNT, --input_account INPUT_ACCOUNT
                        scinet account e.g user@login.scinet.science
  -p INPUT_PATH, --input_path INPUT_PATH
                        path of RNA_annotation output files on Scinet
  -bam INPUT_BAM, --input_bam INPUT_BAM
                        bam file name
  -bigwig INPUT_BIGWIG, --input_bigwig INPUT_BIGWIG
                        bigwig file name
  -bai INPUT_BAI, --input_bai INPUT_BAI
                        indexed bam file name
  -bed INPUT_BED, --input_bed INPUT_BED
                        indexed bed file name
  -track INPUT_TRACK, --input_track INPUT_TRACK
                        trackList.json file path
  -s SOURCE, --Source SOURCE
                        Source.txt file name

move_data.py [-h] [-Node1a NODE1_ACCOUNT] [-s SOURCE]

optional arguments:
  -h, --help            show this help message and exit
  -Node1a NODE1_ACCOUNT, --node1_account NODE1_ACCOUNT
                        apollo-nodea account e.g user@apollo-
                        node1,nal.usda.gov
  -s SOURCE, --Source SOURCE
                        Source.txt file path
```

## Example
**Rnannot**
- `download_sra_metadata.py -t 1049336 -o 1049336.tsv`
- `download_sra_metadata_by_accessions.py -a SRX12101703 SRX12101702 SRX12101701 -o SRX12101703.tsv SRX12101702.tsv SRX12101701.tsv`
- `wget "https://i5k.nal.usda.gov/data/Arthropoda/ephdan-(Ephemera_danica)/BCM-After-Atlas/1.Genome%20Assembly/BCM-After-Atlas/Scaffolds/Edan07162013.scaffolds.fa.gz"`
- `RNAseq_annotate.py -i ./example/1049336.tsv -g ./Edan07162013.scaffolds.fa.gz -a Edan_2.0`

**Add_trackList**
- `add_trackList.py -a user@login.scinet.science -p /project/nal_genomics/user-name/NAL_RNA_seq_annotation_pipeline/rnannot/2020-05-20 -bam Tricas_Tcas5.2_RNA-Seq-alignments_2020-05-20.bam -bai Tricas_Tcas5.2_RNA-Seq-alignments_2020-05-20.bam.bai -bigwig Tricas_Tcas5.2_RNA-Seq-alignments_2020-05-20.bigwig -bed Tricas_Tcas5.2_RNA-Seq-alignments_2020-05-20.bed -track /app/data/other_species/tricas/Tcas5.2/jbrowse/data/trackList.json -s Source.txt`
- `move_data.py -Node1a user@apollo-node1.nal.usda.gov -s SOURCE.txt`

## Run on Docker container
We provide docker image which includes all of the prerequisites and has everything installed. It also contains the whole repo, so you don't need to clone this repo if you use this docker container. The working directory of this image is set to **/opt/output** and all of the repo files are in **/opt/RNA_repo**

To get this docker image, you can:

1. Build this image by Dockerfile. 
   clone this repo and run `sudo docker build -t [your_image_tag_name] .`
   
   or
   
2. Pull this image from docker hub.
   run `sudo docker pull k2025242322/i5k_rna_seq_annotation_pipeline:latest`
   
**Docker commands:**   
- run `sudo docker run --rm --mount type=bind,source=[The directory you would like to bind],target=/opt/output [your_image_tag_name] python3 /opt/RNA_repo/rnannot/download_sra_metadata.py -t [tax_id] -o [tax_id.tsv]`
- run `wget [genome file URL]` (download your file in the binding directory) or `sudo docker run --rm --mount type=bind,source=[The directory you would like to bind],target=/opt/output [your_image_tag_name] wget [genome file URL]`
- run `sudo docker run --rm --mount type=bind,source=[The directory you would like to bind],target=/opt/output [your_image_tag_name] python3 /opt/RNA_repo/rnannot/RNAseq_annotate.py -i /opt/output/[tax_id.tsv] -g /opt/output/[genome file] -a [assembly_name]`

## Run on Ceres (by conda virtual environment)
**1. Setup conda env**
- https://scinet.usda.gov/guide/conda/
- Used conda to create an new env.
- Package Pysam is not included in the module list of Ceres. Use conda to install them into env. (Python3 and Perl5 may be insatlled into env at the same time) Pysam: https://anaconda.org/bioconda/pysam

We also provide a conda venv which is ready to be used. If you don't want to create your conda env, you can skip step1~4 and run the following command.
- `module load minicanda` and `conda activate /lustre/project/nal_genomics/hsiukang/rnannot_venv`
- Note that you may need to specify the full path of the command for it to work. 

**2. Git clone RNA_annotation_pipeline into working directory**

**3. Activate conda env**
- Do `module load minicanda` and `conda activate [env_name]`

**4. Run setup.py**
- Do `python3 setup.py install`
(If meet "ValueError: bad marshal data", delete .pyc file may solve this problem.)
find /home/[user_name] -name '*.pyc' -delete

**5. Use bash script to submit job**
- Download your genome file by `wget [URL of your genome file]`
- Edit your command in the sbatch.sh file.
- Do `sbatch sbatch.sh`

## Run on Ceres (by singularity container)
You can find more information about singularity here: https://scinet.usda.gov/guide/singularity

**1. Pull docker image from docker hub**
- Do `salloc` before you run singularity command
- Do `SINGULARITY_CACHEDIR=[the directory for storaging blob files] singularity -d build rnannot.img docker://k2025242322/i5k_rna_seq_annotation_pipeline`
  [the directory for storaging blob files] can not under your home directory. You may not have enough quota for all blob files.  

**2. Edit bash script**
- Download your genome file by `wget [URL of your genome file]`
- Edit your command in the sbatch.sh file.
- example:
- `singularity exec [the PATH of rnannot.img] python3 /opt/RNA_repo/rnannot/download_sra_metadata.py -t 1049336 -o 1049336.tsv`
- `wget [URL of your genome file]`
- `singularity exec [the PATH of rnannot.img] python3 /opt/RNA_repo/rnannot/NAseq_annotate.py -i 1049336.tsv -g Edan07162013.scaffolds.fa.gz -a Edan`
  
## Notes

- The input tsv should have at least five columns, including `Run`, `Platform`, `Model`, `LibraryLayout` (header must be presented), and `download_path`.
  - `Run` column represents the paths to the SRA files. You can use either relative path to your current directory or absolute path. To make less confusion, we recommned to use absolute path.
  - `Platform` column represents the sequencer's brand. We will recognize `ILLUMINA` and `ABI_SOLID` (although we will not process `ABI_SOLID`) in this field, because it determines the adapters used in the pipeline with `Model`.
  - `Model` column represents the sequencer's model. We will recongize `Illumina HiSeq ...`, `Illumina MiSeq ...`, and `Illumina Genome Analyzer II ...`, because it determines the adapters used in the pipeline with `Platform`. Don't forget that if you have any space in the name of the model. You need to escape them using quoting such as `"Illumina Hiseq 2000"` or back slash. 
  - `LibraryLayout` column represents what's the strategy of RNA-Seq experiment. It can be only `SINGLE` or `PAIRED`.
    - Currently, the `ABI_SOLID` sequencer is not supported.
    - For paired-end layout, `Trimmomatic` will produces four fastq files: forward\_paired, forward\_unpaired, reverse\_paired, reverse\_unpaired, but we will only use the paired data in alignment (by HISAT2)
  - `download_path` column represents where we can download the SRA files.
- When using on server, make sure you use the `JAVA_TOOL_OPTIONS` environment to set the maximum memory usage like `export JAVA_TOOL_OPTIONS="-Xmx2g"` when running the toolkit. You can also check an example [here](example/example_script.sh).
- The format of the output name of bam file, indexed bam file, bigwig file and bed file is [gggsss] _ [assembly Name] _ RNA-Seq-alignments _ [date]. The name of the new folders created in apollo server is also based on this format. Changing this format may affect the downstream workflow(add_trackList.py and move_data.py). 

## Tests

### Test environment

- `python -m unittest -f tests/test_setup.py`

### Test parser

- `python -m unittest -f tests/test_parser.py`
