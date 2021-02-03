# Jorg
A MAG Circularization Method
By Lauren Lui, Torben Nielsen, and Adam Arkin

<p align="center">
<img src="https://genomics.lbl.gov/~lmlui/Images/StrikingImage-Jorg5b.png" width="50%" >
</p>

This is a method to help circularize genomes from shotgun metagenomics data. We named this project Jorg after J&ouml;rgmungandr, <em>aka</em> the Midgard Serpent, from Norse mythology. It is an example of an ouroboros, a snake biting its own tail.


Contact Lauren with questions (lmlui at lbl dot gov).


Please see the manuscript ["A method for achieving complete microbial genomes and better quality bins from metagenomics data" on bioRxiv](https://www.biorxiv.org/content/10.1101/2020.03.05.979740v2.full) and cite it if you use this method.

We thank Sean Jungbluth for helping extend the code for use in the DOE Systems Biology KnowledgeBase.

<h1>Overview of Method </h1>

<p align="center">
<img src="https://www.biorxiv.org/content/biorxiv/early/2020/03/07/2020.03.05.979740/F1.large.jpg" width="50%">
</p>

<font size="6">
<b>This method assumes that you already have a pipeline that you like to use for assembling your metagenomes and creating bins.  We like to use BBtools for trimming and read filtering, metaSPAdes for assembly, and MetaBat 2 for binning. What we describe here is steps D-G in the above figure.</b>
</font>

<h1> Installation </h1>

<h2> System Requirements </h2>
<ul>
  <li>At least 8 Gb of RAM</li>
  <li>At least enough free disk space as 2x the size of the input read file</li>
</ul>

<h2> Dependencies </h2>
<ul>
  <li>MIRA</li>
  <li>seqtk</li>
  <li>BWA (optional)</li>
  <li>LAST (optional)</li>
</ul>
<h2> Optional for final checks after circularization </h2>
<ul>
  <li>Pilon</li>
  <li>Infernal </li>
</ul>

<h3>Installing MIRA</h3>

* We used MIRA 5.0rc1.  Binaries and source can be found here: https://github.com/bachev/mira/releases/tag/V5rc1.  We recommend that you download and use the binaries.  You can learn more about MIRA in the [Definitive Guide to MIRA]( http://mira-assembler.sourceforge.net/docs/DefinitiveGuideToMIRA.html) written by its author, Bastien Chevreux.  Note that the versions of MIRA on SourceForge are not up to date.
* After unpacking or installing MIRA, make sure that mirabait and mira are in your path. These are found in the bin directory when you download MIRA.  

<h3>Installing seqtk</h3>
We recommend installing seqtk by either of these methods:

1. Install using conda. `conda install -c bioconda seqtk`
1. Install from source.  Follow the directions from the [seqtk Github site](https://github.com/lh3/seqtk).

After installing seqtk make sure that it is in your path.

<h3>Installing BWA (optional)</h3>

Information on BWA can be found here: http://bio-bwa.sourceforge.net/.  You can install bwa by either of these methods

1. Download from SourceForge (https://sourceforge.net/projects/bio-bwa/). Unzip and go into the directory. Type `make`.
1. Using conda. `conda install bwa`.


BWA is used to determine the coverage of the bin.  Most binning software should also output coverage values. However not all binning software will output the same values, especially if you are changing parameters!  We recommend using BWA or checking the contig coverage values output by MetaBat 2 (`<contigfile>.depth.txt` file).

<h3>Installing LAST (optional)</h3>

LAST is designed to find regions of sequence similarity in large datasets.  We build a LAST DB to help determine if a genome has been circularized.  

1. Installation using conda. `conda install -c bioconda last`
1. See the [LAST website](http://last.cbrc.jp/) for more information and [directions for manual installation](http://last.cbrc.jp/doc/last.html).



<h3>Installing Pilon (optional) </h3>

You can find the source and binaries for Pilon on the [Pilon Github page](http://github.com/broadinstitute/pilon/wiki) by the Broad Institute. You can also install Pilon using conda. Although this is optional, it is useful for checking for any misasssemblies not corrected by the MIRA assembly.

<h3> Installing Infernal (optional) </h3>

Binaries and directions for installing Infernal can be found on the [Infernal homepage](http://eddylab.org/infernal/).

One of the final checks is to find a full complement of ribosomal RNAs (16S, 23S, 5S), tRNAs (all amino acids represented) and RNase P RNA. [PROKKA](http://github.com/tseemann/prokka) can be used to find tRNAs and rRNAs and [tRNAscanSE](http://lowelab.ucsc.edu/tRNAscan-SE/) can be used to find tRNAs, but you will need to use Infernal to find RNase P RNA. There are two types of bacterial RNase P RNA and two types of archaeal RNase P RNA.  Each have different models in RFAM (https://rfam.org/family/RF00010).



<h1> General guide </h1>

<h2> Assemble your metagenome and bin contigs</h2>

We use [BBtools](https://jgi.doe.gov/data-and-tools/bbtools/) for cleaning the reads, [SPAdes](http://cab.spbu.ru/software/spades/) for assembly, and [MetaBat 2](https://bitbucket.org/berkeleylab/metabat) for binning.

<h2> Pick a bin to circularize</h2>
Criteria for picking a bin:
<ol>
  <li> <b> A bin with <10 contigs. </b> We recommend picking a bin that has fewer than 10 contigs to increase success, but you can pick any bin and likely it will be improved using this method. We made exceptions for bins that looked promising, such as a bin with many contigs, but with one or two large contigs that comprise most of the bin’s sequence length  </li>
  <li> <b> The bin has an average coverage >30X. </b> Higher coverage is better for success.  If you didn't get the coverage from your binning output, use bwa to map reads to the longest contig to help determine the coverage. Typically this is a good coverage estimate.
 </ol>   
<h2> Use the Jorg script to iterate mapping reads to the bin with mirabait and reassemble with MIRA. Note: typically it takes at least several hours for each iteration of Jorg to run, so jobs using a high number of iterations may take multiple days.</h2>

  <h3> Overview</h3>
 

  The Jorg script will do the following:

1. Create a manifest file for MIRA
1. Use `mirabait` to map reads to a bin
1. Use `mira` to reassemble the reads
1. Repeat steps 2 and 3 until reaching the number of iterations indicated

  <h3>Parameters</h3>

  The `jorg` script requires that you specify a kmer value for baiting the reads and a minimum coverage value for filtering contigs.  Here are some guidelines for choosing these parameters:

 <h4>Kmer value for baiting</h4>
 mirabait requires a kmer value for baiting reads.  We recommend starting with 31 or 33. Increasing the kmer value will make the read recruitment more strict if you are worried that you are picking up reads that do not belong in your bin.

 <h4>Minimum coverage</h4>

 During each iteration contigs that do not meet the minimum coverage are filtered out. We recommend starting with a minimum coverage of 75% of the top contig. Use BWA to map reads to get coverage values or see the `totalAvgDepth` field in the `<contigfile>.depth.txt` file in MetaBat 2 output.

 <h3>Using the script</h3>

* Make sure that the `manifest_template.conf` file is in the same directory.
* An example of running the script is

```bash
jorg -b bin.186.fa -r SRX3307784_clean.fastq.gz -k 33 -c 50 -i 5 --high_contig_num no 
```
where 33 is the kmer value, bin.186.fa is the fasta file with contigs, SRX3307784_clean.fastq.gz are your interleaved sequencing reads that have been trimmed and quality checked, 50 is the minimum coverage value, and 5 is the number of iterations.
* Output:
  * `mirabait.log` - log file from `mirabait`.  This is extremely helpful if you are getting errors at this step.
  * `mira.log` - log file from `mira`.
  * `iterations.txt` - contig stats for the assembly from each iteration
  * Directory called `Iterations` with the assemblies from each iteration
  * `<binID>.out.fasta` -  Assembly from the last iteration


<h2> Check assembly stats and repeat as necessary</h2>

* The `jorg` script will output a file called `iterations.txt` with contig stats.  Check this file to see if the contigs are getting longer.  You may also want to remove contigs that appear to be contamination, e.g. those that are short and are not extending, before the next set of iterations.  If you need to continue iterating, use the `<binID>.out.fasta` file as input to the next round.

* Note that sometimes the best assembly might not be the final iteration.

* Stop when you reach one of these three outcomes:

<h3>Circularization</h3>
You find a single contig with a significant - and exact - repeat at the ends. In addition, we required that the repeat be at least 100 nt in length, is longer than any other repeat in the contig, and does not match any of the other repeats.

See the script `circle_check_using_last` for an automated look at the location and length of repeats in a genome.  This script requires that LAST is installed.

<h3>Idempotence</h3>
You observe no change in the assembled contigs after a round of read pair extraction and reassembly with MIRA.

<h3>Chaos</h3>
There are cases where a bin is shattered into a multitude of pieces. We are not certain as to the exact cause, but this result is likely due to misassemblies from the initial SPAdes assembly (discussed in more depth in the manuscript). Chaos appears strongly correlated with GC and tends to occur more often when the GC content is high. We believe that Chaos bins are caused by lack of read coherence in the contigs.

<h2> Use Pilon as a final check for misasssemblies </h2>

* After circularizing a contig, we do final checks for misassemblies with Pilon. We use Pilon on the contig and then we rotate it by half the length to ensure that the ends were in the middle and apply Pilon again. See the script `fasta_rotate`. We rotate the genomes because Pilon is not capable of covering the ends of a contig.

* Pilon has a detailed [Github wiki page](https://github.com/broadinstitute/pilon/wiki/Requirements-&-Usage) that details how to use it and understand its output.

<h2> Final checks for non-coding RNAs </h2>

To look for tRNAs, rRNAs, and RNase P RNA, scan genomes with models from RFAM by using `cmsearch` from Infernal. If your genome is missing any of these RNAs, it might be falsely circularized.
