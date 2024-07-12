
# Dive Deeper into Yi-9B

*Author: 01.ai*
*Date: 2024-03-18*

Since introducing our Yi-6B and Yi-34B models, we've received valuable feedback from the open-source community. Kudos! A common piece of feedback highlights improvements in coding and mathematics within the Yi series models.

We’re thrilled to introduce the Yi-9B and its 200K variant. You might wonder, "Why Yi-9B?" Let's dive into the reasons and insights behind its development.

**TL;DR:** 

Yi-9B builds upon the Yi-6B model, employing a combination of model depth expansion and multi-stage incremental training. This approach results in significant improvements in coding and mathematics, alongside maintaining excellent bilingual abilities.

## What is Yi-9B

Yi-9B is an extension of Yi-6B, with boosted performance in coding and mathematics, while maintaining strong bilingual abilities. Yi-9B features:

- **Parameter size:** Yi-9B has approximately 8.8B parameters.

- **Context length:** Supports a context length of 4096 tokens.

- **Training data:** Yi-9B was further trained on 0.8 trillion tokens, adding to the 3.1 trillion tokens used for Yi-6B, with data cutoff date up to June 2023.

## Why 9B?

Yi-6B and Yi-34B were pre-trained with 3.1T tokens of English and Chinese corpora. Compared to similarly sized models, Yi-6B and Yi-34B showed better performance in evaluations, but had room for improvement in coding and mathematical abilities.

To enhance these abilities, inspired by scaling laws that link model performance with data volume and model size, our team explored the following two approaches.

### Approach 1: Increase data volume

Our team first experimented with incremental training for Yi-6B. However, the improvements in coding and mathematical skills fell short of our expectations, leading us to adopt a second approach.

### Approach 2: Increase model size, then increase data volume

#### Increasing model size

Before further training Yi-6B, we expanded the model to 9B parameters and then conducted multi-stage incremental training with additional data.

#### Increasing data volume (multi-stage)

- Stage 1 utilized 0.4 trillion tokens (including general corpora and code), with the same data ratio as Yi-6B.

- Stage 2 utilized another 0.4 trillion tokens (including general corpora, code, and math), with an increased proportion of code and mathematics data.

#### Optimization and Tuning Methods

- Instead of commonly used learning rate decay, our team set a fixed value based on **An Empirical Model of Large-Batch Training** [2] and **Don't Decay the Learning Rate, Increase the Batch Size** [3].  We set the learning rate to 3e-5 and gradually increase the batch size, starting from 4M tokens. The batch size is increased whenever the model loss stops decreasing, allowing the model to learn more thoroughly and better convergence.

## Expansion Method

### Why Expand the Model?

Before performing model expansion, our team analyzed trends in the model structure and training process.

#### Model Structure

In testing and analyzing Yi-6B/34B and Llama2-70B, we calculated the cosine similarity of token embeddings through different layers to observe the models' ability to extract token context features. 

The following figure shows the input/output cosine values of each layer of each token in the text "Write a quiz about bits". We observed that:

- Yi-34B and Llama2-70B have more layer cosine values close to 1, that is, some layers are of little help for feature extraction of token context features and are not fully trained.

- Yi-6B, however, showed larger differences in input and output cosine values across all layers, indicating that each layer has been fully trained.

<p align="center">
    <img src="https://cdn-uploads.huggingface.co/production/uploads/6413d7be996b2e426f230fb7/b21aLi1VSFMXIDFwXq_bK.png">
    <em>Figure 1: Yi-34B layer cosine value is close to 1.</em>
</p>
<p align="center">
    <img src="https://cdn-uploads.huggingface.co/production/uploads/6413d7be996b2e426f230fb7/6RUpt3kNgSTUCrJdNeDRR.png">
    <em>Figure 2: Llama2-70B layer cosine value is close to 1.</em>
</p>
<p align="center">
    <img src="https://cdn-uploads.huggingface.co/production/uploads/6413d7be996b2e426f230fb7/Z9M5eyKFwEUAuIfLG8tLE.png">
    <em>Figure 3: The cosine similarity value of the Yi-6B layer is not close to 1.</em>
</p>

#### Training Process Trends

We introduced a metric to quantify the cumulative overall model input/output cosine distance, observing changes in Yi-6B/34B's performance during training. 

