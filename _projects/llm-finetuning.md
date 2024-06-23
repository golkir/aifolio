---
layout: page
title: LLM Finetunning
description: Finetuning Llama2 on the Dolly 15k instruction dataset
img: assets/img/llama.jpg
importance: 1
category: work
related_publications: false
---


## Overview
In this project, I fine-tuned the Llama2 language model on a 1000-sample subset of the [Dolly 15k instruction dataset](https://huggingface.co/datasets/databricks/databricks-dolly-15k). The fine-tuning process was conducted using Supervised Fine-Tuning (SFT) and implemented with QLoRA 4-bit precision to optimize the model's performance and efficiency.

## Objectives
The primary objectives of this project were:

- To enhance the Llama2 model's ability to understand and generate instructions by leveraging a specialized dataset.
- To implement and evaluate the effectiveness of QLoRA 4-bit precision in reducing computational requirements while maintaining model accuracy.
- To demonstrate the application of advanced fine-tuning techniques in improving language model capabilities for specific tasks.


<div class="row justify-content-sm-center">
    <div class="col-sm-6 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/llama.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm-6 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/ai.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    LLM is king
</div>

## Supervised Fine-Tuning (SFT):

- Employed SFT to train the Llama2 model on the prepared dataset. SFT involves using labeled data to guide the model in learning specific tasks, in this case, understanding and generating instructional content.
- Monitored training metrics such as loss and accuracy to assess the model's learning progress and make necessary adjustments.

## QLoRA 4-bit Precision:

- Implemented QLoRA (Quantized Low-Rank Approximation) to perform fine-tuning with 4-bit precision. This technique reduces the computational load and memory footprint, making the training process more efficient without significantly compromising the model's performance.
- Evaluated the impact of QLoRA on the model’s performance, ensuring that the reduced precision did not degrade the quality of the fine-tuned model.

## Results
- The fine-tuned Llama2 model exhibited improved performance in generating accurate and coherent instructional content compared to its pre-fine-tuned state.
- The use of QLoRA 4-bit precision significantly reduced the computational resources required for fine-tuning, demonstrating its effectiveness in optimizing the training process.
- The model showed enhanced generalization capabilities across various types of instructions, making it more versatile and useful for practical applications.


### Repos

[Github](https://github.com/golkir/llama2-7b-minidatabricks)

[Finetuned model on HuggingFace ](https://huggingface.co/kirillgoltsman/llama-2-7b-minidatabricks)

[Dataset](https://huggingface.co/datasets/kirillgoltsman/databricks-dolly-llama2-1k)

