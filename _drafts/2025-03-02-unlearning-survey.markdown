---
layout: post
title:  "Some Recent Progress in Machine Unlearning for LLMs"
date:   2025-03-02 14:13:17 -0800
categories: math
---

_Status: Draft_

## Motivation

Over the last few years, frontier machine learning models have been trained on larger and larger datasets comprising of text, image, and video content. While this trend has contributed to increased model capability and generalization, there are a few concerns: if it turns out that a model has learned information it should not have learned, how do we remove the knowledge from the model? This is what the field of machine unlearning is meant to address.

In this post, I focus primarily on unlearning in large language models.

## Problem Formulation

One formulation of the LLM unlearning problem might be as follows (suggested by Liu et al.[1]): “How can we efficiently and effectively eliminate the influence of specific ‘unlearning targets’ and remove associated model capabilities while preserving model performance for non-targets.”

“Unlearning targets”: Different unlearning tasks will have different unlearning targets. For example, privacy-focused unlearning might attempt to remove the influence of all data points containing personally identifiable information in the training datasets. Model safety-focused unlearning might attempt to remove all model capabilities related to, say, gain-of-function biology research (along with all data points related to these capabilities).

“Preserving model performance for non-targets”: This is a requirement for successful unlearning: the greater the “unlearning tax” on the overall model performance, the less likely any such techniques will be used in real-world scenarios. (If we only cared about unlearning the undesired data, the easiest way would be to discard the model entirely!)

“Effectively”: As we’ll soon discuss, evaluating whether an unlearned model has truly unlearned some unlearning targets or model capabilities is one of the big challenges with robust unlearning. Effective unlearning strategies should also take into account specific threat models: different techniques will be relevant if an attacker has access to query a model versus whitebox access versus access to finetune the model.

“Efficiently”: Similar to preserving model performance on non-targets: the more expensive unlearning is to perform, the less likely such techniques will be used during model training/fine-tuning.

## Strategies for Unlearning

Retraining from scratch
Input/output methods
Finetuning based approaches
Gradient ascent
NPO
Latent adversarial training
Degrading representations (?)
Adversarial training?
Influence functions (?)
Problems with finetuning strategies for unlearning
Model edits/lesions
Other mechanistic tricks
Localization-informed unlearning
Linearity-based approaches
Misc
Gradient routing
Architecture modification?
RAG-based approaches?
Compression/distillation
Metalearning?

The gold-standard technique for unlearning is retraining the original model on the retain set. While this generally isn’t a feasible solution given the costs of retraining, a model trained from scratch can be a useful benchmark to evaluate approximate unlearning techniques.

### Input-output techniques

This category of techniques provides “guardrails” for model inputs and outputs to simulate model unlearning. Note that these types of techniques generally don’t modify the underlying model. Such techniques are generally cheaper and more efficient than retraining or finetuning a model. They can often provide useful baselines to compare with other unlearning methods.

Pawelczyk et al [32] study in-context unlearning, where a language model is provided with retain examples and their true labels alongside forget examples and incorrect labels in context before being queried. Thaker et al [33] investigate how well simple prompt prefixes and postprocessing filters can be at simulating unlearning to an “honest but curious” adversary. Liu et al [34] propose a system where a classifier is trained to evaluate whether a prompt is related to previously “unlearned” data, and if so, the prompt is corrupted in the embedding space so the model behaves similar to an unlearned model on the same prompt.

### Finetuning approaches

These techniques involve finetuning a base model to unlearn specific information.

Gradient ascent ([2], [35], [38]) is one common approach to unlearning. Given a set of tokens from the “forget set”, a model is finetuned on the training objective to maximize the log-likelihood of the token sequences instead of minimizing it. In essence, this technique uses a negation of the standard training objective to degrade a model’s knowledge on a specific topic. The loss function looks like the following:

Negative preference optimization ([36]) was motivated by issues with catastrophic collapse in gradient ascent. This technique modifies the loss function from direct preference optimization ([37]) and omits the term corresponding to the positive response and only including the negative response. Instead of optimizing

(the standard DPO loss), they instead optimize
 .  NPO appears to have better theoretical guarantees of divergence speed and avoid catastrophic collapse during empirical evaluation compared with gradient ascent.

Representation Misdirection for Unlearning (RMU) ([3]) attempts to simultaneously preserve a model’s representations on the retain dataset while degrading the model’s representations on the unlearning dataset. This technique uses the following loss function and optimizes only on a few layers:
Forget loss,
Retain loss: ,
Full loss:
 is the number of tokens in ,  is the set of hidden states of the unlearned model at some layer ,  is the corresponding set of hidden states of the original, frozen model at that same layer ,  is a random unit vector.

