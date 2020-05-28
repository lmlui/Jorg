# Jorg
A MAG Circularization Method
By Lauren Lui, Torben Nielsen, and Adam Arkin

This is a method to help circularize genomes from shotgun metagenomics data. We named this project Jorg after J&ouml;rgmungandr, <em>aka</em> the Midgard Serpent, from Norse mythology. It is an example of an ouroboros, a snake biting its own tail.

Contact Lauren with questions (lmlui at lbl dot gov)

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
</ul>
<h2> Optional for final checks after circularization </h2>
<ul>
  <li>Pilon</li>
  <li>PROKKA or Infernal </li>
</ul>
  
<h3>Installing MIRA</h3>

We used MIRA 5.0rc1.  Binaries and source can be found here: https://github.com/bachev/mira/releases/tag/V5rc1.  We recommend that you download and use the binaries.  You can learn more about MIRA in the [Definitive Guide to MIRA]( http://mira-assembler.sourceforge.net/docs/DefinitiveGuideToMIRA.html) written by its author, Bastien Chevreux.  Note that the versions on MIRA on SourceForge are not up to date.

<h3>Installing seqtk</h3>
We recommend installing seqtk by either of these methods:
<ol>
  <li>Install using conda. `conda install seqtk`, or if you have not installed the bioconda channel, `conda install -c bioconda seqtk`  </li>
  <li>Install from source.  Follow the directions from the Github site: https://github.com/lh3/seqtk.</li>
</ol>



<h1> Tutorial </h1>
<h2> Pick a bin to circularize </h2>
We recommend picking a bin that has fewer than 10 contigs.  However, you can pick any bin and likely it will be improved using  this method.
<h2> Use mirabait to map reads to a bin </h2>
We recommend starting with a minimum coverage of 75% of the top contig.
<h2> Reassemble reads using MIRA </h2>
Generate a manifest file.
We recommend starting with a kmer value of 31.
<h2> Check assembly features </h2>
Check the iterations to see if the contigs have gotten longer and if there is any potential contamination
<h2> Iterate as necessary </h2>
<h2> Final checks for non-coding RNAs </h2>
Run PROKKA or Infernal to look for tRNAs, rRNAs, and RNase P RNA.

