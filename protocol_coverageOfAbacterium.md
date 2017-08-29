Following the pipeline as described in [Richardson 2012](http://journals.plos.org/plosgenetics/article?id=10.1371/journal.pgen.1003129).

1. map nonDrosophila reads back to the reference genome to calculate mean depth of coverage and breadth of coverage. see coverage.sh

		export PATH=$PATH:/home/hfan/build/bbmap
		bbwrap.sh ref=/media/backup_2tb/Data/FlyMicrobiome/Microbes/$3.fa in=/media/backup_2tb/Data/FlyMicrobiome/nonDrosophila/Round5/$1/$2_R#_unmapped.fq.gz out=$2_$3.sam.gz kfilter=22 subfilter=15 maxindel=80
		samtools view -bS $2_$3.sam.gz | samtools sort > $2_$3_sorted.bam
		samtools mpileup $2_$3_sorted.bam > coverage_$2_$3.txt
		python ~/scripts/coverage2region_general.py coverage_$2_$3.txt > stats_$2_$3.txt
		
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
 
 