### Model edits/lesions

### Misc

## Evaluation methods

Revisiting the original problem formulation, there are a few dimensions to evaluate an unlearning technique:

- Unlearning effectiveness: how thoroughly is the knowledge or capability removed from the model?
- Model performance on non-targets: how well does the model retain knowledge and capabilities unrelated to the unlearning task?
- Efficiency: this might include computational efficiency, such as the number of FLOPs needed to perform the unlearning, and sample efficiency, such as how much unlearning data is needed to unlearn the model.

Of these evaluation dimensions, the effectiveness of unlearning might be the most tricky to evaluate. After all, it’s difficult to distinguish between some knowledge or capability truly being removed from a model versus the model possessing that knowledge but simply being prompted poorly. We discuss this more below.

Evaluating model performance on non-targets often looks similar to standard LLM evaluations (MMLU, GLUE, GPQA, MATH, HumanEval, etc). One important distinction is the importance of testing ‘hard’ non-target examples: examples that are similar to the knowledge or capabilities to be unlearned yet that should still be retained ([1], [7]). For example, if we are attempting to unlearn information that might be used to develop a bioweapon [3], a ‘hard’ non-target example might be benign biology knowledge.

The gold standard for LLM unlearning is to retrain the model from scratch without the offending data. In theory, we would expect any successful unlearning technique should result in an unlearned model that is as indistinguishable as possible from a retrained model. (In practice of course, such an evaluation effort would likely be infeasible due to the cost of retraining a model.)

However, note that when the goal of unlearning is to remove harmful capabilities, there will likely be dual-use knowledge that is useful for both benign use cases (and therefore important for model performance on non-targets) yet can contribute to harmful capabilities [5]. Such cases might not have a clear `gold standard`.

### Benchmarks

Currently, there are a few common benchmarks to evaluate new unlearning techniques.

Maini et al. [2] propose the TOFU: Task of Fictitious Unlearning benchmark, which provides a dataset of facts about fictitious authors. This dataset was synthetically produced by prompting GPT-4 to curate a set of fictitious authors with specific predefined attributes (birthplace, gender, writing genre, etc.) and then again prompting GPT-4 to generate question-answer pairs for each author, before validating that the data was indeed fabricated. TODO: I don’t totally understand how this is evaluated.

Li et al. [3] introduce the WMDP Benchmark, which consists of multiple-choice questions in biosecurity, chemistry, and cybersecurity written by subject-matter experts. These questions serve as proxies for hazardous knowledge in these three domains; as such, successful unlearning methods should result in models being unable to answer these questions correctly while retaining performance on nonhazardous biosecurity, chemistry, and cybersecurity questions. The main metric considered is accuracy on this test set.

Eldan et al. [4] introduced a benchmark for unlearning information from the Harry Potter book series. Unlearning is evaluated by performing sentence-completion on a set of prompts related to the Harry Potter universe and evaluated by classifying how familiar the output was to the Harry Potter universe using GPT-4. Model performance on non-targets was assessed using benchmarks such as WinoGrande, HellaSwag, and piqa.

However, Thaker et al. [6] argue that these empirical benchmarks are often limited measures of progress and may often be misleading. For example, evaluations often distinguish between a “forget” set (knowledge that an unlearned model should forget) and a “retain” set (knowledge that an unlearned model should retain). When evaluation queries include dependencies on knowledge from both the forget and the retain sets (such as simply asking one question about a concept from a forget set and another question about a concept from the retain set), many unlearning algorithms perform poorly.

### Unlearning Attacks and other Evaluation Methods

#### Threat Models For Unlearning Attacks

Depending on the level of access an attacker has for an unlearned model, different attacks can be performed to attempt to retrieve “unlearned” knowledge. Here are a few common threat models considered in LLM unlearning papers:

Input/Output queries: the attacker has access to query a model and see its outputs, but not much more than that. In-context learning-based attacks and jailbreak attacks often fall into this category.
Finetuning access: the attacker can finetune a model on a given dataset, such as via a fine-tuning API. For example, relearning attacks often fall into this category.
White-box access: the attacker has access to the model weights. In such cases, the attacker likely will have finetuning and input/output access as well.

Successful unlearning methods should be robust to a wide variety of attacks to be useful in real-world cases.

#### Input/output Attacks

