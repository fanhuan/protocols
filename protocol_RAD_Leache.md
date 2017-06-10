### This protocol is used for the second example of the RAD paper
 
1. Download the data  
Did not work:  
		
		/media/backup_2tb/Data/RAD$ sudo apt-get install sra-toolkit 
Maybe version issue.
Downloaded the newest sra-toolkit from nicbi website and put it in build.  
Configure it following [this](https://github.com/ncbi/sra-tools/wiki/Toolkit-Configuration).

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
		for name in ZI197N ZI256N ZI274N ZI366N ZI403N ZI418N KF8 KF18 KF21 KF22 KF24 FR109N FR110N FR112N FR11N FR126N FR157N FR198N FR219N FR312N FR59N KF10 KF11 KF1 KF20 KF23 KF2 KF3 KF4 KF5 KF6 KF7 KF9 ZI31N
		bbmap.sh in1=../bbsplit/AMBIGUOUS_${ref}_1.fq in2=../bbsplit/AMBIGUOUS_${ref}_2.fq ref=/media/backup_2tb/Data/FlyMicrobiome/Microbes/$ref.fa fastareadlen=500 minhits=1 minratio=0.56 maxindel=20 qtrim=rl untrim=t trimq=6 out=ambg_$ref.sam

>>> flag = []
>>> with open('178901.sam.flag') as fh:
...     for line in fh:
...             if line.split()[1] not in flag:
...                     flag.append(line.split()[1])
...                     print line.split()[1]
... 
