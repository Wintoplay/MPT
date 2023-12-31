---
license: apache-2.0
---
# LoRA-MPT
A repo to make it so that you can easily fine tune MPT-7B using LoRA. This uses the alpaca dataset but can easily be adapted to use another. 

## Setup

Ensure you are using either Linux or WSL for Windows. Mac might work as well but I don't have one to test it, but Bitsandbytes won't definitely won't run for Windows.


To use as a library in another project/directory simply clone the repo, navigate to the folder and run:
```
pip install -e ./
```

or if you want to build a project within this directly just do a git clone and run and (if needed) modify the files in the src folder. 

## Multi-Gpu Note

For all of the following commands, substitute deepspeed for python to use with multi-gpu systems. Make sure you have deepspeed installed for this (a simple "pip install deepspeed" will normally accomplish this.)

## Fine Tuning

To fine tune the MPT-7B model on the Alpaca dataset from Stanford using LoRA use the following command:
```
python src/finetune.py --base_model 'mosaicml/mpt-7b-instruct' --data_path 'yahma/alpaca-cleaned' --output_dir './lora-mpt' --lora_target_modules '[Wqkv]'
```

The hyperparameters can be tweaked using the following flags as well:

```
python src/finetune.py \
    --base_model 'mosaicml/mpt-7b-instruct' \
    --data_path 'yahma/alpaca-cleaned' \
    --output_dir './lora-mpt' \
    --batch_size 128 \
    --micro_batch_size 4 \
    --num_epochs 3 \
    --learning_rate 1e-4 \
    --cutoff_len 512 \
    --val_set_size 2000 \
    --lora_r 8 \
    --lora_alpha 16 \
    --lora_dropout 0.05 \
    --lora_target_modules '[Wqkv]' \
    --train_on_inputs \
    --group_by_length
    --use_gradient_checkpointing True \ 
    --load_in_8bit True
```

To speed up training at the expense of GPU memory run with --use_gradient_checkpointing False.

load_in_8bit defaults to True so to disable it just run the command with load_in_8bit set to False. 

## Inference

A Gradio Interface was also created which can be used to run the inference of the model, once fine-tuned, using:

```
python src/generate.py --load_8bit --base_model 'mosaicml/mpt-7b-instruct' --lora_weights 'lora-mpt'
```

## Evaluation

For the eval library to work there are a few extra steps.

You need to install this repo from source so cd in this repo's directory on your computer and run:
```
pip install -e ./
```

Then, git clone my fork of EleutherAI's Evaluation Harness at https://github.com/mikeybellissimo/lm-evaluation-harness/tree/master#language-model-evaluation-harness and follow the instructions to download the library (Pretty much just clone it, cd into it and "pip install -e ."). 

Once that's done, switch back into MPT-Lora Directory and run:

```
python src/eval.py --model mpt-causal --model_args pretrained=mosaicml/mpt-7b-instruct,trust_remote_code=True,load_in_8bit=True,peft=lora-mpt --tasks hellaswag 
```

To evaluate on the hellaswag task, for example, using the LoRA weights defined in lora-mpt. You can change the task to whatever one you'd like. 

## MosaicML Platform

If you're interested in [training](https://www.mosaicml.com/training) and [deploying](https://www.mosaicml.com/inference) your own MPT or LLMs on the MosaicML Platform, [sign up here](https://forms.mosaicml.com/demo?utm_source=huggingface&utm_medium=referral&utm_campaign=mpt-7b).

Note: I left this in as a thank you to MosaicML for open-sourcing their model.

## Attributions 

I would like to thank the wonderful people down at MosaicML for releasing this model to the public. I believe that the future impacts of AI will be much better if its development is democratized. 

```
@online{MosaicML2023Introducing,
    author    = {MosaicML NLP Team},
    title     = {Introducing MPT-7B: A New Standard for Open-Source, 
    ly Usable LLMs},
    year      = {2023},
    url       = {www.mosaicml.com/blog/mpt-7b},
    note      = {Accessed: 2023-03-28}, % change this date
    urldate   = {2023-03-28} % change this date
}
```

This repo also adapted/built on top of code from Lee Han Chung https://github.com/leehanchung/mpt-lora which was adapted from tloen's repo for training LLaMA on Alpaca using LoRA https://github.com/tloen/alpaca-lora so thank you to them as well. 
"# MPT" 
