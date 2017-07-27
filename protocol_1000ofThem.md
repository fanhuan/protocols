1. Download data.

Accession numbers were aquired from Lack 2016 Table S2. Note that if the accession number is in SRX(instead of SRR), manual look-up from NCBI sra is neccessary. A helpful [post](https://edwards.sdsu.edu/research/fastq-dump/) on fastq-dump (part of the sra-toolkit) options.

	fastq-dump --gzip --split-files --readids --dumpbase --skip-technical –clip --read-filter pass SRR_ID 

2. Quality control using [Trimmomatic](http://www.usadellab.org/cms/?page=trimmomatic). Make sure the adaptor file is in the workin directory. (only uses one core, should be parrallized)

		cp ~/build/Trimmomatic-0.36/adapters/TruSeq3-PE.fa ./
		java -jar ~/build/Trimmomatic-0.36/trimmomatic-0.36.jar PE -phred33 ../round2/EA_inbred/EA59N_R1.fq.gz ../round2/EA_inbred/EA59N_R2.fq.gz EA59N_R2_paired.fq.gz EA59N_R2_unpaired.fq.gz EA59N_R1_paired.fq.gz EA59N_R1_unpaired.fq.gz ILLUMINACLIP:TruSeq3-PE.fa:2:30:10 LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 MINLEN:36
		
3. Split between Drosophila and Wolbachia using [bbsplit](http://seqanswers.com/forums/showthread.php?t=41288).  

		export PATH=$PATH:/home/hfan/build/bbmap
		bbsplit.sh in=/media/backup_2tb/Data/FlyMicrobiome/Drosophila/Trimmomatic/EA59N_R#_paired.fq.gz ref=/media/backup_2tb/Data/FlyMicrobiome/Microbes/Wolbachia.fa,/media/backup_2tb/Data/FlyMicrobiome/Drosophila/Drosophila_melanogaster.fa basename=%_#.fq.gz ambig2=split outu1=EA59N_R1_unmapped.fq.gz outu2=EA59N_R2_unmapped.fq.gz
		
