# Reanalysis_MinION_Grubaugh2019
This repo develops machine learning models to cluster the true and false positive variants called in the MinION experiments in [Grubaugh2019](https://genomebiology.biomedcentral.com/articles/10.1186/s13059-018-1618-7). 

One author—[Nicholas J. Loman](https://github.com/nickloman/zika-isnv)—from [Grubaugh2019](https://genomebiology.biomedcentral.com/articles/10.1186/s13059-018-1618-7) published a Github repo showing the process how they classified the true and false positive MinION variants using a logistic regression model. 

My repo here contains two folders. 
1. The first folder ["start_from_variants"]() starts from three variants tables taken from [Nicholas J. Loman](https://github.com/nickloman/zika-isnv). The three tables are 
- "BC01.variants.0.03.txt" 
- "BC02.variants.0.03.txt"
- "BC03.variants.0.03.txt"

These three tables were generated using the following commands from [Nicholas J. Loman](https://github.com/nickloman/zika-isnv):
- python scripts/freqs.py --snpfreqmin 0.03 BC01.trimmed.sorted.bam refs/ZIKV_REF.fasta > BC01.variants.0.03.txt
- python scripts/freqs.py --snpfreqmin 0.03 BC02.trimmed.sorted.bam refs/ZIKV_REF.fasta > BC02.variants.0.03.txt
- python scripts/freqs.py --snpfreqmin 0.03 BC03.trimmed.sorted.bam refs/ZIKV_REF.fasta > BC03.variants.0.03.txt

In ["start_from_variants"](), the variants tables contain two colomns—ForwardVariantCov and ReverseVariantCov, and the strand bias is calculated using the following equation:
StrandAF = pmin(ForwardVariantCov, ReverseVariantCov) / pmax(ForwardVariantCov, ReverseVariantCov). 

We found this calculation disturbing because it does not incorporate ForwardRefCov, ReverseRefCov. [Guo2012](The effect of strand bias in Illumina short-read sequencing data) described three ways to calculate strand bias. We decided to adapt these three methods to re-calcualte the strand bias. However, since ForwardRefCov and ReverseRefCov are not present in the three variants tables in ["start_from_variants"](). We need to generate new variants tables by starting the bam files. This is the reason why we have the second folder ["start_from_trimmed.sorted.bam"]() included in this repo. 

2. The second folder ["start_from_trimmed.sorted.bam"]() starts from the three trimmed.sorted.bam files taken from [Nicholas J. Loman](https://github.com/nickloman/zika-isnv). These are: 
- "BC01.trimmed.sorted.bam"
- "BC02.trimmed.sorted.bam"
- "BC03.trimmed.sorted.bam"

