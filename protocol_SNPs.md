Following the pipeline described in Dave Tang's [blog](https://davetang.org/muse/2015/08/26/samtools-mpileup/)

	$bwa index /media/backup_2tb/Data/FlyMicrobiome/Microbes/Acetobacter_pomorum.fa  
	$fastq-dump --gzip --split-files --dumpbase --skip-technical –clip --read-filter pass SRR3734079  
	#[bam_sort] Use -T PREFIX / -o FILE to specify temporary and final output files
	$bwa mem -t 4 /media/backup_2tb/Data/FlyMicrobiome/Microbes/Acetobacter_pomorum.fa SRR3734079_pass_1.fastq.gz SRR3734079_pass_2.fastq.gz | samtools view -bSu - | samtools sort -T SRR3734079_pass_1
	#I tried -t 40, 8,6,4,2,1, 4 is the most efficient in terms of real sec.