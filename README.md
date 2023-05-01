# lab-notebook
Lab notebook for GEN711 :)

# git clone
pulls features of repos to vscode
if you do ls file-name, directories wil show up

# fork
make a copy of main repo

# pull requests

4/21

# qiime
combines all fastq files into 1 file so that you can run alaysis

# fastp
removes poly-g tails


remove primer sequences

# denoising reads (END HERE)
look at reads and look at diff sequences in the files
what species is this - compare to BLAST

# CODE:

cp /tmp/gen711_project_data/fastp-single.sh ~/fastp-single.sh 

chmod +x ~/fastp-single.sh

mkdir trimmed_fastqs

./fastp-single.sh 120 /tmp/gen711_project_data/FMT_3/fmt-tutorial-demux-1 trimmed_fastqs

mkdir trimmed_fastqs2

./fastp-single.sh 120 /tmp/gen711_project_data/FMT_3/fmt-tutorial-demux-2 trimmed_fastqs2

conda activate qiime2-2022.8

IMPORT FASTQS TO QIIME
1: qiime tools import --type "SampleData[SequencesWithQuality]" \
--input-format CasavaOneEightSingleLanePerSampleDirFmt \
--input-path trimmed_fastqs \
--output-path trimmed_fastqs/FMT_trimmed_fastqs1.qza

2: qiime tools import --type "SampleData[SequencesWithQuality]" \
--input-format CasavaOneEightSingleLanePerSampleDirFmt \
--input-path trimmed_fastqs2 \
--output-path trimmed_fastqs2/FMT_trimmed_fastqs2.qza

CUTADAPT
1: qiime cutadapt trim-single \
--i-demultiplexed-sequences trimmed_fastqs/FMT_trimmed_fastqs1.qza \
--p-front TACGTATGGTGCA \
--p-discard-untrimmed \
--p-match-adapter-wildcards \
--verbose \
--o-trimmed-sequences trimmed_fastqs/FMT_cutadapt1.qza

2: qiime cutadapt trim-single \
--i-demultiplexed-sequences trimmed_fastqs2/FMT_trimmed_fastqs2.qza \
--p-front TACGTATGGTGCA \
--p-discard-untrimmed \
--p-match-adapter-wildcards \
--verbose \
--o-trimmed-sequences trimmed_fastqs2/FMT_cutadapt2.qza

DEMUX
1: qiime demux summarize \
--i-data trimmed_fastqs/FMT_cutadapt1.qza \
--o-visualization trimmed_fastqs/FMT_demux1.qzv

2: qiime demux summarize \
--i-data trimmed_fastqs2/FMT_cutadapt2.qza \
--o-visualization trimmed_fastqs2/FMT_demux2.qzv

DENOISE
1: qiime dada2 denoise-single \
--i-demultiplexed-seqs trimmed_fastqs/FMT_cutadapt1.qza \
--p-trunc-len 50 \
--p-trim-left 13 \
--p-n-threads 4 \
--o-denoising-stats denoising-stats1.qza \
--o-table feature_table1.qza \
--o-representative-sequences rep-seqs1.qza

2: qiime dada2 denoise-single \
--i-demultiplexed-seqs trimmed_fastqs2/FMT_cutadapt2.qza \
--p-trunc-len 50 \
--p-trim-left 13 \
--p-n-threads 4 \
--o-denoising-stats denoising-stats2.qza \
--o-table feature_table2.qza \
--o-representative-sequences rep-seqs2.qza

METADATA
1: qiime metadata tabulate \
--m-input-file denoising-stats1.qza \
--o-visualization denoising-stats1.qzv

2: qiime metadata tabulate \
--m-input-file denoising-stats2.qza \
--o-visualization denoising-stats2.qzv

TABULATE
1: qiime feature-table tabulate-seqs \
--i-data rep-seqs1.qza \
--o-visualization rep-seqs1.qzv

2: qiime feature-table tabulate-seqs \
--i-data rep-seqs2.qza \
--o-visualization rep-seqs2.qzv

TAXONOMY
mkdir FMT_merged

qiime feature-table merge-seqs \
--i-data rep-seqs1.qza \
--i-data rep-seqs2.qza \
--o-merged-data FMT_merged/merged.rep-seqs.qza

qiime feature-classifier classify-sklearn \
--i-classifier /tmp/gen711_project_data/reference_databases/classifier.qza \
--i-reads FMT_merged/merged.rep-seqs.qza \
--o-classification FMT_merged/FMT-taxonomy.qza

qiime taxa barplot \
--i-table trimmed_fastqs/feature_table1.qza \
--i-taxonomy FMT_taxonomy.qza \
--o-visualization FMT_merged/barplot1.qzv

qiime taxa barplot \
--i-table trimmed_fastqs2/feature_table2.qza
--i-taxonomy FMT_merged/FMT-taxonomy.qza
--o-visualization FMT_merged/barplot2.qzv




