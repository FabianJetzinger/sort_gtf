# sort_gtf

This script (attempts to) sort .gtf files in the following manner:

1. sort chromosomes
2. sort all entries within a chromosome by gene start position (if no gene_id, use the entry's own start position)
    
    --> this means, if e.g. gene 1 and 2 overlap, all entries associated with gene 1 should come before gene 2, even if e.g. the start position of the last exon of gene 1 is later than the start of gene 2
3. sort all entries within the same gene start position (either same gene or two genes with same start) by transcript start position (or its ownstart position if no transcript_id)
    
    --> same logic but on transcript level; tie between gene entry and its transcripts with the same start position
4. sort by gene_id
    
    --> for genes with the same start position, sort them (and all entries belonging to them) by their ID alphanumerically
5. sort by transcript_id
    
    --> same logic but on transcript level; tie between: gene entry + first transcript + all its exons
6. sort by start position of the entry itself
    
    --> puts exons of the transcript in order
7. sort by kind of entry, where gene > transcript > others > exon
   
    --> puts entries with remaining ties in order so gene is on top, then its first transcript, then any other entries associated with the transcript, then its first exon
8. remaining ties between entries that are not "gene", "transcript", or "exon", are resolved by alphanumerically sorting the entry type, so e.g. "CDS" will be before "start_codon".

## Requirements
This script relies on extracting gene_id and transcript_id fields from the ninth column of the .gtf. By, default, the following patterns are used to find these fields:

```
gene_id_pattern='gene_id "([^"]+)"' 
transcript_id_pattern='transcript_id "([^"]+)"' 
```

If your .gtf contains gene and transcript ids in the ninth column, but in a different format, you can supply your own patterns with the -g and -t options.

## Usage

Clone this repository or simply copy/download the `sort_gtf.sh` file. Some examples for how to call it:

```
./sort_gtf.sh < test/test.gtf > test/test.sorted.gtf
./sort_gtf.sh -i test/test2.gtf -o test/test2.sorted.gtf -g 'gene "([^"]+)"' -t 'transcript "([^"]+)"'
```

## Why did I make this?
I was working with TAMA Merge, for which I needed to convert some .gtf files to a specific .bed format, for which TAMA provides its own converters. However, I encountered a problem where some of my .gtf files, namely the outputs of the tools TALON and IsoQuant, sorted exons on the "-" strand in a descending manner. I understand that the logic behind this decision is that, because they are on the negative strand, they get read in that order; however, the converter I was using could not work with this, and explicitly needed all the exons in ascending manner. After looking for a suitable tool to handle this sorting for me, I read some discussion that this is easily done with the standard Unix sort command. Naively, I tried to do this, but quickly realized I would need a little bit of pre-processing before the sorting, which eventually turned into this quite complex awk script in order to handle edge cases and hopefully be more useful for other people & use cases as well. All in all, this could probably be done more efficiently in a different language, but I am still providing it here in case someone else finds it useful too. 