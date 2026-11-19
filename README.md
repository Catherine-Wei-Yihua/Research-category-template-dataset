# Template-Based Mathematical Reasoning Dataset

A synthetic, template-based dataset for studying the **validity of mathematical reasoning steps**. It pairs mathematical problems with candidate transformations that are correct, clearly incorrect, or deceptively similar to correct steps.

**Research context:** Developed within the research framework of Dr. Hanjia Gao and Prof. Annie Qu at the University of California, Santa Barbara. Template expansion and Python/SymPy-based dataset generation by Yihua Wei.

## Motivation

LLMs often skip intermediate steps in mathematical solutions, making it difficult to verify **the logical relation between consecutive steps**, even when the final answer seems correct.

To study step-level reasoning without relying on incomplete LLM-generated traces, we designed **human-specified mathematical templates** that define problem structures and valid or invalid transformations. Sampling numerical parameters produces controlled examples with known labels. These examples support **tracking step-to-step logical relations**, detecting errors, and evaluating mathematical step-verification models.

> **Scope:** The current dataset contains problem–candidate-response pairs, mainly **single-step transformations**, rather than complete multi-step solution traces. The templates serve as building blocks for studying or constructing such sequences.

## Dataset at a Glance

The released [`Templates_2100_Dataset.json`](Templates_2100_Dataset.json) contains:

- **2,100** generated mathematical problems
- **6** mathematical categories
- **42** parameterized templates/subtypes
- **50** problem instances per template
- Three response groups for every problem: `positive`, `negative_hard`, and `negative_easy`

| Category (`category`) | Template IDs | Templates | Problems |
| --- | --- | ---: | ---: |
| Simplification | `S1`–`S5` | 5 | 250 |
| Expansion | `E1`–`E10` | 10 | 500 |
| Quadratic equations | `Q1`–`Q7` | 7 | 350 |
| Differentiation | `D1`–`D6` | 6 | 300 |
| Integration | `I1`–`I6` | 6 | 300 |
| Limits | `L1`–`L8` | 8 | 400 |
| **Total** | | **42** | **2,100** |

Across the current JSON release, there are **3,400 positive**, **19,503 hard-negative**, and **42,000 easy-negative** candidate responses. Individual records contain 1–3 positive responses, 8–10 hard negatives, and 20 easy negatives.

## Data Structure

The dataset is a JSON **array of records**. Each record represents one instantiated mathematical problem and contains the following fields:

| Field | Type | Description |
| --- | --- | --- |
| `index` | Integer | Zero-based record identifier within this dataset (`0`–`2099`). |
| `category` | String | Mathematical operation family, such as `expansion` or `differentiation`. |
| `template` | String | ID of the specific parameterized template used to generate the problem (for example, `E2` or `D1`). |
| `problem` | String | Original mathematical expression, equation, or task to be transformed or solved. Stored as mathematical text, often using LaTeX syntax. |
| `responses` | Object | Groups of candidate mathematical responses with different validity labels. |
| `responses.positive` | Array of strings | Mathematically valid transformations or results under the template's intended operation. A problem may have more than one valid representation. |
| `responses.negative_hard` | Array of strings | Invalid candidates designed to be more challenging, often resembling a plausible transformation while introducing mathematical errors. |
| `responses.negative_easy` | Array of strings | Clearly invalid or unrelated mathematical expressions, commonly drawn from different expression families. |

### Example record (abbreviated)

The example below uses entries from record `0`; response arrays are shortened for readability.

```json
{
  "index": 0,
  "category": "simplification",
  "template": "S1",
  "problem": "(-2)x^2+(4)x+(6)-(-2)x^2-(-8)x-(-1)",
  "responses": {
    "positive": [
      "7 + 12 x"
    ],
    "negative_hard": [
      "- 4 x^{2} + 12 x + 7"
    ],
    "negative_easy": [
      "\\frac{\\left|{2 x - 10}\\right|}{8}"
    ]
  }
}
```

**How to interpret it:** The `problem` is the original expression. A string in `responses.positive` is a valid transformation/result; strings in the two negative groups are candidate responses that should **not** be accepted as valid transformations of that problem. The arrays represent **alternatives**, not a chronological sequence of reasoning steps.

## How the Dataset Was Generated

1. **Design templates:** Specify mathematical operations, parameterized problem forms, and valid/invalid transformation patterns. See [`Template_Library.pdf`](Template_Library.pdf).
2. **Sample parameters:** Use Python to generate numerical coefficients and other parameters for each template.
3. **Construct positive candidates:** Instantiate the intended mathematical rules to produce valid transformations or answers.
4. **Construct negative candidates:** Generate easy negatives from a pool of unrelated expression forms and hard negatives from incorrect transformation patterns and parameter variations.
5. **Check candidates:** Use SymPy-based equivalence checks appropriate to the mathematical category to screen valid and invalid candidates.
6. **Export records:** Save the generated examples, template identifiers, and grouped responses in JSON format.

The generation notebook uses a random seed of `45` with `50` instances per template for this release.

## Repository Files

| File | Description |
| --- | --- |
| [`Templates_2100_Dataset.json`](Templates_2100_Dataset.json) | The generated dataset: 2,100 problems and their labeled candidate responses. |
| [`Template_Library.pdf`](Template_Library.pdf) | The six operation categories, 42 template definitions, and examples of valid and invalid transformations. |
| [`Template_Python_Generator.ipynb`](Template_Python_Generator.ipynb) | Python/SymPy generation and validation code. |

## Loading the Dataset

```python
import json

with open("Templates_2100_Dataset.json", "r", encoding="utf-8") as f:
    dataset = json.load(f)

example = dataset[0]
print(example["category"], example["template"])
print("Problem:", example["problem"])
print("Positive candidates:", example["responses"]["positive"])
print("Hard negatives:", len(example["responses"]["negative_hard"]))
print("Easy negatives:", len(example["responses"]["negative_easy"]))
```

To regenerate the dataset, open `Template_Python_Generator.ipynb`, install its `sympy` dependency if necessary, and run the generation cells. **Note:** The notebook's final export cell currently includes a machine-specific absolute output path; update it to a path appropriate for your environment before running.

## Intended Research Uses

- Evaluating whether a proposed mathematical step is logically valid relative to an input problem or expression.
- Comparing detection of obvious errors against more plausible incorrect transformations.
- Studying step-level mathematical reasoning, transformation consistency, and error localization.
- Providing controlled examples for training or evaluating mathematical step-verification methods.

## Limitations

- **Template coverage:** The dataset reflects the selected 42 template families rather than the full diversity of mathematical problems or natural LLM responses.
- **Automatic validation:** SymPy-based checks are useful but do not replace exhaustive manual review of every mathematical expression or edge case in complex-step scenarios.
- **Representation overlap:** Symbolic simplification may sometimes produce equivalent or duplicate positive representations.
- **Negative sampling:** Constructed negatives are controlled examples and may not reproduce the distribution of mistakes made by real LLMs.

## Credits

**Yihua Wei** — Undergraduate Research Assistant, University of California, Santa Barbara  
**Faculty Advisor:** Prof. Annie Qu  
**Research Mentor:** Dr. Hanjia Gao