![](https://github.com/01-ai/Yi/blob/main/assets/img/9b.png?raw=true)

Equation: Quantify cumulative overall model input/output cosine values

In this equation, 
- N represents the number of tokens.
- L represents the number of layers.
- Layer (input/output) cosine = the average of the cosine values of all tokens except the first token.
- Model (input/output) cosine = the average cosine value of the layer excluding the first and last 2 layers.

As can be seen from the figure below, Yi-6B showed a gradually flattening trend in the decline of layer cosine values for English, Chinese, and coding abilities, indicating that the model has been trained quite sufficiently, and it is predicted that if additional tokens are provided, the model would still yield limited benefits. (Errata: The unit of the x-axis for tokens in the figure below is 'B', not 'TB') 

<p align="center">
    <img src="https://cdn-uploads.huggingface.co/production/uploads/6413d7be996b2e426f230fb7/SnhzWQHnGdF2EarayLZVb.png">
    <em>Figure 4: Yi-6B training process - layer cosine downward trend gradually flattens - English ability.</em>
</p>

<p align="center">
    <img src="https://cdn-uploads.huggingface.co/production/uploads/6413d7be996b2e426f230fb7/6p361zC2T9Qtsh_TV55UI.png">
    <em>Figure 5: Yi-6B training process - layer cosine downward trend gradually flattens - Chinese ability.</em>
</p>

<p align="center">
    <img src="https://cdn-uploads.huggingface.co/production/uploads/6413d7be996b2e426f230fb7/L8Use4DHzlJF8gUjSRul1.png">
    <em>Figure 6: Yi-6B training process - layer cosine downward trend gradually flattens - code capability.</em>
</p>

In contrast, Yi-34B still showed a declining trend, indicating after training with more tokens, the model converges better.

<p align="center">
    <img src="https://cdn-uploads.huggingface.co/production/uploads/6413d7be996b2e426f230fb7/ahyxtZvJVyKF0_H6kYEeS.png">
    <em>Figure 7: Yi-34B training process - the downward trend of layer cosine is still steep - English ability.</em>
</p>

<p align="center">
    <img src="https://cdn-uploads.huggingface.co/production/uploads/6413d7be996b2e426f230fb7/9MBurxpAATNpPLot6ymmJ.png">
    <em>Figure 8: Yi-34B training process - layer cosine downward trend is still steep - Chinese ability.</em>
</p>

<p align="center">
    <img src="https://cdn-uploads.huggingface.co/production/uploads/6413d7be996b2e426f230fb7/GwpUfFbfgwR41spE-L2jO.png">
    <em>Figure 9: Yi-34B training process - the downward trend of layer cosine is still steep - code capability.</em>
</p>

Therefore, we predicted that continuing training Yi-6B might not meet expectations, opting instead to expand the model size to 9B before continuing training.

## Why Choose Depth Expansion?

When increasing the model size, our team experimented with expanding depth and width. We found:

- After width expansion, there was a significant decline in the model's performance.

- After in-depth amplification of the original model (by selecting appropriate layers), the closer the input/output cosine of the new layers is to 1.0, that is, the performance of the amplified model can maintain the performance of the original model. The performance loss is slight.

### How to Perform Depth Expansion?

Compared to **Solar's method** [4], which duplicates the middle 16 layers (8-24) of Mistral 7B, our approach involved duplicating a later set of 16 layers (12-28) from Yi-6B, creating a 48-layer Yi-9B. These layers had cosine values closer to 1.0, indicating smaller performance loss. 

### Effects of Depth Expansion

Yi-9B Init, which includes 16 newly added layers (28-44), demonstrates cosine values close to 1, indicating minimal performance loss. Thus, Yi-9B was trained based on Yi-9B Init.

<p align="center">
    <img src="https://cdn-uploads.huggingface.co/production/uploads/6413d7be996b2e426f230fb7/BhLH7TWRj0FkTyXnIShl-.png">
    <em>Figure 10: Comparison of layer cosine values.</em>
</p>

We were able to conclude that Yi-9B Init (Yi method) outperforms Yi-9B Init (Solar method) in model performance. Yi-9B, based on Yi-9B Init (Yi method), inherited and surpassed Yi-6B's performance.

<p align="center">
    <img src="https://cdn-uploads.huggingface.co/production/uploads/6413d7be996b2e426f230fb7/xJMb_eD--nmguDnr6vUTI.png">
    <em>Figure 11: Performance comparison.</em>
</p>

## Results

- In terms of comprehensive ability (Mean-All), Yi-9B performs the best among open-source models of similar size, outperforming DeepSeek-Coder, DeepSeek-Math, Mistral-7B, SOLAR-10.7B, and Gemma-7B.

<p align="center">
    <img src="https://cdn-uploads.huggingface.co/production/uploads/6413d7be996b2e426f230fb7/umn-xZI5jYCK8j2yQoXVv.png">
    <em>Figure 12: Performance comparison - overall capabilities.</em>
</p>

- For coding ability (Mean-Code), Yi-9B's performance is second only to DeepSeek-Coder-7B, outperforming Yi-34B, SOLAR-10.7B, Mistral-7B, and Gemma-7B.

<p align="center">
    <img src="https://cdn-uploads.huggingface.co/production/uploads/6413d7be996b2e426f230fb7/M2bGEj5ITAZ4nC1-N8A8U.png">
    <em>Figure 13: Performance comparison - coding.</em>
</p>

- For mathematical ability (Mean-Math), the performance of Yi-9B is second only to DeepSeek-Math-7B, outperforming SOLAR-10.7B, Mistral-7B, and Gemma-7B.

<p align="center">
    <img src="https://cdn-uploads.huggingface.co/production/uploads/6413d7be996b2e426f230fb7/tJWZbyV0OGwoV-vepZNqX.png">
    <em>Figure 14: Performance comparison - mathematics.</em>
</p>

- For common sense and reasoning ability (Mean-Text), Yi-9B's performance is comparable to Mistral-7B, SOLAR-10.7B, and Gemma-7B.
  
<p align="center">
    <img src="https://cdn-uploads.huggingface.co/production/uploads/6413d7be996b2e426f230fb7/PbsCMha3-6PbtGqO-373t.png">
    <em>Figure 15: Performance comparison - common sense and reasoning.</em>
</p>

- Meanwhile, Yi-9B maintains excellent bilingual language ability. Here, the Mean (All) score is the average of all evaluations (except CMMLU). For Mean (En-Text), we averaged four English evaluations (ARC-c, HellaSwag, Winogrande, and MMLU).

<p align="center">
    <img src="https://cdn-uploads.huggingface.co/production/uploads/6413d7be996b2e426f230fb7/AV5xbYRYCSykGxDHAoGGf.png">
    <em>Figure 16: Performance comparison - detailed data</em>
</p>

- Note that we employed the greedy decoding generation method to perform evaluation in-house. Some results may differ slightly from those reported in the original work. 

- **Parameter Count**: The model name is determined by the total parameter count (Embedding Parameters * 2 + Non-Embedding Parameters), where Non-Embedding Parameters are the effective parameters determining model performance. We take a holistic approach in naming our models, considering both Embedding and Non-Embedding Parameters. Yi-9B using the full parameter amount and rounding it up (0.26 *2 + 8.31 = 8.83 B). 

<p align="center">
    <img src="https://cdn-uploads.huggingface.co/production/uploads/6413d7be996b2e426f230fb7/Z6j7LSE68lLjUYrwcDFd-.jpeg" style="width: 50%;">
</p>
<p align="center"><em>Figure 17: Comparison of parameters</em></p>

We warmly invite the open-source community to provide feedback, share insights, or engage in discussions about the Yi model series. Your contributions are invaluable to our continuous improvement and innovation. 

Join us on [Discord](https://discord.gg/hYUwWddeAu) and check out our [Tech Report](https://arxiv.org/abs/2403.04652).

## 📌 Related Resources

[1] Yi-9B README, https://huggingface.co/01-ai/Yi-9B

[2] An Empirical Model of Large-Batch Training, https://arxiv.org/abs/1812.06162

[3] Don't Decay the Learning Rate, Increase the Batch Size, https://arxiv.org/abs/1711.00489

[4] SOLAR 10.7B: Scaling Large Language Models with Simple yet Effective Depth Up-Scaling, https://arxiv.org/abs/2312.15166