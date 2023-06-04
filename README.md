# To Do
- [DeepLearning.AI](https://www.deeplearning.ai/short-courses/), founded by [Prof. Andrew Ng](https://www.andrewng.org/) , has released three new videos.
  - LangChain for LLM Application Development
  - How Diffusion Models Work
  - Building Systems with the ChatGPT API
- All three will be online courses in application development using the Large Language Model (LLM)
- However, as of 6/4/2023, the transcription function ("TRANSCRIPT" button circled in blue below) is not available, so the video will be transcribed and translated into Japanese using Whisper and DeepL API.
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/67194/1b8fe6d4-3dcc-6509-625e-18b50627d01d.png)

- The environment will be on Google Colaboratory.

# Overview
1. video downloading & conversion to audio data
2. transcription of audio files
3. translation of text data
4. file output
5. check the result

# 1. video downloading & conversion to audio data
- Press "︙" at the bottom right of [Video](https://learn.deeplearning.ai/langchain/lesson/1/introduction) to select "Download" to download the video file.
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/67194/fbf4efa9-fd8d-6a38-3683-ca9a5bd4eb98.png)
- After downloading all the videos, we will convert them to audio data
sequential numbers are given at the beginning of the downloaded file names (0_XXX.mp4, 1_YYYY.mp4)
And then, I converted the mp4 to mp3 files at a website called [Audio Converter](https://online-audio-converter.com/ja/)
- Upload the mp3 file to any folder on Google Drive
(It will be used later on Google Colab)

# 2. transcription of audio files
- Each mp3 file is transcribed using [Whisper](https://openai.com/research/whisper), an automatic speech recognition (ASR) system provided by OpenAI.
- Run Google Colab and change "Runtime Type" to GPU
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/67194/86e0c17c-c10a-2000-5432-2cadd2d433db.png)

- Install Whisper
```bash
!pip install git+https://github.com/openai/whisper.git
```

- Select the model of Whisper to be used for transcription.
In this case we use 'base', but there are several models available, as [follows](https://github.com/openai/whisper#available-models-and-languages) 
```python
model = whisper.load_model("base")
```
| Size | Parameters | English-only model | Multilingual model | Required VRAM | Relative speed |
| --- | --- | --- | --- | --- | --- |
| tiny | 39 M | tiny.en | tiny | ~1 GB | ~32x |
| base | 74 M | base.en | base | ~1 GB | ~16x |
| small | 244 M | small.en | small | ~2 GB | ~6x |
| medium | 769 M | medium.en | medium | ~5 GB | ~2x |
| large | 1550 M | N/A | large | ~10 GB | 1x |


- Get the path to the mp3 files stored in step #1
Mount Google Drive and then get the path to all mp3 files
```python
# mount google drive
from google.colab import drive
drive.mount('/content/drive')

# file path
files = glob.glob("/content/drive/My Drive/<folder containing the mp3 files of #1>/*.mp3")
```

- Transcribe all files.
In #1, sequential numbers were added to the beginning of file names (0_XXX.mp3, 1_YYYY.mp3), and these initial numbers are used as keys for results (dict type).
```python
results = {}

# transcription of all file
for file in files:
    results[(os.path.basename(file))[:1]] = model.transcribe(file)
```

# 3. translation of text data
- Translate transcribed text data into Japanese
- Translation is available via an API service called "DeepL API Free" published by [DeepL](https://www.deepl.com/translator).
Translate up to 500,000 words per month for free!
- To use "DeepL API Free", you need to create an account on DeepL and get an API key, so I referred to the following website (in Japanese).

https://www.kkaneko.jp/ai/online/deeplapi.html

- Install DeepL
```bash
!pip install --upgrade deepl
```

- Use the API key you got to translate
See [GitHub](https://github.com/DeepLcom/deepl-python#usage) for Translator usage

```python
# preparing the translator
auth_key = "<DeepL API Key>"
translator = deepl.Translator(auth_key)

results_ja = {}

# translation of all data
for key in results.keys():
    results_ja[key] = translator.translate_text(results[key]["text"], target_lang="JA").text
```

# 4. file output
- Output the result to a text file
（Line breaks are inserted after a period(.) in English, and after a reading point(。) in Japanese）

- Set output folder (any folder)
```python
# output path
out_path = "/content/drive/My Drive/<folder containing the mp3 files of #1>/output-text/"
```

- Output transcribed English text by audio data
```Python
for key in results.keys():
    tmp = results[key]['text'].replace('. ', '.\n')

    with open(out_path + str(key) + "_en.txt", 'w', encoding='utf-8') as f:
        f.write(tmp)
```

- Output transcribed & translated Japanese text by audio data
```Python
for key in results_ja.keys():
    tmp = results_ja[key].replace('。', '。\n')

    with open(out_path + str(key) + "_ja.txt", 'w', encoding='utf-8') as f:
        f.write(tmp)
```


# 5. check the result
- A quick look at the results shows that the following new words (?) are not correctly transcribed, but they seem to be generally good.
    - Lanchane -> LangChain
    - LLO -> LLM
 
```text:0_en.txt
Welcome to this short course on Lanchane for large language model application development.
By prompting an LOM or large language model is now possible to develop AI applications much faster than ever before.
But an application can require prompting an LOM multiple times and parsing as output.
・・・
```

```text:0_ja.txt
大規模言語モデルアプリケーション開発のためのLanchaneに関するこのショートコースへようこそ。
LOMや大規模言語モデルをプロンプトにすることで、AIアプリケーションをこれまでよりずっと速く開発することができるようになりました。
しかし、アプリケーションでは、LOMを何度もプロンプトし、出力としてパースする必要があります。
・・・
```


# Qiita （in Japanese）
https://qiita.com/hiraku00/items/0723f02da96fb1ffcdf4
