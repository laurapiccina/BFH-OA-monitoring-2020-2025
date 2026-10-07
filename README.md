# BFH-OA-monitoring-2018-2025
BFH OA monitoring 2018-2025

import pandas as pd
import re

df = pd.read_excel("RIS, Arbor, OA mixed 7 ott.xlsx")

df['type'].count()
3437

df1 = pd.read_excel("Arbor collection with types as exported.xlsx")

# Clean DOI column
df1['doi'] = (
    df1['doi']
    .fillna('')
    .astype(str)
    .str.lower()
    .str.extract(r'(10\.\d{4,9}/[^\s|]+)', expand=False)
)
​
# Convert empty strings to <NA>
df1['doi'] = df1['doi'].replace('', pd.NA)

# Remove rows where both DOI columns are missing, empty, or "nan"
df1 = df1[
    ~(
        df1['doi'].fillna('').astype(str).str.strip().str.lower().isin(['', 'nan']) &
        df1['publisher doi'].fillna('').astype(str).str.strip().str.lower().isin(['', 'nan'])
    )
].copy()

df1['doi']
0        10.24451/arbor.22531
1        10.24451/arbor.14464
2        10.24451/arbor.17999
3        10.24451/arbor.15858
4       10.24451/dspace/11361
                ...          
7308     10.24451/arbor.18476
7310     10.24451/arbor.15794
7311     10.24451/arbor.18027
7312     10.24451/arbor.12624
7313     10.24451/arbor.19670
Name: doi, Length: 6228, dtype: object

import pandas as pd
import re
​
# ---------------------------------------------------------
# Clean and standardise publisher DOIs
# ---------------------------------------------------------
​
df1['publisher doi'] = (
    df1['publisher doi']
    .fillna('')
    .astype(str)
    .str.strip()
    .str.lower()
)
​
# Remove common DOI prefixes
df1['publisher doi'] = (
    df1['publisher doi']
    .str.replace(r'^https?://(dx\.)?doi\.org/', '', regex=True)
    .str.replace(r'^doi:\s*', '', regex=True)
    .str.replace(r'^doi\s+', '', regex=True)
)
​
# Extract the DOI itself
df1['publisher doi'] = (
    df1['publisher doi']
    .str.extract(
        r'(10\.\d{4,9}/[-._;()/:a-z0-9]+)',
        expand=False
    )
)
​
# Remove trailing punctuation that is not part of the DOI
df1['publisher doi'] = (
    df1['publisher doi']
    .str.rstrip('.,;:)')
)
​
# Convert empty values to <NA>
df1['publisher doi'] = df1['publisher doi'].replace('', pd.NA)

df1['publisher doi'].dropna().head(20)
1           10.1016/j.gaitpost.2021.02.008
4          10.23919/uia60812.2024.10716033
5            10.1080/19455224.2024.2347201
6                       10.3390/nu12061853
15    10.24840/2183-8976_2023-0008_0001_13
16                       10.1111/grs.12325
18              10.24894/978-3-7965-5336-3
20                     10.54916/rae.142567
25              10.1016/j.clnu.2021.03.013
28              10.1504/ijscor.2024.144592
30               10.1163/9789004751163_004
33                      10.1111/jocn.16478
34            10.1007/978-3-658-32323-3_17
36                     10.3390/jfmk5040074
39                  10.1055/s-0041-1736664
44                     10.3390/su132212931
49              10.1515/spircare-2022-0040
53             10.1007/978-3-658-37306-1_5
60            10.4229/eupvsec2023/3av.3.25
61               10.1024/1662-9027/a000192
Name: publisher doi, dtype: object

# Create lookup tables from df1
doi_to_type = (
    df1.dropna(subset=['doi'])
       .drop_duplicates('doi')
       .set_index('doi')['type']
)
​
publisher_doi_to_type = (
    df1.dropna(subset=['publisher doi'])
       .drop_duplicates('publisher doi')
       .set_index('publisher doi')['type']
)
​
# Fill missing type values in df using DOI
df['type'] = df['type'].fillna(
    df['doi'].map(doi_to_type)
)
​
# For any type values still missing, use publisher DOI
df['type'] = df['type'].fillna(
    df['publisher doi'].map(publisher_doi_to_type)
)

# Clean and standardize the 'type' column
df['type'] = (
    df['type']
    .fillna('')
    .astype(str)
    .str.strip()
    .str.lower()
    # Replace all types of hyphens/dashes with a normal hyphen
    .str.replace(r'[\u002D\u2010\u2011\u2012\u2013\u2014\u2212]', '-', regex=True)
    # Replace multiple spaces with one space
    .str.replace(r'\s+', ' ', regex=True)
    # Convert spaces and hyphens to underscores
    .str.replace(r'[\s-]+', '_', regex=True)
)
​
# Standardize categories
df['type'] = df['type'].replace({
    'journal_article': 'article',
    'journal-article': 'article',
    'book-chapter': 'book_section',
    'book_chapter': 'book_section',
    'working_paper': 'working_paper',
    'music': 'audio_visual'
})
​
# Empty values back to missing
df['type'] = df['type'].replace('', pd.NA)

