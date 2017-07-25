## Download raw reads from JGI

python3 ~/scripts/jgi-query.py http://genome.jgi.doe.gov/AttbisABBM2_FD/AttbisABBM_FD.info.html

## Maxbin

	for name in ABBM2 ACBM1 ACBM2 ACBM3 ALBM1 ALBM3 ASBM1 ASBM2 ASBM3; do ~/build/MaxBin-2.2.4/run_MaxBin.pl -contig /media/backup_2tb/Data/fungus_garden_bacteria/BAscaff/${name}_BAscaff/${name}_BAscaff.fa -reads $name.fq.gz -out ${name}_maxbin -thread 30; done
	
## checkM

	
	checkm lineage_wf folder_with_binned_fasta output_folder -x fasta 
	e.g. $checkm lineage_wf ABBM1_bin ABBM1_checkM -x fasta

