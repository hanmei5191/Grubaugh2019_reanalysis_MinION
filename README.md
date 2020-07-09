# Reanalysis_MinION_Grubaugh2019
This repo develops machine learning models to cluster the true and false positive variants called in the MinION experiments in [Grubaugh2019](https://genomebiology.biomedcentral.com/articles/10.1186/s13059-018-1618-7). 

One author—[Nicholas J. Loman](https://github.com/nickloman/zika-isnv)—from [Grubaugh2019](https://genomebiology.biomedcentral.com/articles/10.1186/s13059-018-1618-7) published a Github repo showing the process how they classified the true and false positive MinION variants using a logistic regression model. 

## My repo here contains three folders. 
### The first folder [start_from_variants](https://github.com/hanmei5191/Grubaugh2019_reanalysis_MinION/tree/master/start_from_variants) starts from three variants tables taken from [Nicholas J. Loman](https://github.com/nickloman/zika-isnv). The three tables are 
- BC01.variants.0.03.txt 
- BC02.variants.0.03.txt
- BC03.variants.0.03.txt

These three tables were generated using the following commands from [Nicholas J. Loman](https://github.com/nickloman/zika-isnv):
- python scripts/freqs.py --snpfreqmin 0.03 BC01.trimmed.sorted.bam refs/ZIKV_REF.fasta > BC01.variants.0.03.txt
- python scripts/freqs.py --snpfreqmin 0.03 BC02.trimmed.sorted.bam refs/ZIKV_REF.fasta > BC02.variants.0.03.txt
- python scripts/freqs.py --snpfreqmin 0.03 BC03.trimmed.sorted.bam refs/ZIKV_REF.fasta > BC03.variants.0.03.txt

In [start_from_variants](https://github.com/hanmei5191/Grubaugh2019_reanalysis_MinION/tree/master/start_from_variants), the variants tables contain two colomns—ForwardVariantCov and ReverseVariantCov, and the strand bias was calculated using the following equation:
StrandAF = pmin(ForwardVariantCov, ReverseVariantCov) / pmax(ForwardVariantCov, ReverseVariantCov). 

We reproduced the logistic regression model by [Nicholas J. Loman](https://github.com/nickloman/zika-isnv). In addition, we developed two more models—KNN and SVM. The analysis is described in start_from_variants.ipynb. 

However, we found this strand bias calculation possibly inaccurate because it does not incorporate ForwardRefCov and ReverseRefCov. [Guo2012](https://link.springer.com/article/10.1186/1471-2164-13-666) described three ways to calculate strand bias. We decided to adapt these three methods, and re-calcualte the strand bias. However, since ForwardRefCov and ReverseRefCov are needed in [Guo2012](https://link.springer.com/article/10.1186/1471-2164-13-666), but are not present in the three variants tables in [start_from_variants](https://github.com/hanmei5191/Grubaugh2019_reanalysis_MinION/tree/master/start_from_variants). We need to generate new variants tables by starting from the bam files. This is the reason why we have the second folder [start_from_trimmed.sorted.bam](https://github.com/hanmei5191/Grubaugh2019_reanalysis_MinION/tree/master/start_from_trimmed.sorted.bam) included in this repo. 

### The second folder [start_from_trimmed.sorted.bam](https://github.com/hanmei5191/Grubaugh2019_reanalysis_MinION/tree/master/start_from_trimmed.sorted.bam) starts from the three trimmed.sorted.bam files taken from [Nicholas J. Loman](https://github.com/nickloman/zika-isnv). These are: 
- BC01.trimmed.sorted.bam
- BC02.trimmed.sorted.bam
- BC03.trimmed.sorted.bam

We modified the python script scripts/freqs.py by [Nicholas J. Loman](https://github.com/nickloman/zika-isnv) to generate variants tables containing three more columns—RefCov, ForwardRefCov, and ReverseRefCov. The modified script is scripts/freqs_modified.py. The changes in code are: 

at line 46: 

print ("Pos\tQual\tFreq\tRef\tBase\tUngappedCoverage\tTotalCoverage\tVariantCov\tForwardVariantCov\tReverseVariantCov") 

-> 

print ("Pos\tQual\tFreq\tRef\tBase\tUngappedCoverage\tTotalCoverage\tVariantCov\tForwardVariantCov\tReverseVariantCov<span style="color:red;">\tRefCov\tForwardRefCov\tReverseRefCov</span>")

at line 96: 

print ("%s\t%s\t%s\t%s\t%s\t%s\t%s\t%s\t%s\t%s" % (pileupcolumn.pos+1, q, float(freqs[base].count) / float(nonindel_coverage), ref, base, nonindel_coverage, total_nonindel_coverage, freqs[base].count, freqs[base].forward, freqs[base].reverse))

-> 

print ("%s\t%s\t%s\t%s\t%s\t%s\t%s\t%s\t%s\t%s\t%s\t%s\t%s" % (pileupcolumn.pos+1, q, float(freqs[base].count) / float(nonindel_coverage), ref, base, nonindel_coverage, total_nonindel_coverage, freqs[base].count, freqs[base].forward, freqs[base].reverse, freqs[ref].count, freqs[ref].forward, freqs[ref].reverse))

Then the new variants tables 
- BC01_modified.variants.0.03.txt
- BC02_modified.variants.0.03.txt
- BC03_modified.variants.0.03.txt

are generated using the following commands: 
- python scripts/freqs_modified.py --snpfreqmin 0.03 BC01.trimmed.sorted.bam refs/ZIKV_REF.fasta > BC01_modified.variants.0.03.txt
- python scripts/freqs_modified.py --snpfreqmin 0.03 BC02.trimmed.sorted.bam refs/ZIKV_REF.fasta > BC02_modified.variants.0.03.txt
- python scripts/freqs_modified.py --snpfreqmin 0.03 BC03.trimmed.sorted.bam refs/ZIKV_REF.fasta > BC03_modified.variants.0.03.txt

The analysis is described in start_from_trimmed.sorted.bam.ipynb. 

We found that BC01/02/03.variants.0.03.txt and BC01/02/03_modified.variants.0.03.txt do not contain the same number of variants. This indicates BC01/02/03.variants.0.03.txt was not generated from BC01/02/03.trimmed.sorted.bam in the same [repo](https://github.com/nickloman/zika-isnv), but from some other bam files. We decided to move on using BC01/02/03.trimmed.sorted.bam provided by [Nicholas J. Loman](https://github.com/nickloman/zika-isnv), since our goal is to test classifying models by
- ALT allele freq
- the strand bias calculated using [Guo2012](https://link.springer.com/article/10.1186/1471-2164-13-666)'s methods. 

### The thrid folder [start_from_reads](https://github.com/hanmei5191/Grubaugh2019_reanalysis_MinION/tree/master/start_from_reads) starts from the raw MinION reads. 

The analysis is described in start_from_reads.ipynb. 

[Conclusion] Compare 

variants tables BC01/02/03_modified/confirm.variants.0.03.new.txt (called from raw reads) 

to 

variants tables BC01/02/03_modified/confirm.variants.0.03.txt (called from BC01/02/03.trimmed.sorted.bam provided by [Nicholas J. Loman](https://github.com/nickloman/zika-isnv))

-> 

only 1–2 positions were seen as inconsistent. 
