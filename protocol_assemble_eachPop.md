### This protocol is used to assemble the non-drosophila reads that were extracted from protocol_Fly2Yeasts. The only difference from protocal_assemble_nonDrosophla.md is that this is to assemble and annotate each genome seperately.
 
1. Run a k-mer composition cluster of the non-Dros reads.

		mkdir reads
		cd reads
		for name in ZI197N ZI256N ZI274N ZI366N ZI403N ZI418N KF8 KF18 KF21 KF22 KF24 FR109N FR110N FR112N FR11N FR126N FR157N FR198N FR219N FR312N FR59N KF10 KF11 KF1 KF20 KF23 KF2 KF3 KF4 KF5 KF6 KF7 KF9 ZI31N
		do
			mkdir $name
			mv ../${name}_nonDrosophila_*.fq.gz $name
		done
		nohup python /home/hfan/scripts/aaf_phylokmer.py -k 21 -t 7 -o nonDros_round4_k21 -d reads -G 200 > nonDros_round4_k21_20170505.log 2>&1 </dev/null & 

		cat *_1.fq.gz >> nonDrosophila_1.fq.gz
		cat *_2.fq.gz >> nonDrosophila_2.fq.gz
		
2. assembly (megahit)  

		megahit -r nonDrosophila_1.fq.gz -r nonDrosophila_2.fq.gz -o nonDrosophila_round4 --out-prefix nonDrosophila_round4

	Stats: 42514 contigs, total 41367840 bp, min 200 bp, max 237365 bp, avg 973 bp, N50 1479 bp  

3. blast (diamond) 

		diamond blastx -d /media/backup_2tb/Data/nr_protein/nr -q nonDrosophila_round4.contigs.fa -o nonDrosophila_round4.m8 --sensitive -p 30
Total time = 16627.6s
Reported 912475 pairwise alignments, 912475 HSSPs. This is why I don't like blast. 42514 contigs, 912475 alignments...  
Output format(tabular):   
query 	subject 	%id 	alignment
length
	mismatches 	gap
openings 	query
start query
end subject
start subject
end 	E
value
	bit
