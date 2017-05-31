Based on results from protocol_Fly2Yeasts.md, the ambigious reads shared between drosophila and yeats are very limited (<1% of the yeast reads). Therefore in this version we are not going to split reads between drosophila and yeast, but only drosophila and nonDrosophila (nonDros,不作死). Instead, since we might expand Richard 2012 to 1000, here we use Wolbachia as another library for bbsplit.
 
1. Get raw reads from NCBI based on accssion # in Table S2 of Lack 2016.  

		fastq-dump --gzip --split-files SRR*
		
2. Quality control using [Trimmomatic](http://www.usadellab.org/cms/?page=trimmomatic). Make sure the adaptor file is in the workin directory. (only uses one core, should be parrallized)

		cp ~/build/Trimmomatic-0.36/adapters/TruSeq3-PE.fa ./
		java -jar ~/build/Trimmomatic-0.36/trimmomatic-0.36.jar PE -phred33 ../round2/EA_inbred/EA59N_R1.fq.gz ../round2/EA_inbred/EA59N_R2.fq.gz EA59N_R2_paired.fq.gz EA59N_R2_unpaired.fq.gz EA59N_R1_paired.fq.gz EA59N_R1_unpaired.fq.gz ILLUMINACLIP:TruSeq3-PE.fa:2:30:10 LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 MINLEN:36
		
3. Split between Drosophila and Wolbachia using [bbsplit](http://seqanswers.com/forums/showthread.php?t=41288).  

		export PATH=$PATH:/home/hfan/build/bbmap
		bbsplit.sh in=/media/backup_2tb/Data/FlyMicrobiome/Drosophila/Trimmomatic/EA59N_R#_paired.fq.gz ref=/media/backup_2tb/Data/FlyMicrobiome/Microbes/Wolbachia.fa,/media/backup_2tb/Data/FlyMicrobiome/Drosophila/Drosophila_melanogaster.fa basename=%_#.fq.gz ambig2=split outu1=EA59N_R1_unmapped.fq.gz outu2=EA59N_R2_unmapped.fq.gz
		
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

