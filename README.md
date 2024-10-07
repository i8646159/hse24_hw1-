Выполнила задание в колабе.

Посмотрим, что содержат файлы с ридами:

```!head /content/blya/oilMP_S4_L001_R1_001.fastq```
![factq](https://github.com/user-attachments/assets/c8849e5e-2ffd-454f-a42a-1ab83838d5e1)
Рисунок 1 – Вывод кода.

Мы видим, что каждая запись в файле формата .fastq предоставляет последовательность нуклеотидов и информацию о качестве этой последовательности, они состоят из 4-х строк:
1. Первая строка начинается с символа “@” – это заголовок записи. Он содержит информацию о считывании, включая идентификатор последовательности, информацию о потоке, номер потока, номер кластера.
2. Вторая строка – это последовательность нуклеотидов, полученная в результате секвенирования. Она может содержать символы A, T, C, G, а также N (недоступные или неопределенные нуклеотиды).
3. Третья строка, содержащая только символ “+” – разделитель.
4. Четвёртая строка – строка качества, которая соответствует нуклеотидам из второй строки. Каждый символ в этой строке представляет собой качество соответствующего нуклеотида, закодированное в формате ASCII. 

Установим *seqtk* для задания и запустим:
```
!apt-get install -y seqtk

!seqtk sample -s118 /content/hw1/oilMP_S4_L001_R1_001.fastq 1500000 > /content/hw1/msub1.fq
!seqtk sample -s118 /content/hw1/oilMP_S4_L001_R2_001.fastq 1500000 > /content/hw1/msub2.fq
!seqtk sample -s118 /content/hw1/oil_R1.fastq 5000000 > /content/hw1/psub1.fq
!seqtk sample -s118 /content/hw1/oil_R2.fastq 5000000 > /content/hw1/psub2.fq
```
Теперь у нас есть 4 файла, содержащие ограниченное количество случайно выбранных чтений. Оценим качество исходных чтений и получим по ним общую статистику с помощью программы *fastQC* и *multiQC*.
```
!wget https://www.bioinformatics.babraham.ac.uk/projects/fastqc/fastqc_v0.11.9.zip
!unzip fastqc_v0.11.9.zip

!chmod +x FastQC/fastqc
```
– Установка.
```
!mkdir fastqreport
!mkdir multiqcreport

!FastQC/fastqc -o fastqreport /content/hw1/psub1.fq
!FastQC/fastqc -o fastqreport /content/hw1/psub2.fq
!FastQC/fastqc -o fastqreport /content/hw1/msub1.fq
!FastQC/fastqc -o fastqreport /content/hw1/msub2.fq
```
– Запуск.
```
!pip install --upgrade --force-reinstall git+https://github.com/MultiQC/MultiQC.git
```
– Установка.
```
!multiqc fastqreport --filename "multiqc_report" --outdir "multiqcreport"
```
– Запуск.

Получились репорты в формате .html – посмотрим, что там.
![multiqc1](https://github.com/user-attachments/assets/cab7b255-9440-421d-9386-eb5cefbf4366)
Рисунок 2 – Репорт *multiQC*.
#### msub1.fq:
+ Процент дупликаций: 4.4% — невысокий процент дупликаций, что говорит о хорошем качестве данных.
+ Процент GC: 44% — типичный уровень для большинства организмов.
+ Длина прочтений: 251 bp — длинные прочтения, что хорошо для
сборки.
#### msub2.fq:
+ Процент дупликаций: 3.9% — также низкий процент, что говорит о низком
уровне дублирования и высоком качестве данных.
+ Процент GC: 44% — как и в msub1.fq.
#### psub1.fq:
+ Процент дупликаций: 32.1% — более высокий процент дупликаций по
сравнению с mate-pair, что может указывать на избыточность данных
или артефакты.
+ Процент GC: 46% — немного выше, чем у mate-pair данных, что может быть
особенностью фрагмента.
+ Длина прочтений: 101 bp — короткие прочтения, стандартные для
paired-end данных.
#### psub2.fq:
+ Процент дупликаций: 30.6% — также высокий процент дупликаций, схожий с
#### pe1_sub.
+ Процент GC: 46% — как и в psub1.fq.


Рисунок 3 – Репорт 
```
!platanus_trim psub1.fq psub2.fq -q 25 -t 16 -l 100
!platanus_trim msub1.fq msub2.fq -q 25 -t 16 -l 100
!fastqc *.trimmed
!platanus assemble -o platanus -f psub1.fq.trimmed psub2.fq.trimmed -t 16
!mv platanus* contigs 
!platanus assemble -o plat -f msub1.fq.trimmed msub2.fq.trimmed -t 16 
!mv plat* contigs
!platanus scaffold -o platanus_scaffold -c contigs/platanus_contig.fa -IP1 psub1.fq.trimmed psub2.fq.trimmed -t 16 
!platanus scaffold -o plat_scaffold -c contigs/plat_contig.fa -IP1 msub1.fq.trimmed msub2.fq.trimmed -t 16
!platanus gap_close -o gap_close -c platanus_scaffold_scaffold.fa -IP1 psub1.fq.trimmed psub2.fq.trimmed -t 16
!platanus gap_close -o gap_closeS -c plat_scaffold_scaffold.fa -IP1 msub1.fq.trimmed msub2.fq.trimmed -t 16
```
