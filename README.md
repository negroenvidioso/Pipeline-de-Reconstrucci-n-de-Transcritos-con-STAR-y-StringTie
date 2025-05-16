# 🧬 Pipeline de Reconstrucción de Transcritos con STAR y StringTie

Este repositorio contiene un pipeline automatizado para la alineación de lecturas y la reconstrucción de transcritos a partir de datos de RNA-seq, utilizando **STAR** y **StringTie**. Está diseñado para ser fácilmente adaptable a distintos proyectos y especies.

---

## 📁 Estructura del Proyecto

```
.
├── align_reads_STAR.sh       # Script generado para alinear lecturas con STAR
├── run_stringtie.sh          # Script generado para ensamblar transcritos con StringTie
├── fastq/                    # Archivos FASTQ filtrados
├── alignment/STAR/           # Archivos BAM alineados
├── stringtie/STAR/           # Archivos GTF y abundancias génicas
└── README.md                 # Este archivo
```

---

## ⚙️ Requisitos

- STAR (≥ 2.7) (https://github.com/alexdobin/STAR)
- StringTie (≥ 2.2) (https://github.com/gpertea/stringtie)
- GNU awk, gzip, bash
- Referencia genómica indexada para STAR
- Anotación de referencia en formato GFF/GTF

---

## 🚀 Ejecución del Pipeline

### 1. Preparación de archivos FASTQ

#### Comprimir archivos FASTQ (opcional)

```bash
for file in *.fastq; do gzip "$file"; done
```

#### Descomprimir archivos FASTQ (si es necesario)

```bash
for file in *.fastq.gz; do gzip -d "$file"; done
```

#### Calcular tamaño total descomprimido

```bash
for file in *.fastq.gz; do
  gzip -dc "$file" | wc -c
done | awk '{s+=$1} END {print "Total uncompressed size: " s/1024/1024/1024 " GB"}'
```

---

### 2. Alineación con STAR

#### Generar script de alineación

```bash
ls *_1_filt.fastq.gz | awk '{
  split($0,x,"_1_filt.fastq.gz");
  out=x[1];
  print "STAR --genomeDir /ruta/a/índice/STAR --readFilesIn "$0"\t"out"_2_filt.fastq.gz --runThreadN 20 --outFileNamePrefix /ruta/a/salida/alineamiento/"out" --outSAMtype BAM SortedByCoordinate"
}' > align_reads_STAR.sh

chmod +x align_reads_STAR.sh
```

> 🔁 **Reemplaza** `/ruta/a/índice/STAR` y `/ruta/a/salida/alineamiento/` con las rutas correspondientes en tu sistema.

#### Ejecutar alineación

```bash
./align_reads_STAR.sh
```

---

### 3. Reconstrucción de transcritos con StringTie

#### Generar script de ensamblado

```bash
ls *.bam | awk '{
  split($0,x,".bam");
  out=x[1];
  print "stringtie "out".bam -o /ruta/a/salida/stringtie/"out".gtf -p 10 -f 0.05 -m 200 --conservative -t -s 3.0 -l ncRNA -A /ruta/a/salida/stringtie/gene_abundances_"out".txt -c 0.5 -u"
}' > run_stringtie.sh
```

> 🔁 **Reemplaza** `/ruta/a/salida/stringtie/` con la ruta deseada para guardar los resultados.

#### Ejecutar ensamblado

```bash
./run_stringtie.sh
```

---

## 📌 Notas

- Asegúrate de que los nombres de los archivos FASTQ sigan el patrón `*_1_filt.fastq.gz` y `*_2_filt.fastq.gz`.
- El índice de STAR debe estar previamente generado con la referencia del genoma correspondiente.
- La anotación de referencia (`ref_ann.gff`) debe estar disponible para StringTie si se desea usar con la opción `-G`.

## 📞 Contacto

Para dudas o sugerencias, puedes abrir un issue en este repositorio o contactar al autor del pipeline.

Ph.D.(c) Allan Peñaloza - Otárola
allan.penaloza@ug.uchile.cl
