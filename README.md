# dedup

dedup selectively removes redundant reads from paired-end libraries generated with random base molecular indices.

## About:

Random base molecular indices (also called unique molecular identifiers or UMIs) are molecular tags that can be introduced adjacent to your fragments of interest during sequencing library preparation. Depending on the number of random bases (or Ns) added around your insert, it is possible to retain otherwise identical reads generated from separate starting molecules while filtering out inappropriate duplicates, such as from PCR amplification. Many library prep kits (especially for RNA-Seq) now offer these indices, and we find them to be essential for PRO-Seq, PRO-Cap, and other sequencing methods that observe promoter proximal Pol II. This script was developed to handle the task of "deduplicating" our paired-end datasets and can be used as long as the molecular index is present and untrimmed in the pre-alignment FASTQ files.

This script accepts mapped paired-end reads in SAM format, extracts the molecular indices flanking these reads from pre-alignment R1 and R2 FASTQ files, sorts all entries by genomic position and molecular index base composition, and then collapses non-unique entries. This script outputs a deduplicated bed file, which is otherwise ready for use with [UCSC Genome Browser](http://genome.ucsc.edu/) or utilities such as [bedtools](http://bedtools.readthedocs.io/en/latest/) or [deepTools](http://deeptools.readthedocs.io/en/latest/).

Default parameters are hard-coded at the beginning of the script. We recommend manually changing `Threads` and `BufferSize` to match your environment. The other parameters should instead be adjusted as needed on a per-run basis.

## Usage:

dedup is a bash script that relies on common Linux utilities, sort, awk, [samtools](https://github.com/samtools/samtools/releases/latest), and [bedtools](http://bedtools.readthedocs.io/en/latest/). It is known to work with samtools v1.4.1 and bedtools v2.26.0.

Before using dedup, you must first perform an alignment. While these steps will vary depending on your experiment and software preferences, a possible paired-end workflow is to eliminate 3' sequencing adapters from both reads with [Trim Galore](https://github.com/FelixKrueger/TrimGalore/releases/latest), and then perform an alignment with [bowtie](https://github.com/BenLangmead/bowtie/releases/latest) using the `--trim5 <n>` and `--trim3 <n>` options, where `<n>` is the number of bases in your UMIs. Regardless of your choice, your alignment output must be in SAM format.

dedup is intended to be run from the command-line and expects the following syntax:

```
dedup -S <reads.sam> -1 <R1.fq> -2 <R2.fq> [optional parameters]
```

### Required parameters:

```
-S <reads.sam>	paired-end alignment file in SAM format
-1 <R1.fq>	R1 reads with untrimmed 5' n bp molecular index
-2 <R2.fq>	R2 reads with untrimmed 5' n bp molecular index
```

This script is intended to work with paired-end sequencing data in SAM format. Output will be saved to the same directory as this file.

This script also extracts the first -n bases of the R1 and R2 FASTQ files. These files can be gzipped or uncompressed, and may be stored in a different directory. While it is highly recommended to trim adapter sequences from the 3' ends of reads before aligning, this script will only look at the first -n bases of `<R1.fq>` and `<R2.fq>`.

Because some SAM files inappropriately contain whitespace in the QNAME field, this script is hard-coded to use `\t` as its expected field separator. As long as the QNAMEs match between the SAM and FASTQ files, this should be acceptable.

### Optional parameters:

```
-n <int>	molecular index length [default 4]
-g 		use if R1 and R2 files are gzipped
-f		use to flip strands of final BED file
-p		use to subtract 1 nt from 3' pause site for PRO-Seq
-k		use to keep temporary files when troubleshooting
-t <int>	number of threads for sort [default 8]
-m <size>	size of memory buffer per thread for sort [default 1G]
```

Depending on your environment, you will likely need to change -t (the number of CPU threads) and -m (the memory buffer used for sort). Because these options are hard-coded, you should adjust them to match your computer. These are used for the sort and samtools sort utilities.

Another parameter that will likely require adjustment is -n (the number of random bases flanking your insert). Although this script assumes that equal length molecular indices are present on both R1 and R2, it will work as long as -n is large enough to encompass your longest UMI, but not longer than your shortest read after adapter-trimming. For example, if you have a single UMI of 8 nt and a 2 nt spacer before your sequences begin on R1, your alignment should use the parameters `--trim5 10 --trim3 0` and you would set dedup to use the option `-n 10`.

If you experience issues or your output is unusually small in size, -k can be used to retain temporary files. The INDICES file is generated from your FASTQ files and should contain the QNAME, the R1 UMI sequence, and the R2 UMI sequence for all reads (not just mapped reads). The MAPPED_READS file is generated from your SAM file and should contain the QNAME and genomic position information of your mapped reads. Finally, the MAPPED_READS_AND_INDICES file is generating by joining lines with matching QNAMEs from the first two temporary files. This temporary file will ultimately be sorted by genomic position and collapsed to generate a deduplicated bed file. If dedup fails after generating the first two temporary files, please check if the QNAME formats match. If your output contains very few reads, make sure your FASTQ files actually contain molecular indices, and that the full length of these indices are present in the first temporary file.

### Output:

dedup outputs a deduplicated bed file, which is ready for the next steps in your pipeline.

## Citation:

dedup was developed by [Kyle Nilson](https://github.com/kylenilson) in the lab of [David Price](https://github.com/P-TEFb) at the University of Iowa (https://price.lab.uiowa.edu/). It was inspired by [dqRNASeq](http://www.biooscientific.com/Next-Gen-Sequencing/Illumina-RNA-Seq-Library-Prep-Kits/NEXTflex-qRNA-Seq-Kit), developed by Weihong Xu.

It was first described in the following manuscript:

```
Soon....
```
