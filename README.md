# Human Lab Journal
**Проект H+: Анализ персонального генома человека**  
**Курс: Анализ геномных данных**

---

## Скачивание данных

Скачал данные генотипирования 23andMe и Genotek с Google Drive.

```bash
pip install gdown --break-system-packages
export PATH=$PATH:/home/fade000_rus/.local/bin

mkdir human_project
cd human_project

# 23andMe
gdown 1QJkwJe5Xl_jSVpqdTSNXP7sqlYfI666j

# Genotek
gdown 1piKi0FxVao-IvG81TAXhDKlHXvHE6uTm
```

Получены файлы: SNP_raw_v4_Full_20170514175358.txt.zip (4.2MB) и genotek.vcf (26MB).

---

##  Конвертация в VCF

Распаковал архив 23andMe и конвертировал в VCF с помощью PLINK.

```bash
unzip SNP_raw_v4_Full_20170514175358.txt.zip

# Скачать PLINK
wget https://s3.amazonaws.com/plink1-assets/plink_linux_x86_64_20231211.zip
unzip plink_linux_x86_64_20231211.zip
chmod +x plink
./plink --version  # PLINK v1.90b7.2

# Конвертация
./plink --23file SNP_raw_v4_Full_20170514175358.txt \
        --recode vcf \
        --out snps_clean \
        --output-chr MT \
        --snps-only just-acgt
```

Результат: 595 401 вариант загружен, 171 041 прошёл фильтры QC.  
PLINK определил пол: **мужской**.

---

##  Фильтрация позиций 0/0

Удалил позиции идентичные референсу.

```bash
conda install -c bioconda bcftools -y
bcftools view -i 'GT!="0/0"' snps_clean.vcf > snps_filtered.vcf
grep -v "^#" snps_filtered.vcf | wc -l
```

**Результат: 171 041 SNP** сохранено для дальнейшего анализа.

---

##  Определение гаплогрупп

### Материнская гаплогруппа (мтДНК)
Извлёк митохондриальные SNP и отправил на сайт https://dna.jameslick.com/mthap/

```bash
grep -E "^rs|^i" SNP_raw_v4_Full_20170514175358.txt | grep -w "MT" > mtdna_snps.txt
cat mtdna_snps.txt
```

**Результат: H(T152C)** — одна из наиболее распространённых гаплогрупп в Западной и Восточной Европе.

### Отцовская гаплогруппа (Y-хромосома)
Отправил данные на https://cladefinder.yseq.net/

**Результат: R-M417 (R1a)** — характерна для славянских народов (русские, поляки, украинцы).

---

##  Предсказание цвета глаз

Проверил генотипы по пяти ключевым SNP согласно модели Liu et al. (2013):

```bash
grep -E "rs12913832|rs12203592|rs12896399|rs16891982|rs6119471" \
     SNP_raw_v4_Full_20170514175358.txt
```

| SNP | Генотип |
|-----|---------|
| rs12913832 | A/G |
| rs12203592 | C/T |
| rs12896399 | G/G |
| rs16891982 | C/G |

**Результат: зелёные или серо-зелёные глаза** — гетерозиготный паттерн по rs12913832 указывает на не-голубой цвет, остальные SNP дают промежуточный фенотип.

---

##  Аннотация SNP через VEP

Загрузил файл snps_filtered.vcf на https://grch37.ensembl.org/Tools/VEP  
Параметры: сборка GRCh37, Phenotypes включены.

Обработано: **164 638 вариантов**  
Перекрытых генов: **27 595**

Скачал результаты в формате TXT и отфильтровал клинически значимые варианты по полю CLIN_SIG.

---

##  Отбор кандидатов для CRISPR-Cas9

Из результатов VEP отобрал 5 SNP для гипотетической коррекции:

| SNP | Ген | Позиция | Текущий аллель | Замена | Заболевание |
|-----|-----|---------|---------------|--------|-------------|
| rs460897 | CFH | chr1:196716319 | C/T | C/C | Макулярная дегенерация |
| rs10757274 | CDKN2B-AS1 | chr9:22096055 | A/G | A/A | Ишемическая болезнь сердца |
| rs2004640 | IRF5 | chr7:128578301 | G/T | G/G | Системная красная волчанка |
| rs28371699 | CYP2D6 | chr22:42526484 | A/C | A/A | Нарушение метаболизма лекарств |
| rs4402960 | IGF2BP2 | chr3:185511687 | G/T | G/G | Сахарный диабет 2 типа |

Все пять позиций гетерозиготны — CRISPR-Cas9 должен исправить риск-аллель на одной из хромосом.
