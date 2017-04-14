### This protocol is used to assemble the non-drosophila reads that were extracted from protocol_Fly2Yeasts.
 
1. Cat the reads together

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
i. find the contigs that has annotation from this taxid.

		for name in ZI197N ZI256N ZI274N ZI366N ZI403N ZI418N KF8 KF18 KF21 KF22 KF24 FR109N FR110N FR112N FR11N FR126N FR157N FR198N FR219N FR312N FR59N KF10 KF11 KF1 KF20 KF23 KF2 KF3 KF4 KF5 KF6 KF7 KF9 ZI31N
		bbmap.sh in1=../bbsplit/AMBIGUOUS_${ref}_1.fq in2=../bbsplit/AMBIGUOUS_${ref}_2.fq ref=/media/backup_2tb/Data/FlyMicrobiome/Microbes/$ref.fa fastareadlen=500 minhits=1 minratio=0.56 maxindel=20 qtrim=rl untrim=t trimq=6 out=ambg_$ref.sam

