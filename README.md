# Protocols from different projects

### Fly2Yeasts
This is for setting aside drosophila reads from other reads.

### assemble_nonDrosophila
After setting aside non-drosophila reads, assemble them altogether to see what's there.


### protocol_FlyWolbachia.md
Based on results from protocol_Fly2Yeasts.md, the ambigious reads shared between drosophila and yeats are very limited (<1% of the yeast reads). Therefore in this version we are not going to split reads between drosophila and yeast, but only drosophila and nonDrosophila (nonDros,不作死). Instead, since we might expand Richard 2012 to 1000, here we use Wolbachia as another library for bbsplit.