df['type'].value_counts()
article             3178
magazine_article    1164
conference_item      902
book_section         775
report               258
book                 109
other                 83
working_paper         72
journal_series        64
audio_visual          11
thesis                 7
patent                 1
Name: type, dtype: int64

import matplotlib.pyplot as plt
​
counts = df['type'].value_counts()
​
ax = counts.plot(
    kind='bar',
    figsize=(10, 6),
    title='Publications by Resource Type'
)
​
ax.set_xlabel('Resource Type')
ax.set_ylabel('Number of Publications')
​
for i, v in enumerate(counts):
    ax.text(i, v + 20, str(v), ha='center')
​
plt.xticks(rotation=45, ha='right')
plt.tight_layout()
plt.show()


#Total number of publications (restricted and open) per resource type, per year 
​
result = (
    df.assign(type=df['type'].str.strip().str.lower())
       .groupby('date')['type']
       .value_counts()
)
​
print(result.to_string())
date  type            
2018  article             263
      book_section         21
      conference_item      17
      magazine_article      6
2019  article             296
      book_section         31
      conference_item      26
      report                5
      book                  3
2020  article             419
      magazine_article    145
      conference_item     118
      book_section        105
      report               37
      other                36
      book                 14
      working_paper         5
      journal_series        4
      thesis                1
2021  article             472
      conference_item     135
      book_section        118
      magazine_article    110
      report               45
      other                19
      working_paper        17
      book                 16
      journal_series        6
      audio_visual          1
      thesis                1
2022  article             434
      magazine_article    248
      conference_item     161
      book_section         95
      report               38
      book                 16
      working_paper        14
      journal_series       11
      other                 7
      audio_visual          1
      thesis                1
2023  article             399
      magazine_article    251
      conference_item     163
      book_section        132
      report               39
      book                 18
      journal_series       12
      working_paper        12
      other                 5
      audio_visual          3
      thesis                2
2024  article             468
      magazine_article    218
      conference_item     172
      book_section        159
      report               54
      book                 27
      working_paper        17
      journal_series       15
      other                 8
      audio_visual          4
      patent                1
2025  article             427
      magazine_article    186
      book_section        114
      conference_item     110
      report               40
      journal_series       16
      book                 15
      other                 8
      working_paper         7
      audio_visual          2
      thesis                2

#Total number of restricted publications per resource type, per year
​
result = (
    df.assign(type=df['type'].str.strip().str.lower())
       .loc[df['fulltext status'].str.strip().str.lower() == 'restricted']
       .groupby('date')['type']
       .value_counts()
)
​
print(result.to_string())
date  type            
2018  article             150
      book_section         14
      conference_item       9
      magazine_article      4
2019  article             158
      book_section         24
      conference_item      14
      report                4
2020  article             190
      conference_item      89
      magazine_article     65
      book_section         53
      report               18
      other                 6
      working_paper         4
      book                  1
2021  article             184
      conference_item      95
      book_section         47
      magazine_article     44
      report               16
      other                12
      working_paper         4
      book                  2
      audio_visual          1
      thesis                1
2022  article             130
      conference_item     104
      magazine_article     54
      book_section         44
      report               15
      working_paper         4
      other                 3
      audio_visual          1
      book                  1
2023  article             123
      conference_item     100
      magazine_article     43
      book_section         32
      report               15
      audio_visual          3
      working_paper         3
      other                 1
      thesis                1
2024  article             134
      conference_item     123
      magazine_article     35
      book_section         31
      report                6
      working_paper         6
      other                 5
      audio_visual          3
      journal_series        2
      book                  1
      patent                1
2025  article             108
      conference_item      63
      magazine_article     28
      book_section         26
      report                7
      book                  4
      audio_visual          2
      thesis                2
      journal_series        1
      other                 1
      working_paper         1

#Total number of open publications per resource type, per year
​
result = (
    df.assign(type=df['type'].str.strip().str.lower())
       .loc[df['fulltext status'].str.strip().str.lower() == 'open.access']
       .groupby('date')['type']
       .value_counts()
)
​
print(result.to_string())
date  type            
2018  article             113
      conference_item       8
      book_section          7
      magazine_article      2
2019  article             138
      conference_item      12
      book_section          7
      book                  3
      report                1
2020  article             229
      magazine_article     79
      book_section         52
      other                30
      conference_item      28
      report               19
      book                 12
      journal_series        4
      thesis                1
      working_paper         1