Rephrasing the original prompt: This might be the simplest form of “attack” where a model is queried many times with slightly different prompts. Sometimes, this attack can be an effective demonstration that an unlearning method is not robust ([24]).
A variation of this might change the type of questions an unlearned model is queried with. For example, if a technique attempts to unlearn text-generation abilities for some unlearning target, the evaluation might test the model’s ability to answer multiple choice questions about the topic.

General jailbreaks: There are a number of papers demonstrating that language models, even with significant effort put into safety finetuning and red teaming, can still be jailbroken to produce objectionable content to arbitrary prompts ([11], [12]). Such jailbreaks can also be used to retrieve “unlearned” knowledge from a model.
For example, Zou et al. [12] propose a class of adversarial attacks where an attacker takes a prompt and appends a suffix to it. The goal of the attack is to identify a suffix that results in the model producing a response that begins with an affirmative response (“Sure, here is how to build a bomb:”). The attacker then optimizes over many prompts and many models. Note that while this attack may require white-box model access in order to optimize the suffix, once this suffix has been generated, the attack often transfers well to other models that may only allow input/output access.
Another example includes querying a model in different languages. LLM safety training often does not generalize to different languages ([8], [10]). For example, Yong et al. [9] find that GPT-4 often honestly answers unsafe prompts translated into low-resource languages.

In-context relearning: Unlearning techniques should be robust to in-context relearning attacks ([8], [13], [14], [15]). If an attacker prompts a model with knowledge that was unlearned, the model should not “relearn” that knowledge in-context when providing a response. Lynch et al. [8] demonstrate that when Llama-2 is unlearned via Who’s Harry Potter [4] unlearning techniques, in-context relearning can recover much of the model performance on Harry Potter related queries.

Challenging forget-set queries: When evaluating unlearned models, it is useful to consider the worst-case data subsets for evaluation ([16]). For example, prompts that include knowledge from both the retain and forget set can pose challenges for unlearned models ([6]). In general, the closer the retain dataset is to the forget dataset, the more difficult it will be to effectively unlearn that dataset.

#### Finetuning Attacks

Finetuning can remove safety training: If fine-tuning access to a model is given, one primary risk comes from finetuning being used to undo safety-training ([17],[18],[19], [22]). For example, Lermen et al. [19] demonstrate that LoRA finetuning can efficiently undo safety training for Llama 2 and Mistral models while retaining general performance. Note that there is a line of work developing tamper-resistant safeguards to prevent these sorts of fine-tuning attacks ([23]).

Relearning via fine-tuning: The analogous risk for unlearning methods is knowledge relearning ([8], [20], [21]), where finetuning an unlearned model on a some knowledge that was unlearned leads to a model relearning significantly more unlearned knowledge.

#### Whitebox access attacks

If an adversary has white-box access to a model, they may also be able to recover knowledge from a model’s weights and activations directly ([8]). Patil et al. ([24]) show that even when model editing techniques like ROME [25] are used to “delete” knowledge from a model, traces of the deleted knowledge still persist in the model’s intermediate hidden states. They project these hidden states onto model vocabulary embeddings to extract knowledge from these intermediate states. Hong et al. [30] propose a general evaluation methodology using similar ideas and demonstrate that many unlearning methods may instead be rendering “unlearned” knowledge harder to access rather than truly removing the knowledge from models.
Zhong et al. [31] show that while existing knowledge editing technique can often edit facts accurately, the techniques often perform poorly for answering multi-hop questions. For example, if a model’s knowledge about the current president of the United States is edited, the model should provide an updated answer to the question “Who is the spouse of the current president of the United States”, which might not be true in reality.
Knowledge probes can also be used to recover knowledge from hidden states in both supervised ([26], [27], [28]) and unsupervised manners ([29]).

## Applications

## Alternatives

- Data markets
- RAG

## Misc interesting details

- Quirks that are interesting?
- Uncategorized

## Prior work

While unlearning for large language models is a relatively young field, machine unlearning more broadly has been around much longer. For example, prior work has been done for unlearning in image classification, image generation, federated learning, graph neural networks, recommendation systems, and differential privacy settings.

## References

