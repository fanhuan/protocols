Following the pipeline described in Dave Tang's [blog](https://davetang.org/muse/2015/08/26/samtools-mpileup/)

	$bwa index /media/backup_2tb/Data/FlyMicrobiome/Microbes/Acetobacter_pomorum.fa  
	$fastq-dump --gzip --split-files --dumpbase --skip-technical –clip --read-filter pass SRR3734079  
	#[bam_sort] Use -T PREFIX / -o FILE to specify temporary and final output files
	$bwa mem -t 4 /media/backup_2tb/Data/FlyMicrobiome/Microbes/Acetobacter_pomorum.fa SRR3734079_pass_1.fastq.gz SRR3734079_pass_2.fastq.gz | samtools view -bSu - | samtools sort -T SRR3734079_pass_1
	#I tried -t 40, 8,6,4,2,1, 4 is the most efficient in terms of real sec.
	
	
##Popoolation2
	for name in EA EG FR KF SP ZI
	do
		bbwrap.sh ref=ref_short.fa in=/media/backup_2tb/Data/FlyMicrobiome/nonDrosophila/Round5/${name}_inbred/${name}_inbred_unmapped#.fq.gz out=$name.sam.gz kfilter=22 subfilter=15 maxindel=80
		samtools view -q 20 -bS $name.sam.gz | samtools sort > ${name}_sorted.bam
	done  
	samtools mpileup -B *_sorted.bam > round5_unmapped.mpileup
	java -ea -Xmx7g -jar ~/build/popoolation2_1201/mpileup2sync.jar --input round5_unmapped.mpileup --output round5_unmapped_java.sync --fastq-type illumina --min-qual 20 --threads 20
	sed '/null/d' round5_unmapped_java.sync > round5_unmapped_noNull.sync
	perl ~/build/popoolation2_1201/snp-frequency-diff.pl --input round5_unmapped_noNull.sync --output-prefix round5_unmapped_noNull --min-count 1 --min-coverage 1 --max-coverage 200
	perl ~/build/popoolation2_1201/fst-sliding.pl --input FR_KF_noNull.sync --output FR_KF_noNull.fst --suppress-noninformative --min-count 1 --min-coverage 1 --max-coverage 200 --min-covered-fraction 1 --window-size 1 --step-size 1 --pool-size 500