2021  article             288
      book_section         70
      magazine_article     65
      conference_item      35
      report               29
      book                 14
      working_paper        12
      other                 7
      journal_series        6
2022  article             304
      magazine_article    193
      book_section         50
      conference_item      45
      report               23
      book                 13
      journal_series       11
      working_paper        10
      other                 4
      thesis                1
2023  article             270
      magazine_article    205
      book_section         97
      conference_item      46
      report               24
      book                 18
      journal_series       12
      working_paper         8
      other                 2
      thesis                1
2024  article             332
      magazine_article    181
      book_section        127
      report               48
      conference_item      39
      book                 25
      journal_series       13
      working_paper        11
      other                 3
      audio_visual          1
2025  article             308
      magazine_article    156
      book_section         87
      conference_item      47
      report               33
      journal_series       15
      book                 11
      other                 7
      working_paper         6

# Total, restricted, and open publications
# per resource type, per year
​
# Clean type and fulltext status
df_temp = df.copy()
​
df_temp['type'] = df_temp['type'].str.strip().str.lower()
​
df_temp['fulltext status'] = (
    df_temp['fulltext status']
    .fillna('')
    .str.strip()
    .str.lower()
)
​
# Calculate all three counts
result = (
    df_temp
    .groupby(['date', 'type'])
    .agg(
        total=('type', 'size'),
        restricted=('fulltext status', lambda x: (x == 'restricted').sum()),
        open=('fulltext status', lambda x: (x == 'open.access').sum())
    )
    .reset_index()
    .sort_values(['date', 'type'])
)
​
# Display as a table in Jupyter
display(result)
date	type	total	restricted	open
0	2018	article	263	150	113
1	2018	book_section	21	14	7
2	2018	conference_item	17	9	8
3	2018	magazine_article	6	4	2
4	2019	article	296	158	138
...	...	...	...	...	...
69	2025	magazine_article	186	28	156
70	2025	other	8	1	7
71	2025	report	40	7	33
72	2025	thesis	2	2	0
73	2025	working_paper	7	1	6
74 rows × 5 columns


result.to_excel('publications_by_type_year.xlsx', index=False)

import pandas as pd
import matplotlib.pyplot as plt
​
# Total, restricted, and open publications
# per resource type, per year
​
# Clean type and fulltext status
df_temp = df.copy()
​
df_temp['type'] = (
    df['type']
    .str.strip()
    .str.lower()
)
​
df_temp['fulltext status'] = (
    df_temp['fulltext status']
    .fillna('')
    .str.strip()
    .str.lower()
)
​
# Calculate all three counts
result = (
    df_temp
    .groupby(['date', 'type'])
    .agg(
        total=('type', 'size'),
        restricted=('fulltext status', lambda x: (x == 'restricted').sum()),
        open=('fulltext status', lambda x: (x == 'open.access').sum())
    )
    .reset_index()
    .sort_values(['date', 'type'])
)
​
# ---------------------------------------------------------
# 1. Print table for copying into Excel
# ---------------------------------------------------------
​
print(result.to_csv(sep='\t', index=False))
​
​
# ---------------------------------------------------------
# 2. Prepare data for graphics
# ---------------------------------------------------------
​
plot_data = result.pivot(
    index='date',
    columns='type',
    values='total'
).fillna(0)
​
​
# ---------------------------------------------------------
# 3. Total publications by resource type and year
# ---------------------------------------------------------
​
ax = plot_data.plot(
    kind='bar',
    figsize=(12, 6)
)
​
ax.set_title('Total publications by resource type and year')
ax.set_xlabel('Year')
ax.set_ylabel('Number of publications')
ax.legend(
    title='Resource type',
    bbox_to_anchor=(1.02, 1),
    loc='upper left'
)
​
plt.tight_layout()
plt.show()
​
​
# ---------------------------------------------------------
# 4. Open vs. restricted publications by year
# ---------------------------------------------------------
​
status_by_year = (
    result
    .groupby('date')[['open', 'restricted']]
    .sum()
)
​
ax = status_by_year.plot(
    kind='bar',
    stacked=True,
    figsize=(11, 6)
)
​
ax.set_title('Open and restricted publications by year')
ax.set_xlabel('Year')
ax.set_ylabel('Number of publications')
ax.legend(title='Full-text status')
​
plt.tight_layout()
plt.show()
date	type	total	restricted	open
2018	article	263	150	113
2018	book_section	21	14	7
2018	conference_item	17	9	8
2018	magazine_article	6	4	2
2019	article	296	158	138
2019	book	3	0	3
2019	book_section	31	24	7
2019	conference_item	26	14	12
2019	report	5	4	1
2020	article	419	190	229
2020	book	14	1	12
2020	book_section	105	53	52
2020	conference_item	118	89	28
2020	journal_series	4	0	4
2020	magazine_article	145	65	79
2020	other	36	6	30
2020	report	37	18	19
2020	thesis	1	0	1
2020	working_paper	5	4	1
2021	article	472	184	288
2021	audio_visual	1	1	0
2021	book	16	2	14
2021	book_section	118	47	70
2021	conference_item	135	95	35
2021	journal_series	6	0	6
2021	magazine_article	110	44	65
2021	other	19	12	7
2021	report	45	16	29
2021	thesis	1	1	0
2021	working_paper	17	4	12
2022	article	434	130	304
2022	audio_visual	1	1	0
2022	book	16	1	13
2022	book_section	95	44	50
2022	conference_item	161	104	45
2022	journal_series	11	0	11
2022	magazine_article	248	54	193
2022	other	7	3	4
2022	report	38	15	23
2022	thesis	1	0	1
2022	working_paper	14	4	10
2023	article	399	123	270
2023	audio_visual	3	3	0
2023	book	18	0	18
2023	book_section	132	32	97
2023	conference_item	163	100	46
2023	journal_series	12	0	12
2023	magazine_article	251	43	205
2023	other	5	1	2
2023	report	39	15	24
2023	thesis	2	1	1
2023	working_paper	12	3	8
2024	article	468	134	332
2024	audio_visual	4	3	1
2024	book	27	1	25
2024	book_section	159	31	127
2024	conference_item	172	123	39
2024	journal_series	15	2	13
2024	magazine_article	218	35	181
2024	other	8	5	3
2024	patent	1	1	0
2024	report	54	6	48
2024	working_paper	17	6	11
2025	article	427	108	308
2025	audio_visual	2	2	0
2025	book	15	4	11
2025	book_section	114	26	87
2025	conference_item	110	63	47
2025	journal_series	16	1	15
2025	magazine_article	186	28	156
2025	other	8	1	7
2025	report	40	7	33
2025	thesis	2	2	0
2025	working_paper	7	1	6




