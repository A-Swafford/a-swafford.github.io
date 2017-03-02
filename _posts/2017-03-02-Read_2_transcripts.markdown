---
layout: post
title: "Identifying Reads that Assemble a Transcript"
subtitle: "A quick way to assess quality of specific transcripts"
date: 2017-3-02 10:10:00
author: "Andrew Swafford"
comments: true
category: Tutorials
tags: [Illumina,Read,Transcript,Tutorial]
---  

## Programs & Tools needed
* [Bowtie2 v2.3.0](https://sourceforge.net/projects/bowtie-bio/files/bowtie2/2.3.0/)
* [Samtools v1.3.1](http://www.htslib.org/download/)
* [Integrated Genomics Viewer (IGV) v2.3](http://software.broadinstitute.org/software/igv/)

## How to map reads back to the transcripts
### Prepare input files
Firstly, you will need to organize several input files: (1) The bowtie2 index of your focal transcripts, and (2) the read files from which those transcripts were assembled.  
Your focal transcripts should be unaligned in fasta format and placed in a clean directory for easy access.  For the purposes of this tutorial, we'll call this fasta file of our focal transcripts `focal_t.fasta`.  Open a command line editor, navigate to the directory where `focal_t.fasta` is located and execute the following command:
```
bowtie2-build focal_t.fasta focal_transcripts
```
Here, `focal_transcripts` can be replaced with whatever you would like, just be sure to replace instances of 'focal_t.fasta' in this tutorial with whatever name you chose.  Double check that the command has worked and built the bowtie2 index files (.bt2) in your working directory.

Next, ensure you either have your readfiles in the same directory as your bowtie2 index files, or you know the exact path to your readfiles.  In this tutorial, I am using paired end, Illumina reads in fastq format with my forward reads in a file named `read1.fq` and reverse reads in a file named `read2.fq`.  If your reads are in **fasta** format, replace the `-q` option with a `-a` option. 

### Map reads and visualize outputs

Once you have your inputs prepared, use the following command to start mapping:
```
bowtie2 --local --no-unal -x focal_transcripts -q -1 read1.fq -2 read2.fq > focal_transcripts.bam
```
Depending on how large your reads and transcripts are, this may take some time.  Once finished, we need to prepare the output for visualization in IGV with a few further commands.  First, we need to sort the reads we've mapped:
```
samtools sort focal_transcripts.bam -o sorted_focal_t.sorted
```
Next, we create an index of these reads, giving IGV positional data about where the reads map on our focal transcripts:
```
samtools index sorted_focal_t.sorted
```
This will output an index file named `sorted_focal_t.sorted.bai`.

:warning:It is important to note that *ALL* of these bowtie2 outputs must stay in the same directory as your original file of focal transcripts. This is needed for IGV to correctly read the index files we're making.:warning:

Lastly, we will need to prepare the original transcripts for visualization in Integrated Genomics Viewer (IGV) through it's own menu.  Open IGV (the first initialization may take a while depending on your processor and internet connection) and navigate to the "Create .genome File" option under the Genomes menu: `Genomes > "Create .genome File"`.  A prompt will appear with several required fields, here is how you can fill them in:  
* `Unique Identifier`: Any unique string of alphanumeric characters. (e.g. 12345678ABCDE)  
* `Descriptive Name`: A human readable, short, name of your 'genome'. (e.g. Cyclopsin_focal_transcripts)
* `FASTA File`: Either type in the path to your original transcript file, `focal_t.fasta`, or use thier UI to navigate to and select it.

You can ignore the rest of the input fields.  Once done providing the required information, save your genome somewhere accessible (the default location works well) and it will automatically open.  If you have more than one focal transcripts, nothing will appear until you choose a locus (i.e. transcript) to examine.  You can do this at the top of the IGV user interface, in the second drop down menu.  The first menu is the current genome, and selecting a locus other than 'all' will create a track with that locus' sequence.

Lastly, load in your reads by navigating to `File > Load from File` and selecting your sorted bam file, e.g. `sorted_focal_t.sorted` (*not the index file*!).  With any luck, you should see the reads appear in the IGV window mapped onto your set of focal transcripts.

If this does not work, ensure all your bowtie2 outputs are in the same folder, and that you have actually selected the sorted bam file, and not an index file.