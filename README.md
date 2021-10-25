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

3) Удаление ненужных файлов:
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

4) С помощью программ platanus_trim и platanus_internal_trim подрезаем чтения по качеству и удаляем праймеры:
```
platanus_trim pe_R1.fastq pe_R2.fastq 
platanus_internal_trim mp_R1.fastq mp_R2.fastq  
```
5) Удаляем ненужные файлы
```
ls -1 *.fastq | xargs -tI{} rm -r {}
```
6) Оценка качества подрезанных чтений и получение по ним общей статистики с помощью программы fastQC и multiQC :
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
https://user-images.githubusercontent.com/93148620/138770934-30630aac-e679-4971-9b35-d6c3bcdbf3f9.png
После:
![alt](![image](https://user-images.githubusercontent.com/93148620/138771024-8aca81f5-4932-4a9e-b06c-2b3300e83a92.png)![image](https://user-images.githubusercontent.com/93148620/138771183-197016ad-7038-44c6-880c-3b84b23105d2.png))

До:
![alt]([image](https://user-images.githubusercontent.com/93148620/138771233-95b325cb-507a-4952-be6c-1a0fe3529569.png))
После:
![alt](![image](https://user-images.githubusercontent.com/93148620/138771358-8fc8e266-46b9-4457-aacb-249f337c0684.png))

До:
![alt](![image](https://user-images.githubusercontent.com/93148620/138771443-f7cce033-26df-4eab-b47b-e816e3bf5878.png))
После:
![alt](![image](https://user-images.githubusercontent.com/93148620/138772053-0f89e027-a2b7-47e5-b15f-71a89aba5cb2.png))


7) С помощью программы “platanus assemble” собираем контиги из подрезанных чтений:
```
time platanus assemble -o Poil -f trimmed_fastq/pe_R1.fastq.trimmed trimmed_fastq/pe_R2.fastq.trimmed 2> assemble.log
```
8) Анализ полученных контигов (общее кол-во контигов, их общая длина, длина самого длинного контига, N50):

![alt](![image](https://user-images.githubusercontent.com/93148620/138772535-a2583802-f9d0-4807-a81c-44c8b21af55c.png))

9) С помощью программы “ platanus scaffold” собрать скаффолды из контигов, а также из подрезанных чтений:
```
time platanus scaffold -o Poil -c Poil_contig.fa -IP1 trimmed_fastq/pe_R1.fastq.trimmed  trimmed_fastq/pe_R2.fastq.trimmed -OP2 trimmed_fastq/mp_R1.fastq.int_trimmed trimmed_fastq/mp_R2.fastq.int_trimmed 2> scaffold.log
```
10) Анализ полученных скаффолдов и количество гэпов:

![alt](![image](https://user-images.githubusercontent.com/93148620/138772717-17d3a1e9-8352-4179-a0f6-46e0c364638d.png))
![alt](![Uploading image.png…])


11) Создание файла с одним самым большим размером:
```
echo scaffold1_len3834575_cov231 > name_scaff.txt
seqtk subseq Poil_scaffold.fa name_scaff.txt > BigScaff.fna
rm -r name_scaff.txt
```

11) С помощью программы “ platanus gap_close” уменьшаем кол-во гэпов с помощью подрезанных чтений:
```
time platanus gap_close -o Poil -c Poil_scaffold.fa -IP1 trimmed_fastq/pe_R1.fastq.trimmed  trimmed_fastq/pe_R2.fastq.trimmed -OP2 trimmed_fastq/mp_R1.fastq.int_trimmed trimmed_fastq/mp_R2.fastq.int_trimmed 2> gapclose.log
```
12) Анализ полученных скаффолдов с уменьшением гэпов:

jupyter_notebook

Создаем файл с одним самым большим размером:
```
echo scaffold1_cov231 > name_scaff.txt
seqtk subseq Poil_gapClosed.fa name_scaff.txt > longest.fna
rm -r name_scaff.txt
```
