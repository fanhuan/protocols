### This protocol is used to set aside non-drosophila reads. 
Two yeast genomes (Candida for while flies and Saccharomyces for lab flies) are included in the reference because there are reads that map equally good to drosophla and yeast based on preliminary analysis (using Saccharomyces only). In order to have reads that are possibly from yeast aside for yeast composition analysis, reads that have multiple mappings are considered non-drosophila reads.
 
1. Construct reference (should use HanseniasporaUvarum as well, will update later)

		cat CandidaKrusei.fa > Fly2Yeasts.fa
		cat Saccharomyces.fa >> Fly2Yeasts.fa
		cat Drosophila_melanogaster.BDGP6.dna_sm.toplevel.fa >> Fly2Yeasts.fa 
		
2. bbsplit (see instructions [here](http://seqanswers.com/forums/showthread.php?t=41288))

	fastq files:

		bbsplit.sh in=/media/backup_2tb/Data/FlyMicrobiome/Drosophila/Trimmomatic/KF5_R#.trimpair.fastq.gz ref=/media/backup_2tb/Data/FlyMicrobiome/Microbes/CandidaKrusei.fa,/media/backup_2tb/Data/FlyMicrobiome/Microbes/Saccharomyces.fa,/media/backup_2tb/Data/FlyMicrobiome/Drosophila/Drosophila_melanogaster.BDGP6.dna_sm.toplevel.fa basename=o%_#.fq ambig2=split outu1=unmapped1.fq outu2=unmapped2.fq
	non-drosophila reads:  
	
		python ~/scripts/ambigous_consensus.py CandidaKrusei,Saccharomyces
		cat ambigous_consensus_1.fq CandidaKrusei_1.fq Saccharomyces_1.fq unmapped1.fq > nonDrosophila_1.fq  
		cat ambigous_consensus_2.fq CandidaKrusei_2.fq Saccharomyces_2.fq unmapped2.fq > nonDrosophila_2.fq
		
	re-split between 3 yeasts(bbsplit_huan.sh):  
	
		bbsplit.sh in=nonDrosophila_#.fq ref=/media/backup_2tb/Data/FlyMicrobiome/Microbes/CandidaKrusei.fa,/media/backup_2tb/Data/FlyMicrobiome/Microbes/Saccharomyces.fa,/media/backup_2tb/Data/FlyMicrobiome/Microbes/HanseniasporaUvarum.fa basename=%_#.fq ambig2=split outu1=unmapped_1.fq outu2=unmapped_2.fq
		bbsplit.sh in=nonDrosophila_#.fq ref=/media/backup_2tb/Data/FlyMicrobiome/Microbes/CandidaKrusei.fa,/media/backup_2tb/Data/FlyMicrobiome/Microbes/Saccharomyces.fa,/media/backup_2tb/Data/FlyMicrobiome/Microbes/HanseniasporaUvarum.fa basename=%.sam ambig2=split
		
		
4. Coverage of the three yeast genomes (min=no ambigous, max= plus all ambigous reads)  

		for ref in CandidaKrusei Saccharomyces HanseniasporaUvarum
		do
		samtools view -bS $ref.sam | samtools sort > sorted_$ref.bam
		samtools mpileup sorted_$ref.bam > coverage_min_$ref.txt
		python ~/scripts/coverage2region_general.py coverage_min_$ref.txt > stats_min_$ref.txt
		#Step 1: map all the ambigous reads to the respective genome using bbmap and same parameters used in bbsplit.
		bbmap.sh in1=../bbsplit/AMBIGUOUS_${ref}_1.fq in2=../bbsplit/AMBIGUOUS_${ref}_2.fq ref=/media/backup_2tb/Data/FlyMicrobiome/Microbes/$ref.fa fastareadlen=500 minhits=1 minratio=0.56 maxindel=20 qtrim=rl untrim=t trimq=6 out=ambg_$ref.sam
		samtools view -bS ambg_$ref.sam | samtools sort > sorted_ambg_$ref.bam
		samtools merge sorted_max_$ref.bam sorted_$ref.bam sorted_ambg_$ref.bam
		samtools mpileup sorted_max_$ref.bam > coverage_max_$ref.txt
		python ~/scripts/coverage2region_general.py coverage_max_$ref.txt > stats_max_$ref.txt
		done
4. kslam  
Annotate all the unmappled reads using kslam and record the major groups.

