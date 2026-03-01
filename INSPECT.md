# 🔍 CUAD Benchmark — Inspection Notes

> **Contract Understanding Atticus Dataset (CUAD)**  
> A legal contract review benchmark annotated by legal experts.  
> Paper: [CUAD: An Expert-Annotated NLP Dataset for Legal Contract Review](https://arxiv.org/abs/2103.06268)

---

## 📁 Repository Files

| File | Format | Purpose |
|------|--------|---------|
| `CUADv1.json` | JSON | Full dataset (train + test) |
| `test.json` | JSON | Test split |
| `train_separate_questions.json` | JSON | Training split |
| `category_descriptions.csv` | CSV | Descriptions of 41 legal clause categories |
| `evaluate.py` | Python | Evaluation script (AUPR, Precision@Recall) |
| `train.py` | Python | Fine-tuning script (HuggingFace Transformers) |
| `utils.py` | Python | Prediction + scoring utilities |
| `contract_review.png` | Image | README illustration only — **not benchmark data** |

---

## 🧩 What is the Task?

> **"Finding needles in a haystack"**

Given a full legal contract and a specific legal question about one of **41 clause categories**, the model must:
- **Extract the exact text span** from the contract that answers the question, OR
- **Return empty** if the clause does not exist in the contract

This is **extractive question answering** (SQuAD v2 format).

---

## 📥 Input Format

### ❌ What is NOT in the input
- No images
- No CSV files fed to the model
- No structured metadata
- No tables or specifications

> `category_descriptions.csv` is only used by the **evaluation script**, not by the model.  
> `contract_review.png` is only a README diagram.

### ✅ What IS in the input — Two plain text strings

```
1. context  → Full plain-text of a legal contract (~5,000–15,000 characters)
2. question → Natural language question about one specific clause category
```

---

## 📄 Data File Structure (JSON — SQuAD v2 format)

```json
{
  "version": "aok_v1.0",
  "data": [
    {
      "title": "LohaCompanyltd_20191209_F-1_EX-10.16_..._Supply Agreement",
      "paragraphs": [
        {
          "context": "<full plain-text of the contract>",
          "qas": [
            {
              "id": "...Supply Agreement__Governing Law",
              "question": "Highlight the parts (if any) of this contract related to \"Governing Law\" ...",
              "is_impossible": false,
              "answers": [
                {
                  "text": "It will be governed by the law of the People's Republic of China ...",
                  "answer_start": 10691
                }
              ]
            },
            {
              "id": "...Supply Agreement__Non-Compete",
              "question": "Highlight the parts (if any) of this contract related to \"Non-Compete\" ...",
              "is_impossible": true,
              "answers": []
            }
          ]
        }
      ]
    }
  ]
}
```

---

## 🔬 One Concrete Example

### Contract
> **Supply Agreement** — Shenzhen LOHAS Supply Chain Management Co., Ltd. (buyer) vs. unnamed seller. Filed with the SEC.

---

### Input ① — Contract Text (excerpt)

```
Exhibit 10.16 SUPPLY CONTRACT Contract No: Date:
The buyer/End-User: Shenzhen LOHAS Supply Chain Management Co., Ltd.
...
21. Law application
It will be governed by the law of the People's Republic of China,
otherwise it is governed by United Nations Convention on Contract
for the International Sale of Goods.
...
23. The Contract is valid for 5 years, beginning from and ended on .
```

---

### Input ② — Legal Question

```
Highlight the parts (if any) of this contract related to "Governing Law"
that should be reviewed by a lawyer.
Details: Which state/country's law governs the interpretation of the contract?
```

---

### ✅ Expected Output — Extracted Span

```
It will be governed by the law of the People's Republic of China,
otherwise it is governed by United Nations Convention on Contract
for the International Sale of Goods.
```
> Character offset: `10691` in the full contract text

---

### ❌ When No Clause Exists (e.g., "Non-Compete")

```json
{
  "answers": [],
  "is_impossible": true
}
```
> Model should return **empty string**.

---

## 📊 The 41 Legal Clause Categories (`category_descriptions.csv`)

| # | Category | Answer Format |
|---|----------|---------------|
| 1 | Document Name | Contract Name |
| 2 | Parties | Entity/individual names |
| 3 | Agreement Date | Date (mm/dd/yyyy) |
| 4 | Effective Date | Date (mm/dd/yyyy) |
| 5 | Expiration Date | Date / Perpetual |
| 6 | Renewal Term | Number of years/months |
| 7 | Notice Period to Terminate Renewal | Number of days/months |
| 8 | Governing Law | US State / Country |
| 9 | Most Favored Nation | Yes/No |
| 10 | Non-Compete | Yes/No |
| 11 | Exclusivity | Yes/No |
| 12 | No-Solicit of Customers | Yes/No |
| 13 | Competitive Restriction Exception | Yes/No |
| 14 | No-Solicit of Employees | Yes/No |
| 15 | Non-Disparagement | Yes/No |
| 16 | Termination for Convenience | Yes/No |
| 17 | Rofr/Rofo/Rofn | Yes/No |
| 18 | Change of Control | Yes/No |
| 19 | Anti-Assignment | Yes/No |
| 20 | Revenue/Profit Sharing | Yes/No |
| 21 | Price Restrictions | Yes/No |
| 22 | Minimum Commitment | Yes/No |
| 23 | Volume Restriction | Yes/No |
| 24 | IP Ownership Assignment | Yes/No |
| 25 | Joint IP Ownership | Yes/No |
| 26 | License Grant | Yes/No |
| 27 | Non-Transferable License | Yes/No |
| 28 | Affiliate License-Licensor | Yes/No |
| 29 | Affiliate License-Licensee | Yes/No |
| 30 | Unlimited/All-You-Can-Eat License | Yes/No |
| 31 | Irrevocable or Perpetual License | Yes/No |
| 32 | Source Code Escrow | Yes/No |
| 33 | Post-Termination Services | Yes/No |
| 34 | Audit Rights | Yes/No |
| 35 | Uncapped Liability | Yes/No |
| 36 | Cap on Liability | Yes/No |
| 37 | Liquidated Damages | Yes/No |
| 38 | Warranty Duration | Number of months/years |
| 39 | Insurance | Yes/No |
| 40 | Covenant Not to Sue | Yes/No |
| 41 | Third Party Beneficiary | Yes/No |

---

## 🔄 Full Inference Flow

```
┌──────────────────────────────────────────────────────────────┐
│  INPUT                                                       │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  context:  [Full plain-text of legal contract]         │  │
│  │  question: "Highlight parts related to 'Governing Law' │  │
│  │             ...Which country's law governs?"           │  │
│  └────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────┘
                            ↓
         Transformer model (RoBERTa-base / RoBERTa-large
                           / DeBERTa-xlarge)
                            ↓
┌──────────────────────────────────────────────────────────────┐
│  OUTPUT                                                      │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  Extracted text span from the contract                 │  │
│  │  (or empty string if the clause does not exist)        │  │
│  └────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────┘
```

> Each contract gets **41 questions** asked against it → 41 input-output pairs per contract.  
> Dataset total: **510 contracts × 41 questions = ~13,101 QA pairs**

---

## 📏 Evaluation Metrics (`evaluate.py`)

| Metric | Description |
|--------|-------------|
| **AUPR** | Area Under Precision-Recall curve (primary metric) |
| **Precision @ 80% Recall** | Precision when recall ≥ 0.80 |
| **Precision @ 90% Recall** | Precision when recall ≥ 0.90 |
| **IoU threshold** | 0.5 (Jaccard similarity between predicted and gold span) |

---

## 🏗️ Loading CUAD into a Lakehouse (Presto / Spark)

### Why flattening is needed

The raw JSON is **3 levels deep** (contract → paragraphs → qas → answers).  
It must be normalized into flat relational tables before loading into a lakehouse.

---

### Proposed Relational Schema

**Table 1: `cuad_questions`**

| Column | Type | Description |
|--------|------|-------------|
| `contract_id` | STRING | Contract filename/title |
| `question_id` | STRING | Unique QA pair ID |
| `clause_category` | STRING | e.g. "Governing Law" |
| `question_text` | STRING | Full question string |
| `contract_context` | STRING | Full plain-text of the contract |
| `is_impossible` | BOOLEAN | True if clause absent from contract |

**Table 2: `cuad_answers`**

| Column | Type | Description |
|--------|------|-------------|
| `question_id` | STRING | FK → `cuad_questions.question_id` |
| `answer_text` | STRING | Extracted clause text span |
| `answer_start` | INT | Character offset in `contract_context` |

---

### Step 1 — Flatten JSON and Load with PySpark

```python
import json
from pyspark.sql import SparkSession, Row

spark = SparkSession.builder.appName("CUAD").getOrCreate()

with open("CUADv1.json") as f:
    raw = json.load(f)

question_rows = []
answer_rows = []

for contract in raw["data"]:
    contract_id = contract["title"]
    context = contract["paragraphs"][0]["context"]
    for qa in contract["paragraphs"][0]["qas"]:
        clause = qa["id"].split("__")[-1]
        question_rows.append(Row(
            contract_id=contract_id,
            question_id=qa["id"],
            clause_category=clause,
            question_text=qa["question"],
            contract_context=context,
            is_impossible=qa["is_impossible"]
        ))
        for ans in qa["answers"]:
            answer_rows.append(Row(
                question_id=qa["id"],
                answer_text=ans["text"],
                answer_start=ans["answer_start"]
            ))

df_questions = spark.createDataFrame(question_rows)
df_answers   = spark.createDataFrame(answer_rows)

# Write as Delta tables to lakehouse
df_questions.write.format("delta").saveAsTable("cuad_questions")
df_answers.write.format("delta").saveAsTable("cuad_answers")
```

---

### Step 2 — SQL Queries (Presto / Spark SQL)

**Query 1: Get all Governing Law clauses**

```sql
SELECT
    q.contract_id,
    q.clause_category,
    a.answer_text,
    a.answer_start
FROM cuad_questions q
JOIN cuad_answers a ON q.question_id = a.question_id
WHERE q.clause_category = 'Governing Law'
ORDER BY q.contract_id;
```

**Query 2: Clause coverage analysis across all contracts**

```sql
SELECT
    clause_category,
    COUNT(DISTINCT contract_id)                                          AS total_contracts,
    SUM(CASE WHEN is_impossible = false THEN 1 ELSE 0 END)               AS contracts_with_clause,
    ROUND(
        100.0 * SUM(CASE WHEN is_impossible = false THEN 1 ELSE 0 END)
        / COUNT(DISTINCT contract_id), 1
    )                                                                    AS pct_with_clause
FROM cuad_questions
GROUP BY clause_category
ORDER BY pct_with_clause DESC;
```

---

### Step 3 — Python Code on the DataFrame

```python
# Pull SQL result into a Pandas DataFrame
df = spark.sql("""
    SELECT q.contract_id, q.clause_category, a.answer_text
    FROM cuad_questions q
    JOIN cuad_answers a ON q.question_id = a.question_id
    WHERE q.is_impossible = false
""").toPandas()

# Average answer length per clause category
df["answer_length"] = df["answer_text"].str.len()
avg_len = (
    df.groupby("clause_category")["answer_length"]
    .mean()
    .sort_values(ascending=False)
)
print(avg_len)

# Count contracts with a Non-Compete clause
non_compete = df[df["clause_category"] == "Non-Compete"]
print(f"Contracts with Non-Compete: {non_compete['contract_id'].nunique()}")

# Feed a specific contract's context + question to the NLP model
sample = spark.sql("""
    SELECT contract_context, question_text
    FROM cuad_questions
    WHERE clause_category = 'Governing Law'
    LIMIT 1
""").toPandas()

context  = sample["contract_context"].iloc[0]
question = sample["question_text"].iloc[0]
# → pass context + question to RoBERTa / DeBERTa model for inference
```

---

### Summary of the Lakehouse Pipeline

```
CUADv1.json
    │
    ▼  (Python: flatten nested JSON)
    │
    ├──► cuad_questions  (Delta/Parquet table)
    └──► cuad_answers    (Delta/Parquet table)
              │
              ▼  (Presto / Spark SQL)
         DataFrame
              │
              ▼  (Pandas / PySpark Python)
         Analysis / NLP Model Input
```

> ⚠️ **Note:** `contract_context` is very long text (~10,000 chars/row).  
> Partition by `clause_category` or `contract_id` to avoid full-table scans.

---

*Generated from CUAD benchmark inspection — 2026-03-01*