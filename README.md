# Analysis of Helicobacter winghamensis genomes

## ABRicate

Run ABRicate with all reference databases. Use thresholds `minid = 30%` and `mincov = 70%`

```bash
export PATH=/home/domenico/Research/izs/salmonella/abricate/bin/abricate:$PATH
```

```bash
for genome in $(ls New_Assembly/*/*.fasta)
do
    echo $genome
    genome_id=$(echo $(basename $genome) | sed 's/\.fasta//g')
    for i in $(abricate --list | awk 'NR>1{print $1}')
    do
        echo $i
        abricate \
        --db $i \
        --minid 30 \
        --mincov 70 \
        $genome  | sed 's/New_Assembly\///g' | sed 's/\.fasta//g' > abricate_210217/${genome_id}_${i}.out
    done
done
```

## Annotate with PROKKA

Prokka contig ID must be shorter than 20 characters.

```python
import glob

samples = [i.split("/")[-1].split(".")[0] for i in glob.glob("New_Assembly/*/*.fasta")]

for s in samples:
    assembly_file = "New_Assembly/{s}/{s}.fasta".format(s=s)
    assembly = open(assembly_file, 'r')
    new_assembly = open(assembly_file.replace(".fasta", ".newID.fasta"), 'w')
    for l in assembly:
        if l.startswith(">"):
            l = "_".join(l.split("_")[:2]) + "\n"
        _ = new_assembly.write(l)

    new_assembly.close()
```

```bash
conda activate prokka

mkdir -p annotation/prokka

for sample in HW228 HW295 HW296 HW294 HW196
do
    prokka \
    --force \
    --prefix ${sample} \
    --cpus 2 \
    --outdir annotation/prokka/${sample} \
    New_Assembly/${sample}/${sample}.newID.fasta
done
```

PFAM

```bash
conda activate pfam_scan

mkdir -p annotation/pfam
mkdir -p logs/annotation/pfam

for sample in HW228 HW295 HW296 HW294 HW196
do
    echo ${sample}
    time pfam_scan.pl \
    -outfile ./annotation/pfam/${sample}.pfam_scan \
    -cpu 2 \
    -fasta ./annotation/prokka/${sample}/${sample}.faa \
    -dir ~/Research/data/pfam/Pfam-A &> logs/annotation/pfam/${sample}.log
done
```
