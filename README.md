# ДЗ1.
## Выполнила Михайлова Ирина Александровна.

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
## Оценка качества исходных данных секвенирования.

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
![multiqc0](https://github.com/user-attachments/assets/f7490f0d-27d0-4094-8031-50542cdd8c8a)
Рисунок 2 – Репорт *multiQC*.

#### msub1.fq:
+ Процент дупликаций: 4.5% — невысокий процент дупликаций, что говорит о хорошем качестве данных.
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

![multiqc1](https://github.com/user-attachments/assets/2a341397-6baf-4cea-a6d1-4bebb4ff491b)
Рисунок 3 – Среднее значение качества по каждой базовой позиции при чтении.

![p1](https://github.com/user-attachments/assets/74039563-9dc4-461b-9dee-b4ff83c86282) 
Рисунок 4 – Per base sequence quality для *psub1.fq*.
![p2](https://github.com/user-attachments/assets/5c7210a1-c81d-4862-8568-2619c5508634)
Рисунок 5 – Per base sequence quality для *psub2.fq*.

Таким образом, *psub1.fq* и *psub2.fq* прошли проверку качества, так как имеют высокое качество, без значительных ошибок или деградации по позициям.

![m1](https://github.com/user-attachments/assets/b528eb11-ff89-4bfd-ab94-816abe5d1dfe)
Рисунок 6 – Per base sequence quality для *msub1.fq*.
![m2](https://github.com/user-attachments/assets/fcf1c5f8-d367-4c04-af2a-b95ce0d9d248)
Рисунок 7 – Per base sequence quality для *msub2.fq*.

В свою очередь, *msub1.fq* - имеет среднее качество чтений - возможно, в некоторых позициях последовательности наблюдается ухудшение качества чтений; в *msub2.fq* большая часть ридов имеет средние качественные оценки, которые существенно ниже ожидаемого уровня, там может быть большое количество ошибок и «шумов» так как имеют низкокачественные позиции во многих местах, нужно триммировать риды.

![multiqc2](https://github.com/user-attachments/assets/a99c0e40-5b22-409f-8948-1e57036e5b37)
Рисунок 8 – Оценка качества последовательностей.

График Per Sequence Quality Scores показывает, как распределяется среднее качество прочтений по всему набору данных. Он помогает определить наличие прочтений с низким качеством (в данном случае все риды имеют хорошее качество, так как по оси X достигают больших значений), а также узнать, как качество распределено по всему набору. Если большинство данных имеет высокое качество, то пик на графике будет находиться на стороне высоких значений (ближе к 40), что и произошло в нашем случае.

![multiqc5](https://github.com/user-attachments/assets/0c63ed7b-f71f-4a78-b710-0c1c344151e9)
Рисунок 9 – Наличие адаптерных последовательностей.

По графику Adapter Content видно, что msub1.fq и msub2.fq содержат адаптерные последовательности, которые могут исказить результаты анализа, так как не являются частью исходного генома. Возможно из-за этого качество ридов плохое. Для этих последовательностей нужно проведём тримминг - удалим адаптеры с концов прочтений.

С помощью программы *platanus* проведём тримминг ридов и соберём контиги и скаффолды.

```
!platanus_trim psub1.fq psub2.fq -q 30 -t 16 -l 100
!platanus_trim msub1.fq msub2.fq -q 30 -t 16 -l 100
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

## Анализ качества триммированных ридов.

![multiqc02](https://github.com/user-attachments/assets/6b65e1b5-faa8-4376-9607-e1855ce9f06c)
Рисунок 10 – Репорт *multiQC* для триммированных ридов.

+ Мы видим, что снизился процент дублирования - после тримминга срезались адаптеры (что мы видим на рисунке 11) и низкокачественные последовательности (рисунки 12-16), тем не менее, много дубликатов осталось.
+ Длина прочтений после тримминга сократилась, так как срезались адаптеры и низкокачественные участки в конце прочтений.
+ Количество прочтений после тримминга существенно уменьшилось для всех образцов. Это также ожидаемо, так как в процессе тримминга удаляются не только низкокачественные участки, но и некоторые прочтения, которые не соответствуют заданным критериям качества.

![multiqc7](https://github.com/user-attachments/assets/bf5b44eb-6a99-4c9e-be91-54dfbbc5fa54)
Рисунок 11 – Наличие адаптерных последовательностей.
![multiqc12](https://github.com/user-attachments/assets/e4963a07-9d1c-4498-ae7c-1fd96956830d)
Рисунок 12 – Среднее значение качества по каждой базовой позиции при чтении.
![p11](https://github.com/user-attachments/assets/86d5e9e0-fc1f-4fdf-969a-c013dce52459)
Рисунок 13 – Per base sequence quality для *psub1.fq*.
![p22](https://github.com/user-attachments/assets/c954b865-2ea4-48cc-95fd-0bf655b7c673)
Рисунок 14 – Per base sequence quality для *psub2.fq*.
![m11](https://github.com/user-attachments/assets/e77c8820-cbe9-4b00-a4da-841b8e720f1c)
Рисунок 15 – Per base sequence quality для *msub1.fq*.
![m22](https://github.com/user-attachments/assets/a7a5c51a-2915-4c9a-ad0e-4a18bab9f185)
Рисунок 16 – Per base sequence quality для *msub2.fq*.
![multiqc22](https://github.com/user-attachments/assets/627375f1-1999-422d-aebc-232ac89aa399)
Рисунок 17 – Оценка качества последовательностей.

Качество последовательностей улучшилось.

Качество секвенирования, анализируемое по плитке (tile) на секвенаторе сомнительное.  Показатель Per tile sequence quality для mp1_sub выявил несколько красных плиток на графике. Возможные причины: 
+ Неисправности в секвенаторе - могут влиять на качество прочтений в определенных местах. 
+ Наличие пыли, грязи, пузырьков воздуха на поверхности секвенируемой плитки может препятствовать процессу секвенирования.
+ Неравномерное распределение кластеров ДНК на плитке или их недостаток может также привести к снижению качества секвенирования в определенных зонах.

## Анализ полученных контигов

```
f = open("/content/mate_contigs.fasta", "r", encoding='utf-8')
print(f.read().count('>'))
f.seek(0)
lines = f.readlines()
s = 0
l = []
for line in lines:
    if line[0] != '>':
        continue
    j = line.find('length') + 3
    k = line.find('_', j)
    n = int(line[j:k])
    s += n
    l.append(n)
print(s)
l = sorted(l,reverse=True)
print(l[0])
s2, i = 0, 0
while s2 < s * 0.5:
    s2 += l[i]
    i += 1
print(l[i-1])
f.close()
```
- Код для нахождения количества контигов, 
