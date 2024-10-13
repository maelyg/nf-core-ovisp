# ONTViSc-hands-on-training
**Friday 1st November 2024 10-12 am**  
**In person workshop for DAFF staff**  
**QUT Garden Point P block level 8 room 810**  

## Aims of this workshop
- Learn to install and run ONTViSc with test data on the HPC Lyra
- Run and review QC
- Run the 4 different modes
- If time permits test with datasets from participants

## What is Oxford Nanopore data?
Using long nanopore DNA/RNA sequencing reads researchers can:
- Resolve complex structural variants and repetitive regions
- Simplify de novo genome assembly and improve existing reference genomes
- Study linkage and phasing
- Enhance metagenomic identification of closely related species and distinguish plasmid from genome
- Sequence entire microbes in single reads – in real-time
- Explore epigenetic modifications using direct, long-read DNA sequencing

## How does it work?
Nanopore sequencing is a unique, scalable technology that enables direct, real-time analysis of long DNA or RNA fragments. It works by monitoring changes to an electrical current as nucleic acids are passed through a protein nanopore. The resulting signal is decoded to provide the specific DNA or RNA sequence.  
A range of nanopore sequencing devices are available, providing high-yields and scalable sample throughput to suit all requirements — from portable analysis using Flongle and MinION, through to flexible, high-throughput benchtop sequencing on GridION and PromethION. MinION Starter Packs are available from just $1,000 providing low-cost access to the benefits of long-read, real-time DNA sequencing.  
Unlike traditional DNA sequencing platforms, which deliver data in bulk at the end of a sequencing run, nanopore DNA sequencing data is streamed in real time — providing immediate access to results. Advantages of real-time data streaming include rapid access to time critical information (e.g. pathogen identification), the generation of early sample insights, and the facility to stop sequencing once a result has been achieved — enabling washing and reuse of the flow cell.

Nanopore technology can be applied across all DNA sequencing techniques.
- Whole genome sequencing: Accurately characterise all genomic variants and generate complete genome assemblies.
- Targeted sequencing: Sequence long targeted regions and expand on the limitations of traditional targeted sequencing approaches. Use amplification-free strategies to preserve base modifications.
- Metagenomic sequencing: More accurately identify microbes and their abundance with real-time results.
- Epigenetics: dentify base modifications alongside DNA sequence using direct sequencing approaches.

## ONTViSc
eresearchqut/ontvisc is a Nextflow-based bioinformatics pipeline designed to help diagnostics of viruses and viroid pathogens for biosecurity. It takes fastq.gz files generated from either amplicon or whole-genome sequencing using Oxford Nanopore Technologies as input.  

The pipeline can either: 1) perform a direct search on the sequenced reads, 2) generate clusters, 3) assemble the reads to generate longer contigs or 4) directly map reads to a known reference.  

The reads can optionally be filtered from a plant host before performing downstream analysis.  

The ONTViSc pipeline is written in Nextflow.  

