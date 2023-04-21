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

./fastp-single.sh 150 /tmp/gen711_project_data/FMT_3/fmt-tutorial-demux-1 trimmed_fastqs

./fastp-single.sh 150 /tmp/gen711_project_data/FMT_3/fmt-tutorial-demux-2 trimmed_fastqs

conda activate qiime2-2022.8

qiime tools import --type "SampleData[PairedEndSequencesWithQuality]" \
--input-format CasavaOneEightSingleLanePerSampleDirFmt \
--input-path trimmed_fastqs \
--output-path trimmed_fastqs/FMT_trimmed_fastqs

qiiime cutadapt trim-paired \
--i-demultiplexed-sequences FMT_trimmed_fastqs/FMT_trimmed_fastqs \
--p-cores 4 \
--p-front-f TACGTATGGTGCA \
--p-discard-untrimmed \
--p-match-adapter-wildcards \
--verbose \
--o-trimmed-sequences trimmed_fastqs/FMT_trimmed_fastqs.qza

qiime demux summarize \
--i-data FMT_trimmed_fastqs/FMT_trimmed_fastqs.qza \
--o-visualization trimmed_fastqs/FMT_trimmed_fastqs.qzv

qiime dada2 denoise-paired \
--i-demultiplexed-seqs qiime_out/${run}_demux_cutadapt.qza \
--p-trunc-len-f ${trunclenf} \
--p-trunc-len-r ${trunclenr} \
--p-trim-left-f 0 \
--p-trim-left-r 0 \
--p-n-threads 4 \
--o-denoising-stats FMT_trimmed_fastqs/denoising-stats.qza \
--o-table FMT_trimmed_fastqs/feature_table.qza \
--o-representative-sequences FMT_trimmed_fastqs/rep-seqs.qza

qiime metadata tabulate \
--m-input-file FMT_trimmed_fastqs/denoising-stats.qza \
--o-visualization FMT_trimmed_fastqs/denoising-stats.qzv

qiime feature-table tabulate-seqs \
--i-data FMT_trimmed_fastqs/rep-seqs.qza \
--o-visualization FMT_trimmed_fastqs/rep-seqs.qzv

