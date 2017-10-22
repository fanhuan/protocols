## Download raw reads from JGI

python3 ~/scripts/jgi-query.py http://genome.jgi.doe.gov/AttbisABBM2_FD/AttbisABBM_FD.info.html

## Organizing files


## Maxbin

	for name in ABBM2 ACBM1 ACBM2 ACBM3 ALBM1 ALBM3 ASBM1 ASBM2 ASBM3; do ~/build/MaxBin-2.2.4/run_MaxBin.pl -contig /media/backup_2tb/Data/fungus_garden_bacteria/BAscaff/${name}_BAscaff/${name}_BAscaff.fa -reads $name.fq.gz -out ${name}_maxbin -thread 30; done
	
	#rawFastq
	for name in ABBM1 ABBM3 ACBM1 ACBM2 ACBM3 ALBM1 ALBM2 ASBM1 ASBM2 ASBM3
	do
	mv $name.fq.gz rawFastq/
	#MaxBin
	for name in ABBM3 ACBM1 ACBM2 ACBM3 ALBM1 ALBM2 ASBM1 ASBM2 ASBM3
	do
		mkdir MaxBin/$name
		mv ${name}_* MaxBin/$name
	done

	
## checkM

	checkm lineage_wf ABBM1_bin ABBM1_checkM -x fasta -t #threads
	
	for name in ABBM2 ABBM3 ACBM1 ACBM2 ACBM3 ALBM1 ALBM2 ALBM3 ASBM1 ASBM2 ASBM3
	do
		checkm lineage_wf MaxBin/$name checkM/${name}_checkM -x fasta -t 30
	done

## Circle plot
map nonDrosophila reads back to the reference genome to calculate mean depth of coverage and breadth of coverage. see coverage.sh

		export PATH=$PATH:/home/hfan/build/bbmap
		bbwrap.sh ref=/media/backup_2tb/Data/FlyMicrobiome/Microbes/$3.fa in=/media/backup_2tb/Data/FlyMicrobiome/nonDrosophila/Round5/$1/$2_R#_unmapped.fq.gz out=$2_$3.sam.gz kfilter=22 subfilter=15 maxindel=80
		samtools view -q 40 -bS ${sample}_$ref.sam.gz | samtools sort > ${sample}_$ref.bam
    	samtools mpileup -B ${sample}_$ref.bam > ${sample}_$ref.mpileup
    	python ~/scripts/coverage2region_general.py ${sample}_$ref.mpileup > stats_${sample}_$ref.txt
		


