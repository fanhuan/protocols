### This protocol is used to set aside non-drosophila reads. 
Two yeast genomes (Candida for while flies and Saccharomyces for lab flies) are included in the reference because there are reads that map equally good to drosophla and yeast based on preliminary analysis (using Saccharomyces only). In order to have reads that are possibly from yeast aside for yeast composition analysis, reads that have multiple mappings are considered non-drosophila reads.
 
1. Construct reference

		cat CandidaKrusei.fa > Fly2Yeasts.fa
		cat Saccharomyces.fa >> Fly2Yeasts.fa
		cat Drosophila_melanogaster.BDGP6.dna_sm.toplevel.fa >> Fly2Yeasts.fa 
		
2. Index for bwa
				
		bwa index Fly2Yeasts.fa 
		
	
