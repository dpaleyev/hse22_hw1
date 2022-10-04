# hse22_hw1

## Обязательная часть

1. Создадим символьные ссылки чтоб избежать копирования.

```
ln -s /usr/share/data-minor-bioinf/assembly/oil_R1.fastq
ln -s /usr/share/data-minor-bioinf/assembly/oil_R2.fastq
ln -s /usr/share/data-minor-bioinf/assembly/oilMP_S4_L001_R2_001.fastq
ln -s /usr/share/data-minor-bioinf/assembly/oilMP_S4_L001_R1_001.fastq
```

2. Установим seed (в моем случае это 721) и через seqtk выбираем случайно 5 миллионов чтений типа paired-end и 1.5 миллиона чтений типа mate-pairs

```
seqtk sample -s721 oil_R1.fastq 5000000 > sub1.fastq
seqtk sample -s721 oil_R2.fastq 5000000 > sub2.fastq
seqtk sample -s721 oilMP_S4_L001_R1_001.fastq 1500000 > matepairs.fastq
seqtk sample -s722 oilMP_S4_L001_R2_001.fastq 1500000 > matepairs2.fastq
```


3. С помощью программы fastQC оцениваем качество исходных чтений и выводим по ним общую статистику c помощью multiQC.
```
mkdir fastqc
ls sub* matepairs* | xargs -tI{} fastqc -o fastqc {}
mkdir multiqc
multiqc -o multiqc fastqc
```
![Скрин](https://github.com/dpaleyev/hse22_hw1/blob/master/screenshots/screen1.png)
![Скрин2](https://github.com/dpaleyev/hse22_hw1/blob/master/screenshots/screen2.png)

4. С помощью программ platanus_trim и platanus_internal_trim подрезаем чтение по качеству.
```
platanus_trim sub*
platanus_internal_trim matepair*
```
5. Удаляем исходники.
```
rm sub1.fastq
rm sub2.fastq
rm matepairs.fastq 
rm matepairs2.fastq
```

6. С помощью программы fastQC оцениваем качество исходных чтений и выводим по ним общую статистику c помощью multiQC.
```
mkdir fastqc_trim
ls sub* matepairs*| xargs -tI{} fastqc -o fastqc_trim {}
mkdir multqctrim
multiqc -o multqctrim fastqc_trim
```
![Скрин3](https://github.com/dpaleyev/hse22_hw1/blob/master/screenshots/screen3.png)
![Скрин4](https://github.com/dpaleyev/hse22_hw1/blob/master/screenshots/screen4.png)

7. С помощью программы “platanus assemble” собираем контиги.
```
time platanus assemble -o Poil -f sub1.fastq.trimmed sub2.fastq.trimmed 2> assemble.log
```

8. С помощью программы “platanus scaffold” собираем скаффолды из контигов.
```
time platanus scaffold -o Poil -c Poil_contig.fa -IP1 sub1.fastq.trimmed sub2.fastq.trimmed -OP2 matepairs.fastq.int_trimmed matepairs2.fastq.int_trimmed 2> scaffold.log
```
9. C помощью программы “platanus gap_close” уменьшаем промежутки.
```
time platanus gap_close -o Poil -c Poil_scaffold.fa -IP1 sub1.fastq.trimmed sub2.fastq.trimmed -OP2 matepairs.fastq.int_trimmed  matepairs2.fastq.int_trimmed 2> gapclose.log
```

Ссылка на ноутбук: https://github.com/dpaleyev/hse22_hw1/blob/master/src/hw.ipynb
