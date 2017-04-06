### This protocol is used to set aside non-drosophila reads. 
Two yeast genomes (Candida for while flies and Saccharomyces for lab flies) are included in the reference because there are reads that map equally good to drosophla and yeast based on preliminary analysis (using Saccharomyces only). In order to have reads that are possibly from yeast aside for yeast composition analysis, reads that have multiple mappings are considered non-drosophila reads.
 
1. Construct reference

		cat CandidaKrusei.fa > Fly2Yeasts.fa
		cat Saccharomyces.fa >> Fly2Yeasts.fa
		cat Drosophila_melanogaster.BDGP6.dna_sm.toplevel.fa >> Fly2Yeasts.fa 
		
2. Index for bwa
				
		bwa index Fly2Yeasts.fa 
		bwa mem Fly2Yeasts.fa /media/backup_2tb/Data/FlyMicrobiome/Drosophila/Trimmomatic/KF5_R1.gz.trimpair.fastq.gz /media/backup_2tb/Data/FlyMicrobiome/Drosophila/Trimmomatic/KF5_R2.gz.trimpair.fastq.gz -t 40 > KF5_Fly2Yeasts.sam

3. bbsplit (see instructions [here](http://seqanswers.com/forums/showthread.php?t=41288))

	fastq files:

		bbsplit.sh in=/media/backup_2tb/Data/FlyMicrobiome/Drosophila/Trimmomatic/KF5_R#.trimpair.fastq.gz ref=/media/backup_2tb/Data/FlyMicrobiome/Microbes/CandidaKrusei.fa,/media/backup_2tb/Data/FlyMicrobiome/Microbes/Saccharomyces.fa,/media/backup_2tb/Data/FlyMicrobiome/Drosophila/Drosophila_melanogaster.BDGP6.dna_sm.toplevel.fa basename=o%_#.fq ambig2=split outu1=unmapped1.fq outu2=unmapped2.fq
	sam files:

		bbsplit.sh in=/media/backup_2tb/Data/FlyMicrobiome/Drosophila/Trimmomatic/KF5_R#.trimpair.fastq.gz ref=/media/backup_2tb/Data/FlyMicrobiome/Microbes/CandidaKrusei.fa,/media/backup_2tb/Data/FlyMicrobiome/Microbes/Saccharomyces.fa,/media/backup_2tb/Data/FlyMicrobiome/Drosophila/Drosophila_melanogaster.BDGP6.dna_sm.toplevel.fa basename=o%.sam ambig2=split
		
4. Coverage of the Saccharomyces genome and Candida genome.

		samtools view -bS oCandidaKrusei.sam | samtools sort > sorted_oCandidaKrusei.bam
		samtools mpileup sorted_oCandidaKrusei.bam > coverage_oSaccharomyces.txt
		
5. Ambiguous coverage

