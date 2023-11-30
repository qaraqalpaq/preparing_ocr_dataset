
# Adding Karakalpak Language to Tesseract OCR

---
[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/gist/davron112/ee9777c2c692d894ec7f6d1f9558539c/training_kaa_latin.ipynb)


## Overview
This guide details the process of adding support for the Karakalpak language to the Tesseract OCR engine. It involves setting up the environment, preparing training data, training the model, and generating the Karakalpak `.traineddata` file.

# STEP-1: Info


# Dataset Requirements for Tesseract OCR Training

This document outlines the necessary components and formats for creating a dataset for Tesseract OCR training, specifically tailored for the Karakalpak language.

## Training Text (`.training_text`)

### Requirements
- Large, diverse set of sentences or phrases in Karakalpak.
- Covers a wide range of vocabulary and sentence structures.
- Represents actual language usage, including common phrases and expressions.

### Example

```
Meniń atym Jánibek.
Búgın hawa ayaz.
Adamlar paydalanatuǵın gezlemler tiykarınan tábiyǵıy hám ximiyalıq
talshıqlardan alınadı.
Men Qaraqalpaqstanda jasayman
```

## Wordlist (`.wordlist`)

### Requirements
- List of words in Karakalpak, one per line.
- Includes a comprehensive range of commonly used words.
- Special focus on words with unique Karakalpak characters.

### Example

```
men
sen
ol
biz
sizler
olar
kitap
qalam
mektep
adamlar
```

## Unicharset (`.unicharset`)

### Requirements
- Lists all unique characters found in the training text.
- Each line represents a character and its properties.

### Example

```
a 0 Common 0
b 1 Common 0
Á 2 Latin 1
...
```

## Unicharambigs (`.unicharambigs`)

### Requirements
- Specifies ambiguities in character recognition.
- Helps Tesseract understand potentially confusing character sequences.

### Example

```
1    sh    s h    0
```

## Config Files

### Requirements
- Contain settings and parameters for Tesseract training.
- Include language-specific configurations.
- Tailored to the Karakalpak language.

### Example

```
tessedit_char_whitelist 0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZÁǴÍŃÓÚShCh
```

## Box/Tiff Files for Training

### Requirements
- Box files map text regions in an image.
- Tiff files are corresponding image files.
- Both should be accurately aligned.

### Example
- `image.tif` - Image file.
- `image.box` - Corresponding box file.

## Font Properties File (`.font_properties`)

### Requirements
- Information about fonts used in training.
- Includes font name, styles (italic, bold, etc.).

### Example

```
Arial 0 0 0 0 0
TimesNewRoman 0 0 0 1 0
```

Adjust these examples as needed to fit the specific characteristics of the Karakalpak language.



#STEP-2: Start

## Prerequisites
- Google Colab or a similar environment.
- Basic Python programming and command-line knowledge.
- Tesseract OCR and related dependencies.

## Setup Instructions

### 1. Environment Setup

Mount your Google Drive and navigate to your Tesseract directory:

```
from google.colab import drive
drive.mount('/content/drive')
```

# Navigate to Tesseract directory

```
git clone https://github.com/qaraqalpaq/preparing_ocr_dataset.git tesseract
chmod 777 -R /content/drive/MyDrive/tesseract
cd /content/drive/MyDrive/tesseract
```

### 2. Install Required Libraries


```
sudo apt install tesseract-ocr
sudo apt-get install tesseract-ocr libtesseract-dev libleptonica-dev pkg-config
sudo apt-get install libicu-dev libpango1.0-dev libcairo2-dev
pip install pytesseract
```

### 3. Preparing Training Data

Check the training text file format:

```
def check_file_requirements(file_path):
    # Implementation here
```

Generate training images:

```bash
text2image --text=./langdata/kaa/kaa.training_text --outputbase=./box-training/kaa --font='Times New Roman' --fonts_dir=./fonts
```

### 4. Training the Model

Begin the training process:

```bash
tesseract ./kaa.tif ./lstmf/kaa_model_output --psm 6 lstm.train
```

