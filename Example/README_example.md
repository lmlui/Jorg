
<h1> Introduction </h1>

This is an example on how to circularize a MAG.  This is one of the MAGs that is in the manuscript.

<h1> Data </h1>
  
Data from this example is from the dataset 

<h1> Picking a bin to circularize </h2>

* Look for a bin that has less than 10 contigs. The bin.186 has 4 contigs. Two of the contigs are of decent length at ~241Kb and ~215Kb
* (Optional) Look at classification of the bin and see if the bin is approximately the right length.  The bin.186 is classified in the Patescibacteria phylum (aka Candidate Phyla Radiation phyla) by GTDB-tk.  Species in this taxon tend to have genomes between 0.5-1.2 Mb. Bin.186 is ~0.58 Mb.
* Check the coverage of the longest contig. The coverage is also good at ~162X. We recommend a coverage of at least 30X but 150X or higher improves the odds of success.  Data on the contigs from the depth.txt file from MetaBat 2 output:
```
contigName              contigLen  totalAvgDepth	
SRX3307784_contig_86    241519     162.663	      
SRX3307784_contig_105   215963     164.673	      
SRX3307784_contig_435   99259      162.843	      
SRX3307784_contig_4243  22132      158.726	      
```

<h1> Run jorg </h1>

We'll start with a kmer baiting value of 33 and 10 iterations.  Since the average coverage was ~162X, we want to pick a minimum coverage value that is ~75-80% of that.  We'll try 130.

`jorg 33 bin.186.fa SRX330 130 10`

<h1> Evaluate Iterations </h1>

Check the `iterations.txt` file output.  You can see that the bin starts to improve and probably reaches circularization between iterations 6-8.

```
contig_name    length	GC_percent    cumulative_length
Iteration 1    
bin_186_c1     579736   46.24        579736
Iteration 2                          
bin_186_c1     581681   46.27        581681
Iteration 3                          
bin_186_c1     339699   46.05        339699
bin_186_c2     242284   46.59        581983
Iteration 4                          
bin_186_c1     340017   46.05        340017
bin_186_c2     242519   46.59        582536
Iteration 5                          
bin_186_c1     340373   46.05        340373
bin_186_c2     242525   46.59        582898
Iteration 6                          
bin_186_c1     580209   46.25        580209
bin_186_c6       1391   47.23        581600
Iteration 7                          
bin_186_c1     579241   46.24        579241
Iteration 8                          
bin_186_c1     580957   46.26        580957
Iteration 9                          
bin_186_c1     338135   46.03        338135
bin_186_c3     242418   46.59        580553
bin_186_c6       1391   47.23        581944
Iteration 10                         
bin_186_c1     339699   46.05        339699
bin_186_c2     242284   46.59        581983
```

<h1> Check for Circularization </h1>

Use the `make_assembly_db` script to check for a long repeat at the end that is longer than all other repeats.

<h1> Final Checks </h1>
* Check for 16S, 23S, 5S, tRNAs, and RNase P RNA.
