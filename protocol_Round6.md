In this round, we are not going to use bbsplit but bbwrap right away to extract nonDrosophila reads. Last time when we used bbsplit, though discarding ambigous ones with Wolbachia, the assembly of haploid ones were swamped by Drosophila contigs. Thus we will use that as an indicator of our parameter setting.


1. Map onto drosophila reference genome using bbwrap and extrac nonDrosophila reads following [this post](https://github.com/voutcn/megahit/wiki/An-example-of-real-assembly).  

		bbwrap.sh ref=/media/backup_2tb/Data/FlyMicrobiome/Drosophila/Drosophila_melanogaster.fa in=/media/backup_2tb/Data/FlyMicrobiome/Drosophila/Trimmomatic/$1/$2_R1_paired.fq.gz in2=/media/backup_2tb/Data/FlyMicrobiome/Drosophila/$1/$2_R2_paired.fq.gz out=$2.sam.gz kfilter=22 subfilter=15 maxindel=80
		samtools view -u -f4 $2.sam.gz | samtools-1.2/samtools bam2fq -s $2_unmapped.se.fq - > $2_unmapped.pe.fq
		
However when I lowered the threshold to kfilter=21 subfilter=35, I still have more reads as unmapped than bbsplit... OK never mind. Maybe it's something about pooled assembly. Try annotate each read now.

### Annotate each read by blast (only the three stocks we did isolation with).

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
 