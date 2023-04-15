The entire training process includes three parts: vocabulary expansion, pre-training, and instruction fine-tuning. The training code refers to the [run_clm.py](https://github.com/huggingface/transformers/blob/main/examples/pytorch/language-modeling/run_clm.py) in 🤗transformers and the relevant parts of dataset processing in the [Stanford Alpaca](https://github.com/tatsu-lab/stanford_alpaca) project.

### Preparation: Vocabulary Expansion

Due to the limited support for Chinese (and other non-English languages) in the original LLaMA,

- We further expanded the Chinese vocabulary based on training with the general Chinese corpus using [sentencepiece](https://github.com/google/sentencepiece) to create a 20K Chinese vocabulary, which was then merged with the original LLaMA model's 32K vocabulary. 
- After removing duplicate tokens, the final Chinese LLaMA vocabulary size is 49,953.
- It should be noted that during the fine-tuning stage, Alpaca has one more pad token than LLaMA, so the Chinese Alpaca vocabulary size is 49,954.

For more information on the motivation behind expanding the Chinese vocabulary, please refer to the [FAQ](#FAQ).

### Pre-training

In the pre-training phase, the general Chinese corpora (consistent with the corpora used in [Chinese BERT-wwm](https://github.com/ymcui/Chinese-BERT-wwm), [MacBERT](https://github.com/ymcui/MacBERT), [LERT](https://github.com/ymcui/LERT), [PERT](https://github.com/ymcui/PERT)) were used for further pre-training based on the original LLaMA weights. This process is divided into two stages:

1. Stage One: Fix the parameters of the transformer part of the model and only train the embedding, adapting the newly added Chinese word vectors without disturbing the original model as much as possible.
2. Stage Two: Use LoRA technology to add LoRA weights (adapter) to the model, and train the embedding while updating LoRA parameters.

### Instruction Fine-tuning

1. The task format of the instruction fine-tuning phase is basically the same as that of [Stanford Alpaca](https://github.com/tatsu-lab/stanford_alpaca). The training scheme also used LoRA for efficient fine-tuning and further increased the number of trainable parameters.
2. We follow the original prompt by [Stanford Alpaca](https://github.com/tatsu-lab/stanford_alpaca) that without "input". For the data that contains "input" values, we simply concatenate them in the form of`f"{instruction}+\n+{input}"`.

### Training Data

During the instruction fine-tuning phase, about 2M data were used for 7B model, and 3M data for 13B model. Details:
| Dataset                   | Size |                             Source                             | Description                                                    |
| ---------------------- | :--: | :----------------------------------------------------------: | ------------------------------------------------------- |
| Chinese-English Translation            | 500K | [link](https://github.com/brightmart/nlp_chinese_corpus#5翻译语料translation2019zh) | sampled and cleaned from original dataset                 |
| pCLUE              | 300K |        [link](https://github.com/CLUEbenchmark/pCLUE)        | sampled and cleaned from original dataset                  |
| Stanford Alpaca data | 50K  |     [link](https://github.com/tatsu-lab/stanford_alpaca)     |  Original training data of Stanford Alpaca                               |
| Stanford Alpaca data (Chinese) | 50K  |                 Provided in our proj => [link](./data)                 | We translate original data into Chinese using ChatGPT  |
| Self-instruction data   | 1-2M |                         N/A                        | We use ChatGPT API to get these data, see below               |

This project provides a script `script/crawl_prompt.py` for dynamically generating prompts of different domains and instruction types.

```bash
python script/crawl_prompt.py output-file
```

- The idea is similar to the approach used in [Stanford Alpaca](https://github.com/tatsu-lab/stanford_alpaca#data-generation-process). It generates 20 sets of data at a time (you can modify the templates), reducing the cost of crawling.
- The generated file contains data crawled through `gpt-3.5-turbo` (you must have an OpenAI API key to use it).
- Although the instruction template requires the output to be in JSON format, the system does not always return valid JSON, so you need to clean it up according to the returned data.
- Since crawling takes a long time, it is recommended to run this script in the background. When running multiple threads, pay attention to the [call limit of the OpenAI API](https://platform.openai.com/docs/guides/rate-limits/overview).

### Experimental Setups

| Settings          | Pre-training Stage One | Pre-training Stage Two | Instruction Fine-tuning |
| :----------------------- | :--------------------: | :--------------------: | :---------------------: |
| Batch Size               |          1024          |          1024          |           512           |
| Initial Learning Rate    |          2e-4          |          1e-4          |          1e-4           |
| Training Steps           |           3K           |           6K           |         6K-10K          |
| Max Length               |          512           |          512           |           512           |
| Trainable Parameters (%) |         2.97%          |         6.06%          |          6.22%          |
| Training Device          |    8 × A100     |    16 × A100     |     16 × A100     |
| Distributed Training     | DeepSpeed Zero-2 | DeepSpeed Zero-2 | DeepSpeed Zero-2 |