- [1]: Liu, Sijia, Yuanshun Yao, Jinghan Jia, Stephen Casper, Nathalie Baracaldo, Peter Hase, Yuguang Yao et al. "Rethinking machine unlearning for large language models." arXiv preprint arXiv:2402.08787 (2024).
- [2]: Maini, Pratyush, Zhili Feng, Avi Schwarzschild, Zachary C. Lipton, and J. Zico Kolter. "Tofu: A task of fictitious unlearning for llms." arXiv preprint arXiv:2401.06121 (2024).
- [3]: Li, Nathaniel, Alexander Pan, Anjali Gopal, Summer Yue, Daniel Berrios, Alice Gatti, Justin D. Li et al. "The wmdp benchmark: Measuring and reducing malicious use with unlearning." arXiv preprint arXiv:2403.03218 (2024).
- [4]: Eldan, Ronen, and Mark Russinovich. "Who's Harry Potter? Approximate Unlearning in LLMs." arXiv preprint arXiv:2310.02238 (2023).
- [5]: Barez, Fazl, Tingchen Fu, Ameya Prabhu, Stephen Casper, Amartya Sanyal, Adel Bibi, Aidan O'Gara et al. "Open Problems in Machine Unlearning for AI Safety." arXiv preprint arXiv:2501.04952 (2025).
- [6]: Thaker, Pratiksha, Shengyuan Hu, Neil Kale, Yash Maurya, Zhiwei Steven Wu, and Virginia Smith. "Position: LLM Unlearning Benchmarks are Weak Measures of Progress." arXiv preprint arXiv:2410.02879 (2024).
- [7]: Liu, Chris Yuhao, Yaxuan Wang, Jeffrey Flanigan, and Yang Liu. "Large Language Model Unlearning via Embedding-Corrupted Prompts." arXiv preprint arXiv:2406.07933 (2024).
- [8]: Lynch, Aengus, Phillip Guo, Aidan Ewart, Stephen Casper, and Dylan Hadfield-Menell. "Eight methods to evaluate robust unlearning in llms." arXiv preprint arXiv:2402.16835 (2024).
- [9]: Yong, Zheng-Xin, Cristina Menghini, and Stephen H. Bach. "Low-resource languages jailbreak gpt-4." arXiv preprint arXiv:2310.02446 (2023).
- [10]: Kotha, Suhas, Jacob Mitchell Springer, and Aditi Raghunathan. "Understanding catastrophic forgetting in language models via implicit inference." arXiv preprint arXiv:2309.10105 (2023).
- [11]: Zou, Andy, Zifan Wang, Nicholas Carlini, Milad Nasr, J. Zico Kolter, and Matt Fredrikson. "Universal and transferable adversarial attacks on aligned language models." arXiv preprint arXiv:2307.15043 (2023).
- [12]: Wei, Alexander, Nika Haghtalab, and Jacob Steinhardt. "Jailbroken: How does llm safety training fail?." Advances in Neural Information Processing Systems 36 (2024).
- [13]: Shumailov, Ilia, Jamie Hayes, Eleni Triantafillou, Guillermo Ortiz-Jimenez, Nicolas Papernot, Matthew Jagielski, Itay Yona, Heidi Howard, and Eugene Bagdasaryan. "Ununlearning: Unlearning is not sufficient for content regulation in advanced generative ai." arXiv preprint arXiv:2407.00106 (2024).
- [14]: Xhonneux, Sophie, David Dobre, Jian Tang, Gauthier Gidel, and Dhanya Sridhar. "In-context learning can re-learn forbidden tasks." arXiv preprint arXiv:2402.05723 (2024).
- [15]: Wei, Zeming, Yifei Wang, Ang Li, Yichuan Mo, and Yisen Wang. "Jailbreak and guard aligned language models with only few in-context demonstrations." arXiv preprint arXiv:2310.06387 (2023).
- [16]: Fan, Chongyu, Jiancheng Liu, Alfred Hero, and Sijia Liu. "Challenging forgets: Unveiling the worst-case forget sets in machine unlearning." In European Conference on Computer Vision, pp. 278-297. Springer, Cham, 2025.
- [17]: Qi, Xiangyu, Yi Zeng, Tinghao Xie, Pin-Yu Chen, Ruoxi Jia, Prateek Mittal, and Peter Henderson. "Fine-tuning aligned language models compromises safety, even when users do not intend to!." arXiv preprint arXiv:2310.03693 (2023).
- [18]: Yang, Xianjun, Xiao Wang, Qi Zhang, Linda Petzold, William Yang Wang, Xun Zhao, and Dahua Lin. "Shadow alignment: The ease of subverting safely-aligned language models." arXiv preprint arXiv:2310.02949 (2023).
- [19]: Lermen, Simon, Charlie Rogers-Smith, and Jeffrey Ladish. "Lora fine-tuning efficiently undoes safety training in llama 2-chat 70b." arXiv preprint arXiv:2310.20624 (2023).
- [20]: Hu, Shengyuan, Yiwei Fu, Steven Wu, and Virginia Smith. "Jogging the Memory of Unlearned LLMs Through Targeted Relearning Attacks." In Neurips Safe Generative AI Workshop 2024.
- [21]: Lo, Michelle, Shay B. Cohen, and Fazl Barez. "Large language models relearn removed concepts." arXiv preprint arXiv:2401.01814 (2024).
- [22]: Zhan, Qiusi, Richard Fang, Rohan Bindu, Akul Gupta, Tatsunori Hashimoto, and Daniel Kang. "Removing rlhf protections in gpt-4 via fine-tuning." arXiv preprint arXiv:2311.05553 (2023).
- [23]: Tamirisa, Rishub, Bhrugu Bharathi, Long Phan, Andy Zhou, Alice Gatti, Tarun Suresh, Maxwell Lin et al. "Tamper-resistant safeguards for open-weight llms." arXiv preprint arXiv:2408.00761 (2024).
- [24]: Patil, Vaidehi, Peter Hase, and Mohit Bansal. "Can sensitive information be deleted from llms? objectives for defending against extraction attacks." arXiv preprint arXiv:2309.17410 (2023).
- [25]: Meng, Kevin, David Bau, Alex Andonian, and Yonatan Belinkov. "Locating and editing factual associations in GPT." Advances in Neural Information Processing Systems 35 (2022): 17359-17372.
- [26]: Liu, Kevin, Stephen Casper, Dylan Hadfield-Menell, and Jacob Andreas. "Cognitive Dissonance: Why Do Language Model Outputs Disagree with Internal Representations of Truthfulness?." arXiv preprint arXiv:2312.03729 (2023).
- [27]: Gurnee, Wes, and Max Tegmark. "Language models represent space and time." arXiv preprint arXiv:2310.02207 (2023).
- [28]: Belinkov, Yonatan. "Probing classifiers: Promises, shortcomings, and advances." Computational Linguistics 48, no. 1 (2022): 207-219.
- [29]: Burns, Collin, Haotian Ye, Dan Klein, and Jacob Steinhardt. "Discovering latent knowledge in language models without supervision." arXiv preprint arXiv:2212.03827 (2022).
- [30]: Hong, Yihuai, Lei Yu, Haiqin Yang, Shauli Ravfogel, and Mor Geva. "Intrinsic evaluation of unlearning using parametric knowledge traces." arXiv preprint arXiv:2406.11614 (2024).
- [31]: Zhong, Zexuan, Zhengxuan Wu, Christopher D. Manning, Christopher Potts, and Danqi Chen. "Mquake: Assessing knowledge editing in language models via multi-hop questions." arXiv preprint arXiv:2305.14795 (2023).
- [32]: Pawelczyk, Martin, Seth Neel, and Himabindu Lakkaraju. "In-context unlearning: Language models as few shot unlearners." arXiv preprint arXiv:2310.07579 (2023).
- [33]: Thaker, Pratiksha, Yash Maurya, Shengyuan Hu, Zhiwei Steven Wu, and Virginia Smith. "Guardrail baselines for unlearning in llms." arXiv preprint arXiv:2403.03329 (2024).
- [34]: Liu, Chris Yuhao, Yaxuan Wang, Jeffrey Flanigan, and Yang Liu. "Large Language Model Unlearning via Embedding-Corrupted Prompts." arXiv preprint arXiv:2406.07933 (2024).
- [35]: Jang, Joel, Dongkeun Yoon, Sohee Yang, Sungmin Cha, Moontae Lee, Lajanugen Logeswaran, and Minjoon Seo. "Knowledge unlearning for mitigating privacy risks in language models." arXiv preprint arXiv:2210.01504 (2022).
- [36]: Zhang, Ruiqi, Licong Lin, Yu Bai, and Song Mei. "Negative preference optimization: From catastrophic collapse to effective unlearning." arXiv preprint arXiv:2404.05868 (2024).
- [37]: Rafailov, Rafael, Archit Sharma, Eric Mitchell, Christopher D. Manning, Stefano Ermon, and Chelsea Finn. "Direct preference optimization: Your language model is secretly a reward model." Advances in Neural Information Processing Systems 36 (2023): 53728-53741.
- [38]: Yao, Yuanshun, Xiaojun Xu, and Yang Liu. "Large language model unlearning." arXiv preprint arXiv:2310.10683 (2023).
