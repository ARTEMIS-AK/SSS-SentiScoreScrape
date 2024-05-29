# SSS-SentiScoreScrape

## Web Data Extraction and Text Analysis

This project demonstrates how to extract data from web pages, process and analyze the text content, and generate various text metrics using Python libraries such as BeautifulSoup, TextBlob, and NLTK.

## Project Description

### Libraries and Setup
1. **Import Required Libraries**: The necessary libraries are imported for data handling, web scraping, and text analysis.
    ```python
    import pandas as pd
    import nltk
    from textblob import TextBlob
    from nltk.tokenize import sent_tokenize, word_tokenize
    import requests
    from bs4 import BeautifulSoup
    ```

2. **Download NLTK Data**: The required NLTK data packages are downloaded.
    ```python
    nltk.download('punkt')
    nltk.download('averaged_perceptron_tagger')
    ```

### Function to Count Syllables
A function is defined to count the syllables in a word, which is useful for calculating text complexity metrics.
```python
def count_syllables(word):
    vowels = 'aeiouy'
    count = 0
    word = word.lower()
    if word[0] in vowels:
        count += 1
    for index in range(1, len(word)):
        if word[index] in vowels and word[index - 1] not in vowels:
            count += 1
    if word.endswith('e'):
        count -= 1
    if count == 0:
        count += 1
    return count
```

### Mount Google Drive (Optional)
If using Google Colab, mount Google Drive to access files.
```python
from google.colab import drive
drive.mount('/content/drive')
```

### Reading Input Data
The input data is read from a CSV file using pandas.
```python
input_data = pd.read_csv('/content/drive/MyDrive/BlackCoffer/Coffer2/Input.xlsx - Sheet1.csv')
```

### Data Extraction from URLs
For each URL in the input data, the HTML content is fetched, parsed, and saved to a text file.
```python
for index, row in input_data.iterrows():
    url = row['URL']
    url_id = row['URL_ID']

    response = requests.get(url)
    if response.status_code == 200:
        soup = BeautifulSoup(response.content, 'html.parser')
        with open(f"{url_id}.txt", 'w', encoding='utf-8') as file:
            title = soup.find('title').get_text()
            article_text = " ".join([p.get_text() for p in soup.find_all('p')])
            file.write(f"{title}\n\n{article_text}")
    else:
        print(f"Failed to fetch data from {url}")

print("Data extraction complete.")
```

### Text Analysis
For each extracted text file, various text metrics are calculated, including sentiment scores, complexity metrics, and word/sentence counts.
```python
import os
output_data = []

for index, row in input_data.iterrows():
    url_id = row['URL_ID']
    filename = f"{url_id}.txt"

    if os.path.exists(filename):
        with open(filename, 'r', encoding='utf-8') as file:
            content = file.read()

        sentences = sent_tokenize(content)
        words = word_tokenize(content)
        num_sentences = len(sentences)
        num_words = len(words)
        avg_sentence_length = num_words / num_sentences

        blob = TextBlob(content)
        polarity = blob.sentiment.polarity
        subjectivity = blob.sentiment.subjectivity

        complex_word_count = sum(count_syllables(word) > 3 for word in words)
        percentage_complex_words = (complex_word_count / num_words) * 100 if num_words > 0 else 0

        fog_index = 0.4 * (avg_sentence_length + percentage_complex_words)

        word_count = len(words)
        positive_score = (polarity + 1) / 2
        negative_score = (1 - polarity) / 2
        overall_polarity = abs(polarity)
        syllables_per_word = sum(count_syllables(word) for word in words) / num_words if num_words > 0 else 0

        personal_pronouns = sum(1 for word in words if word.lower() in ['i', 'me', 'my', 'mine', 'myself', 'you', 'your', 'yours', 'yourself', 'he', 'him', 'his', 'himself', 'she', 'her', 'hers', 'herself', 'we', 'us', 'our', 'ours', 'ourselves', 'they', 'them', 'their', 'theirs', 'themselves'])

        avg_word_length = sum(len(word) for word in words) / num_words if num_words > 0 else 0

        output_data.append({
            'URL_ID': url_id,
            'POSITIVE SCORE': positive_score,
            'NEGATIVE SCORE': negative_score,
            'POLARITY SCORE': overall_polarity,
            'SUBJECTIVITY SCORE': subjectivity,
            'AVG SENTENCE LENGTH': avg_sentence_length,
            'PERCENTAGE OF COMPLEX WORDS': percentage_complex_words,
            'FOG INDEX': fog_index,
            'AVG NUMBER OF WORDS PER SENTENCE': avg_sentence_length,
            'COMPLEX WORD COUNT': complex_word_count,
            'WORD COUNT': num_words,
            'SYLLABLE PER WORD': syllables_per_word,
            'PERSONAL PRONOUNS': personal_pronouns,
            'AVG WORD LENGTH': avg_word_length
        })
    else:
        print(f"File {filename} does not exist. Skipping...")
```

### Output Results
The calculated metrics are stored in a DataFrame and saved to an Excel file.
```python
output_df = pd.DataFrame(output_data)
final_output = pd.merge(input_data, output_df, on='URL_ID')
final_output.to_excel('output1.xlsx', index=False)

print("Text analysis complete. Output saved in output.xlsx.")
```

## Dependencies
- pandas
- nltk
- textblob
- requests
- beautifulsoup4

This project showcases the process of web scraping, text processing, and analysis to generate useful metrics from web content, making it a comprehensive example of end-to-end text data analysis in Python.