## Requirements
If you want to familiarise yourself with Nextflow, please review the material covered in the workshop [Introduction to Nextflow](https://eresearchqut.atlassian.net/wiki/spaces/EG/pages/2261090311/2024-S2+eResearch+-+Session+3+Introduction+to+Nextflow)
A generic [user guide](https://mantczakaus.github.io/ontvisc_guide) on how to set up and execute OntViSc is also available. It covers how to run Nextflow from tower, which will not be covered in this workshop.
Nextflow can be used on any POSIX compatible system (Linux, OS X, etc). It requires Bash 3.2 (or later) and Java 11 (or later, up to 21) to be installed.
1. Log in to Lyra ```ssh [username]@lyra.qut.edu.au```
2. Start an interactive session: ```qsub -I -S /bin/bash -l walltime=10:00:00 -l select=1:ncpus=1:mem=4gb```
3. Load java: ```module load java```
4. If you haven't done so before, install [Nextflow](https://www.nextflow.io/docs/latest/getstarted.html#installation)
    - Download the executable package by copying and pasting either one of the following commands in your terminal window: ```curl -s https://get.nextflow.io | bash```
     This will create the nextflow main executable file in the current directory.
    - Make the binary executable on your system by running chmod +x nextflow. Optionally, move the nextflow file to a directory accessible by your $PATH variable (this is only required to avoid remembering and typing the full path to nextflow each time you need to run it): ```mv nextflow $HOME/bin```
PLease note: If you have installed Nextflow before on the HPC then you will have to run: ```nextflow self-update```
5. Once you have installed Nextflow on Lyra, there are some settings that should be applied to your $HOME/.nextflow/config to take advantage of the HPC environment at QUT.
To create a suitable config file for use on the QUT HPC, copy and paste the following text into your Linux command line and hit ‘enter’. This will make the necessary changes to your local account so that Nextflow can run correctly:
6. If you haven't done so before, install [Singularity](https://docs.sylabs.io/guides/3.0/user-guide/quick_start.html#quick-installation-steps).
```
[[ -d $HOME/.nextflow ]] || mkdir -p $HOME/.nextflow

cat <<EOF > $HOME/.nextflow/config
singularity {
    cacheDir = '$HOME/.nextflow/NXF_SINGULARITY_CACHEDIR'
    autoMounts = true
}
conda {
    cacheDir = '$HOME/.nextflow/NXF_CONDA_CACHEDIR'
}
process {
  executor = 'pbspro'
  scratch = false
  cleanup = false
}
includeConfig '/work/datasets/reference/nextflow/qutgenome.config'
EOF
```
## Installing the required indexes and references
Depending on the pipeline analysis mode you are interested to run, you will need to install some databases and references.

| Mode | Index | Description |
| --- | --- | --- |
| --host_filtering | --host_fasta | path to host fasta file to use for read filtering|
| --blast_vs_ref | --reference | path to viral reference sequence fasta file to perform homology search on reads (read_classification), clusters (clustering) or contigs (de novo) |
| --blast_mode localdb | --blastn_db | path to [`viral blast database`](https://zenodo.org/records/10117282) to perform homology search on reads (read_classification), clusters (clustering) or contigs (de novo)|
| --blast_mode ncbi | --blastn_db | path to NCBI nt database, taxdb.btd and taxdb.bti to perform homology search on reads (read_classification), clusters (clustering) or contigs (de novo)|
| --read_classification --kraken2 | --krkdb | path to kraken index folder e.g. PlusPFP|
| --read_classification --kaiju | --kaiju_dbname | path to kaiju_db_*.fmi |
|                           | --kaiju_nodes | path to nodes.dmp |
|                           | --kaiju_names | path to names.dmp |
| --map2ref | --reference | path to viral reference sequence fasta file to perform alignment |

- If you have access to a host genome reference or sequences and want to filter your reads against it/them before running your analysis, specify the `--host_filtering` parameter and provide the path to the host fasta file with `--host_fasta /path/to/host/fasta/file`.

- The homology searches is set by default against the public NCBI NT database in the nextflow.config file (`--blast_mode ncbi`)
Download a local copy of the NCBI NT database, following the detailed steps available at https://www.ncbi.nlm.nih.gov/books/NBK569850/. Create a folder where you will store your NCBI databases. It is good practice to include the date of download. For instance:
  ```
  mkdir blastDB/20230930
  ```
  You will need to use a current update_blastdb.pl script from the blast+  version used with the pipeline (ie 2.13.0).
  For example:
  ```
  perl update_blastdb.pl --decompress nt
  perl update_blastdb.pl taxdb
  tar -xzf taxdb.tar.gz
  ```

  Make sure the taxdb.btd and the taxdb.bti files are present in the same directory as your blast databases.
  Specify the path of your local NCBI blast nt directories in the nextflow.config file.
  For instance:
  ```
  params {
    --blastn_db = '/work/hia_mt18005_db/blastDB/20230930/nt'
  }
  ```
  
-  If you want to run homology searches against a viral database instead, you will need to download it [`here`](https://zenodo.org/records/10183620) by using the following steps:  
```
wget https://zenodo.org/records/10183620/files/VirDB_20230913.tar.gz?download=1
tar -xf VirDB_20230913.fasta.tar.gz
```
Specify the ``--blast_mode localdb`` parameter and provide the path to the database by specifying ``--blastn_db /path/to/viral/db``.

  
- To run nucleotide taxonomic classification of reads using Kraken2, download the pre-built index relevant to your data and provided by [`Kraken2`](https://benlangmead.github.io/aws-indexes/k2) (for example, PlusPFP can be chosen for searching viruses in plant samples).  

- To run protein taxonomic classification using Kaiju, download the pre-built index relevant to your data. Indexes are listed on the README page of [`Kaiju`](https://github.com/bioinformatics-centre/kaiju) (for example refseq, refseq_nr, refseq_ref, progenomes, viruses, nr, nr_euk or rvdb). After the download is finished, you should have 3 files: kaiju_db_*.fmi, nodes.dmp, and names.dmp, which are all needed to run Kaiju.
You will have to specify the path to each of these files (using the ``--kaiju_dbname``, the ``--kaiju_nodes`` and the ``--kaiju_names`` parameters respectively.

On Lyra, we have a copy of NCBI, Kaiju and Kraken predownloaded under:
- /scratch/datasets/blast_db/20240730/ (nt)
- /scratch/datasets/kaiju_databases/ (version kaiju_db_rvdb_2023-05-26)
- /scratch/datasets/kraken_databases/PlusPFP_10_2023

- If you want to align your reads to a reference genome (--map2ref) or blast against a reference (--blast_vs_ref), you will have to specify its path using `--reference`.

## Quality control (QC) of Oxford Nanopore data
The initial step of every sequencing project is the quality control step to assess the quality of your sequencing data. For this reason, we recommend you first run the **--qc-only** mode of the pipeline to perform a preliminary check of your data. 
You will be able to assess how successful your sequencing run was based on certain statistics (e.g. length and quality score distributions of your reads),
as well as identify potential problems with your input DNA/RNA, the sequencing run or the data itself.  
The quality control step will guide you in chosing the right filtering and trimming parameters (i.e. removal of those reads that are too short or too low in quality).
An increasing number of tools is available for sequence data QC and filtering, with different strength and applicaton cases. In this pipeline, we use **Nanoplot**. 
This tool gives a good overall overview of your data by creating several files in html format which any browser can open as a web-page.

**Exercise 1:**
Let's compare the Nanoplot statistic outputs from two ONT samples. The first one is MT001, a plant sample on which whole genome sequencing was performed. The second is ET300, an insect sample from which an amplicon of the Dengue virus was amplified and sequenced at very high depth. Pay attention to the mean read length, n50, mean quality and mean quality.

**MT001 statistics:**
<p align="left"><img src="images/MT001_summary.png" width="750"></p>

**ET300 statistics:**
<p align="left"><img src="images/ONT300_summary.png" width="750"></p>

You can see that overall, the data is of a higher quality for the MT001 amplicon sample.

If we now look at the read length versus average read quality plot:  
**MT001 plot:**  
<p align="left"><img src="images/MT001_nanoplot.png" width="1000"></p>  

**ET300 plot:** 
<p align="left"><img src="images/ONT300_nanoplot.png" width="1000"></p>

If we compare the plot profiles, you can see that the data distribution looks very different between the 2 samples.  
In MT001, most of the reads are short, so we will not perform a quality filtering step as most of the data would otherwise be filtered out.  
In ONT300, we recover a lot of reads that are 10k in length so we can confidently perform a quality filtering step based on length since the sample was an amplicon and was sequenced at very high depth.  

## Example of whole genome sequencing
**Sample MT001, MT002, MT010 and MT011** are samples that were derived using direct cDNA sequencing kit (SQK-DCS109) followed by whole genome sequencing using [Flongle](https://nanoporetech.com/products/sequence/flongle). Double-stranded (ds) cDNA was synthesised using random hexamers.

The samples originate from different plant commodities (citrus, prunus and miscanthus) and contain different virus types:

| Sample name | Host | Virus | Genome type | Dataset |
| --- | --- | ---  | --- | --- |
| MT001 | Citrus | CEVd, (Citrus endogenous pararetrovirus) | sscRNA | /work/hia_mt18005/raw_data/ONT_MinION_NZMPI/MT001_ONT.fastq.gz |
| MT002 | Prunus | PNRSV | ssRNA(+) tripartite | /work/hia_mt18005/raw_data/ONT_MinION_NZMPI/MT002_ONT.fastq.gz |
| MT010 | Miscanthus | MsiMV | ssRNA(+) | /work/hia_mt18005/raw_data/ONT_MinION_NZMPI/MT010_ONT.fastq.gz |
| MT011 | Citrus | CTV, CVd-VI | ssRNA(+), sscRNA| /work/hia_mt18005/raw_data/ONT_MinION_NZMPI/MT011_ONT.fastq.gz |

For these samples, we recommend performing first a direct read homology search using:
- megablast and the NCBI NT database
- direct taxonomic read classification using Kraken2 (nucleotide-based) and Kaiju (protein-based).  
This will provide a quick overview of whether samples are predicted to be infected with any viruses and/or viroids, and warrant further investigation.
We also recommend to filter host reads before performing the direct read searches.
```
# This command will:
# Check for the presence of adapters
# Filter reads against the reference host
# Perform a direct read homology search using megablast and the NCBI NT database.
# Perform a direct taxonomic read classification using Kraken2 and Kaiju.

nextflow run eresearchqut/ontvisc -resume -profile singularity \
                            --adapter_trimming \
                            --host_filtering \
                            --host_fasta /path/to/host/fasta/file \
                            --analysis_mode read_classification \
                            --kraken2 \
                            --krkdb /path/to/kraken2_db \
                            --kaiju \
                            --kaiju_dbname /path/to/kaiju/kaiju.fmi \
                            --kaiju_nodes /path/to/kaiju/nodes.dmp \
                            --kaiju_names /path/to/kaiju/names.dmp \
                            --megablast --blast_mode ncbi \
                            --blast_threads 8 \
                            --blastn_db /path/to/ncbi_blast_db/nt
```

After checking the results of the direct read approach, check if some viruses are present in high abundance.
You will see that we recover high coverage for MsiMV in sample MT010. For these cases, it is possible to try a de novo assembly or clustering approach to see if we can reconstruct their full genome.
To perform a denovo assembly approach on MT010 with the tool Canu, try the following command:
```
#!/bin/bash -l
#PBS -N ontvisc
#PBS -l select=1:ncpus=2:mem=8gb
#PBS -l walltime=6:00:00

cd $PBS_O_WORKDIR
module load java
NXF_OPTS='-Xms1g -Xmx4g'
nextflow run eresearchqut/ontvisc -resume profile singularity \
                                 --denovo_assembly \
                                 --canu \
                                 --canu_genome_size 0.01m \
                                 --canu_options 'useGrid=false maxInputCoverage=2000 minReadLength=200 minOverlapLength=100' \
                                 --blast_threads 8 \
                                 --blastn_db /path/to/ncbi_blast_db/nt
```
Running this command should enable the recovery of most of the MsiMV genome.
```
>tig00000001 len=9578 reads=523 class=contig suggestRepeat=no suggestBubble=no suggestCircular=no trim=0-9578
AATAAACTCGCAACCTTCGTGATAAAATCACTCCAGAGGCCGTCCGTCTAGTGGCTCGAAGCTAGTAAAA
GAACGTACTAAAGAGTTGGAGGTAAACCTCTCTCCTCTGCAGAAATAATACGTACTTACCACATTATACA
TGTATAACTAACGAGGTACAACCTCACCACAATAGGTAACACATATAATATTACTTTACTGCAAACAAGG
TTTCCTCGCATCAATGGTGCTGTTGCACCCCAAGTAAGGAGTGCATATTACGACTGACATCGCCAGCCGT
GTGACGCTCGGTATTCTCCTGAGCTTCTCCGACATTCCCATCAAGTCCGAACATACGGGTGTTTGAACCA
CGTACTGCTGCTGCTTTCATCTGCATATGGGCTTCCTTTGCACGCACTGGTGTCCTTGACGTCATTTCAT
AGAAATCAAAAGCATAGCGCGCAAGATTAAAGTCGGTTAAGTTTCGCTGAAGACCGTATCTTGGCATGTA
TCGTTCTGTTGCATTGCGATATTCAATATATGCTTCAGCTGCATCACTAAAATGATGCATTATTTGTCTA
AAGGTTGGAGATGCATTCTCTATTATTGGTTTCAAAGGGAAGGTTCTCTGTTCTTCTCCATCCATCATAG
TCCAAACCCCATTAATATTTGGTGAGCAACCGTTTTCTATGCACCAAACCATAAGTCCACTCATGACAAC
TGTCATCTGTGTGTCATTAAGTTCGTATTCCTTTAGAACGGCTGCATACCATCTGTCAAACTCAGCTCTC
GTGGCTCGTGTGTTTGATATGTCTTGTTGTTGTGGCTTGTATCCAAGTAGGAAGTCTAAATGTAAAATAT
TCTTCCCCTTTGCTTGCGGTAAACGCATTTTCTTGGACATTGCTTTAAGTTTAGGAACTGCCACTGTTCC
TGAAGTGCCAGCATCAACATCTTTGTCTTTGTTGCCTTGCTGGGCTTTGCTAGCCTCGGATGCTGCCCTA
CTTGCTGCCTCCTCCTCTTGCTTCTTTTTATTAGCAGCCTCTTGTCTTTGTTTTGCTGCTGCCTCTTCTT
CACTCTTCTTTTTCGCTTCAGCATTCCTCTGCGCTTGAGCTGCGGCTTCTCGTTGTCTCTGTGCCGCTGC
GGCATCCTCACTGGCCTTTCGCTGTGCTTCCGTTTGTTGTCCAGCATCGACAGAACCAGTGGCGGCCTGG
TGGATAACGTCCACTGCCTCATCCGAAACATAATCGGGCAAATCCATAATAAATCGTTTGAAGTATTCCT
CAATTTCGCTTGGTACTACTTCCTGCCCAGTGTAGAGATTTCTTAGTGCTGATTCTGCTATGTATGGCGC
GAGGCCCTCCTGCGCAAGCCCAGAGAATGGTTGCATCTCAAGTATCCAAGCATAGAATTTCCTTATTTCC
TTAAGTAAATCAGGGTAGCCCCACGCTTCAACCATCGACGCACATATCGCCTCTAATCTGTATTGTGGTA
GCACCGATCTATCCCATTCAAGGATTGCAACTATTCTCTCTCTTTCCAGTTTTGGAATATACATACCGTC
TTGTAAGACACCTCTGGTTGACATGAACCACAAATCCGACTTTTCCCTCGTTCTGTTCACAAAGTCAAAA
TTCAAGCCAAGGTTTGAAAACGAAGTTGAAAAATTGTCAAGTAAGTATTCGTAGTCTGGATGCACTGCAA
GAAGCAAATCGTCCCCATTTGCAAACATTTTACAAACATTGTCAATCATGTCCTCGTGGATCCCGTTTGA
TATCATTGAATAGTTAAAAGCAAGTATAACCATGAGCGTGTTATCAACGACTGTTGATGGTTGTCCACTA
TTGTTACCTTTGAATTTCTTAATTACAGATCCATCTGGCGTCGCTATCGGTGTGTACACTATTTCAGTGT
ATAAGTTTTTAAGCATTCTTTCTCCTAACTCCCACGGTTCCATGAATGATAACCTGATGTCCAAAATGGC
GTTTAAGAGATAGGGCGTTAAAGAACTATCGAATTGGGATCCATCGGCGTCGCAATAAATCCAGCCCTCT
GGCAATGCCCTCAATAGACGATCCCATCCTCCATAGAACTTTGTTATTCCAACTGTCCATGGTCCTTCTA
AATGGTGCGAGTAGAATTGATTATTGAAGTCATCAACACACACTTTGCCACCAAGTAATGTTTCCAACGG
GGCGGCCGTGAACGTTCTAGTTTTATTAGCTTCTGTCTTTTCGATTGGTCGAATCTCAGCTTTGAGTGAA
CCATTCCAAATGCCAAGTTTCCCGTAATATAAACGTTCGCACGATTGTTTGATTATTTCAGCCTTATCTT
CGTTTGTAAAATCCTTGAAATAATCTTTCTTTTTACCTGTATATAACGCTCCGACTGCTGCGTTTAAATT
TAGTGACTTGAATATTTCCTCCTCTTCTGTAACATAATTACATTGTTGCATACCAACATTGTGTAAAATA
CGCTTGACTAATTGAACTGCTCTCTTGAAAACATTATCGTCAACTTCACCAATAAACGTTGGCTTCGAGT
ATTTCATGAGATCTTTGGTGAAAGCTGCCTTATTCAGCCTGCTCTTATCATATTTGCCCATGAGAGGTTT
AAAGTACGCGTTTGCTTCTTCGTGTGTTGATAAGTACAAAGAGAAATGTGGACAAGGACCCTTCACAACG
TGTTTAGTCACCAGCTGACCAGGGCATTTCGCGACTACTTGTAGGTTTTGCTCAACATGTTGCGTTAGCC
ACGTATTATGAGTCATGCACTGCTCTGTCACGTCAAAAGTTAAATCTTCAATCAATTTTGCAGTTTTAAA
TAATCCCGTTGGCTCTGAATCAACTAAATTGAGACCATTCCATGAAATTAAATTCGGGTTATAATGCCAT
CCTTTCTCCCATGATTGAGCTCCATTCAACTCACATATATATGATTCGAAATCTGGTGGCACTGCAACGA
AGAAGTTTGTGTTACCGTTCACTGTTGTTAAACTGTGGATACCGACAATGGACTTGTCTCTGACATTTAC
TAATGGTAACCCACATTGCCCTTCAGTCGTAGAGATCCAGTGTTTCCAAAAGGTACTGTTTCCTTTAGGT
GCCGTCACACTACTTTCTGATACAATACAAGAACTGTAATTTTGTTGGAAATTCACTCCAACTAAGCAGA
CTCTATCATTCCTATTTGGCTCTGTGAATTTTAATTTGCGTGGGAAAGGTGGAAAGTCCTTTGGCAGTTG
CATTATTACCATATCCCTCCCAGAGATCGGATGTAGCTTAACCTCTACTGAATTTCTAATTTTGAACAAA
CCTCGTGATGATCTTAAAGTTATTTCACCATTATTGTATCTGAACAAATGCGCTGGCACAATTAAGAATG
ATCCGTATCCGATTGCGAATACACACTTTCGTGTGTCGTTAGAGTGGTTTTCTATCATACATATCTGATT
CGAAATCGGTGTGTAGTCACCTAACCCCAACATCATAGATTTTGCTTCATGTGAAACTCCTTCTTCGTGC
CTGTTGGGCACCTGACTTGCTGGAACAATCTGTGCTTTGCCTGTCTGGCGTAATGTCCCTTCATACTCAG
GGAATCCTGCTATGTTATTGTTTGAAACAACACGTAATGGGACATGTGGAGTTAGGTCTACTCTCAAGGC
ATTCTCTGAACCATTTTGTATAAAGAATGCTCGTATGCCTGGGTTCGCATAGATATGTTGTCGATCTAAT
CTATCATCATCTAGTGCAGCTTCTCTAATTTCGCTGAAATGCTCTTGCACTAGATTAATGTCTGCGTGTA
TTTGTTCATCTAAGGTGGCTCCTGTTAACGGATCGACAAATCGAATAAGATTATAATCTTGTGGATCAAA
TCCATACATCATGTGGAATTTGTGCTGCTTAACTCCGAGTCCTACCTTTGTTCCACCTTTCTTTTCCTTT
TTGATGTATGCTGAGCCAAAGTTCTCTTCTATCGTATCCTTGGAACCCGTGACATCGTAGGCAAATTTAT
TATCGCGTGCCTTTTGGAATTTGAGCTTCTGTCTGCTCCGCTTATTCTTTGCCTGATGCGTAACTTTTGT
CTGCGACCATTTTGCAAACAGAAACCATAACATAGTTAAACCTCCTGTGAAAACACCTGCTGCTATCATT
AAATCTCTTTGAATTAATGGTGCATTCCACCGCCCTTCTAATTCAAGACATTGTGCAGTCGCATCCAAAC
TTTGGTGTATGACAGTGTTTAGAGCACCAATTTCCATTAGATCTTCCGCTTTTGTAAACTGCGCTCCTGT
TCCTTGGAATTCCATCAGCTGGTCTCGAACACGCACAAGTTTTTCGATGTTGCCTTTGGAGTGATCCTTC
ATGTAGCGTGAACTTAACAAGCTGACTATGCCGCTAAGAGAGAAGGCGTGAGATGATGAGGGGTTAGCAG
AGATTGAATTGTAATGATTTCTCTTTGCATGCTCTTCTGCTATTAAGGTATTTATAATCGCAATTGTTCG
TGGCAACGAAAAAGGATCAGTTCGTAGTGTATATGCAACCTTTCCTGCACATGCGCTTGATAAGCGGCCG
TAACAGCTAGTTTGACTATATTGCAAGATAATGTCGTACAATTCTGCGTATAGCTTATCTGGAACACCTC
GTACATAATATGGTATTTTAATATAATCGCTCAATTCCAAATCGCACCCAATCTTATTATAATCACCAAC
TGTCAACCAATTATGTACATTAGTATAAGGTATGGCGTTTGGTCTCAACATGACTACAGAATCCCTAAGT
TTATATCTTTTAAGTTTTTCGTGGATTTGCGGGTGCATTGAACCATCAAACTTAACAAGCTCACACATAA
TAAAAGGTGATAACTCGAATTGCATCATTGTTCGCGCTTGTTTCACTGTGCATTTTGCCAGGTGTGTTGT
TGAAACATTGTGTGTTATCACATTTAGCCCATAAGCAAAACAAAGAAATGCCGCTTCTGTTGCTATCATG
GCGGGTATTTCTTGTAGACCCTTCATCGTCACACCCGAACGGATCACACTGCCGGGTTTTGTTCTTCCAA
CTCGGCCAAGTCGTTGTATACGCTCACCGTACGAGACACTAACCTTTTTGTAAATAACTGCTCTGTTATC
GACGTCTAGTTCTGCTGTAACTTTAAGTCCAAAATCGACAACGACGTCGACATCTAATGTGACACCATTT
TCGATAATATTTGTGGCGACAACAAAACACTTCCTCATTGGAGTTCCATTAGTGTGTATACCATTCGTGT
TTTGCTTCATTGTTCTTCCATCAACTTTAATTACTGAGTAGTGTTGTTCAGTTAATGCTCTTGAGAGTGT
GTCAACCTCGTTGTAGCTAGCAACATACACGAGAATATTGTTTCCATATTTCGTTGCATCTGCTTTTGAT
CCAGTGCCTAGCTCAAGAACGAACTGATGTTGAGTTAGATTCTCACATATATGAATGTCCACCGGATGCT
GAGTTGAAAACTCGCACTCTCTCCCAGGTGGTGTTGCAGAAACTTTAATAACTTTCCCTTTGAATTCGTA
TTTCTTGAGCAAGCAGTAAAAAGCCATCGCTGGAGCTTCCATGATATGGCACTCATCAAATATTATAAAA
TCATAGTTAAAAAGTTTGTCTGGATTATTGGCATACATATGCAGAGCAAATCCTGATGTCATTATTGTAA
TTGGGGTTGAGCCAAATGAACTAAGTCCACGCATCTGCAATGTTGGACTTACATTGAAAGGGGCGCCCTG
TAGTTGCCGACATACATTCTCTGCTAAAGGTCTCGTCGGTTCTAGCAGAAGAACTTTGCCACTCTGGCAT
AAATGGTAGGGTAAGCCCGTTGACTTTCCTGATCCAACAGCTCCTCTCAACAGGAACTCAGTTTCCTGGT
GTGTATGTGCTATATCGACACTTACTGTTGCTGCATTTGCTCTTGTGAATTCAATGAATTTTCCCCGAGA
CGGTAATGTGGTACTGTTCTGTTGTTTTCAAGTTGCTTCGTCCACCATGCTTCGAAAGTTACGTCATTGC
TGAATGTGTCTGCGGGTAGGTCTTGATCCGTATCAAAATCCACGGTTAATTCTTTTTCAGTACTTAAGAT
GAGTGGATCATCGAGTTCTACGCTTTGGTGCTTGACGGGTGTGGTGAAAATTGATGTAAAATCAAGTCTA
GGAAATGTTGAGTTCTGTTCGAATGTGTTAATAACCGTCCTCATTTTATTCAAAACCTTGTACACGGCAT
CACTTTTAACAGGGTCAAAAATCATCGTGATTAATGTGCCCAATGCCATCGCTTGTTCAAGATTTGTCTC
CAGGTGTGATTTGCCTTCGTGGATGACACCTGTTCCCGTAAGCTCTATCGTAGCTTCGATTAATCGTGGG
TGGTTTTCTCTGATATATTCAATAAAATCCTCACTTGTGAGATTTTTATCGTGTACTTTAAGCAACTTTG
CATGGATTGCACGCACTTCGTTGATCTCTAATTCGTATTCATCCTCACGCGCTTGTTTTTGCAATCTTTT
GTAGTCTTGCATTGTAGTTATTATAGTGTTAGCAATTGTGGACAATAAGCTCAAAATAAGAAAGATATGT
ATGAGCCTAAATATGTCAGGAACAAACCAATACATAGTGTTTATAGCTTTAACTCTAAGTTTCTCGCATT
TATCACATAAAACGCGGTGGAGTTTGGTTGAAATAGAGCTGACTCGACTTCGACTTTTCTGCACTAAATT
TGATGCCAGATGCGTAACTGATATATTGTACACAGCGTCTAAATCTACGCTTTTTCTCAGGGTTAAAGAT
GGTTTGTAGTACTTCTTGTGTCTGAACACATACCAAATTTGTTGCAATCGTGCACAATATGTTAAATCTG
CCCATGCCTGATTTAATGTCTCTACGTAACTTTTTTCCATTAGAACATACAGTCTTTCGTCGTACAATGC
ATAGCCTTGCTCAATTAGCTCTTTGTTCATCTCTGACCTTGTTGTCATAGCTTCTAAGTGTGACCATAGC
AGGCGCTTTGCTGGGCTAACATGACTTATTCCTGATACCGCAAAACGCAATTGTGTTGAAGCCTTTTCCA
GTATTTGCATTTGCTGCACTAGCAATTCCGCTTGTGATGTCTCTTTTGCTAAGGCCTCAAGTTGAGCAAA
AATTGCTGCTACTCCTTGATTTTTAACAATCCAGTACGTCATTGCTTGCTCAATGTAATTGTTGTTGTAC
AATGATATAAGGATAGTTGGCGATGCGATCGCCATCATCAACAGAAACGGTTCCTCCTCAATTATCTCTT
TGATCTTCAAAGGTTTAAACATATTCTTTCCAAGAAACTTGAGAAGACTACTAAAAGTTCTATGGGTCAA
AGTTCCGCCTACCACGTAATCGCGCATTTCACTCTCCATCGACTCATACTGCATTTTAATTAATTGACCA
ATTGTGTTTGCTTTCAGGATGTGGAATCCGACACTTAAGGAACCATATGAATCAATTACATGCATCGTTT
TGTGCTCATGATCAACAAGAATTGGTGGTAATTCTGCGTTCTTTATCTCAGGAAACATAACTGATAGTGC
GTAACACGCAGTTGCTACGTCTTTGAGCTTTGGCCATTTTCCAAGTCTCTCAATTAATTCATCACGCAGA
AATTTTGTGTAGTCTTTGGCTGACTCCTCGTTAACGTTAATCATTGCTGCTAAAAATATGTTGATGTAGC
AATATCCATCCTTTGCAATATACATCATTGGTGGATCTGTATTGGGAAGGTCCACAATTTTGGGTCAACT
GAATTGCCTATGGTGATGTGCCCTTTCGTTGGTGGTATGATTTCCGAATAAGCGGGTTGTCCGAATTCCG
TCGTCACACAACAACAAGGGTATACGTATGCATTGTCTACTTTACTCAGACAAGCTTTGTTGAGCGGTTG
TCGTGTAATCGACACTCCTGTGAAGGATTTACGTATACGATCGAAGTCGAGCGGAATGACTAATTTACCA
ATCGACAATTTCCGTTGCCCATTTGGATTGAGTCGTGTTATATGTTTCCCATACGCATCTGACGGGTCAA
CAGCTTCAAAATAATTTGTAAAGAATCGCTTAGCGTGGTACTCACGTTTTCCCCATAAGAAATTACCATT
AACATCTAATTGATTATCACACATCAGTGCTGTGTTTATGGTGCTCTTCGGTGATATTTTATTCCTAAAT
GTTGCTAGAGTGTCTTCTTTCGCTGATTCCTTACGATTCTTGTACCATCGTGAAATCTCTAACAGGGCTG
CGCTTGCTTTCACCGCGTCCTCAACCGTCAAAGTGTTTATCTTCAATAATGCTTCATTTACATCCATTAT
TTGTTTCGATTGATTGTCAGTGTAGTGACCAATCGTTTCAGCAACCTGCCATGAAATCGGTAGATGTTCG
AATTTTGGGTTACACCTCTGTTTTATAAAATGCAACGTTCGTTTCAACTTCTGGTCTTCAGCTAAGTAGT
CACTCTGCTGTTTTTCAACGCGCACTATGTTCTTATACAGTTTATCTCCAAATTCATCATCTGCTAGTTC
AAGATCATCGATGTTACAGTGTTTACATGTAATCTCAAACGTTGAATGAAACAGAATTTCAAGTAACGCC
ATTCGTTTTCCACACTGCTCTAAATTAACAGTTGGTGTATGCTCAGTATGGGTTGTTGATATGTTTCGAT
TCGCAATGTATGCATCAGTGTATCCCTTCCAGAATGCATTAGCCTGATGATCGCTAAAGTGTTCAATCTC
ATTGATGAAATTCACTTCATGTATGCTATTGATTAGCTGTCCATTGGCTTTTCCGCGAATTATAAATAAC
TCACCATTTTGCACGTAGGTTATGCCACTGTATCCTTTACAAATTTTTGGCTCCATCTTACTGGACACTG
TGGAATAGCATTTAGACACTGCCACGATGTTGTCAATCCAGTGAGCACTTAGATTAACATCTTTACGCTT
GAATTGTCCCATTTCATGGCGCGTTCGGCAATGAAGGTATGTTTTCTTATTCCTGGTGAGTAATGAAATT
TTTGTGACGTTACGCCTCTTACTTCCGACTAACTCCACTTTCAACCCTGTGGTCAGGCCTATCTCAAGTG
TGTCTCTGATCAACTTATTGATTGAACTTGTGACACTTTGCTTGGGAGTGTCACTCTTCTGCTTATGTAA
TCGTGCGTATACCAAGTGATCTGTCGGCCTATTGTCTGACTTCTCAACCCATGTACGTCTGAGAGGTGTT
GGTATGGCTATGGGAGTTTCTTCAACCTCCTGCACAAACGTCTTCCTTAAGATTTTTGCATTATGCGCTT
CTGCTCTCTCACGAACATACTCTTGATGTTTCGCGGCAAAATGTTCCATCACCTTCTTAACGTCGCGCGG
ATTGTCCAAGTCTGGCCTCCATTTGTAGGAAACAGTGGTCCACGCTCCTGCCATTACGTCTCCAAAGCTT
CGCTTAGGTTTACAGAGAGTTCTTTCAAGGCACCTTGCGCTTGAACCGTGAACTTCTATCTGAGTTTGAA
CGTGATTGTGCTTAATCGTGTTCGGTTGTGTAGCAATACGTAACTGAACGAAGTACAA
```
This contig shows 99% coverage to OL312763.1.
<p align="left"><img src="images/blast_results.png" width="750"></p>


You can also test the Flye assembler. You will want to specify the paramater --meta as this sample contains both host and viral sequences which are present at different concentrations.
```
#!/bin/bash -l
#PBS -N ontvisc
#PBS -l select=1:ncpus=2:mem=8gb
#PBS -l walltime=12:00:00

cd $PBS_O_WORKDIR
module load java
NXF_OPTS='-Xms1g -Xmx4g'
nextflow run eresearchqut/ontvisc -resume profile singularity \
                                  --denovo_assembly \
                                  --flye \
                                  --flye_options '--genome-size 0.01m --meta' \
                                  --blast_threads 8 \
                                  --blastn_db /path/to/ncbi_blast_db/nt
                                                 
```
You can now compare the contigs obtained with Flye and Canu.

## Example of short amplicon product

**Sample ONT009** is a rubus sample which is infected with Rubus yellow net virus. The target is a short amplicon which was derived using degenerate primers that enable to distinguish between endogenous and exogenous RYNVs. The product is ~100 bp and fails Sanger sequencing. 

| Primer Name | Primer type | Primer sequence (5’-3’) |
| --- | --- | --- |
| RdRp-F | Forward | 5’-GGNGARTTYGGNACNTTYTTYTTY-3’ |
| RdRp-R | Reverse | 5’-NCKCCANCCRCARAANARNGG-3’ |


In this scenario, because the amplicon is very short, we recommend using the rattle-based clustering approach as the de novo assembly approach generally fails to recover products which are < 1000 bp. We also recommend using the --adapter_trimming option to make sure no residual adapters are present at the start and end of the sequences.

```
nextflow run eresearchqut/ontvisc -resume profile singularity \
                                  --adapter_trimming \
                                  --analysis_mode clustering \
                                  --rattle_clustering_options '--lower-length 30 --upper-length 120' \
                                  --blast_threads 8 \
                                  --blastn_db /path/to/host/fasta/file
```

## Example of amplicon data derived using 5' and 3' RACE 
For **sample MT483**, 5'and 3' RACE sequencing reactions were derived using a ligation method to amplify overlapping products which cover the full length of a novel genome identified using sRNASeq. The genome size is predicted to be ~7000 bp. Guided by the sequences recovered using sRNASeq, specific primers were used in each RACE which are ~5000 bp products. 
For this example, we are analysing the 5' and 3' RACE together, so you will place all the reads into a single folder. 
You want to run porechop_abi so it detects and removes any remaining adapters so we select --adapter_trimming. Here porechop will search specifically for the RAPID adaper provided in the adapters.txt file. 
Using the --final_primer_check option, a final primer check will be performed after the de novo assembly step to check for the presence of any residual universal RACE primers at the end of the assembled contigs.
You can either blast against NCBI or the predicted nucleotide sequence of the viral genome.

```
nextflow run eresearchqut/ontvisc -resume -profile singularity \
                                 --adapter_trimming \
                                 --porechop_custom_primers --porechop_options '-ddb '
                                 --analysis_mode denovo_assembly \
                                 --canu \
                                 --canu_options 'useGrid=false' \
                                 --canu_genome_size 0.01m \
                                 --final_primer_check \
                                 --blastn_db /work/hia_mt18005/databases/blastDB/20230606/nt
```
Search for the most abundant contig in the MT483_canu_assembly_filtered.fa file under the results/MT483/assembly folder. 
```
>tig00000003 reads=151 class=contig suggestRepeat=no suggestBubble=no suggestCircular=no
GGCCACGCGTCGACTAGTACTTTTTTTTTTTTTTGGAAACTTTATTCAAACCAAAAGCTATAGTAGTATTAAGTTATACCAGCATTAATATTCGGGAGCCTCAATGGCAGGCAGAGTTGGACGTGGAAGGCGTCCTTGGGTTACACCTGCATAGTTGGAGACAGTGTGAAGAGTCTGTTTGGCCGCCTCATGGAGGGATGCCATCTTGTTTGTGCCTTTGGCCAATAGC
TCATTTTGGGTGGGTTGGCGAACCAACCCCTCTTTTGGTTCAAGGGCTGCAGGGTTGGTAACAGCGTCAAAGAAATCGAAAGCGGCGAAGGCTTCAGATTTGCGGTAGTTCCATTTGGCCCAGCCCGCGGGTGGTGATTGCCTCTCAATAAGCAGGTTCCACACTATCTTTGCATAGTATGAGCAGAATTTGCGCAGTGTGGTGTGTTTCTTGGTGATGCGGGCAAGAG
TGTCATATGATACTGAGAGGTCGGAGATTGAGCCCTCAAACTCAGTTTGGTCTGAGGAGCCATTGTCTGCGCAGTAGATGGCGAGGCTCCAAGCTGCGAGCGCCACTTGGTCAGCGGGGACGCCGTTTTCAACCCATTCTTTAGCGATGGCCTTCACTGTCTCAGGTGATGCAACAGAGTTGGAGTGTGTTTTGTAAGTTATCCTCTTCAAATCTGCTAGAGATGGACT
AATGGTAGGGTCTGAGAGGTCGGAGAGAATTACAGTTGAAGCAGTAGCTGGATCTGAGGTGTCCATAACTCTATTCAGAATTGATTGGAAACTTAACGGCGTGTAGAGCCTCTCTGTTGCTGAGTATGTTGGCAAGTTCCTGAGTGACTGGGCAACCATGTGCTGTGACTCGCTCCCCTGTGATGATGATGGTACAGGGTTCATAGGGTCTGGGAAGAGCAAGTAGTAC
AAGTATGCCCAGGGTTATTCCTAGTAGTAGCTTTATGTAGAAGGAAAATGACAAGTGGAAGAATGCAAACAAGGATGGCGGGGAGGTGTGCTCGTTGTGCATGAGAGGGGAAGAAGGTACCTTGACCTTTGGGTGGTAAGTAGTTAATTCTCTTGGTGCCGTCGATGTATGAGCCACCGAAAGGAAGACTGTGGATGTTGTCACCAGTGTGGGGTAGAGTGTTGCGGGT
GCAAGTGAAGATGGCTAGAACTAATGCAAAACCTAGTAGAAATGGTGTCAATTCCCTAATGTGGTTAGGCGGTGGGGTGAGAGGCATTTGATGATGAGTTTGTTGAGGGAGCGGGTGCAGGCGATGTAGTACAAATGGGTGGGTTTGGGCTCGAGAGTGATGACGGTGACAGTGTCAAATTGGAGTCCTAGAGCCTGGCATGGTCGGAGATAGTCAGCTTGGTGGGAGT
TGAGGAGGCGGATGATTTCAGGCTCCCAAGCTATGATGGTACCAATGGGTTCACCTTGGAAGATATTTTCAAATGTTAAAAAACCCGCTTTGGTTGAATGGATGTCAAATGTGAGTGACCTTAGATGGTTACAGATGGGTTCAGGCACACGGTGGGTGAAGTTGCTGGTAAAGTGTGGATACGGTTCGGTGATGTTGTGTTGGATAGGGTCAGCGAAACAGAAGTCAGC
TGCTTGGTAGTCAGTGGCTAGTGGGTATTCGTCAAGGAGGGTTGTGGTGCCTGGTCTTTCACTCTTCCAGGGTAGGATGTGAGTGCCAGTAAGGGAGGGGGGGTCGGGAGTGCCACAGGTGAAGGCTCTAATGGTGTGATGGATGGTTAGGAGTTTTCTGATGATGGTGGTCTTGCCAGCGCCTGCTACCAGGTGGATTATGATGGGTTTGGAGAGGGGACGGGATGTG
CGTGTTAGTTGCAAACGGGTTAATTCGGTAATGACTAAGTTGTCCATTTCAAACTAAGATCAACTTAAACCCTCCAATAGGTGCCCTGTTCCTATCTTGTGAATCTTTCTGATGAGTGTTTGGTGCATGCCACACTCCTTCTCTGAAAGTACTTCTTGCAATTTGTCCCCAAGTTGGTAAGCAAATTTGGCATCGAGGGCATAACTGGTAGCTACTTCCTTGGTTTTGC
CAGTTTTAATGGCTAGTTCAAGGCTGGCCAAAATTTTGGTAGGGTTCTTGAAGATGCCCAGTCTGCTGATAGTCCACCCGCAGAAGTCGGCCCAGTCGCCCGGTTTTTGTTGGTACGTGACTGGTTTGGACGTGAGATTCAGTTTATTCTGGATCAGTTGGAAGCTTTTCTTTTCAATTGCAGGATGGTCTTGGGCGGAATCATCACCAGCATAAAGTTGGGCCACATT
GTCTGGCACATGGAACCTGGTGTGGTGGTAGGCAATGGAACATTCAGTGTTAGCATCGAAAGTGGGGCCTTCACCTGTAAGTCTCATTATGGCCAGGGTGCCAAGGAAGATTGTGGCATGTGTTTTAAGGTGCACATATCCATCAATGATGTCAGCTGGAATGTTGAAGTGTTTGGCCTTAAGTACCTCAAACTGGAGCATGGCACCATCTTGTGATTGGTCAAAGGCT
GTGAAGTCATTAGAGTATGCAGTTCTTGAAAAGTTCCAGTTTTCGAGCACGAATTGGTTGAGGTCTTCAGGAGTCTTTTCACAGTTGATGAAGATGTTCTTGGGTTGGAATACTGCTCTCATTTTCCTCAGGTAGCGGGCCATTGTTCCGTATAGCATTACAGTTTCCTGCATGAAGGCTGCTATAGTTTGGCCGGGCTTTACTTTCAGAGCACCCAATTTCTCAACTT
TTTTAACCCACTGTGACTTTAGGAAGAGTGCAATTTTCTCCTTGTCAAAATCAGGAGATTGTCGTTGAGCGGCATTGACCAGCATGTTGATGGGCTTCTCCAAGTATTTGTTGCGGACTTCAGTGGCAGACAGCTCCCACAGTCGGGGATCAAAAGGCACAGGTTGGTCAGGCAAATGCATGGCTCTCTTGTAGTTGAGGAAGAGCAAGTCACCAATGTCTTTTTTTAA
TGCCAGCTCTTTCAGGTTTTGCTCCGGTGATGCAGTCTTGATGCGAGCCTCAATGGTAGCCCAGAGAAGAGGCTCATCCTTAGCCTGCTGGTGTTGGAAGATTTGCACTAATTGGTCATCAGTTTGTATACAGTTGGTGTGACCGTGCCTGCTGTCAAAGATCTCTCGGTCATACTTTTCCTTTTGCCCCTCAATGTAGTCCTCAAGCTGTATGTCCTGATGCTCAACA
GGGAAGTGTGTGGGGATTTTGGGCTCTGGAACTTGAGGTTCTAGGCTGGTAATTTTTGCCATGGCCTCCTCTCTGGCAGTGTCAAGGAAAGTTTTGAGGTATGGAGTGGCGTCCAACTTGGTCCAGTACTCCTGAGAGTTAGGTCCAGTGTTGATAAAGTGAATCATTTCCACAGATCTGGATAAAGCTGTGTACATCACCTCATCGGAGCAGAATGGGGTGTTGGAAT
CGAGCAGGATTTGAACTTTGGGAGCTGTGAGTCCCTGGCAGCCGGCATAAGTGTATCCTTTGTTGCCCAACTCAGTGATGGCTGTCTTTTTGATCATGGAGGGGGTAAGAATAGGAATTCCTTTAAGCACTTGACTAGAGATTGTGATGGAACACGGTCCCTCAGTCTCGGAGTAGACGCCCAGCTTATTTGCAATGTCTCTCCTGTTCCTGTGTGTGCAGTTCAGATA
GTACCTTGAGTACTGGTCAAAGTGGGAGGGGCCTGGGTCGAGCTTGGAGATGAGGGCCATGTCATTATCCTCATGGTATGAACTTTGTCTGGGGTCCCCGGTTAGAATGGCAAGCTCAACGGCCGGATGCGAGAATATGTATGCCTCAATGAAACCAGCTGGCATTTTCCCATAATCATCAAAAATGACTATTTGTCCTGAACCAGCTATTAGTGCCTTTTCAAAAGTT
TTGATCAGCAGCCTGTTGATGTTGGGAAGTTTTCCCTCCCAATCAAGCCTGAGTTCATTGGTGGGGACGACAATTGTAGCTGGGCAATTGTCTTGTTCAAATGTTTTGAGCCAATTTTGGAGGAAACGGGACTTGCCAGAGCCTCCAGCACCGTGAATCACGGACATCTGCACTGTCATATCAGGGCTCCTTTCACACATAAGGGAGAACTTTTGCTTCCATTCCAAAG
GTTGGGTTTTGCCGAGCAGACCCGTTCTGCAATTCTTCAAGTCAGAAGCATAGGCTGCTGCCCGAGTACCTGAAAGTAGAACTGGTGTGGGAAAGCGGCTGATGGACTTGAGGGCTAGAATGAGATTTGGGTCCAAGTCTTCGAGCCCCTCAGAGCGGGGTAGCTTGTGTTGTCTTGTGATTGGGATTATCAGATTAGTTCTCTTGTGGTCTGAGTACTGCATCTGATT
GCCTTTGAATCCAACAGAGTTCAGTATGGGGAGCCAGGATTTCCAAGGGATTTCAATGCCCAAATCCTCAGGGGCTGAAGGTTTTGGTGAGGCGGAGCCAAGGCCTGCTCTTTTTGGGATTATTTCAATCTTTGGTTCAGTTAGAACCCCTTGGTGTTGTTTGCCTAGACCTTCACCGGGTGTGTAGCCCATTTTTGTCATCATTTTCTCAGCTTTCGAGGGCTCGGGG
CAATCAACTTTCTCTTTGCCCTTTCCGATTGGTTGGGGTTCTGTGTGGGAGTGGTTGGTCGTGGAGTCAGGTTGTGGCTCATTGATCTGAAGGGGCTTGTTGGAAGGGTGTGGCAACGGTGTGGTGACTGAGGCAAGTGTGGGACTGTTCTGTGATGGGGGAACTGTTGTCACCTTAGTTGTGGAGGGTGTAGAGCATTCACCAAGATTTGGCTGTGAGGAGGTGGAGG
GTGTATCTGTGGTTGGCGGGTGTGAGGAGTTGTTTGCTTGCAGCAGAGCGTCCTGAAGTTTGTGCACAAGCATCTTTGGGTCCAGGGATGTGAATGTGGTGACTGTGCCAGGTAGTAGTGTTTCTATGGACAGGTCATGGCCTTGTAGAAAGAAGTCGTGGATCTGGGCTGTTTGGCCCGGGTCCAGGTCGATGTTTGCCTGTCCGCGGTAGCATTGCACCTTGAGCAC
AGCTGGGACGTTGGTGAAGATGAAGGCTCCGCCAACTGGGTCAAATCCAGGAGCAAAGAGCTTTTGGTTGTTGGGTATTGGGCTTGTGGTGTGGGTCTTGACCTGGTCTAGGTGTAGCAGGTTGGCAGCCTCATGAGCATAGGCCAACTTGGTGTGGTGTAGGATGGTGGTGGGTGCTTGGCATAGGCATTTGAAAGTTTTAACTGGCTGCAAACATGTGTTAATGAAG
TCTGGGGTGGTTTGGGTGTCTGTAGGGATGTTGAAACCAGGGGGTGTGGGTGGTGCAGGTAAGGCCTTGGGTGTGAGGGAGCCGGCGGGTTGTGTTGGTAGGGGCTCCGTATTGACATTTTGGAGCTGTGATTCAGACAACTGAGTAGGGTCGGAAAGATCCTCTTCAGAAGGTTCAATTTCAGGTAGTGAGGCCTTGGTCCAGAGCTCTTTTGAGTTCTCTTTGAATG
TGACTGAGTACTGCTCAGTCTTAATTGAATGTGAAAATGTTTTCCATTCAAGTGCCTTCATGAGTTGAAGGAAATCTCTCTGACCAAAGATCTTCTCATACCAGACAATAAATTTGTTCTTTATGGGCATGAGAAACTTCTTGAAGCAGGACATGGAAAGAATTTTGTCAAAAGAGTTGGTTGAGTCCAATTCAGCCATGAAGGAGAAATAATTAGCCATATGAATTAT
TTCATCTTGGGAGAAATTGGCTAATTGGTTAGTTGGGATCAGTTGCCTAATTTTAGCATAAATATCCCGCTCCGTTACAGTCTTGAGTGATTTGCAGTAAAGAAGCAACTGCATGGCAAGAGTTTTGCTGATTGGTTTGACAGCGTTGTACTCTGAGGGGTGGAAGATTTGTGGGAGCAAGACCATTTCATCAGCTCTGAAAGTCCTAATGGGTGGGGTGAACATTTTT
CCCCTTTTGAATATGTACAAATGGTTAGCAGCTAAACTTTCTACAATTTCAGCGGTGATGGTTACGTTGGGTGAGATGATATGATCATACTTTAACCAGTCTAGAGTTTTCATCTCATGGTGGTAAGCGGCACCACCGCAACCTCCTGGCAAGTATTGGTACCCGCCGTACTGGGAGTAGTTGAGGGTGTAGAGGTTGGGGTAGAGAGATTGCATCCTGTAGAGTGCCT
CTGGGGGTAGAACCAGAGTTGCATAGAGTACATCAAGTGCAGGGGATGCCTCAAATAGCCTGATGAGATATTCTGGGCTTAAAAAGTGCAGTGTGTCACTAATGTAAGCCGTGGTGGTTGGGATTTGTAGATCCTCTTTTATAAGTGTGTCTTCAGAATATCTGGCTGCGTCTCTAGGAGTGATCATTTGGTTCATGAAGTAATCCTTGAGTCTAGGATCTCTCCTGAG
AAATCTTAACTTGCTGCGTTTGAGGTAGAGGAATGTGGTTCTCTCTCTGCGGCAAGTATTCGCCTACGATTCCAAGGAGTTTGTTCTCAATTGCTTTCACACCAGCGTGTGTGTGAAGTGCAATGGAGTAGGGATTGGTGATGATTCCTAGTTTTTCTAGGTCATCAGCAGCCGCTTCTGGAATGGCGTAGGGGTTGACAAGTTGAGCTTGTTTGAGGATGGGTCGTAT
AGTTTTGTAAGCTTCTTCTTGTACAACTGCCCTGACTGAGGCGTCAGTGACGTTTGCAAGTACCTTGGAAACCAAAGCCATAATAACTGGACGGAGGTCAAAGACCACTAGTGTTGTTATAAAGGGCTGGTTTGAGGTAGGTTGTGGTGGGTTGGTTTGTGGTGTGTTTTCC
```

If you notice that there is a polyT tail at the start of the seqeunce, you can use [reverse complement](https://www.bioinformatics.org/sms/rev_comp.html) to reverse complement it before blasting it to NCBI, for ease of interpretation.
The panels below shows the blast results:
<p align="left"><img src="images/blast_results2.png" width="750"></p>

```
Query: None Query ID: lcl|Query_121611 Length: 6813


>Adenium obesum virus X isolate B27, complete genome
Sequence ID: OR039325.1 Length: 6799
Range 1: 1 to 6795

Score:12364 bits(6695), Expect:0.0, 
Identities:6762/6795(99%),  Gaps:2/6795(0%), Strand: Plus/Plus

Query  1     GGAAAACACACCACAAACCAACCCACCACAACCTACCTCAAACCAGCCCTTTATAACAAC  60
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  1     GGAAAACACACCACAAACCAACCCACCACAACCTACCTCAAACCAGCCCTTTATAACAAC  60

Query  61    ACTAGTGGTCTTTGACCTCCGTCCAGTTATTATGGCTTTGGTTTCCAAGGTACTTGCAAA  120
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  61    ACTAGTGGTCTTTGACCTCCGTCCAGTTATTATGGCTTTGGTTTCCAAGGTACTTGCAAA  120

Query  121   CGTCACTGACGCCTCAGTCAGGGCAGTTGTACAAGAAGAAGCTTACAAAACTATACGACC  180
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  121   CGTCACTGACGCCTCAGTCAGGGCAGTTGTACAAGAAGAAGCTTACAAAACTATACGACC  180

Query  181   CATCCTCAAACAAGCTCAACTTGTCAACCCCTACGCCATTCCAGAAGCGGCTGCTGATGA  240
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  181   CATCCTCAAACAAGCTCAACTTGTCAACCCCTACGCCATTCCAGAAGCGGCTGCTGATGA  240

Query  241   CCTAGAAAAACTAGGAATCATCACCAATCCCTACTCCATTGCACTTCACACACACGCTGG  300
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  241   CCTAGAAAAACTAGGAATCATCACCAATCCCTACTCCATTGCACTTCACACACACGCTGG  300

Query  301   TGTGAAAGCAATTGAGAACAAACTCCTTGGAATCGTAGGCGAATACTTGCCGCAGA--GA  358
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||  ||
Sbjct  301   TGTGAAAGCAATTGAGAACAAACTCCTTGGAATCGTAGGCGAATACTTGCCGCAGAGGGA  360

Query  359   GAGAACCACATTCCTCTACCTCAAACGCAGCAAGTTAAGATTTCTCAGGAGAGATCCTAG  418
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  361   GAGAACCACATTCCTCTACCTCAAACGCAGCAAGTTAAGATTTCTCAGGAGAGATCCTAG  420

Query  419   ACTCAAGGATTACTTCATGAACCAAATGATCACTCCTAGAGACGCAGCCAGATATTCTGA  478
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  421   ACTCAAGGATTACTTCATGAACCAAATGATCACTCCTAGAGACGCAGCCAGATATTCTGA  480

Query  479   AGACACACTTATAAAAGAGGATCTACAAATCCCAACCACCACGGCTTACATTAGTGACAC  538
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  481   AGACACACTTATAAAAGAGGATCTACAAATCCCAACCACCACGGCTTACATTAGTGACAC  540

Query  539   ACTGCACTTTTTAAGCCCAGAATATCTCATCAGGCTATTTGAGGCATCCCCTGCACTTGA  598
             ||| ||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  541   ACTACACTTTTTAAGCCCAGAATATCTCATCAGGCTATTTGAGGCATCCCCTGCACTTGA  600

Query  599   TGTACTCTATGCAACTCTGGTTCTACCCCCAGAGGCACTCTACAGGATGCAATCTCTCTA  658
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  601   TGTACTCTATGCAACTCTGGTTCTACCCCCAGAGGCACTCTACAGGATGCAATCTCTCTA  660

Query  659   CCCCAACCTCTACACCCTCAACTACTCCCAGTACGGCGGGTACCAATACTTGCCAGGAGG  718
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  661   CCCCAACCTCTACACCCTCAACTACTCCCAGTACGGCGGGTACCAATACTTGCCAGGAGG  720

Query  719   TTGCGGTGGTGCCGCTTACCACCATGAGATGAAAACTCTAGACTGGTTAAAGTATGATCA  778
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  721   TTGCGGTGGTGCCGCTTACCACCATGAGATGAAAACTCTAGACTGGTTAAAGTATGATCA  780

Query  779   TATCATCTCACCCAACGTAACCATCACCGCTGAAATTGTAGAAAGTTTAGCTGCTAACCA  838
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  781   TATCATCTCACCCAACGTAACCATCACCGCTGAAATTGTAGAAAGTTTAGCTGCTAACCA  840

Query  839   TTTGTACATATTCAAAAGGGGAAAAATGTTCACCCCACCCATTAGGACTTTCAGAGCTGA  898
             ||||||||||||||||||||| ||||||||||||||||||||||||||||||||||||||
Sbjct  841   TTTGTACATATTCAAAAGGGGGAAAATGTTCACCCCACCCATTAGGACTTTCAGAGCTGA  900

Query  899   TGAAATGGTCTTGCTCCCACAAATCTTCCACCCCTCAGAGTACAACGCTGTCAAACCAAT  958
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  901   TGAAATGGTCTTGCTCCCACAAATCTTCCACCCCTCAGAGTACAACGCTGTCAAACCAAT  960

Query  959   CAGCAAAACTCTTGCCATGCAGTTGCTTCTTTACTGCAAATCACTCAAGACTGTAACGGA  1018
             |||||||||||||||||||||||||||||| |||||||||||||||||||||||||||||
Sbjct  961   CAGCAAAACTCTTGCCATGCAGTTGCTTCTCTACTGCAAATCACTCAAGACTGTAACGGA  1020

Query  1019  GCGGGATATTTATGCTAAAATTAGGCAACTGATCCCAACTAACCAATTAGCCAATTTCTC  1078
             ||||||||||||||||||||||||||||||||||||||| ||||||||||||||||||||
Sbjct  1021  GCGGGATATTTATGCTAAAATTAGGCAACTGATCCCAACCAACCAATTAGCCAATTTCTC  1080

Query  1079  CCAAGATGAAATAATTCATATGGCTAATTATTTCTCCTTCATGGCTGAATTGGACTCAAC  1138
             |||||||||||||||||||||||||||||| |||||||||||||||||||||||||||||
Sbjct  1081  CCAAGATGAAATAATTCATATGGCTAATTACTTCTCCTTCATGGCTGAATTGGACTCAAC  1140

Query  1139  CAACTCTTTTGACAAAATTCTTTCCATGTCCTGCTTCAAGAAGTTTCTCATGCCCATAAA  1198
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  1141  CAACTCTTTTGACAAAATTCTTTCCATGTCCTGCTTCAAGAAGTTTCTCATGCCCATAAA  1200

Query  1199  GAACAAATTTATTGTCTGGTATGAGAAGATCTTTGGTCAGAGAGATTTCCTTCAACTCAT  1258
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  1201  GAACAAATTTATTGTCTGGTATGAGAAGATCTTTGGTCAGAGAGATTTCCTTCAACTCAT  1260

Query  1259  GAAGGCACTTGAATGGAAAACATTTTCACATTCAATTAAGACTGAGCAGTACTCAGTCAC  1318
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  1261  GAAGGCACTTGAATGGAAAACATTTTCACATTCAATTAAGACTGAGCAGTACTCAGTCAC  1320

Query  1319  ATTCAAAGAGAACTCAAAAGAGCTCTGGACCAAGGCCTCACTACCTGAAATTGAACCTTC  1378
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  1321  ATTCAAAGAGAACTCAAAAGAGCTCTGGACCAAGGCCTCACTACCTGAAATTGAACCTTC  1380

Query  1379  TGAAGAGGATCTTTCCGACCCTACTCAGTTGTCTGAATCACAGCTCCAAAATGTCAATAC  1438
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  1381  TGAAGAGGATCTTTCCGACCCTACTCAGTTGTCTGAATCACAGCTCCAAAATGTCAATAC  1440

Query  1439  GGAGCCCCTACCAACACAACCCGCCGGCTCCCTCACACCCAAGGCCTTACCTGCACCACC  1498
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  1441  GGAGCCCCTACCAACACAACCCGCCGGCTCCCTCACACCCAAGGCCTTACCTGCACCACC  1500

Query  1499  CACACCCCCTGGTTTCAACATCCCTACAGACACCCAAACCACCCCAGACTTCATTAACAC  1558
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  1501  CACACCCCCTGGTTTCAACATCCCTACAGACACCCAAACCACCCCAGACTTCATTAACAC  1560

Query  1559  ATGTTTGCAGCCAGTTAAAACTTTCAAATGCCTATGCCAAGCACCCACCACCATCCTACA  1618
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  1561  ATGTTTGCAGCCAGTTAAAACTTTCAAATGCCTATGCCAAGCACCCACCACCATCCTACA  1620

Query  1619  CCACACCAAGTTGGCCTATGCTCATGAGGCTGCCAACCTGCTACACCTAGACCAGGTCAA  1678
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  1621  CCACACCAAGTTGGCCTATGCTCATGAGGCTGCCAACCTGCTACACCTAGACCAGGTCAA  1680

Query  1679  GACCCACACCACAAGCCCAATACCCAACAACCAAAAGCTCTTTGCTCCTGGATTTGACCC  1738
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  1681  GACCCACACCACAAGCCCAATACCCAACAACCAAAAGCTCTTTGCTCCTGGATTTGACCC  1740

Query  1739  AGTTGGCGGAGCCTTCATCTTCACCAACGTCCCAGCTGTGCTCAAGGTGCAATGCTACCG  1798
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  1741  AGTTGGCGGAGCCTTCATCTTCACCAACGTCCCAGCTGTGCTCAAGGTGCAATGCTACCG  1800

Query  1799  CGGACAGGCAAACATCGACCTGGACCCGGGCCAAACAGCCCAGATCCACGACTTCTTTCT  1858
             ||||||| ||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  1801  CGGACAGACAAACATCGACCTGGACCCGGGCCAAACAGCCCAGATCCACGACTTCTTTCT  1860

Query  1859  ACAAGGCCATGACCTGTCCATAGAAACACTACTACCTGGCACAGTCACCACATTCACATC  1918
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  1861  ACAAGGCCATGACCTGTCCATAGAAACACTACTACCTGGCACAGTCACCACATTCACATC  1920

Query  1919  CCTGGACCCAAAGATGCTTGTGCACAAACTTCAGGACGCTCTGCTGCAAGCAAACAACTC  1978
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  1921  CCTGGACCCAAAGATGCTTGTGCACAAACTTCAGGACGCTCTGCTGCAAGCAAACAACTC  1980

Query  1979  CTCACACCCGCCAACCACAGATACACCCTCCACCTCCTCACAGCCAAATCTTGGTGAATG  2038
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  1981  CTCACACCCGCCAACCACAGATACACCCTCCACCTCCTCACAGCCAAATCTTGGTGAATG  2040

Query  2039  CTCTACACCCTCCACAACTAAGGTGACAACAGTTCCCCCATCACAGAACAGTCCCACACT  2098
             ||||||||||||||||||||||||||||||| | || |||||||||||||||||||||||
Sbjct  2041  CTCTACACCCTCCACAACTAAGGTGACAACAATCCCTCCATCACAGAACAGTCCCACACT  2100

Query  2099  TGCCTCAGTCACCACACCGTTGCCACACCCTTCCAACAAGCCCCTTCAGATCAATGAGCC  2158
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  2101  TGCCTCAGTCACCACACCGTTGCCACACCCTTCCAACAAGCCCCTTCAGATCAATGAGCC  2160

Query  2159  ACAACCTGACTCCACGACCAACCACTCCCACACAGAACCCCAACCAATCGGAAAGGGCAA  2218
             |||||||||| ||| |||||||||||||||||||||||||||||||||||||||||||||
Sbjct  2161  ACAACCTGACCCCATGACCAACCACTCCCACACAGAACCCCAACCAATCGGAAAGGGCAA  2220

Query  2219  AGAGAAAGTTGATTGCCCCGAGCCCTCGAAAGCTGAGAAAATGATGACAAAAATGGGCTA  2278
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  2221  AGAGAAAGTTGATTGCCCCGAGCCCTCGAAAGCTGAGAAAATGATGACAAAAATGGGCTA  2280

Query  2279  CACACCCGGTGAAGGTCTAGGCAAACAACACCAAGGGGTTCTAACTGAACCAAAGATTGA  2338
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  2281  CACACCCGGTGAAGGTCTAGGCAAACAACACCAAGGGGTTCTAACTGAACCAAAGATTGA  2340

Query  2339  AATAATCCCAAAAAGAGCAGGCCTTGGCTCCGCCTCACCAAAACCTTCAGCCCCTGAGGA  2398
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  2341  AATAATCCCAAAAAGAGCAGGCCTTGGCTCCGCCTCACCAAAACCTTCAGCCCCTGAGGA  2400

Query  2399  TTTGGGCATTGAAATCCCTTGGAAATCCTGGCTCCCCATACTGAACTCTGTTGGATTCAA  2458
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  2401  TTTGGGCATTGAAATCCCTTGGAAATCCTGGCTCCCCATACTGAACTCTGTTGGATTCAA  2460

Query  2459  AGGCAATCAGATGCAGTACTCAGACCACAAGAGAACTAATCTGATAATCCCAATCACAAG  2518
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  2461  AGGCAATCAGATGCAGTACTCAGACCACAAGAGAACTAATCTGATAATCCCAATCACAAG  2520

Query  2519  ACAACACAAGCTACCCCGCTCTGAGGGGCTCGAAGACTTGGACCCAAATCTCATTCTAGC  2578
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  2521  ACAACACAAGCTACCCCGCTCTGAGGGGCTCGAAGACTTGGACCCAAATCTCATTCTAGC  2580

Query  2579  CCTCAAGTCCATCAGCCGCTTTCCCACACCAGTTCTACTTTCAGGTACTCGGGCAGCAGC  2638
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  2581  CCTCAAGTCCATCAGCCGCTTTCCCACACCAGTTCTACTTTCAGGTACTCGGGCAGCAGC  2640

Query  2639  CTATGCTTCTGACTTGAAGAATTGCAGAACGGGTCTGCTCGGCAAAACCCAACCTTTGGA  2698
             ||||||||||||||||||||||||||||||||| ||||||||||||||||||||||||||
Sbjct  2641  CTATGCTTCTGACTTGAAGAATTGCAGAACGGGCCTGCTCGGCAAAACCCAACCTTTGGA  2700

Query  2699  ATGGAAGCAAAAGTTCTCCCTTATGTGTGAAAGGAGCCCTGATATGACAGTGCAGATGTC  2758
             ||||||||||||||||||||| ||||||||||||||||||||||||||||||||||||||
Sbjct  2701  ATGGAAGCAAAAGTTCTCCCTCATGTGTGAAAGGAGCCCTGATATGACAGTGCAGATGTC  2760

Query  2759  CGTGATTCACGGTGCTGGAGGCTCTGGCAAGTCCCGTTTCCTCCAAAATTGGCTCAAAAC  2818
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  2761  CGTGATTCACGGTGCTGGAGGCTCTGGCAAGTCCCGTTTCCTCCAAAATTGGCTCAAAAC  2820

Query  2819  ATTTGAACAAGACAATTGCCCAGCTACAATTGTCGTCCCCACCAATGAACTCAGGCTTGA  2878
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  2821  ATTTGAACAAGACAATTGCCCAGCTACAATTGTCGTCCCCACCAATGAACTCAGGCTTGA  2880

Query  2879  TTGGGAGGGAAAACTTCCCAACATCAACAGGCTGCTGATCAAAACTTTTGAAAAGGCACT  2938
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  2881  TTGGGAGGGAAAACTTCCCAACATCAACAGGCTGCTGATCAAAACTTTTGAAAAGGCACT  2940

Query  2939  AATAGCTGGTTCAGGACAAATAGTCATTTTTGATGATTATGGGAAAATGCCAGCTGGTTT  2998
             ||||||||||||||||||||||||||| ||||||||||||||||||||||||||||||||
Sbjct  2941  AATAGCTGGTTCAGGACAAATAGTCATCTTTGATGATTATGGGAAAATGCCAGCTGGTTT  3000

Query  2999  CATTGAGGCATACATATTCTCGCATCCGGCCGTTGAGCTTGCCATTCTAACCGGGGACCC  3058
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  3001  CATTGAGGCATACATATTCTCGCATCCGGCCGTTGAGCTTGCCATTCTAACCGGGGACCC  3060

Query  3059  CAGACAAAGTTCATACCATGAGGATAATGACATGGCCCTCATCTCCAAGCTCGACCCAGG  3118
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  3061  CAGACAAAGTTCATACCATGAGGATAATGACATGGCCCTCATCTCCAAGCTCGACCCAGG  3120

Query  3119  CCCCTCCCACTTTGACCAGTACTCAAGGTACTATCTGAACTGCACACACAGGAACAGGAG  3178
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  3121  CCCCTCCCACTTTGACCAGTACTCAAGGTACTATCTGAACTGCACACACAGGAACAGGAG  3180

Query  3179  AGACATTGCAAATAAGCTGGGCGTCTACTCCGAGACTGAGGGACCGTGTTCCATCACAAT  3238
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  3181  AGACATTGCAAATAAGCTGGGCGTCTACTCCGAGACTGAGGGACCGTGTTCCATCACAAT  3240

Query  3239  CTCTAGTCAAGTGCTTAAAGGAATTCCTATTCTTACCCCCTCCATGATCAAAAAGACAGC  3298
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  3241  CTCTAGTCAAGTGCTTAAAGGAATTCCTATTCTTACCCCCTCCATGATCAAAAAGACAGC  3300

Query  3299  CATCACTGAGTTGGGCAACAAAGGATACACTTATGCCGGCTGCCAGGGACTCACAGCTCC  3358
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  3301  CATCACTGAGTTGGGCAACAAAGGATACACTTATGCCGGCTGCCAGGGACTCACAGCTCC  3360

Query  3359  CAAAGTTCAAATCCTGCTCGATTCCAACACCCCATTCTGCTCCGATGAGGTGATGTACAC  3418
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  3361  CAAAGTTCAAATCCTGCTCGATTCCAACACCCCATTCTGCTCCGATGAGGTGATGTACAC  3420

Query  3419  AGCTTTATCCAGATCTGTGGAAATGATTCACTTTATCAACACTGGACCTAACTCTCAGGA  3478
             |||||||||||||||||| |||||||||||||||||||||||||||||||||||||||||
Sbjct  3421  AGCTTTATCCAGATCTGTAGAAATGATTCACTTTATCAACACTGGACCTAACTCTCAGGA  3480

Query  3479  GTACTGGACCAAGTTGGACGCCACTCCATACCTCAAAACTTTCCTTGACACTGCCAGAGA  3538
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  3481  GTACTGGACCAAGTTGGACGCCACTCCATACCTCAAAACTTTCCTTGACACTGCCAGAGA  3540

Query  3539  GGAGGCCATGGCAAAAATTACCAGCCTAGAACCTCAAGTTCCAGAGCCCAAAATCCCCAC  3598
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  3541  GGAGGCCATGGCAAAAATTACCAGCCTAGAACCTCAAGTTCCAGAGCCCAAAATCCCCAC  3600

Query  3599  ACACTTCCCTGTTGAGCATCAGGACATACAGCTTGAGGACTACATTGAGGGGCAAAAGGA  3658
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  3601  ACACTTCCCTGTTGAGCATCAGGACATACAGCTTGAGGACTACATTGAGGGGCAAAAGGA  3660

Query  3659  AAAGTATGACCGAGAGATCTTTGACAGCAGGCACGGTCACACCAACTGTATACAAACTGA  3718
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  3661  AAAGTATGACCGAGAGATCTTTGACAGCAGGCACGGTCACACCAACTGTATACAAACTGA  3720

Query  3719  TGACCAATTAGTGCAAATCTTCCAACACCAGCAGGCTAAGGATGAGCCTCTTCTCTGGGC  3778
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  3721  TGACCAATTAGTGCAAATCTTCCAACACCAGCAGGCTAAGGATGAGCCTCTTCTCTGGGC  3780

Query  3779  TACCATTGAGGCTCGCATCAAGACTGCATCACCGGAGCAAAACCTGAAAGAGCTGGCATT  3838
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  3781  TACCATTGAGGCTCGCATCAAGACTGCATCACCGGAGCAAAACCTGAAAGAGCTGGCATT  3840

Query  3839  aaaaaaaGACATTGGTGACTTGCTCTTCCTCAACTACAAGAGAGCCATGCATTTGCCTGA  3898
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  3841  AAAAAAAGACATTGGTGACTTGCTCTTCCTCAACTACAAGAGAGCCATGCATTTGCCTGA  3900

Query  3899  CCAACCTGTGCCTTTTGATCCCCGACTGTGGGAGCTGTCTGCCACTGAAGTCCGCAACAA  3958
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  3901  CCAACCTGTGCCTTTTGATCCCCGACTGTGGGAGCTGTCTGCCACTGAAGTCCGCAACAA  3960

Query  3959  ATACTTGGAGAAGCCCATCAACATGCTGGTCAATGCCGCTCAACGACAATCTCCTGATTT  4018
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||| ||
Sbjct  3961  ATACTTGGAGAAGCCCATCAACATGCTGGTCAATGCCGCTCAACGACAATCTCCTGACTT  4020

Query  4019  TGACAAGGAGAAAATTGCACTCTTCCTAAAGTCACAGTGGGTTAAAAAAGTTGAGAAATT  4078
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  4021  TGACAAGGAGAAAATTGCACTCTTCCTAAAGTCACAGTGGGTTAAAAAAGTTGAGAAATT  4080

Query  4079  GGGTGCTCTGAAAGTAAAGCCCGGCCAAACTATAGCAGCCTTCATGCAGGAAACTGTAAT  4138
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  4081  GGGTGCTCTGAAAGTAAAGCCCGGCCAAACTATAGCAGCCTTCATGCAGGAAACTGTAAT  4140

Query  4139  GCTATACGGAACAATGGCCCGCTACCTGAGGAAAATGAGAGCAGTATTCCAACCCAAGAA  4198
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  4141  GCTATACGGAACAATGGCCCGCTACCTGAGGAAAATGAGAGCAGTATTCCAACCCAAGAA  4200

Query  4199  CATCTTCATCAACTGTGAAAAGACTCCTGAAGACCTCAACCAATTCGTGCTCGAAAACTG  4258
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  4201  CATCTTCATCAACTGTGAAAAGACTCCTGAAGACCTCAACCAATTCGTGCTCGAAAACTG  4260

Query  4259  GAACTTTTCAAGAACTGCATACTCTAATGACTTCACAGCCTTTGACCAATCACAAGATGG  4318
             |||||| |||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  4261  GAACTTCTCAAGAACTGCATACTCTAATGACTTCACAGCCTTTGACCAATCACAAGATGG  4320

Query  4319  TGCCATGCTCCAGTTTGAGGTACTTAAGGCCAAACACTTCAACATTCCAGCTGACATCAT  4378
             ||||||||||||||||||||||||||||||||||||||||||||| || |||||||||||
Sbjct  4321  TGCCATGCTCCAGTTTGAGGTACTTAAGGCCAAACACTTCAACATCCCGGCTGACATCAT  4380

Query  4379  TGATGGATATGTGCACCTTAAAACACATGCCACAATCTTCCTTGGCACCCTGGCCATAAT  4438
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  4381  TGATGGATATGTGCACCTTAAAACACATGCCACAATCTTCCTTGGCACCCTGGCCATAAT  4440

Query  4439  GAGACTTACAGGTGAAGGCCCCACTTTCGATGCTAACACTGAATGTTCCATTGCCTACCA  4498
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  4441  GAGACTTACAGGTGAAGGCCCCACTTTCGATGCTAACACTGAATGTTCCATTGCCTACCA  4500

Query  4499  CCACACCAGGTTCCATGTGCCAGACAATGTGGCCCAACTTTATGCTGGTGATGATTCCGC  4558
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  4501  CCACACCAGGTTCCATGTGCCAGACAATGTGGCCCAACTTTATGCTGGTGATGATTCCGC  4560

Query  4559  CCAAGACCATCCTGCAATTGAAAAGAAAAGCTTCCAACTGATCCAGAATAAACTGAATCT  4618
             |||||||||||||||||||||||||||||||||||||||||||||||| |||||||||||
Sbjct  4561  CCAAGACCATCCTGCAATTGAAAAGAAAAGCTTCCAACTGATCCAGAACAAACTGAATCT  4620

Query  4619  CACGTCCAAACCAGTCACGTACCAACAAAAACCGGGCGACTGGGCCGACTTCTGCGGGTG  4678
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  4621  CACGTCCAAACCAGTCACGTACCAACAAAAACCGGGCGACTGGGCCGACTTCTGCGGGTG  4680

Query  4679  GACTATCAGCAGACTGGGCATCTTCAAGAACCCTACCAAAATTTTGGCCAGCCTTGAACT  4738
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  4681  GACTATCAGCAGACTGGGCATCTTCAAGAACCCTACCAAAATTTTGGCCAGCCTTGAACT  4740

Query  4739  AGCCATTAAAACTGGCAAAACCAAGGAAGTAGCTACCAGTTATGCCCTCGATGCCAAATT  4798
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  4741  AGCCATTAAAACTGGCAAAACCAAGGAAGTAGCTACCAGTTATGCCCTCGATGCCAAATT  4800

Query  4799  TGCTTACCAACTTGGGGACAAATTGCAAGAAGTACTTTCAGAGAAGGAGTGTGGCATGCA  4858
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  4801  TGCTTACCAACTTGGGGACAAATTGCAAGAAGTACTTTCAGAGAAGGAGTGTGGCATGCA  4860

Query  4859  CCAAACACTCATCAGAAAGATTCACAAGATAGGAACAGGGCACCTATTGGAGGGTTTAAG  4918
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  4861  CCAAACACTCATCAGAAAGATTCACAAGATAGGAACAGGGCACCTATTGGAGGGTTTAAG  4920

Query  4919  TTGATCTTAGTTTGAAATGGACAACTTAGTCATTACCGAATTAACCCGTTTGCAACTAAC  4978
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  4921  TTGATCTTAGTTTGAAATGGACAACTTAGTCATTACCGAATTAACCCGTTTGCAACTAAC  4980

Query  4979  ACGCACATCCCGTCCCCTCTCCAAACCCATCATAATCCACCTGGTAGCAGGCGCTGGCAA  5038
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  4981  ACGCACATCCCGTCCCCTCTCCAAACCCATCATAATCCACCTGGTAGCAGGCGCTGGCAA  5040

Query  5039  GACCACCATCATCAGAAAACTCCTAACCATCCATCACACCATTAGAGCCTTCACCTGTGG  5098
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  5041  GACCACCATCATCAGAAAACTCCTAACCATCCATCACACCATTAGAGCCTTCACCTGTGG  5100

Query  5099  CACTCCCGAcccccccTCCCTTACTGGCACTCACATCCTACCCTGGAAGAGTGAAAGACC  5158
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  5101  CACTCCCGACCCCCCCTCCCTTACTGGCACTCACATCCTACCCTGGAAGAGTGAAAGACC  5160

Query  5159  AGGCACCACAACCCTCCTTGACGAATACCCACTAGCCACTGACTACCAAGCAGCTGACTT  5218
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  5161  AGGCACCACAACCCTCCTTGACGAATACCCACTAGCCACTGACTACCAAGCAGCTGACTT  5220

Query  5219  CTGTTTCGCTGACCCTATCCAACACAACATCACCGAACCGTATCCACACTTTACCAGCAA  5278
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  5221  CTGTTTCGCTGACCCTATCCAACACAACATCACCGAACCGTATCCACACTTTACCAGCAA  5280

Query  5279  CTTCACCCACCGTGTGCCTGAACCCATCTGTAACCATCTAAGGTCACTCACATTTGACAT  5338
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  5281  CTTCACCCACCGTGTGCCTGAACCCATCTGTAACCATCTAAGGTCACTCACATTTGACAT  5340

Query  5339  CCATTCAACCAAAGCGGGTTTTTTAACATTTGAAAATATCTTCCAAGGTGAACCCATTGG  5398
             |||||||||||||||||| |  ||||||||||||||||||||||||||||||||||||||
Sbjct  5341  CCATTCAACCAAAGCGGGATCCTTAACATTTGAAAATATCTTCCAAGGTGAACCCATTGG  5400

Query  5399  TACCATCATAGCTTGGGAGCCTGAAATCATCCGCCTCCTCAACTCCCACCAAGCTGACTA  5458
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||| ||||
Sbjct  5401  TACCATCATAGCTTGGGAGCCTGAAATCATCCGCCTCCTCAACTCCCACCAAGCTCACTA  5460

Query  5459  TCTCCGACCATGCCAGGCTCTAGGACTCCAATTTGACACTGTCACCGTCATCACTCTCGA  5518
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  5461  TCTCCGACCATGCCAGGCTCTAGGACTCCAATTTGACACTGTCACCGTCATCACTCTCGA  5520

Query  5519  GCCCAAACCCACCCATTTGTACTACATCGCCTGCACCCGCTCCCTCAACAAACTCATCAT  5578
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  5521  GCCCAAACCCACCCATTTGTACTACATCGCCTGCACCCGCTCCCTCAACAAACTCATCAT  5580

Query  5579  CAAATGCCTCTCACCCCACCGCCTAACCACATTAGGGAATTGACACCATTTCTACTAGGT  5638
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  5581  CAAATGCCTCTCACCCCACCGCCTAACCACATTAGGGAATTGACACCATTTCTACTAGGT  5640

Query  5639  TTTGCATTAGTTCTAGCCATCTTCACTTGCACCCGCAACACTCTACCCCACACTGGTGAC  5698
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  5641  TTTGCATTAGTTCTAGCCATCTTCACTTGCACCCGCAACACTCTACCCCACACTGGTGAC  5700

Query  5699  AACATCCACAGTCTTCCTTTCGGTGGCTCATACATCGACGGCACCAAGAGAATTAACTAC  5758
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  5701  AACATCCACAGTCTTCCTTTCGGTGGCTCATACATCGACGGCACCAAGAGAATTAACTAC  5760

Query  5759  TTACCACCCAAAGGTCAAGGTACCTTCTTCCCCTCTCATGCACAACGAGCACACCTCCCC  5818
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  5761  TTACCACCCAAAGGTCAAGGTACCTTCTTCCCCTCTCATGCACAACGAGCACACCTCCCC  5820

Query  5819  GCCATCCTTGTTTGCATTCTTCCACTTGTCATTTTCCTTCTACATAAAGCTACTACTAGG  5878
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  5821  GCCATCCTTGTTTGCATTCTTCCACTTGTCATTTTCCTTCTACATAAAGCTACTACTAGG  5880

Query  5879  AATAACCCTGGGCATACTTGTACTACTTGCTCTTCCCAGACCCTATGAACCCTGTACCAT  5938
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  5881  AATAACCCTGGGCATACTTGTACTACTTGCTCTTCCCAGACCCTATGAACCCTGTACCAT  5940

Query  5939  CATCATCACAGGGGAGCGAGTCACAGCACATGGTTGCCCAGTCACTCAGGAACTTGCCAA  5998
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  5941  CATCATCACAGGGGAGCGAGTCACAGCACATGGTTGCCCAGTCACTCAGGAACTTGCCAA  6000

Query  5999  CATACTCAGCAACAGAGAGGCTCTACACGCCGTTAAGTTTCCAATCAATTCTGAATAGAG  6058
             ||||||||||||||||||||| ||||||||||||||||||||||||||||||||||||||
Sbjct  6001  CATACTCAGCAACAGAGAGGCCCTACACGCCGTTAAGTTTCCAATCAATTCTGAATAGAG  6060

Query  6059  TTATGGACACCTCAGATCCAGCTACTGCTTCAACTGTAATTCTCTCCGACCTCTCAGACC  6118
             ||||||||||||||||  |||  || ||||| ||||||||||||||||||||||||||||
Sbjct  6061  TTATGGACACCTCAGACACAGACACCGCTTCGACTGTAATTCTCTCCGACCTCTCAGACC  6120

Query  6119  CTACCATTAGTCCATCTCTAGCAGATTTGAAGAGGATAACTTACAAAACACACTCCAACT  6178
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  6121  CTACCATTAGTCCATCTCTAGCAGATTTGAAGAGGATAACTTACAAAACACACTCCAACT  6180

Query  6179  CTGTTGCATCACCTGAGACAGTGAAGGCCATCGCTAAAGAATGGGTTGAAAACGGCGTCC  6238
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  6181  CTGTTGCATCACCTGAGACAGTGAAGGCCATCGCTAAAGAATGGGTTGAAAACGGCGTCC  6240

Query  6239  CCGCTGACCAAGTGGCGCTCGCAGCTTGGAGCCTCGCCATCTACTGCGCAGACAATGGCT  6298
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  6241  CCGCTGACCAAGTGGCGCTCGCAGCTTGGAGCCTCGCCATCTACTGCGCAGACAATGGCT  6300

Query  6299  CCTCAGACCAAACTGAGTTTGAGGGCTCAATCTCCGACCTCTCAGTATCATATGACACTC  6358
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  6301  CCTCAGACCAAACTGAGTTTGAGGGCTCAATCTCCGACCTCTCAGTATCATATGACACTC  6360

Query  6359  TTGCCCGCATCACCAAGAAACACACCACACTGCGCAAATTCTGCTCATACTATGCAAAGA  6418
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  6361  TTGCCCGCATCACCAAGAAACACACCACACTGCGCAAATTCTGCTCATACTATGCAAAGA  6420

Query  6419  TAGTGTGGAACCTGCTTATTGAGAGGCAATCACCACCCGCGGGCTGGGCCAAATGGAACT  6478
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  6421  TAGTGTGGAACCTGCTTATTGAGAGGCAATCACCACCCGCGGGCTGGGCCAAATGGAACT  6480

Query  6479  ACCGCAAATCTGAAGCCTTCGCCGCTTTCGATTTCTTTGACGCTGTTACCAACCCTGCAG  6538
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  6481  ACCGCAAATCTGAAGCCTTCGCCGCTTTCGATTTCTTTGACGCTGTTACCAACCCTGCAG  6540

Query  6539  CCCTTGAACCAAAAGAGGGGTTGGTTCGCCAACCCACCCAAAATGAGCTATTGGCCAAAG  6598
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  6541  CCCTTGAACCAAAAGAGGGGTTGGTTCGCCAACCCACCCAAAATGAGCTATTGGCCAAAG  6600

Query  6599  GCACAAACAAGATGGCATCCCTCCATGAGGCGGCCAAACAGACTCTTCACACTGTCTCCA  6658
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  6601  GCACAAACAAGATGGCATCCCTCCATGAGGCGGCCAAACAGACTCTTCACACTGTCTCCA  6660

Query  6659  ACTATGCAGGTGTAACCCAAGGACGCCTTCCACGTCCAACTCTGCCTGCCATTGAGGCTC  6718
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  6661  ACTATGCAGGTGTAACCCAAGGACGCCTTCCACGTCCAACTCTGCCTGCCATTGAGGCTC  6720

Query  6719  CCGAATATTAATGCTGGTATAACTTAATACTACTATAGCTTTTGGTTTGAATAAAGTTTC  6778
             ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Sbjct  6721  CCGAATATTAATGCTGGTATAACTTAATACTACTATAGCTTTTGGTTTGAATAAAGTTTC  6780

Query  6779  Caaaaaaaaaaaaaa  6793
             |||||||||||||||
Sbjct  6781  CAAAAAAAAAAAAAA  6795
```

You can now go in the canu folder (results/MT483/assembly/canu) to check the original canu contig (tig00000003), in the MT483_canu_assembly.fasta file. If you comapre it to the final contig after the cutadapt step was applied, you will notice that without the final primer check, the contig had an extra stretch of nucleotide at its 3' end (CCCGCGTACTCTGCGTTGATACCACTGCTT) which matches the Template-switching oligonucleotide (AAGCAGTGGTATCAACGCAGAGTACGCGGG) which was used during the RACE library preparation.

Reads can also be directly mapped to the predicted amplicon sequence if it is known:
```
nextflow run maelyg/ontvisc -resume -profile singularity \
                            --analysis_mode map2ref \
                            --reference /work/eresearch_bio/test_datasets/MT483_amplicon_RACE_new_chemistry_ligation/AobVX.fasta
```

## Example of direct RNA sequencing

The **SRR17660991** sample is from a Nicotiana tabaccum specimen infected with tomato spotted wilt orthotospovirus. This sample is part of a study that was published in 2022 in Frontiers in Microbiology ([https://www.ncbi.nlm.nih.gov/pmc/articles/PMC913109](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC9131090/). The raw data (inc fastq files) are available at https://www.ncbi.nlm.nih.gov/sra/?term=SRR17660991. Total RNA was DNAse treated and polyA tailing were applied for further library preparation using Oxford Nanopore Technologies kit DirectRNA (SQK-RNA002) for sequencing.

For this sample, you can use a de novo approach or a clustering approach.


```
nextflow run eresearchqut/ontvisc -resume -profile singularity \
                                  --analysis_mode clustering \
                                  --host_filtering \
                                  --qual_filt --chopper_options '-q 10' \
                                  --blast_threads 8 --blastn_db /work/hia_mt18005/databases/blastDB/20230606/nt \
                                  --rattle_clustering_options '--rna --lower-length 1000 --upper-length 150000' \
                                  --rattle_polishing_options '--rna' \
                                  --host_fasta ~/code/micropipe/test_data/Plant_host_sequences11_ed.fasta
```

## Example of dengue virus sample sequenced at very high depth
This was sequenced using an amplicon approach. The amplicon size is expected to be ~10,000 bp. Raw data is available in folder /work/eresearch_bio/test_datasets/ET300 on Lyra.

The strategy here is to only retain high quality data as it was sequenced at very high depth. After checking the QC profile, we can see that by performing harsh quality filtering step, we should still retain a decent amount of reads. Chopper retains 14,921 reads using the options '-q 18 -l 9500 --maxlength 11000'.
```
nextflow run eresearchqut/ontvisc -resume -profile singularity \
                                  --qual_filt \
                                  --chopper_options '-q 18 -l 9500 --maxlength 11000' \
                                  --blast_threads 8 --blastn_db /work/hia_mt18005/databases/blastDB/20230606/nt \
                                  --clustering  --rattle_clustering_options '--lower-length 9000 --upper-length 11000'
```
