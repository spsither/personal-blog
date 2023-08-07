---
title: 'Fine-tuining Wav2Vec2 for tibetan ASR'
excerpt: 'Experiences from fine-tuining facebook's Wav2Vec2 model for Tibetan ASR.'
coverImage: '/assets/blog/tibetan-asr-wav2vec2/cover.png'
date: '2023-08-07T16:40:0Z'
author:
  name: spsither
  picture: '/assets/blog/authors/sp.jpeg'
ogImage:
  url: '/assets/blog/tibetan-asr-wav2vec2/cover.png'
---

As part of the OpenPecha project, I spent the last few weeks fine-tuining facebook's open source model Wav2Vec2 for tibetan Automatic Speech Recognition (ASR). In this blog I share some of the problems we faced and how to avoid the same mistakes. 

## Fine-Tune Wav2Vec2

There are plenty of resources on how to Fine-Tune Wav2Vec2, such as [this](https://huggingface.co/blog/fine-tune-wav2vec2-english) blog from HuggingFace. Although the blog is conprehensive we faced some issues when addapting the notebook to train with our Tibetan training data.

###  Using your own data
The data we had was in the form of a tsv file with two cloumns, `path` and `sentence`. Where `path` is the filename of the audio segment and `sentence` is the transcription of what was said in the audio file. All the audio segments are short clips of about 1 to 12 seconds. 


The HuggingFace blog uses `load_dataset` method from `datasets` library from HugginFace but if you want to use local data from an file, you can use `pandas` to read the tsv file and then use `Dataset.from_pandas` to make a dataset. Like so 
```python
from datasets import Dataset
train_ds = Dataset.from_pandas(train_df)
```

### Jupyter Notebook ram limit 
When using Jupyter notebook on a conputer with decent specs the notebook keeps restarting the kernel especially when we ran the `prepare_dataset` in a map.

After I came accross [this article](https://towardsdatascience.com/leveraging-the-power-of-jupyter-notebooks-26b4b8d7c622), I realized that we have to configure the Jupyter notebook RAM limit. Suppose you have 32GB ram than you can use the follownig command to create config file and then set a new ram limit. The iopub_data_rate_limit is to set the limit of output your notebook can show. 
```bash
!jupyter notebook --generate-config
!jupyter notebook --NotebookApp.max_buffer_size=32000000000
!jupyter notebook --NotebookApp.iopub_data_rate_limit=10000000000
```

### Prepare dataset in serial 
Another expensive mistake we made was to change the `prepare_dataset` function to operate not in batches but sequentially. Out notebook kernel kept restarting before the operation could finish on the training dataset so we thought it was a good idea to change the function into sequestial. 


- CTC infinite eval loss
- pushing the model online
