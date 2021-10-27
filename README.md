# hse21_hw1
1) Создаем папку hw и создаем ссылки на нужные данные:
```
mkdir hw
cd hw
mkdir 1
cd 1
ls -1 /usr/share/data-minor-bioinf/assembly/* | xargs -tI{} ln -s {}
```
2) Выбор случайных чтений:
```
seqtk sample -s419 oil_R1.fastq 5000000 > pe_R1.fastq
seqtk sample -s419 oil_R2.fastq 5000000 > pe_R2.fastq
seqtk sample -s419 oilMP_S4_L001_R1_001.fastq 1500000 > mp_R1.fastq
seqtk sample -s419 oilMP_S4_L001_R2_001.fastq 1500000 > mp_R2.fastq
```
3) Удаление ненужных файлов:
```
rm -r oil_R1.fastq
rm -r oil_R2.fastq
rm -r oilMP_S4_L001_R1_001.fastq
rm -r oilMP_S4_L001_R2_001.fastq
```
4) Оценка качества исходных чтений и получение по ним общей статистики с помощью программы (fastQC и multiQC):
```
mkdir fastqc
ls *.fastq | xargs -P 4 -tI{} fastqc -o fastqc {}

mkdir multiqc
multiqc -o multiqc fastqc
```
Для скачивания файлов с сервера была использована программа WinSCP

5) С помощью программ platanus_trim и platanus_internal_trim подрезаем чтения по качеству и удаляем праймеры:
```
platanus_trim pe_R1.fastq pe_R2.fastq 
platanus_internal_trim mp_R1.fastq mp_R2.fastq  
```
6) Удаляем ненужные файлы
```
ls -1 *.fastq | xargs -tI{} rm -r {}
```
7) Оценка качества подрезанных чтений и получение по ним общей статистики с помощью программы fastQC и multiQC :
```
mkdir trimmed_fastq
mv -v *trimmed trimmed_fastq/
```
```
mkdir trimmed_fastqc
ls trimmed_fastq/* | xargs -P 4 -tI{} fastqc -o trimmed_fastqc {}
```
```
mkdir trimmed_multiqc
multiqc -o trimmed_multiqc trimmed_fastqc
```
До:
![image](https://user-images.githubusercontent.com/93148620/138774274-ebe7a729-4d3e-4c3e-9077-3e302e504bfd.png)
После:
![image](https://user-images.githubusercontent.com/93148620/138773889-c2424f13-6901-4cc6-9918-9909fddec3b5.png)

До:
![image](https://user-images.githubusercontent.com/93148620/138773978-bb020ffb-2f95-45f8-b16e-10ad1ba558e3.png)
После:
![image](https://user-images.githubusercontent.com/93148620/138774033-531dc199-2055-444a-b5ef-fb8550108383.png)

До:
![image](https://user-images.githubusercontent.com/93148620/138774084-269a2444-7d06-4412-b063-e7a61d1196fa.png)
После:
![image](https://user-images.githubusercontent.com/93148620/138774132-42f60f8c-f1a2-4ed8-82e8-dccdb8c1012a.png)


8) С помощью программы “platanus assemble” собираем контиги из подрезанных чтений:
```
time platanus assemble -o Poil -f trimmed_fastq/pe_R1.fastq.trimmed trimmed_fastq/pe_R2.fastq.trimmed 2> assemble.log
```
9) Анализ полученных контигов (общее кол-во контигов, их общая длина, длина самого длинного контига, N50):

![image](https://user-images.githubusercontent.com/93148620/139102723-265d33ae-c9c5-4020-bfe2-c5b4a4d8adc2.png)

10) С помощью программы “ platanus scaffold” собрать скаффолды из контигов, а также из подрезанных чтений:
```
time platanus scaffold -o Poil -c Poil_contig.fa -IP1 trimmed_fastq/pe_R1.fastq.trimmed  trimmed_fastq/pe_R2.fastq.trimmed -OP2 trimmed_fastq/mp_R1.fastq.int_trimmed trimmed_fastq/mp_R2.fastq.int_trimmed 2> scaffold.log
```
11) Анализ полученных скаффолдов и количество гэпов:
![image](https://user-images.githubusercontent.com/93148620/139102877-2baa77ea-612a-40bf-9f9b-b00e2e74708a.png)
![image](https://user-images.githubusercontent.com/93148620/139102971-2e600d26-0040-4813-9b6f-2fe3834ada84.png)


12) Создание файла с самым длинным скаффолдом:
```
echo scaffold1_len3834575_cov231 > name_scaff.txt
seqtk subseq Poil_scaffold.fa name_scaff.txt > BigScaff.fna
rm -r name_scaff.txt
```

13) С помощью программы “ platanus gap_close” уменьшаем кол-во гэпов с помощью подрезанных чтений:
```
time platanus gap_close -o Poil -c Poil_scaffold.fa -IP1 trimmed_fastq/pe_R1.fastq.trimmed  trimmed_fastq/pe_R2.fastq.trimmed -OP2 trimmed_fastq/mp_R1.fastq.int_trimmed trimmed_fastq/mp_R2.fastq.int_trimmed 2> gapclose.log
```
14) Снова достаем самый длинный скаффолд:
```
echo scaffold1_cov231 > name_scaff.txt
seqtk subseq Poil_gapClosed.fa name_scaff.txt > longest.fna
rm -r name_scaff.txt
```
