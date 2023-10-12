# Bovine Long Read Consortium Guidelines for Run 1

*Version: 14/09/2023*

These specifications describe the data requirements and steps to process sequence reads to GCVF and SNF files for the Bovine Long Read Consortium. Please follow these instructions closely. **NOTE:** GVCF and SNF files will not be accepted if they do not meet these specifications.

These specifications have been agreed to by a Quality Control Subcommittee comprising of Marta Godia-Perello, Elizabeth Ross, Loan Nguyen, Iona MacLeod, Tuan Nguyen, Oscar Gonzalez-Recio, James Prendergast, Hubert Pausch, Didier Boichard, Ben Rosen, Andrea Talenti, Guilherme Neumann, Christian Reimer, Johannes Geibel, Mekki Boussaha, Ben Hayes, Amanda Chamberlain.

## Data Requirements for Inclusion in Run 1

1. For inclusion in Run1, participants agree to make their data public on publication of results. All Run1 collaborators will meet to jointly plan approaches/tasks towards publication of results. Animals that contributors wish to remain private cannot be included.
2. Minimum set of animals (small number for Run 1) = 10
3. ONT minimum base call requirement: Guppy V6.4 & Super High Accuracy mode (if FAST5 files available)
4. PacBio minimum mapped read depth: CLR 20x, HiFi 10x
5. ONT minimum mapped read depth: 10x (but if basecalled with old version of Guppy & FAST5 files deleted - then prefer 20x)
6. Minimum N50 read length = 10 Kb. For future sequencing aim for minimum N50 = 20 Kb. Most SV are not huge, so for population-scale the above read length is OK.
7. If you have a breed not previously LR sequenced, then consider the possibility of doing at least one animal at higher coverage (and lengths) and contact Ben Rosen [ben.rosen@usda.gov](mailto:ben.rosen@usda.gov) at the Pan Genome Consortium project to discuss breed inclusion.
8. SNP chip genotypes preferably available for sequenced animals (higher density preferable) to check genotype call accuracy. For some, there may be short read data available, this could provide an alternate mechanism.
9. Provide high quality records of meta-data of samples sequenced
10. Lab protocols: Participants can choose ONT or PacBio sequencing. QC committee members have agreed to share their experiences with these technologies. Email inquiries to cattleLRC@gmail.com

## Run 1 Process