df.to_csv('df.csv', index=False)
# Publications with a DOI: total, open, and closed per year

df_temp = df.copy()

# Clean DOI and fulltext status
df_temp['doi'] = (
    df_temp['doi']
    .fillna('')
    .astype(str)
    .str.strip()
)

df_temp['fulltext status'] = (
    df_temp['fulltext status']
    .fillna('')
    .astype(str)
    .str.strip()
    .str.lower()
)

# Keep only records with a DOI and years 2018–2025
df_temp = df_temp[
    (df_temp['doi'] != '') &
    (pd.to_numeric(df_temp['date'], errors='coerce').between(2018, 2025))
].copy()

df_temp['year'] = pd.to_numeric(df_temp['date'], errors='coerce').astype(int)

# Calculate total, open, and closed publications
result_doi = (
    df_temp
    .groupby('year')
    .agg(
        total=('doi', 'size'),
        open=('fulltext status', lambda x: (x == 'open.access').sum()),
        closed=('fulltext status', lambda x: (x == 'restricted').sum())
    )
    .reset_index()
    .sort_values('year')
)

display(result_doi)
# Publications with a DOI: total, open, and closed per year
​
df_temp = df.copy()
​
# Clean DOI and fulltext status
df_temp['doi'] = (
    df_temp['doi']
    .fillna('')
    .astype(str)
    .str.strip()
)
​
df_temp['fulltext status'] = (
    df_temp['fulltext status']
    .fillna('')
    .astype(str)
    .str.strip()
    .str.lower()
)
​
# Keep only records with a DOI and years 2018–2025
df_temp = df_temp[
    (df_temp['doi'] != '') &
    (pd.to_numeric(df_temp['date'], errors='coerce').between(2018, 2025))
].copy()
​
df_temp['year'] = pd.to_numeric(df_temp['date'], errors='coerce').astype(int)
​
# Calculate total, open, and closed publications
result_doi = (
    df_temp
    .groupby('year')
    .agg(
        total=('doi', 'size'),
        open=('fulltext status', lambda x: (x == 'open.access').sum()),
        closed=('fulltext status', lambda x: (x == 'restricted').sum())
    )
    .reset_index()
    .sort_values('year')
)
​
display(result_doi)
year	total	open	closed
0	2018	701	315	385
1	2019	861	353	502
2	2020	884	455	426
3	2021	940	526	406
4	2022	1027	654	357
5	2023	1037	684	321
6	2024	1143	780	347
7	2025	953	691	248

import matplotlib.pyplot as plt
​
ax = result_doi.set_index('year')[['open', 'closed']].plot(
    kind='bar',
    stacked=True,
    figsize=(10, 6)
)
​
ax.set_title('Publications with DOI by Open Access Status, 2018–2025')
ax.set_xlabel('Year')
ax.set_ylabel('Number of Publications')
ax.legend(['Open', 'Closed'])
​
plt.xticks(rotation=0)
plt.tight_layout()
plt.show()
