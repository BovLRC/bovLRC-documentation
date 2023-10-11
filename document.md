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
7. If you have a breed not previously LR sequenced, then consider the possibility of doing at least one animal at higher coverage (and lengths) and contact Ben Rosen (ben.rosen@usda.gov) at the Pan Genome Consortium project to discuss breed inclusion.
8. SNP chip genotypes preferably available for sequenced animals (higher density preferable) to check genotype call accuracy. For some, there may be short read data available, this could provide an alternate mechanism.
9. Provide high quality records of meta-data of samples sequenced
10. Lab protocols: Participants can choose ONT or PacBio sequencing. QC committee members have agreed to share their experiences with these technologies. Email inquiries to cattleLRC@gmail.com

## Run 1 Process

**Figure 1:** Flowchart of the agreed process for run 1 of the Bovine Long Read Consortium.

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
```shell
filtlong --min_length 200 ${input}.fastq.gz | bgzip > ${output}.fastq.gz
porechop_abi -abi -i ${input}.fq.gz -o ${output}_QC.fq.gz