![BovLRC_flow.png
](https://raw.githubusercontent.com/BovLRC/bovLRC-documentation/main/images/BovLRC_flow.png)

*Figure 1:* Flowchart of the agreed process for run 1 of the Bovine Long Read Consortium.

**Reference to use for Run 1**

ARS-UCD2.0 is the reference genome to be used in this project. This reference comprises ARS-UCD1.2 (Rosen, Bickhart et al. 2018) and the Y chromosome assembly from the T2T assembly from Wagyu. It can be downloaded from [NCBI](https://www.ncbi.nlm.nih.gov/assembly/GCF_002263795.3). This exact copy of the reference genome must be used to ensure GVCF & SNF files are compatible with the BovLRC pipeline. Non-conforming files will be excluded from the run. The annotation for this assembly is also available at [NCBI](https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/002/263/795/GCF_002263795.3_ARS-UCD2.0/) for downstream analysis.

**Software required for member processing of FASTQ to SNF and GVCF files**

- **QC:** There are many software available for trimming and quality control. We recommend the following for various QC applications:
  - [FiltLong](https://github.com/rrwick/Filtlong) (ONT)
  - [Chopper](https://github.com/wdecoster/chopper) (ONT)
  - [Porechop_ABI](https://github.com/bonsai-team/Porechop_ABI) (ONT)
  - [NanoStats](https://github.com/wdecoster/nanostat) (ONT)
  - [SMRTlink](https://www.pacb.com/support/software-downloads/) (PacBio CLR)
  - baz2bam (PacBio CLR)
  - [FastQC v11.7](http://www.bioinformatics.babraham.ac.uk/projects/download.html#fastqc) (ONT and PacBio)
- **Alignment:** Minimap2 version >2.22 [Minimap2](https://github.com/lh3/minimap2)
- **Sort and index Bam file:** [Samtools](http://www.htslib.org/dc/samtools.html)
- **SV Calling:** [Sniffles v2.2](https://github.com/fritzsedlazeck/Sniffles)
- **SNP Calling:** [Clair3 v1.4](https://github.com/HKU-BAL/Clair3) (Clair3 models: [ONT](https://github.com/nanoporetech/rerio/tree/master/clair3_models), [PacBio](https://github.com/HKU-BAL/Clair3))

**Sample and File Naming Convention**

All files should be named according to the animal's BioSample or International ID. If an animal’s sequence has not yet been submitted to a public archive and therefore does not have a BioSample ID you will be required to provide it prior to publication. If your animal does not have an international ID, you should create one that conforms to Interbull standards, ie 3 character breed code + 3 character country code + sex code (M or F) + 12 character animal ID, eg HOLCANM000000352790. See [Interbull Breed Codes](http://www.interbull.org/ib/icarbreedcodes). All files should be named with either the Biosample or International ID as a prefix.

**Trim and Filter FASTQ Files**

Trimming can be personalized based on platform/quality. It is crucial that any adapter sequences are removed and that the library meets the specifications detailed in the data requirements section above. Read length n50 should be provided in the checklist (see below).
- For ONT data we recommend discarding reads < 200bp with FiltLong and remove adapter sequence with Porechop_ABI. NanoStats is useful to check quality before & after QC.
- For PacBio CLR data we recommend using baz2bam to check quality.
- PacBio Hifi data does not require any further QC.

*Example of FiltLong & Porechop_ABI command:*
```
filtlong --min_length 200 ${input}.fastq.gz | bgzip > ${output}.fastq.gz
porechop_abi -abi -i ${input}.fq.gz -o ${output}_QC.fq.gz
```

*Example NanoPlot command*

```
NanoPlot -t $task.cpus --fastq ${input}.fastq.gz --outdir {ouputdirectory} -p ${prefix}_ --loglength --plots dot
```

*An example baz2bam command*

```
baz2bam ${input}.baz --metadata ${input}.metadata.xml -j 32 -b 8 --inlinePbi --progress --silent --maxInputQueueMB 70000 –zmwBatchMB 50000 –zmwHeaderBatchMB 30000 –maxOutputQueueMB 15000 -o ${output_dir}
```

## Map FASTQ

First the reference FASTA should be indexed by Minimap2. Map trimmed reads (that pass above QC) to the reference using Minimap2. Sort resulting bam file with samtools sort and index your sorted bam file with samtools index. Using the correct reference will ensure the bam files are sorted correctly.

Where multiple bam files are generated for an individual you can use [Picard MergeSamFiles] (https://broadinstitute.github.io/picard/command-line-overview.html#MergeSamFiles) to merge them. 

*An example of Minimap2 and Samtools commands*

```
minimap2 -x ${X} -d ont ARS-bov-ont.mmi ARS-UCD2.0.fasta.gz
minimap2 -ax ${X} ARS-bov-ont.mmi -t 24 ${fastq} --MD | samtools sort -@ 24 - -o ${INTERNATIONALID}.sorted.bam 
```

where `X` can be `map-ont`, `map-pb` or `map-hifi` depending on your sequence

*An example of Picard MergeSamFiles command*

```
java -Xmx80G -jar /usr/local/picard/2.1.0/picard.jar MergeSamFiles ${BAMlist} O=${INTERNATIONALID}.sorted.bam VALIDATION_STRINGENCY=LENIENT ASSUME_SORTED=true MERGE_SEQUENCE_DICTIONARIES=true
```

## Variant Calling

Use Sniffles2 to call structural variants within individual. This will generate a SNF file, use bgzip to compress it. This snf.gz file should be submitted to AgVic, keep a copy for your own reference. 

An example of Sniffles2 command

```
sniffles --input ${INTERNATIONALID}.sorted.bam \
         --vcf ${INTERNATIONALID}_SV.vcf \
         --snf ${INTERNATIONALID}.snf \
         --threads $task.cpus \
         --minsvlen 50 \
         --mapq 20 \
         --reference ARS-UCD2.0.fa         
```

Use Clair3 to call small variants within individual. Ensure the appropriate Clair3 models are used for your data type (using `--model_path option`). Models can be found for various data types on the [Clair3 github](https://github.com/HKU-BAL/Clair3). For ONT flowcell version 10.4 and above, users should download the newest Clair3 models from [https://github.com/nanoporetech/rerio/tree/master/clair3_models](https://github.com/nanoporetech/rerio/tree/master/clair3_models). Note that there is currently no model for PacBio CLR data. If you have this data type please email us at [cattleLRC@gmail.com](mailto:cattleLRC@gmail.com) for alternative instructions. 

The resulting GVCF file should be zipped using bgzip and submitted to AgVic, keep a copy for your own reference.

*An example of Clair3 command*

```
run_clair3.sh --bam_fn=${INTERNATIONALID}.sorted.bam \
    --ref_fn=ARS-UCD2.0.fa \
    --threads=24 \
    --platform=${platform}\
    --sample_name=${INTERNATIONALID} \
    --model_path=${model_path} \
    --ctg_name=${chr} \
    --remove_intermediate_dir \
    --gvcf \
    --output=${outputdirectory}
```

Where ${platform} is ont or hifi and ${model_path} is the path to the Clair3 model for your data type.

*An example of compressing command*

```
bgzip -@ $task.cpus ${INTERNATIONALID}.snf
bgzip -@ $task.cpus ${INTERNATIONALID}_SV.vcf  
```

## Calculate read coverage

It is important to know the coverage for a few reasons, one being to ensure compliance with the coverage requirements. Mosdepth can be used to calculate coverage. Coverage should be provided in the checklist (see below).

*An example Mosdepth command*

```
mosdepth --threads $task.cpus ${INTERNATIONALID}${INTERNATIONALID}.sorted.bam
```

## Create md5sum for all Files to be Transferred

An md5sum must be created for all files shared with the consortium. An md5sum is a 128 bit checksum which will be unique for each file. Two non-identical files will not have the same md5sum and therefore the md5sum can be used to cross verify the integrity of a file after download or transfer.


## BovLRC Checklist

- Consult the file submission checklist (provided to all project partners) as well as these specifications before preparing data. Checklist and ID key spreadsheets must then be filled in and submitted via email to [cattleLRC@gmail.com](mailto:cattleLRC@gmail.com).

- The inclusion of genotype information aids the quality checking process and can identify problems with libraries or alignments. If you have BovineSNP50 or BovineHD (or equivalent) data for your samples, it would be very beneficial to share these. Genotype data files should be provided as Illumina GenomeStudio output in TOPTOP (preferred) and FORWARD/FORWARD format. Please contact us if you have Affymetrix or other high-density (>100,000 loci per chip) genotype data and would like to contribute it. Alternatively, short read sequence genotypes can be provided in VCF format. If the animal is part of the 1000 Bull Genomes dataset, just advise us of the animal ID in that dataset. Please inform us if you have any of these data types in the checklist.

## Submission of Files

GVCF files (.g.vcf.gz), md5sum files (.md5), and SNF files (.snf.gz) may be transferred electronically by uploading them to your consortium server account. The following procedure must be followed:

1. Contact Tuan Nguyen ([tuan.nguyen@agriculture.vic.gov.au](mailto:tuan.nguyen@agriculture.vic.gov.au)) and let him know the timing and size of files to be transferred. Only start uploads once AgVic confirms the server has the required capacity.
2. Files must be uploaded to your account, e.g., `username@203.12.194.81:/home/username`.
