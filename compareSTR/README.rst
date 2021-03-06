CompareSTR
==========

CompareSTR is a tool for comparing TR callsets. Example use cases include:

* Comparing calls to a "ground truth" set, e.g. from capillary electrophoresis data
* Comparing calls for the same tool using different parameter settings
* Comparing calls for different tools. This only works if they used the same set of reference TRs.

It outputs overall, per-locus, and per-sample concordance information. It optionally will stratify results based on a user-specified FORMAT field (e.g. depth, or quality score) and by repeat motif length.

Usage
-----
CompareSTR takes as input two VCF files with overlapping TRs and samples and outputs metrics and plots based on comparing calls across the two VCFs. The input VCF files must be sorted, indexed, and have the appropriate `##contig` header lines. The VCF files must also have alleles specified in the same way. e.g. GangSTR-style using full allele sequences, or ExpansionHunter style with SV tags.

To run compareSTR use the following command::

  compareSTR \
    --vcf1 <vcf file> --vcf2 <vcf file> \
    --out test \
    [additional options]

Required Parameters:

* :code:`--vcf1 <VCF>`: First VCF file to compare (must be sorted, bgzipped, and indexed).
* :code:`--vcf2 <VCF>`: Second VCF file to compare (must be sorted, bgzipped, and indexed).
* :code:`--out <string>`: Prefix to name output files

Recommended Parameters:

* :code:`--vcftype1 <string>`: Type of VCF file 1.
* :code:`--vcftype2 <string>`: Type of VCF file 2.

Filtering Options:

* :code:`--samples <string>`: File containing list of samples to include. If not specified, all samples are used.
* :code:`--region <string>`: Restrict to this region chrom:start-end

Metrics to stratify results:

* :code:`--stratify-fields`: Comma-separated list of FORMAT fields to stratify by. e.g. DP,Q
* :code:`--stratify-binsizes`: Comma-separated list of min:max:binsize to stratify each field on. Must be same length as :code:`--stratify-fields`. e.g. 0:50:5,0:1:0.1
* :code:`--stratify-file`: Specify which file to look at the :code:`--stratify-fields` in. If set to 0, apply to both files. If set to 1, apply only to :code:`--vcf1`. If set to 2, apply only to :code:`--vcf2`.
* :code:`--period`: Report results overall and also stratified by repeat unit length (period).

Plotting options:
* :code:`--bubble-min`: Minimum x/y axis value to display on bubble plots.
* :code:`--bubble-max`: Maximum x/y axis value to display on bubble plots.

Other options:
* :code:`--verbose`: Print helpful debugging info
* :code:`--noplot`: Don't output any plots. Only produce text output.

Outputs
-------

In output files, compareSTR reports the following metrics:

* Length concordance: % of genotypes concordant between the two VCF files when only considering TR allele lengths
* Sequence concordance: % of genotypes concordant between the two VCF files when considering TR allele sequence. Currently only relevant for HipSTR. Otherwise will be identical to length concordance
* R2: Pearson r2 between the sum of allele lengths at each call compared between the two VCF files.

compareSTR outputs the following text files and plots:

* outprefix+"-overall.tab": Has columns period, concordance-seq, concordance-len, r2, numcalls. Plus additional columns for any FORMAT fields to stratify results on. This file has one line for all results (period="ALL") and a different line for each period analyzed separately. If stratifying by format fields, it will have additional lines for each range of values for each of those fields.
* outprefix+"-bubble-period$period.pdf": "Bubble" plot, which plots the sum of allele lengths for each call in :code:`--vcf1` vs. :code:`--vcf2`. Allele lengths are given in terms of bp difference from the reference genome. The size of each bubble gives the number of calls at each cooordinate. A seperate plot is output for all TRs (period="ALL") and for each period.
* outprefix+"-locuscompare.tab": Has columns chrom, start, metric-conc-seq, metric-conc-len, sample. last column gives the number of samples considered at each locus. There is one line for each TR.
* outprefix+"-locuscompare.pdf": Plots the length concordance metric for each TR locus considered.
* outprefix+"-samplecompare.tab": Has columns sample, metric-conc-seq, metric-conc-len, numcalls. One line per sample
* outprefix+"-samplecompare.pdf": Plots the length concordance metric for each sample considered.

Example compareSTR command
--------------------------

Compare two callsets::

  FILE1=${REPODIR}/test/common/sample_vcfs/compareSTR_vcfs/compare_vcf1.vcf.gz
  FILE2=${REPODIR}/test/common/sample_vcfs/compareSTR_vcfs/compare_vcf2.vcf.gz
  compareSTR \
    --vcf1 ${FILE1} --vcf2 ${FILE2} \
    --out test-compare

where :code:`$REPODIR` points to the root path of this repository.

Similarly, to compare two callsets, but stratify by the DP and Q format fields in the first VCF file and output metrics separately by period (and also modify the bubble plot dimensions)::

  FILE1=${REPODIR}/test/common/sample_vcfs/compareSTR_vcfs/compare_vcf1.vcf.gz
  FILE2=${REPODIR}/test/common/sample_vcfs/compareSTR_vcfs/compare_vcf2.vcf.gz
  compareSTR \
    --vcf1 ${FILE1} --vcf2 ${FILE2} \
    --stratify-fields DP,Q \
    --stratify-binsizes 0:50:10,0:1:0.1 \
    --stratify-file 1 \
    --period \
    --bubble-min -50 --bubble-max 50 \
    --out test-compare
