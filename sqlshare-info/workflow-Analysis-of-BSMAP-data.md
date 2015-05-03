This workflow outlines analysis of data from mapping bisulfite treated DNA (BS-seq) to the oyster genome.

Initially sequencing reads, in this case from oyster sperm, are mapped to the genome using BS-MAP.


    ./bsmap -a /Volumes/betty/filtered_174gm_A_NoIndex_L006_R1.fastq.gz -b /Volumes/betty/filtered_174gm_A_NoIndex_L006_R2.fastq.gz -d /Volumes/betty/oyster_v9.fa -o /Volumes/betty/BiGO_Betty_plain.sam -p 4

_output_:  
Total number of aligned reads:  
pairs:       90102067 (53%)  
single a:    17033002 (9.9%)  
single b:    15975991 (9.3%)  

<http://eagle.fish.washington.edu/cnidarian/BiGO_Betty_plain.sam>


Resulting SAM file then subjected to methratio python script

    python methratio.py -d /Volumes/betty/oyster_v9.fa -u -z -g -o  /Volumes/betty/BiGO_betty_plain_methratio_v1.txt -s /Volumes/Bay3/Software/BSMAP/bsmap-2.73/samtools /Volumes/betty/BiGO_Betty_plain.sam
    
_output_:  
total 167774088 valid mappings, 127192776 covered cytosines, average coverage: 13.23 fold.

<http://eagle.fish.washington.edu/cnidarian/BiGO_betty_plain_methratio_v1.txt>

This file was uploaded to SQLShare using python script  
    
    python singleupload.py sr320@washington.edu c8APIKEYAPIKEYAPIKEY5c15c /Volumes/web/cnidarian/BiGO_betty_plain_methratio_v1.txt

<hr>
<img src="https://www.evernote.com/shard/s10/sh/dd8451a7-99fc-4d00-bf70-071d073a9044/5eb549aff8cbcc2a14eb5d473e317f29/deep/0/Screenshot%205/24/13%2010:34%20AM.png" width = "60%"alt="Screenshot%205/24/13%2010:34%20AM" />


directlink:
<https://sqlshare.escience.washington.edu/sqlshare#s=query/sr320%40washington.edu/BiGO_betty_plain_methratio_v1.txt>
    
<hr>

### Interested in how many CpG loci have methylation information?

The following query will get all lines where CG are in the 3rd and 4th position.  

    SELECT * FROM [sr320@washington.edu].[BiGO_betty_plain_methratio_v1.txt]​ betty where context like '__CG_'​​​​​​​​​​​​​ --_=single character wildcard​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​
 
 
 
 <hr>
    
 <img src="https://www.evernote.com/shard/s10/sh/ea1d91af-c92b-4461-a979-c357dce141ff/285518308334e0dfb6eed38f1eb2902e/deep/0/Screenshot%205/24/13%2010:45%20AM.png" width = "60%" "alt="Screenshot%205/24/13%2010:45%20AM" />
 
  <hr>


### To count the number of instances in this query
 
<img src = "http://25.media.tumblr.com/71b286219406760ca7e8091e3b5b73f6/tumblr_mnbehrZ7b71qmcl6do1_400.png"/>
 
 
   <hr>

### How to convert methratio file to gff format
In this case will make GFF only of CpGs, where there is at least 10x coverage.

    SELECT 
    chr as seqname,  
    'methratio' as source,  
    'CpG' as feature, 
    pos as start,
    pos + 1 as [end],
    ratio as score,  
    strand,  
    '.' as frame,  
    '.' as attribute  
  
    FROM [sr320@washington.edu].     
    [BiGO_betty_plain_methratio_v1.txt] betty 
    where 
    context like '__CG_' --_=single character wildcard
    and
    CT_Count > 9

<img src="https://www.evernote.com/shard/s10/sh/43eba902-672e-4a7a-9c8b-6246f5496232/fbcc05cb68bdf5628760f1bca50a91d7/deep/0/Screenshot%205/25/13%207:47%20AM.jpg" alt="Screenshot%205/25/13%207:47%20AM" width = 60%/>
    
<hr>

​<img src="https://www.evernote.com/shard/s10/sh/f5769f54-2538-4424-8aa8-0322ad91e3ea/28db871037c8fe3993e67e563bfbebd8/deep/0/Screenshot%205/25/13%207:52%20AM.jpg" alt="Screenshot%205/25/13%207:52%20AM" width = "80%" />

<hr>
<https://sqlshare.escience.washington.edu/sqlshare#s=query/sr320%40washington.edu/BiGO_Methylation_oysterv9_GFF>
<hr>
 