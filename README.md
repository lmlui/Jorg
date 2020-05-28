# Jorg
A MAG Circularization Method
By Lauren Lui, Torben Nielsen, and Adam Arkin

This is a method to help circularize genomes from shotgun metagenomics data. We named this project Jorg after J&ouml;rgmungandr, <em>aka</em> the Midgard Serpent, from Norse mythology. It is an example of an ouroboros, a snake biting its own tail.

Contact Lauren with questions (lmlui at lbl dot gov).

Please see the manuscript ["A method for achieving complete microbial genomes and better quality bins from metagenomics data" on bioRxiv](https://www.biorxiv.org/content/10.1101/2020.03.05.979740v2.full) and cite it if you use this method.

<h1> General Overview of Method </h1>

<p align="center">
<img src="https://www.biorxiv.org/content/biorxiv/early/2020/03/07/2020.03.05.979740/F1.large.jpg" width="50%">
</p>

<font size="6">
<b>This method assumes that you already have a pipeline that you like to use for assembling your metagenomes and creating bins.  We like to use BBtools for trimming and read filtering, metaSPAdes for assembly, and MetaBat 2 for binning. What we describe here is steps D-G in the above figure.</b>
</font>

<h1> Installation </h1>

<h2> Dependencies </h2>
<ul>
  <li>MIRA</li> 
  <li>seqtk</li>
  <li>BWA (optional)</li>
</ul>
<h2> Optional for final checks after circularization </h2>
<ul>
  <li>Pilon</li>
  <li>Infernal </li>
</ul>
  
<h3>Installing MIRA</h3>

* We used MIRA 5.0rc1.  Binaries and source can be found here: https://github.com/bachev/mira/releases/tag/V5rc1.  We recommend that you download and use the binaries.  You can learn more about MIRA in the [Definitive Guide to MIRA]( http://mira-assembler.sourceforge.net/docs/DefinitiveGuideToMIRA.html) written by its author, Bastien Chevreux.  Note that the versions of MIRA on SourceForge are not up to date.
* After installing MIRA, make sure that mirabait and mira are in your path.  

<h3>Installing seqtk</h3>
We recommend installing seqtk by either of these methods:

1. Install using conda. `conda install seqtk`. If you have not installed the bioconda channel, then use `conda install -c bioconda seqtk`
1. Install from source.  Follow the directions from the seqtk Github site: https://github.com/lh3/seqtk.

After installing seqtk make sure that it is in your path.

<h3>Installing BWA (optional)</h3>
Information on BWA can be found here: http://bio-bwa.sourceforge.net/.  You can install bwa by
<ol>
  <li> Download from SourceForge (https://sourceforge.net/projects/bio-bwa/). Unzip and go into the directory. Type "make".
  <li> Using conda. `conda install bwa`.
</ol>

BWA is used to determine the coverage of the bin.  Most binning software should also output coverage values. However not all binning software will output the same values, especially if you are changing parameters!  We recommend using BWA or checking the coverage values output by MetaBat 2.

<h3>Installing Pilon (optional) </h3>
You can find the source and binaries for Pilon on the Pilon Github page (http://github.com/broadinstitute/pilon/wiki) by the Broad Institute. You can also install Pilon using conda. Although this is optional, it is useful for checking for any misasssemblies not corrected by the MIRA assembly.

<h3> Installing Infernal (optional) </h3>

Binaries and directions for installing Infernal can be found on the [Infernal homepage](http://eddylab.org/infernal/).

One of the final checks is to find a full complement of ribosomal RNAs (16S, 23S, 5S), tRNAs (all amino acids represented) and RNase P RNA. PROKKA (http://github.com/tseemann/prokka) can be used to find tRNAs and rRNAs and tRNAscanSE (http://lowelab.ucsc.edu/tRNAscan-SE/) can be used to find tRNAs, but you will need to use Infernal to find RNase P RNA. There are two types of bacterial RNase P RNA and two types of archaeal RNase P RNA.  Each have different models in RFAM (https://rfam.org/family/RF00010).



<h1> Tutorial </h1>
<h2> Assemble your metagenome and bin contigs</h2>
We use BBtools(https://jgi.doe.gov/data-and-tools/bbtools/) for cleaning the reads, SPAdes(http://cab.spbu.ru/software/spades/) for assembly, and MetaBat 2(https://bitbucket.org/berkeleylab/metabat) for binning.

<h2> Pick a bin to circularize</h2>
Criteria for picking a bin:
<ol>
  <li> <b> A bin with <10 contigs. </b> We recommend picking a bin that has fewer than 10 contigs to increase success, but you can pick any bin and likely it will be improved using this method. We made exceptions for bins that looked promising, such as a bin with many contigs, but with one or two large contigs that comprise most of the bin’s sequence length  </li>
  <li> <b> The bin has an average coverage >30X. </b> Higher coverage is better for success.  If you didn't get the coverage from your binning output, use bwa to map reads to the longest contig to help determine the coverage. Typically this is a good coverage estimate.
 </ol>   
<h2> Use the Jorg script to iterate mapping reads to the bin with mirabait and reassemble with MIRA</h2>
  <h3> Overview</h3>
  The Jorg script will do the following:
  <ol>
  <li> Create a manifest file for MIRA </li>
  <li> Use mirabait to map reads to a bin </li>
  <li> Use MIRA to reassemble the reads </li>
  <li> Repeat steps 2 and 3 until reaching the number of iterations indicated
  </ol>
  
  <h3>Parameters</h3>
  The Jorg script requires that you specify a kmer value for baiting the reads and a minimum coverage value for filtering contigs.  Here are some guidelines for choosing these parameters:
 <h4>Kmer value for baiting</h4>
 mirabait requires a kmer value for baiting reads.  We recommend starting with 31 or 33. Increasing the kmer value will make the read recruitment more strict if you are worried that you are picking up reads that do not belong in your bin.

 <h4>Minimum coverage</h4>
 During each iteration contigs that do not meet the minimum coverage are filtered out. We recommend starting with a minimum coverage of 75% of the top contig.
 
 <h3>Using the script</h3>
*Make sure that the `manifest_template.conf` file is in the same directory.  
*Then run

```bash
jorg 33 bin1.fasta myreads.fastq.gz 200 10
```
where 33 is the kmer value, bin1.fasta is the fasta file with contigs, myreads.fastq.gz are your interleaved sequencing reads that have been trimmed and quality checked, 200 is the minimum coverage value, and 10 is the number of iterations.

<h2> Check assembly stats and repeat as necessary</h2>
*The jorg script will output a file called "iterations.txt" with contig stats.  Check this file to see if the contigs are getting longer.  You may also want to remove contigs that appear to be contamination, e.g. those that are short and are not extending, before the next set of iterations.  If you need to continue iterating, use the <binID>.out.fasta file as input to the next round.
  
*Stop when you reach one of these three outcomes:
  
<h3>Circularization</h3> You find a single contig with a significant - and exact - repeat at the ends. In addition, we required that the repeat be at least 100 nt in length, is longer than any other repeat in the contig, and does not match any of the other repeats.

<h3>Idempotence</h3> You observe no change in the assembled contigs after a round of read pair extraction and reassembly with MIRA. 

<h3>Chaos</h3> There are cases where a bin is shattered into a multitude of pieces. We are not certain as to the exact cause, but this result is likely due to misassemblies from the initial SPAdes assembly (discussed in more depth in the manuscript). Chaos appears strongly correlated with GC and tends to occur more often when the GC content is high. We believe that Chaos bins are caused by lack of read coherence in the contigs.

<h2> Use Pilon as a final check for misasssemblies </h2>
After circularizing a contig, we do final checks for misassemblies with Pilon. We use Pilon on the contig and then we rotate it by half the length to ensure that the ends were in the middle and apply Pilon again. We rotate the genomes because Pilon is not capable of covering the ends of a contig. 

<h2> Final checks for non-coding RNAs </h2>
Run Infernal to look for tRNAs, rRNAs, and RNase P RNA.

