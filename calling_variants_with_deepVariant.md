# DeepVariant on a human genome

This is the workflow I follow to call variant on any sample "XXX":

## Software versions:
```bash
BWA: 0.7.17-r1194-dirty
Java: openjdk version "11.0.3" 2019-04-16
Picard MarkDuplicates: 2.18.21-SNAPSHOT (https://github.com/broadinstitute/picard/releases/download/2.18.21/picard.jar)
Samtools: samtools 1.7 Using htslib 1.7
DeepVariant: 0.8.0
```

## Install Java
```bash
sudo apt update; sudo apt install default-jdk; java -version;
```
## GRCh38

#### Step 1: Map reads to GRCh38 using bwa mem and samtools
```bash
bwa mem -t 64 -R '@RG\tID:XXX_S1_L001\tPL:ILLUMINA\tPU:xTen\tLB:XXX\tSM:XXX' \
/home/kishwar/data/GRCh38/GRCh38_full_analysis_set_plus_decoy_hla.fa \
/home/kishwar/data/XXX_R1_001.fastq \
/home/kishwar/data/XXX_R2_001.fastq | samtools sort -@64 -o XXX_2_GRCh38.bam -
```
We also need to index it
```bash
samtools index -@64 XXX_2_GRCh38.bam
```

#### Step 2: Remove duplicates

Next remove duplicates using picard
```bash
java -verbose:gc -XX:+UseParallelOldGC -XX:+AggressiveOpts -XX:ParallelGCThreads=64 -Xms50g  -Xmx110g -jar /home/kishwar/software/picard.jar MarkDuplicates \
INPUT=XXX_2_GRCh38.bam \
OUTPUT=XXX_2_GRCh38_duplicates_removed.bam \
METRICS_FILE=/data/tmp/logs/dupmetrics_XXX_2_GRCh38_duplicates_removed.txt \
CREATE_INDEX=true \
REMOVE_DUPLICATES=true \
MAX_RECORDS_IN_RAM=40000000 \
ASSUME_SORT_ORDER=coordinate \
USE_JDK_DEFLATER=true \
USE_JDK_INFLATER=true \
TMP_DIR=/data/tmp 2>&1 | tee /data/tmp/logs/markdup_XXX_GRCh38.log
```

#### Step 3: DeepVariant
Install DeepVariant docker:
```bash
sudo apt -y update
sudo apt-get -y install docker.io
sudo docker pull gcr.io/deepvariant-docker/deepvariant:"0.8.0"
```

Run DeepVariant:
```bash
sudo docker run \
-v "/home/kishwar/data":"/input" \
-v "/home/kishwar/data/deepvariant_output:/output" \
gcr.io/deepvariant-docker/deepvariant:"0.8.0" \
/opt/deepvariant/bin/run_deepvariant \
--model_type=WGS \
--ref=/input/GRCh38/GRCh38_full_analysis_set_plus_decoy_hla.fa \
--reads=/input/XXX_to_GRCH38/XXX_2_GRCh38_duplicates_removed.bam \
--output_vcf=/output/XXX_GRCh38.vcf.gz \
--output_gvcf=/output/XXX_GRCh38.g.vcf.gz \
--num_shards=64
```

## GRCh37
It's the exact same pipeline but for a different reference. For the sake of completeness, I'll repeat the steps again.

#### Step 1: Map reads to GRCh38 using bwa mem and samtools
```bash
bwa mem -t 64 -R '@RG\tID:XXX_S1_L001\tPL:ILLUMINA\tPU:xTen\tLB:XXX\tSM:XXX' \
/home/kishwar/data/GRCh37/human_g1k_v37.fasta \
/home/kishwar/data/fullgenomes/XXX_R1_001.fastq \
/home/kishwar/data/fullgenomes/XXX_R2_001.fastq | samtools sort -@64 -o XXX_2_GRCh37.bam -
```
We also need to index it
```bash
samtools index -@ 64 XXX_2_GRCh37.bam
```

Next remove duplicates using picard
```bash
java -verbose:gc -XX:+UseParallelOldGC -XX:+AggressiveOpts -XX:ParallelGCThreads=64 -Xms50g  -Xmx110g -jar /home/kishwar/software/picard.jar MarkDuplicates \
INPUT=XXX_2_GRCh37.bam \
OUTPUT=XXX_2_GRCh37_duplicates_removed.bam \
METRICS_FILE=/data/tmp/logs/dupmetrics_XXX_2_GRCh37_duplicates_removed.txt \
CREATE_INDEX=true \
REMOVE_DUPLICATES=true \
MAX_RECORDS_IN_RAM=40000000 \
ASSUME_SORT_ORDER=coordinate \
USE_JDK_DEFLATER=true \
USE_JDK_INFLATER=true \
TMP_DIR=/data/tmp 2>&1 | tee /data/tmp/logs/markdup_XXX_GRCh37.log
```

#### Step 3: DeepVariant
Install DeepVariant docker:
```bash
sudo apt -y update
sudo apt-get -y install docker.io
sudo docker pull gcr.io/deepvariant-docker/deepvariant:"0.8.0"
```

Run DeepVariant:
```bash
sudo docker run \
-v "/home/kishwar/data":"/input" \
-v "/home/kishwar/data/deepvariant_output:/output" \
gcr.io/deepvariant-docker/deepvariant:"0.8.0" \
/opt/deepvariant/bin/run_deepvariant \
--model_type=WGS \
--ref=/input/GRCh37/human_g1k_v37.fasta \
--reads=/input/XXX_to_GRCH37/XXX_2_GRCh37_duplicates_removed.bam \
--output_vcf=/output/XXX_GRCh37.vcf.gz \
--output_gvcf=/output/XXX_GRCh37.g.vcf.gz \
--num_shards=64
```
