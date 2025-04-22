# Background and Goals
As a language learner, i use an APP to memorize 55 english words everyday and i benefit from this habit. However, when it comes to the language of Portuguese, there is no such a convenient tool. This is where my work comes in.
This is a set of Anki flashcards designed to help Chinese speakers memorize English and Portuguese vocabulary.

# Project Overview
This project can be divided into three main steps:
1. Finding the appropriate materials
2. Transforming those materials into a CSV file
3. Manipulating the data using SQL

# Step 1 & 2: Data Collection and Transformation
The most challenging parts were the first two steps. After some research and thanks to insights shared on Red Note, I discovered a lesser-known website offering free dictionary resources. All the dictionaries I collected were in the .mdx format, which isn't directly compatible with most data tools.

To make the data usable, I attempted to convert the .mdx files into .csv format using Python. However, I ran into issues due to an incorrectly configured virtual environment on my macOS system. I spent a full week troubleshooting the setup and debugging the code. Eventually, I hired an engineer to help me configure the Python environment properly—and finally, the script worked as intended.
## transform json to csv
```python
import json
import csv
with open("/Users/zhoulingwei/Documents/dict_edit/json_to_csv/BEC_3.json", "r", encoding="utf-8") as file:
    data = [json.loads(line) for line in file]

header = ['wordRank', 'headWord', 'sentence', 'usphone', 'ukspeech', 'synonyms', 'phrase', 'trans']

with open("BEC_3_output.csv", "w", newline="", encoding="utf-8") as csv_file:
    writer = csv.writer(csv_file)
    writer.writerow(header)  # 写入列名

    for entry in data:
      
        word_data = [
            entry.get('wordRank'),
            entry.get('headWord'),
            entry.get('content', {}).get('word', {}).get('content', {}).get('sentence', {}).get('sentences', [{}])[0].get('sContent', ''),
            entry.get('content', {}).get('word', {}).get('content', {}).get('usphone', ''),
            entry.get('content', {}).get('word', {}).get('content', {}).get('ukspeech', ''),
            entry.get('content', {}).get('word', {}).get('content', {}).get('syno', {}).get('synos', [{}])[0].get('tran', ''),
            entry.get('content', {}).get('word', {}).get('content', {}).get('phrase', {}).get('phrases', [{}])[0].get('pContent', ''),
            entry.get('content', {}).get('word', {}).get('content', {}).get('trans', [{}])[0].get('tranCn', '')
        ]
        writer.writerow(word_data)

print("JSON transformed to CSV ！")
```
## transform mdx to csv
```python
import readmdict 
import pandas as pd
import csv

def mdx_to_csv(mdx_file, csv_file):
    # read .mdx 
    dictionary = readmdict.MDX(mdx_file)
    entries = dictionary.items()
    
    # transform the data to DataFrame
    data = []
    for word, definition in entries:
        data.append([word.decode("utf-8").strip(), definition.decode("utf-8").strip()])
    
    df = pd.DataFrame(data, columns=["Word", "Definition"])
    
    
    df.to_csv(csv_file, index=False, quoting=csv.QUOTE_ALL, encoding="utf-8-sig")
    print(f"CSV file was saved as {csv_file}")

if __name__ == "__main__":
    mdx_to_csv(
        r"/Users/zhoulingwei/Documents/葡语学习/葡语词典/Dicionário Vicon [PT-EN-PT].mdx",
        r"/Users/zhoulingwei/Documents/葡语学习/葡语词典/output.csv"
    )
```

# Step 3: SQL Practice
Compared to the earlier steps, the SQL portion was more straightforward. That said, it still offered valuable learning opportunities.

In the past, I had only taken online SQL courses and practiced using pre-built datasets. This was the first time I built a dataset from scratch and manipulated it on my own. Through this project, I enhanced my skills in creating tables, modifying structures, and writing more practical, real-world SQL queries.

```sql

with pt_en_pt as(
select
right(word, char_length(word)-position('.' IN word)) as new_word,
left(definition, position('.' in definition)) as en_word,
right(definition, char_length(definition)-position('.' in definition)) as pt_definition
from dict_edit.no_html)

SELECT pt.en_word, pt.pt_definition,en.phrase, en.translation_cn
FROM pt_en_pt as pt
left join dict_edit.bec_2_outputv3 as en
on en.word = pt.new_word
where en.word is not null;
```
