Based on results from protocol_Fly2Yeasts.md, the ambigious reads shared between drosophila and yeats are very limited (<1% of the yeast reads). Therefore in this version we are not going to split reads between drosophila and yeast, but only drosophila and nonDrosophila (nonDros,不作死). Instead, since we might expand Richard 2012 to 1000, here we use Wolbachia as another library for bbsplit.
 
1. Get raw reads from NCBI based on accssion # in Table S2 of Lack 2016.  

		fastq-dump --gzip --split-files SRR*
		
2. Quality control using [Trimmomatic](http://www.usadellab.org/cms/?page=trimmomatic). Make sure the adaptor file is in the workin directory. (only uses one core, should be parrallized)

		cp ~/build/Trimmomatic-0.36/adapters/TruSeq3-PE.fa ./
		java -jar ~/build/Trimmomatic-0.36/trimmomatic-0.36.jar PE -phred33 ../round2/EA_inbred/EA59N_R1.fq.gz ../round2/EA_inbred/EA59N_R2.fq.gz EA59N_R2_paired.fq.gz EA59N_R2_unpaired.fq.gz EA59N_R1_paired.fq.gz EA59N_R1_unpaired.fq.gz ILLUMINACLIP:TruSeq3-PE.fa:2:30:10 LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 MINLEN:36
		
3. Split between Drosophila and Wolbachia using [bbsplit](http://seqanswers.com/forums/showthread.php?t=41288).  

		export PATH=$PATH:/home/hfan/build/bbmap
		bbsplit.sh in=/media/backup_2tb/Data/FlyMicrobiome/Drosophila/Trimmomatic/EA59N_R#_paired.fq.gz ref=/media/backup_2tb/Data/FlyMicrobiome/Microbes/Wolbachia.fa,/media/backup_2tb/Data/FlyMicrobiome/Drosophila/Drosophila_melanogaster.fa basename=%_#.fq.gz ambig2=split outu1=EA59N_R1_unmapped.fq.gz outu2=EA59N_R2_unmapped.fq.gz
		
### Now we have two dataset: Wolbachia and others.

#### Route 1: Others(unmapped).
1. Assembly from each population.  
In order to use Megahit, we need to change the tag of all unmapped reads to represent their sample source, and then pull them together for each population. See EA_haploid below as an example. The full script is at cat_unmapped.sh in scripts.
 

		for file in *R1*; do sample=`echo $file | cut -d \. -f 1`; zcat $file | sed s/SALLY:261:C0V2YACXX:2/$sample/g | gzip >> EA_haploid_unmapped1.fq.gz; done
		for file in *R2*; do sample=`echo $file | cut -d \. -f 1`; zcat $file | sed s/SALLY:261:C0V2YACXX:2/$sample/g | gzip >> EA_haploid_unmapped2.fq.gz; done
		megahit -r EA_haploid_unmapped1.fq.gz -r EA_haploid_unmapped2.fq.gz -o EA_haploid_megahit --out-prefix EA_haploid_unmapped  
		 
2. Taxonamy assignment  
a. JGI uses [USEARCH](https://academic.oup.com/bioinformatics/article-lookup/doi/10.1093/bioinformatics/btq461):  
"
Genes are associated with KO terms [17] and EC num- bers based on USEARCH 6.0.294 results [18] comparing metagenome proteins against an isolate genome refer- ence database with maxhits of 50 and an e-value of 0.1....One top USEARCH hit per gene is also retained for the Phylogenetic Distri- bution tool in IMG and assignment of phylogenetic lineage to scaffolds and contigs. The latter is assigned as the last common ancestor of USEARCH hits of the genes on the scaffold/contig provided that at least 30 % of the genes have USEARCH hits.
"  
I'm using [DIAMOND](http://www.nature.com/nmeth/journal/v12/n1/full/nmeth.3176.html), which is supposed to be much faster.



#### Route 2: Wolbachia.
		
	