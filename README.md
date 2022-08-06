# Somático EP2

A amostras WP312 agora será executado com a versão do genoma humano HG19.
A única parte que muda do EP1 é a das referências


**AS Referências do Genoma hg19 (FASTA, VCFs)**

Os arquivos de Referência: **Panel of Normal (PoN), Gnomad AF**

> https://console.cloud.google.com/storage/browser/gatk-best-practices/somatic-b37?project=broad-dsde-outreach

```bash
wget -c https://storage.googleapis.com/gatk-best-practices/somatic-b37/Mutect2-WGS-panel-b37.vcf
```

```bash
wget -c https://storage.googleapis.com/gatk-best-practices/somatic-b37/Mutect2-WGS-panel-b37.vcf.idx
```

```bash
wget -c  https://storage.googleapis.com/gatk-best-practices/somatic-b37/af-only-gnomad.raw.sites.vcf
```

```bash
wget -c  https://storage.googleapis.com/gatk-best-practices/somatic-b37/af-only-gnomad.raw.sites.vcf.idx
```

> Arquivo no formato FASTA do genoma humano hg19

Diretório Download UCSC hg19:https://hgdownload.soe.ucsc.edu/goldenPath/hg19/chromosomes/
chr9.fa.gz: https://hgdownload.soe.ucsc.edu/goldenPath/hg19/chromosomes/chr9.fa.gz

```bash
wget -c https://hgdownload.soe.ucsc.edu/goldenPath/hg19/chromosomes/chr9.fa.gz
```

---

# Adicionando chr nos VCFs do Gnomad e PoN

O arquivo `af-only-gnomad.raw.sites.vcf` (do bucket somatic-b37) não tem o `chr`na frente do nome do cromossomo. Precisamos adicionar para não gerar conflito de contigs de referência na hora de executar o GATK.

```bash
grep "\#" af-only-gnomad.raw.sites.vcf > af-only-gnomad.raw.sites.chr.vcf
grep  "^9" af-only-gnomad.raw.sites.vcf |  awk '{print("chr"$0)}' >> af-only-gnomad.raw.sites.chr.vcf
```

**indexing**

```bash
bgzip af-only-gnomad.raw.sites.chr.vcf
tabix -p vcf af-only-gnomad.raw.sites.chr.vcf.gz
```

```bash
grep "\#" Mutect2-WGS-panel-b37.vcf > Mutect2-WGS-panel-b37.chr.vcf 
grep  "^9" Mutect2-WGS-panel-b37.vcf |  awk '{print("chr"$0)}' >> Mutect2-WGS-panel-b37.chr.vcf 
```

```bash
bgzip Mutect2-WGS-panel-b37.chr.vcf 
tabix -p vcf Mutect2-WGS-panel-b37.chr.vcf.gz
```

# GATK4 - Mutect Call (Refs hg19 com chr)

```bash
./gatk-4.2.2.0/gatk GetPileupSummaries \
	-I WP312_sorted_rmdup.bam  \
	-V af-only-gnomad.raw.sites.chr.vcf.gz  \
	-L WP312_coverageBed20x.interval_list \
	-O WP312.table
```

```bash
./gatk-4.2.2.0/gatk CalculateContamination \
-I WP312.table \
-O WP312.contamination.table
```

```bash
./gatk-4.2.2.0/gatk Mutect2 \
  -R chr9.fa \
  -I WP312_sorted_rmdup.bam \
  --germline-resource af-only-gnomad.raw.sites.chr.vcf.gz  \
  --panel-of-normals Mutect2-WGS-panel-b37.chr.vcf.gz \
  --disable-sequence-dictionary-validation \
  -L WP312_coverageBed20x.interval_list \
  -O WP312.somatic.pon.vcf.gz
```
