
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
Ózbekstan - meniń vatanym.
Búgın hawaj ayaz.
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
cd /content/drive/MyDrive/tesseract
chmod 777 -R /content/drive/MyDrive/tesseract
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
text2image --text=<training_text_path> --outputbase=<output_path> --font='Times New Roman' --fonts_dir=<fonts_directory>
```

### 4. Training the Model

Begin the training process:

```bash
tesseract <tif_file_path> <output_path> --psm 6 lstm.train
```

### 5. Additional Training Steps

- Extract unicharset.
- Create a wordlist.
- Combine model components.
- Modify configurations.
- Generate `.traineddata` file.

### 6. Fine-tuning and Evaluating the Model

Fine-tune the model:

```bash
lstmtraining --continue_from <checkpoint_path> --traineddata <traineddata_path> --max_iterations 1000
```

### 7. Additional Scripts and Utilities

- Sentence generator from wordlist.
- Merge unicharsets.
- Parse HTML content.
- Sort unicharset.

### 8. Finalizing the Trained Model

Convert the checkpoint files into a `.traineddata` file:

```bash
lstmtraining --stop_training --continue_from <checkpoint_path> --traineddata <traineddata_path> --model_output <final_traineddata_path>
```

## Conclusion

You should now have a functional `.traineddata` file for Karakalpak language, ready for use with Tesseract OCR.

## Notes

- Ensure accuracy in paths and filenames.
- Regularly save progress.
- Use representative training data for best results.

---
This markdown file is structured to provide clear instructions and descriptions, making it suitable for users with varying levels of expertise. Remember to replace placeholder paths like `<training_text_path>` with actual paths specific to your setup.
