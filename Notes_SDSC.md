Gene calling.  
The JGI way: "The identification of protein-coding genes is per- formed using a consensus of four different ab initio gene prediction tools: prokaryotic GeneMark.hmm (v. 2.8) [11], MetaGeneAnnotator (v. Aug 2008) [12], Prodigal (v. 2.6.2) [13] and FragGeneScan (v. 1.16) [14]. The predictions from all the tools are combined and protein-coding genes with translations shorter than 32 amino acids are deleted. A majority rule-based decision schema is then followed in order to select gene calls. When there is a tie between two or more different gene models, selection is based on the prefer- ence order of gene callers determined by benchmark- ing of the individual gene finders on simulated metagenomic datasets (GeneMark > Prodigal > Meta- GeneAnnotator > FragGeneScan)."  
My way:   
i. Used [MetaGeneAnnotator](http://exon.gatech.edu/GeneMark/meta_gmhmmp.cgi) only. Note: check "Protein sequence" in the "Output options".  
ii. filter for length >= 32  

		python MetaGeneMark_lengthFilter.py prefix 32
		
iii.COG calling: rpsblast
