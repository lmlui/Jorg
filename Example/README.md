
<h1> Introduction </h1>

This is an example on how to circularize a MAG.  This is one of the MAGs that is in the manuscript.

<h1> Data </h1>
  
Data from this example is from processing the metagenome [SRX3307784](https://www.ncbi.nlm.nih.gov/sra/?term=SRX3307784) and is from this study: [Wilhelm RC, Hanson BT, Chandra S, Madsen E. Community dynamics and functional characteristics of naphthalene-degrading populations in contaminated surface sediments and hypoxic/anoxic groundwater. Environ Microbiol. 2018;20: 3543–3559]( https://doi.org/10.1111/1462-2920.14309).

The trimmed and quality filtered reads are provided for this example on [Zenodo](http://dx.doi.org/10.5281/zenodo.3889132).

<h1> Picking a bin to circularize </h2>

* Look for a bin that has less than 10 contigs. The bin.186 has 4 contigs. Two of the contigs are of decent length at ~241Kb and ~215Kb
* (Optional) Look at classification of the bin and see if the bin is approximately the right length.  The bin.186 is classified in the Patescibacteria phylum (aka Candidate Phyla Radiation phyla) by GTDB-tk.  Species in this taxon tend to have genomes between 0.5-1.2 Mb. Bin.186 is ~0.58 Mb.
* Check the coverage of the longest contig. The coverage is also good at ~162X. We recommend a coverage of at least 30X but 150X or higher improves the odds of success.  Data on the contigs from the `depth.txt file` from MetaBat 2 output is below and the full file can be found in the available files for this example.

```
contigName              contigLen  totalAvgDepth	
SRX3307784_contig_86    241519     162.663	      
SRX3307784_contig_105   215963     164.673	      
SRX3307784_contig_435   99259      162.843	      
SRX3307784_contig_4243  22132      158.726	      
```

<h1> Run jorg </h1>

We'll start with a kmer baiting value of 33 and 20 iterations.  Usually 5-10 iterations is a good place to start to get a feel for how many iterations you may need. Since the average coverage was ~162X, we want to pick a minimum coverage value that is ~75-80% of that.  We'll try 130. 

The file of the original bin is included (`bin.186.fa`) and the trimmed and quality filtered reads can be downloaded from [Zenodo](http://dx.doi.org/10.5281/zenodo.3889132).

`jorg 33 bin.186.fa SRX3307784_clean.fastq.gz 130 20`

<h1> Evaluate Iterations </h1>

Check the `iterations.txt` file output (in the `OutputFiles` directory here in this Github repo).  You can see that the bin starts to improve and probably reaches circularization between iterations 13-19. We'll check Iteration 19 for circularity.

```
contig_name    length	GC_percent    cumulative_length
Iteration 13   
bin.186_c1     581912   46.26          581912
Iteration 14                           
bin.186_c2     581382   46.25          581382
Iteration 15                           
bin.186_c3     583110   46.27          583110
Iteration 16                           
bin.186_c1     342043   46.06          342043
bin.186_c2     241772   46.58          583815
Iteration 17                           
bin.186_c1     578886   46.25          578886
bin.186_c2       2865   50.89          581751
bin.186_c4       1391   47.23          583142
Iteration 18                           
bin.186_c1     581826   46.27          581826
bin.186_c7       1391   47.23          583217
Iteration 19                           
bin.186_c1     581938   46.26          581938
Iteration 20                           
bin.186_c1     579241   46.24          579241
bin.186_c3       3080   50.88          582321
```

<h1> Check for Circularization </h1>

The file `19.fasta` in the `Iterations` directory contains the 19th assembly (you can find this file in the `OutputFiles` directory here in this Github repo).  Use the `make_assembly_db` script to check for a long repeat at the end that is longer than all other repeats.

```
make_assembly_db 19.fasta
```

Taking a look at the `19.reduced` file (in the `OutputFiles` directory here in this Github repo), we see that there is a significant repeat on the ends that is greater than 100 bp (295 bp long) and is longer than any other repeat in the genome.

```
# Fields: query id, subject id, % identity, alignment length, mismatches, gap opens, q. start, q. end, s. start, s. end, evalue, bit score, query length, subject length
# batch 0
bin.186_c1      bin.186_c1      100.00  581938  0       0       1       581938  1       581938  0       9.2e+05 581938  581938  581938
bin.186_c1      bin.186_c1      100.00  295     0       0       1       295     581644  581938  8.7e-130        468     581938  581938  295
bin.186_c1      bin.186_c1      100.00  295     0       0       581644  581938  1       295     8.7e-130        468     581938  581938  295
```

<h1> Final Checks </h1>

* Trim the end of the sequence and use Pilon as a final check for misassemblies. 

* Use `fasta_rotate` to rotate the genome half its length and use Pilon again. Pilon may shrink the genome slightly (as in this case) to improve the overlap.

* Check for 16S, 23S, 5S, tRNAs, and RNase P RNA in the final genome to ensure that the genome was not falsely circularized.
