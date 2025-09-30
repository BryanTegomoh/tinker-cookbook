# Prompt Distillation

Prompt distillation is a training technique in which a model is optimized to behave as though it had been provided with a long and complex prompt, without requiring access to that prompt during inference.

At a high level, this procedure involves two main steps:
- **Creation of distillation data**: A teacher prompt, which is typically lengthy and highly detailed, provides explicit, step-by-step instructions. A teacher model uses this prompt to generate responses for a set of queries.
- **Training the student model**: A student model is then trained (or fine-tuned) on the distilled dataset, thereby learning to reproduce the essential behaviors and reasoning encoded in the teacher’s instructions.

---

## Overview

Let $f_T$ and $f_S$ denote the teacher and student models, respectively. Given an instruction prompt $P$ and a query $q_i$, the teacher model generates a response $r_i$:

$$
r_i = f_T([P, q_i])
$$

Here, the prompt $P$ and the query $q_i$ are concatenated to form the input to the teacher model $f_T$. For a dataset of queries $Q = \{q_i \mid 1 \leq i \leq D\}$, we obtain a corresponding set of teacher responses $R = \{r_i \mid 1 \leq i \leq D\}$.

The distillation training dataset is defined as the set of query–response pairs (excluding the original prompt):

$$
T = \{(q_i, r_i) \mid 1 \leq i \leq D\}.
$$

The student model $f_S$ is then trained to minimize the cross-entropy loss:

$$
\ell(f_S(q_i), r_i) = \ell(f_S(q_i), f_T([P, q_i])).
$$

---

## Example

The Tinker Cookbook provides a prompt distillation recipe tailored for a language classification task. The objective is straightforward: given a text query, the model should predict a two-character code corresponding to the language of the input. The set of possible labels is:
```
ar (Arabic), de (German), el (Greek), en (English), es (Spanish), fr (French), hi (Hindi), ru (Russian), tr (Turkish), ur (Urdu), vi (Vietnamese), zh (Chinese - Simplified), ot (Other/Unknown).
```

The recipe in `create_data.py` also includes handling strategies for inputs containing code, numerical content, or multiple languages.

In the example below, the same model (`Qwen/Qwen3-30B-A3B`) is used as both teacher and student, though in general they need not be identical.

---

### Step 1: Generate Training Data

Create prompt distillation data using the teacher model using `create_data.py`:

```bash
python -m tinker_cookbook.recipes.prompt_distillation.create_data \
  output_file=/tmp/tinker-datasets/prompt_distillation_lang.jsonl
```

This command will:
- Use the configured teacher model to generate language classification examples
- Save the distilled dataset to the specified output file
- Create diverse training examples suitable for student model fine-tuning

### Step 2: Train the Student Model

Fine-tune a student model on the distillation data using `train.py`:

```bash
python -m tinker_cookbook.recipes.prompt_distillation.train
```

The training script will:
- Load the generated distillation dataset
- Apply optimized training configurations
- Fine-tune the student model for language classification

### Step 3: Test Your Model

Once training is complete, you can test your distilled model by sampling from the trained model to verify its performance on language classification tasks.

## Advanced Configuration

The prompt distillation recipe can be customized for different scenarios:

- **Teacher model selection**: Choose different base models based on your requirements
- **Sampling strategies**: Adjust temperature and other generation parameters
- **Data volume**: Scale the number of generated examples based on your needs
- **Training hyperparameters**: Fine-tune learning rates and other training settings