Add ready `kaa_model_output.lstmf` file to training_files.txt

```bash
ls ./lstmf/*.lstmf > ./lstmf/training_files.txt
```

### 5. Additional Training Steps

- Extract unicharset

```
!unicharset_extractor --output_unicharset ./langdata/kaa/kaa.unicharset ./box-training/kaa.box
```

- Create a `wordlist` from `kaa.traintext`

```
import re

def create_wordlist_from_train_text(input_path, output_path):
    with open(input_path, 'r', encoding='utf-8') as file:
        text = file.read()

    # Apply your regex transformations here
    # Add your transformations as shown in the previous example

    # Split text into words and remove duplicates
    words = set(re.findall(r'\b[a-zA-ZÁáÚúÓóÍıǴǵÚúŃńÓó]+\b', text))

    # Write the unique words to the output file
    with open(output_path, 'w', encoding='utf-8') as file:
        for word in sorted(words):
            file.write(word + '\n')

# Example usage
train_text_path = './langdata/kaa/kaa.training_text'
wordlist_output_path = './langdata/kaa/kaa.wordlist'
create_wordlist_from_train_text(train_text_path, wordlist_output_path)
```

Combine model components.
```
combine_lang_model \
  --input_unicharset ./langdata/kaa/kaa.unicharset \
  --script_dir ./langdata \
  --words ./langdata/kaa/kaa.wordlist \
  --numbers ./langdata/kaa/kaa.numbers \
  --puncs ./langdata/kaa/kaa.punc \
  --output_dir ./output-combine/ \
  --lang kaa
```

- Modify configurations if needed.

```
combine_tessdata -o ./output-combine/kaa/kaa.traineddata \
  ./langdata/kaa/kaa.unicharset
```

- Extract `.lstm` from english model or old trained model
  ```
  combine_tessdata -e ./eng-model/eng.traineddata ./eng-model/extract_eng/eng.lstm

  ```
  
  or
  
  ```
  combine_tessdata -e ./finish/kaa.traineddata ./eng-model/extract_kaa/kaa.lstm
  ```

### 6. Fine-tuning and Evaluating the Model

- Starting LSTM Training for a New Language using an English Model
  ```bash
  lstmtraining \
  --continue_from ./eng-model/extract-eng/eng.lstm \
  --model_output ./trained_models/kaa \
  --traineddata ./output-combine/kaa/kaa.traineddata \
  --train_listfile ./lstmf/training_files.txt \
  --max_iterations 1000
  ```

- Fine-tuning or Retraining an Existing LSTM Model for a New Language
    ```bash
    lstmtraining \
        --continue_from ./eng-model/extract-kaa/kaa.lstm \
        --old_traineddata ./finish/kaa.traineddata \
        --traineddata ./output-combine/kaa/kaa.traineddata \
        --train_listfile ./lstmf/training_files.txt \
        --model_output ./trained_models/kaa \
        --max_iterations 1000
    ```
    
or 

 ```bash
  lstmtraining \
  --continue_from ./trained_models/kaa.checkpoint \
  --model_output ./trained_models/kaa \
  --traineddata ./output-combine/kaa/kaa.traineddata \
  --train_listfile ./lstmf/training_files.txt \
  --max_iterations 1000
  ```

### 7. Additional Scripts and Utilities

- Sentence generator from wordlist.
- Merge unicharsets.
- Parse HTML content.
- Sort unicharset.

### 8. Finalizing the Trained Model

Convert the checkpoint files into a `.traineddata` file:

```bash
lstmtraining --stop_training \
              --continue_from ./trained_models/kaa.checkpoint \
              --traineddata ./output-combine/kaa/kaa.traineddata \
              --model_output ./finish/kaa.traineddata
```

## Conclusion

You should now have a functional `.traineddata` file for Karakalpak language, ready for use with Tesseract OCR.

## Notes

- Ensure accuracy in paths and filenames.
- Regularly save progress.
- Use representative training data for best results.
