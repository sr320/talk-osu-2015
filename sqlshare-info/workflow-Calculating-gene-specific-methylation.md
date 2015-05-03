Worflow constructed by Dan Halperin to calculate methylation ratios from multiple GFF files and annoation tables (nicely visualized below).

<img src="https://www.evernote.com/shard/s10/sh/c1e0a30c-8742-4bc2-bed2-b230be199672/02abb173e83217cc8b8a20ba27533d0d/deep/0/Screenshot%205/26/13%208:46%20AM.jpg" alt="Screenshot%205/26/13%208:46%20AM" width =70% />

<hr>

**(0) import GFF and clean it up. This is direct (use all the GFF columns, but pull everything after the ';' out and put it in an extra column called Comment).**

	SELECT Seqname
	     , Source
	     , Feature
	     , StartIdx
	     , EndIdx
	     , Score
	     , Strand
	     , Frame
	     , CASE WHEN CHARINDEX(';', GroupID) = 0
	            THEN GroupID
	            ELSE SUBSTRING(GroupID, 1, CHARINDEX(';', GroupID)-1)
	       END AS GroupID
	     , CASE WHEN CHARINDEX(';', GroupID) = 0
	            THEN ''
	            ELSE SUBSTRING(GroupID, CHARINDEX(';', GroupID)+1, LEN(GroupID))
	       END AS Comment
	FROM [dhalperi@washington.edu].[table_oyster.v9.glean.final.rename.mRNA.gff]


**(1&2) Count the number of methylated/total CGs in each gene. Note I wrote my own, simple interval join but it's almost trivial in SQL**

	SELECT oyster.GroupID
	     , COUNT(*) as methCGcnt
	FROM [dhalperi@washington.edu].[oyster.v9.glean.final.rename.mRNA.gff] oyster
	   , [dhalperi@washington.edu].[bivalvia_methylated_20CG_20as_20bed.txt.gff] meth
	WHERE oyster.seqname=meth.seqname
	  AND (oyster.startidx > meth.startidx AND oyster.startidx < meth.endidx
	       OR meth.startidx > oyster.startidx AND meth.startidx < oyster.endidx)
	GROUP BY oyster.groupid​

**(3) Join the methylated/all CGs per gene data, divide the two, and use 0 if a gene didn't appear in the methylated data [the RIGHT OUTER JOIN gets all the rows from the All CGs data, the COALESCE statement puts 0 in for null values].**

	SELECT allCG.GroupID
	     , COALESCE(1.0*methCGcnt/allCGcnt,0) as MethRatio
	FROM [dhalperi@washington.edu].[methylated_CG_gene_count] methCG
	RIGHT OUTER JOIN [dhalperi@washington.edu].[all_CG_gene_count] allCG
	ON methCG.GroupID = allCG.GroupID

**(4) Link methylation with the gene description. Note this includes the trimming the 'ID=' string off in the last line.**

	SELECT geneDesc.*
	     , methRatio.MethRatio
	FROM [dhalperi@washington.edu].[methylation_ratio_CG_gene] methRatio
	JOIN [dhalperi@washington.edu].[TJGR_Gene_SPID_evalue_Description.txt] geneDesc
	  ON geneDesc.Column1 = SUBSTRING(methRatio.GroupID, CHARINDEX('CGI', methRatio.GroupID), LEN(methRatio.GroupID))

**(3&4) Advanced version that both computes the ratio and links with gene description at once. Note that it's basically just the join statement from 4 appended to the query from 3!**

	SELECT geneDesc.*
	     , COALESCE(1.0*methCGcnt/allCGcnt,0) as MethRatio
	FROM [dhalperi@washington.edu].[methylated_CG_gene_count] methCG
	RIGHT OUTER JOIN [dhalperi@washington.edu].[all_CG_gene_count] allCG
	  ON methCG.GroupID = allCG.GroupID
	JOIN [dhalperi@washington.edu].[TJGR_Gene_SPID_evalue_Description.txt] geneDesc
	  ON geneDesc.Column1 = SUBSTRING(allCG.GroupID, CHARINDEX('CGI', allCG.GroupID), LEN(allCG.GroupID))