score  
4. Accesion number to taxid  
   Donwlod prot.accession2taxid.gz from [ncbi](ftp://ftp.ncbi.nih.gov/pub/taxonomy/accession2taxid/)  
   Combine blast result with taxid.  
   	
   		python ~/scripts/accession2taxid.py nonDrosophila_round4.m8 > nonDrosophila_round4_taxid.txt
There are some accesions that are obselete.
Downloaded dead_prot.accession2taxid.gz. Still ended up with 599 accession number unknown. Having Miranda manually looking them up this afternoon.

5. See how well the first one did!
This is the one with the most hits. Acetobacter malorum(taxid=178901).  
i. map non_drosophila reads to the A.malorum genome [here](https://github.com/voutcn/megahit/wiki/An-example-of-real-assembly). 

		export PATH=$PATH:/home/hfan/build/bbmap
		bbwrap.sh ref=/media/backup_2tb/Data/FlyMicrobiome/Microbes/Acetobacter_malorum.fa.gz in=/media/backup_2tb/Data/FlyMicrobiome/nonDrosophila/Round4/nonDrosophila_#.fq.gz out=178901.sam.gz kfilter=22 subfilter=15 maxindel=80
	###samfile flags explained:
	Evert flag is a decimal of a 12-digit bit number. The bit number match the following items from left to right:  
	read paired  
  read mapped in proper pair  
  read unmapped  
  mate unmapped  
  read reverse strand  
  mate reverse strand  
  first in pair  
  second in pair  
  not primary alignment  
  read fails platform/vendor quality checks  
  read is PCR or optical duplicate  
  supplementary alignment  
  for each mapping, the flag reflects the 12 criteria. Say one read that is paired(1) and mapped in proper pair(1), not unmapped(0), mate also not unmapped(0), it is the reverse strand(1), its mate is also on the reverse strand(1), it is the first in pair(1), not the second in pair(0, notice how you can have conflict informaiton here), it is the primary alignment(0), it did not fail platform/vendor quality checks(0), it is not PCR or optical duplicate(0), an not a suplementary alignment(0). Note how the criteria range from most common to less common to speed up the calculation. Here we have a bit score of 000001110011, which translate to 115 in decimal.  
  So in this case, if I want the mapped reads, the only thing I would need is to for column 3 and 4 read unmapped and mate unmapped both to be 0, and other columns do not matter. This includes: 99, 147, 83, 163, 81, 161, 113, 177, 65, 129, 97, 145.  
		
		samtools view -F4 -F8 178901.sam.gz | gzip > 178901_mapped.sam.gz
		samtools view -H 178901.sam.gz > 178901.sam.header
		
		for name in ZI197N ZI256N ZI274N ZI366N ZI403N ZI418N KF8 KF18 KF21 KF22 KF24 FR109N FR110N FR112N FR11N FR126N FR157N FR198N FR219N FR312N FR59N KF10 KF11 KF1: KF20 KF23 KF2: KF3 KF4 KF5 KF6 KF7 KF9 ZI31N
		do
		cp 178901.sam.header ${name}_178901_mapped.sam
		zcat 178901_mapped.sam.gz | grep $name >> ${name}_178901_mapped.sam
		samtools view -bS ${name}_178901_mapped.sam | samtools sort > sorted_${name}_178901_mapped.bam
		samtools mpileup sorted_${name}_178901_mapped.bam > coverage_${name}_178901_mapped.txt
		python ~/scripts/coverage2region_general.py coverage_${name}_178901_mapped.txt > stats_${name}_178901_mapped.txt
		done

Check the mapping similarity (move to ipython notebook: FlyMicrobiome: Percent identity)

~90%! Not bad. This is mainly because "subfilter=15" in bbmap; it only allows 15 mismatches.

I start to notice that Wolbachia, apparently very present in this dataset, had number of hits or total hit lengths ranked around 100. This might be due to that the reads that maps to the Wolbachia genome were not assembled into contigs. Here we will first see how many reads were included in assembly; look at the case of wolbachia to see how many reads that map to the wolbachia genome were included in the assembly; then maybe we should just blast the reads. The problem with blasting reads is that there might be duplicated reads. Maybe we should only blast the reads that are not included in the assembly. But how do we combine the stats from the two parts afterwards? Maybe I should also study megahit to see whether there is a better parameter setting.

1. how many reads were included in assembly?

		export PATH=$PATH:/home/hfan/build/bbmap
		bbwrap.sh ref=/media/backup_2tb/Data/FlyMicrobiome/nonDrosophila/Round4/nonDrosophila_round4/nonDrosophila_round4.contigs.fa in=/media/backup_2tb/Data/FlyMicrobiome/nonDrosophila/Round4/nonDrosophila_#.fq.gz out=megahit_assembly.sam.gz kfilter=22 subfilter=15 maxindel=80
		samtools view -F4 -F8 megahit_assembly.sam.gz | gzip > megahit_assembly_mapped.sam.gz
Header length: 42516  
reads mapped:28297248
non-Drosophila reads: 44734738-42516=44692222
63% of the reads were assembled! Not bad at all.
2. how many wolbachia reads were included in assembly?
	
		filterbyname.sh in=megahit_assembly_mapped.sam.gz names=1317678/1317678_mapped.sam.gz out=sharedWolbachia.sam include=t
There are 5063300 reads mapped to Wolbachia genome (1317678_mapped.sam.gz). Among those, 4861464 reads (96%) were used in assembly. This is pretty amazing! OK. At least for wolbachia, the assembly was very good! So is it a problem with blast? Or processing of blast results? I'm like a detective. OK, so which contigs were those 4861464 reads mapped to? Are those contigs identified as wolbachia?
Yes... But they are identified as different wolbachia. OK, time to assign taxon to each contig!

3. Find the lowest common ancester for each contig

		python ~/GitHub/script/contig2taxon.py ~/Dropbox/Research/Drosophila/Round4/region/nodes.dmp ~/Dropbox/Research/Drosophila/Round4/Contig2Species.txt > contig2LCA.txt
		
	Since I did not download the same version of prot.accession2taxid and nodes.dmp, 5 of the taxid in the prot.accession2taxid are obselete. Manually fix them (upgrading to nodes.dmp) in Contig2Species.txt.  
	482058	1980924  
	1649555	1333996  
	1715259	1334193  
	1855411	1873524  	1917424	1788301
	
4. I have 305 congits, 414,652 bp in total has LCA of root! WTF. Maybe I should do the whole JGI thing, from gene calling to rps-blast. I have that pipeline. And in the end, "The latter is assigned as the last common ancestor of USEARCH hits of the genes on the scaffold/contig provided that at least 30 % of the genes have USEARCH hits."

I looked at those root contigs. The similarities scores are very low. Filtering out similarity scores lower than 70%.