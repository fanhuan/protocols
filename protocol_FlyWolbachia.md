Based on results from protocol_Fly2Yeasts.md, the ambigious reads shared between drosophila and yeats are very limited (<1% of the yeast reads). Therefore in this version we are not going to split reads between drosophila and yeast, but only drosophila and nonDrosophila (nonDros,不作死). Instead, since we might expand Richard 2012 to 1000, here we use Wolbachia as another library for bbsplit.
 
1. Get raw reads from NCBI based on accssion # in Table S2 of Lack 2016.  

		fastq-dump --gzip --split-files SRR*
		
2. Quality control using [Trimmomatic](http://www.usadellab.org/cms/?page=trimmomatic). Make sure the adaptor file is in the workin directory. (only uses one core, should be parrallized)

		cp ~/build/Trimmomatic-0.36/adapters/TruSeq3-PE.fa ./
		java -jar ~/build/Trimmomatic-0.36/trimmomatic-0.36.jar PE -phred33 ../round2/EA_inbred/EA59N_R1.fq.gz ../round2/EA_inbred/EA59N_R2.fq.gz EA59N_R2_paired.fq.gz EA59N_R2_unpaired.fq.gz EA59N_R1_paired.fq.gz EA59N_R1_unpaired.fq.gz ILLUMINACLIP:TruSeq3-PE.fa:2:30:10 LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 MINLEN:36
		
3. Split between Drosophila and Wolbachia using [bbsplit](http://seqanswers.com/forums/showthread.php?t=41288).  

		export PATH=$PATH:/home/hfan/build/bbmap
		bbsplit.sh in=/media/backup_2tb/Data/FlyMicrobiome/Drosophila/Trimmomatic/EA59N_R#_paired.fq.gz ref=/media/backup_2tb/Data/FlyMicrobiome/Microbes/Wolbachia.fa,/media/backup_2tb/Data/FlyMicrobiome/Drosophila/Drosophila_melanogaster.fa basename=%_#.fq.gz ambig2=split outu1=EA59N_R1_unmapped.fq.gz outu2=EA59N_R2_unmapped.fq.gz
		
## Now we have two dataset: Wolbachia and others.

### Others
#### Route 1: Pooled assembly and annotation.
1. Assembly from each population.  
The pooling of reads from different metagenome dataset is proposed in [crAss](https://www.ncbi.nlm.nih.gov/pubmed/23074261). In the paper, it did not discuss which assembler and what parameter settings should be used. It did mention to use assemblers that are less likely to produce chimera contigs. However, those assemblers also tend to form less and shorter contigs thus uses less reads (see a [review](http://journal.frontiersin.org/article/10.3389/fbioe.2015.00141) on different metagenomic assemblers. It was a tough decision: whether to include more reads or be more stringint on chimera contigs.  
Apparently we would like to include more reads. But  
In order to use Megahit, we need to change the tag of all unmapped reads to represent their sample source, and then pull them together for each population. I wrote a python script to generate a shell scrip to do the actual work. Note that we added a parameter to megahit. Smaller (default is 12) --k-step is supposed to be "more friendly to lower coverage" according to the wiki page of [megahit](https://github.com/voutcn/megahit/wiki/Assembly-Tips).

		for name in EA_haploid EA_inbred EG_inbred FR_haploid FR_inbred KF_inbred SP_haploid SP_inbred ZI_haploid ZI_inbred
		do
			python ~/scripts/mergeUnmapped.py $name
			sh ${name}_merge.sh
			megahit -1 ${name}_unmapped1.fq.gz -2 ${name}_unmapped2.fq.gz -o ${name}_megahit_PE --out-prefix ${name}_unmapped_PE --k-step 10 
			sed s/'>'/'>'${name}_/g ${name}_megahit_PE/${name}_unmapped_PE.contigs.fa >> PE.fa
		done
		

		 
2. Taxonamy assignment  
a. JGI uses [USEARCH](https://academic.oup.com/bioinformatics/article-lookup/doi/10.1093/bioinformatics/btq461):  
"
Genes are associated with KO terms [17] and EC num- bers based on USEARCH 6.0.294 results [18] comparing metagenome proteins against an isolate genome refer- ence database with maxhits of 50 and an e-value of 0.1....One top USEARCH hit per gene is also retained for the Phylogenetic Distri- bution tool in IMG and assignment of phylogenetic lineage to scaffolds and contigs. The latter is assigned as the last common ancestor of USEARCH hits of the genes on the scaffold/contig provided that at least 30 % of the genes have USEARCH hits.
"  
I'm using [DIAMOND](http://www.nature.com/nmeth/journal/v12/n1/full/nmeth.3176.html), which is supposed to be much faster. Remember last time we had a lot of issue with obsolete taxid? I decided to update the nr database this time (was updated on June 5th!) and rebuild the diamond database (.dmnd).  

		wget ftp://ftp.ncbi.nlm.nih.gov/blast/db/FASTA/nr.gz
		diamond makedb --in nr.faa -d nr 
		diamond blastx -d /media/backup_2tb/Data/nr_protein/nr -q PE.fa -o PE.m6 --sensitive --taxonmap /media/backup_2tb/Data/nr_protein/prot.accession2taxid.gz --id 70 -e 1e-10 -f 6 qseqid sseqid qlen pident length evalue staxids stitle
						
Lots of contigs in the haploid assembly were identified either as Drosophila or other insects. This suggests the mapping was too stringent. The default parameters in bbsplit.sh are:  

	maxindel=<20>        Don't look for indels longer than this.  Lower is faster.  Set to >=100k for RNA-seq.
	minratio=<0.65>      Fraction of max alignment score required to keep a site.  Higher is faster.
	minhits=<1>          Minimum number of seed hits required for candidate sites.  Higher is faster.

Those are all for sensitivity, not for similarity control. Therefore the parameters used for similarity control are default ones in bbmap.sh. Since Drosophila and Wolbachia did not share much ambigous reads, I decided to only use bbwrap to generate the nonDrosophila reads, thus Round6! Starting another protocal... Really, one day before lab meeting? Huan...

#### Route 2: taxonomy classificaiton of reads ([k-SLAM](https://github.com/aindj/k-SLAM))

### Wolbachia
Following the pipeline as described in [Richardson 2012](http://journals.plos.org/plosgenetics/article?id=10.1371/journal.pgen.1003129).

1. map Wolbachia reads back to the reference genome to calculate mean depth of coverage and breadth of coverage. Almost think I've done this before with some python scripts. (see coverage.sh)

		export PATH=$PATH:/home/hfan/build/bbmap
		bbwrap.sh ref=/media/backup_2tb/Data/FlyMicrobiome/Microbes/Wolbachia.fa in=$1/$2_Wolbachia_R#.fq.gz out=$2_wolbachia.sam.gz kfilter=22 subfilter=15 maxindel=80
		samtools view -bS $2_wolbachia.sam.gz | samtools sort > $2_wolbachia_sorted.bam 
		samtools mpileup $2_wolbachia_sorted.bam > coverage_$2_wolbachia.txt
		python ~/scripts/coverage2region_general.py coverage_$2_wolbachia.txt > stats_$2_wolbachia.txt
		
2. It seems like some of the strains are not wolbachia (50-70% of similarity). Therefore I was curious to see what's actually there. I decided to assemble 16s and see what they are. Ran into [EMIRGE](https://github.com/csmiller/EMIRGE). A bit painful to install but worked. It requires the insert mean and std for pair reads. So I needed to calculate that for each sample...  
		
		for name in 
		cat ../nonDrosophila/Round5/KF_inbred/KF1_R2_unmapped.fq.gz >> KF1_nonDrosophila_R2.fq.gz 
		cp /media/backup_2tb/Data/FlyMicrobiome/Wolbachia/KF1_nonDrosophila_R2.fq.gz ./
		gunzip KF1_nonDrosophila_R2.fq.gz
		./emirge.py KF1 -1 /media/backup_2tb/Data/FlyMicrobiome/Wolbachia/KF1_nonDrosophila_R1.fq.gz -2 KF1_nonDrosophila_R2.fq -b SILVA_128_SSURef_Nr99_tax_silva_trunc.ge1200bp.le2000bp.0.97.fixed -f SILVA_128_SSURef_Nr99_tax_silva_trunc.ge1200bp.le2000bp.0.97.fixed.fasta -l 101 -i 294 -s 91 --phred33 -a 30

3. Emirge worked, but the results are horrible. First of all, the intermidiate strains had no relevant (sometimes none) hits. There are also some weird results such as salmon. Why would there be salmon inside fruit flies? There must be some really rich fly labs.  
4. OK, in order to know whether it is due to insufficient sampling effort or more distantly related wolbachia, we could loose the threshold for mapping. If it is due to insufficient sampling, this should not affect the results much since the missing parts of the genome are lost in the library prep or sequencing process. However, if it is another species, or a very different strain, lower the threshold of mapper should work (coverage_parameter.sh group sample kfilter subfilter).  

	a. kfilter = 21, subfilter = 14



1. Splitting with Wolbachia and other genus that has isolation: Acetobacter, Lactobacillus, Gluconobacter,  Commensalibacter. Based on the tree we made for Acetobactecaea, we choose Acetobacter pasteurianus instead other Acetobacter since it is the furthest from the glocunobacter clade. In Lactobacillus we choose plantarum since it has been isolated 4 times (the highest among other Lactobacillus). The other two only had one isolates. This time we are going to keep the fly reads to make trees (then delete again?).  

		export PATH=$PATH:/home/hfan/build/bbmap
		bbsplit.sh in=/media/backup_2tb/Data/FlyMicrobiome/Drosophila/Trimmomatic/$1/$2_R#_paired.fq.gz ref=/media/backup_2tb/Data/FlyMicrobiome/Microbes/Wolbachia.fa,/media/backup_2tb/Data/FlyMicrobiome/Drosophila/Drosophila_melanogaster.fa basename=%_#.fq.gz ambig2=split outu1=EA59N_R1_unmapped.fq.gz outu2=EA59N_R2_unmapped.fq.gz
 