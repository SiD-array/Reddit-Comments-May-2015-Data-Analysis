# 🔴 Reddit Comments May 2015 – Big Data Analysis Project

<div align="center">

![Reddit](https://img.shields.io/badge/Reddit-FF4500?style=for-the-badge&logo=reddit&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-316192?style=for-the-badge&logo=postgresql&logoColor=white)
![MongoDB](https://img.shields.io/badge/MongoDB-4EA94B?style=for-the-badge&logo=mongodb&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)

**A comprehensive data engineering and analysis project on 50+ million Reddit comments**

[Dataset](https://www.kaggle.com/datasets/kaggle/reddit-comments-may-2015) · [Phase 1](#phase-1-relational-model-postgresql) · [Phase 2](#phase-2-document-model-mongodb) · [Phase 3](#phase-3-data-mining--cleaning)

</div>

---

## 👥 Team Members

| Name | ID |
|------|-----|
| Hrithik Gaikwad | hg3916 |
| Jie Zhang | jz7563 |
| Siddharth Bhople | sb8336 |

---

## 📖 Project Overview

This project implements a complete data pipeline for analyzing the **Reddit Comments May 2015** dataset (~50 million comments, ~30GB). The work spans three phases:

| Phase | Focus | Technology |
|-------|-------|------------|
| **Phase 1** | Relational Data Model | PostgreSQL, SQL DDL |
| **Phase 2** | Document-Oriented Model | MongoDB, PyMongo |
| **Phase 3** | Data Mining & Cleaning | Apriori Algorithm, Association Rules |

---

## 📊 Dataset Information

- **Source**: [Kaggle - Reddit Comments May 2015](https://www.kaggle.com/datasets/kaggle/reddit-comments-may-2015)
- **Size**: ~50 million comments, ~30GB compressed
- **Format**: SQLite database (`database.sqlite`)
- **Time Period**: May 2015
- **Fields**: author, subreddit, body, score, ups, downs, gilded, controversiality, and more

---

## 📁 Project Structure

```
Reddit-Comments-May-2015-Data-Analysis/
│
├── data/                              # Dataset directory (not included in repo)
│   └── reddit-comments-may-2015/
│       └── database.sqlite            # ~30GB SQLite file
│
├── phase1_relational/                 # Phase 1: Relational Model
│   ├── code/
│   │   └── load_data.py               # SQLite → PostgreSQL loader
│   ├── ddl/
│   │   ├── create_ddl_queries.sql     # Schema creation DDL
│   │   └── test_queries.sql           # Validation queries
│   ├── diagrams/
│   │   └── relational_schema.jpg      # ER diagram
│   ├── docs/
│   │   ├── Data Description.docx
│   │   └── Data_Description_Group_7.pdf
│   └── README_phase1.md
│
├── phase2_document_model/             # Phase 2: Document Model
│   ├── code/
│   │   ├── load_to_mongo.py           # SQLite → MongoDB loader
│   │   ├── phase2_queries.py          # Benchmark queries
│   │   ├── sample_queries.js          # MongoDB validation queries
│   │   └── discover_functional_dependencies.py
│   ├── diagrams/
│   │   └── doc_model_visual.png       # Document model diagram
│   └── docs/
│       ├── document_model_report.md
│       ├── functional_dependencies_report.md
│       └── README_Queries.md
│
├── phase3_data_mining/                # Phase 3: Data Mining
│   ├── code/
│   │   ├── association_rule_mining.py # Apriori + Association Rules
│   │   ├── data_cleaner.py            # Data cleaning pipeline
│   │   └── mongo_to_relational.py     # Data migration utility
│   ├── docs/
│   │   └── association_rule_mining_README.md
│   └── sql/
│       ├── dirty_data_finder.sql      # Identify dirty data
│       └── verify_cleaning.sql        # Validate cleaned data
│
├── submission/                        # Final submission package
│   └── CSCI620_Term_Project_Group_7.zip
│
├── kaggle.json                        # Kaggle API credentials
├── requirements.txt                   # Python dependencies
└── README.md                          # This file
```

---

## 🚀 Quick Start

### Prerequisites

- **Python** 3.8+
- **PostgreSQL** 14+
- **MongoDB** 6.0+
- ~50GB disk space for the dataset

### 1. Clone the Repository

```bash
git clone https://github.com/SiD-array/Reddit-Comments-May-2015-Data-Analysis.git
cd Reddit-Comments-May-2015-Data-Analysis
```

### 2. Install Dependencies

```bash
pip install -r requirements.txt
```

### 3. Configure Kaggle API (for auto-download)

Place your `kaggle.json` in the project root:
```json
{
  "username": "your_kaggle_username",
  "key": "your_kaggle_api_key"
}
```

### 4. Download Dataset

The dataset will be auto-downloaded when you run the loaders, or manually:
```bash
kaggle datasets download -d kaggle/reddit-comments-may-2015
```

---

## 📋 Phase 1: Relational Model (PostgreSQL)

### Overview

Phase 1 implements a **normalized relational schema** with 6 tables following 3NF principles.

### Schema Design

| Table | Description | Primary Key |
|-------|-------------|-------------|
| `Users` | Reddit authors and flair info | `author` |
| `Subreddit` | Subreddit metadata | `subreddit_id` |
| `Post` | Post-level data | `link_id` |
| `Post_Link` | Post references | `link_id` |
| `Comment` | All comments with scores | `id` |
| `Moderation` | Moderation actions | `mod_action_id` |

### Setup PostgreSQL Schema

```bash
# Create database
psql -U postgres -c "CREATE DATABASE redditdb;"

# Run DDL
psql -U postgres -d redditdb -f phase1_relational/ddl/create_ddl_queries.sql
```

### Load Data

```bash
python phase1_relational/code/load_data.py \
    --input data/reddit-comments-may-2015/database.sqlite \
    --host localhost \
    --port 5432 \
    --user postgres \
    --password yourpassword \
    --dbname redditdb

# Test with sample data
python phase1_relational/code/load_data.py \
    --input data/reddit-comments-may-2015/database.sqlite \
    --password yourpassword \
    --dbname redditdb \
    --sample 10000
```

### Validate Data

```bash
psql -U postgres -d redditdb -f phase1_relational/ddl/test_queries.sql
```

---

## 📋 Phase 2: Document Model (MongoDB)

### Overview

Phase 2 re-models the dataset into a **hybrid document architecture** optimized for MongoDB.

### Document Collections

| Collection | Purpose | Key Design |
|------------|---------|------------|
| `users` | Author profiles | `_id` = author name |
| `subreddits` | Subreddit metadata | `_id` = subreddit_id |
| `posts` | Posts with embedded comments | Hybrid: embeds top N comments |
| `comments` | All comments (analytics) | Full comment storage |
| `moderation` | Moderation signals | Composite `_id` |

### Key Features

- **Hybrid Embedding**: Top N comments embedded in posts for fast reads
- **Bulk Operations**: Efficient `bulk_write()` for high throughput
- **Streaming Ingestion**: Chunked SQLite reads (50K rows/chunk)
- **Idempotent Upserts**: Safe for re-runs

### Load Data to MongoDB

```bash
python phase2_document_model/code/load_to_mongo.py \
    --input data/reddit-comments-may-2015/database.sqlite \
    --mongo_uri "mongodb://localhost:27017/" \
    --dbname reddit_may2015 \
    --chunksize 50000 \
    --embed-cap 200 \
    --reset
```

### Run Validation Queries

```bash
# MongoDB Shell
mongosh phase2_document_model/code/sample_queries.js

# Python benchmark queries
python phase2_document_model/code/phase2_queries.py \
    --host localhost \
    --user postgres \
    --password yourpassword \
    --dbname redditdb
```

### Discover Functional Dependencies

```bash
python phase2_document_model/code/discover_functional_dependencies.py \
    --input data/reddit-comments-may-2015/database.sqlite \
    --sample 1000000
```

---

## 📋 Phase 3: Data Mining & Cleaning

### Overview

Phase 3 focuses on **data quality** and **pattern discovery** using association rule mining.

### Data Cleaning

The cleaning pipeline handles:

| Issue | Action |
|-------|--------|
| Missing authors | Drop row |
| Empty bodies | Drop row |
| `[deleted]`/`[removed]` content | Drop row |
| Invalid timestamps | Drop row |
| Inconsistent scores | Fix: `score = ups - downs` |
| Symbol-only flair | Set to NULL |

#### Run Data Cleaner

```bash
python phase3_data_mining/code/data_cleaner.py \
    --host localhost \
    --user postgres \
    --password yourpassword \
    --dbname redditdb \
    --batch-size 50000
```

### Association Rule Mining

Discovers patterns in Reddit comments using the **Apriori algorithm**.

#### Transaction Definition

Each comment becomes a transaction with:
- **Subreddit**: `subreddit:AskReddit`
- **Score Category**: `very_high_score`, `high_score`, `medium_score`, `low_score`, `negative_score`
- **Status Flags**: `gilded`, `controversial`, `edited`, `distinguished`, `archived`

#### Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| `min_support` | 0.01 (1%) | Minimum frequency threshold |
| `min_confidence` | 0.5 (50%) | Minimum rule confidence |

#### Run Association Rule Mining

```bash
python phase3_data_mining/code/association_rule_mining.py \
    --host localhost \
    --user postgres \
    --password yourpassword \
    --dbname redditdb \
    --min-support 0.03 \
    --min-confidence 0.5 \
    --sample 100000
```

#### Output Metrics

| Metric | Description |
|--------|-------------|
| **Support** | Frequency of itemset in all transactions |
| **Confidence** | P(consequent \| antecedent) |
| **Lift** | Strength of association (>1 = positive) |
| **Conviction** | Expected error of the rule |

---

## 📈 Performance Benchmarks

### Phase 2 Query Benchmarks

| Query | Before Index | After Index | Speedup |
|-------|-------------|-------------|---------|
| AskReddit latest 50 posts | 2.34s | 0.12s | 19.5x |
| Top 20 subreddits by avg score | 5.67s | 0.89s | 6.4x |
| Top 20 authors by post count | 3.21s | 0.45s | 7.1x |
| Gilded but not archived posts | 1.89s | 0.08s | 23.6x |
| Posts by authors containing 'cat' | 4.56s | 0.67s | 6.8x |
| Avg comments per post (top 10) | 8.23s | 1.34s | 6.1x |

---

## 🛠️ Dependencies

```
psycopg2-binary>=2.9.0   # PostgreSQL adapter
pandas>=1.3.0            # Data manipulation
requests>=2.25.0         # HTTP library
tqdm>=4.60.0             # Progress bars
kaggle                   # Dataset download
mlxtend>=0.22.0          # Association rule mining
pymongo                  # MongoDB driver
```

---

## 📚 Documentation

| Document | Location |
|----------|----------|
| Phase 1 README | `phase1_relational/README_phase1.md` |
| Document Model Report | `phase2_document_model/docs/document_model_report.md` |
| Functional Dependencies | `phase2_document_model/docs/functional_dependencies_report.md` |
| Query Documentation | `phase2_document_model/docs/README_Queries.md` |
| Association Rule Mining | `phase3_data_mining/docs/association_rule_mining_README.md` |

---

## 🔧 Troubleshooting

| Issue | Solution |
|-------|----------|
| Connection refused | Ensure PostgreSQL/MongoDB is running |
| Missing kaggle.json | Download API token from Kaggle account |
| Permission denied | Check database user privileges |
| Disk space full | Dataset requires ~50GB during processing |
| Memory error | Use `--sample` parameter for testing |
| Download timeout | Ensure stable internet (~20GB download) |

---

## 📄 License

This project is for educational purposes as part of **CSCI-620: Big Data** course work.

---

## 🙏 Acknowledgments

- [Kaggle](https://www.kaggle.com) for hosting the Reddit Comments dataset
- [Reddit](https://www.reddit.com) for the original data
- RIT CSCI-620 course staff for guidance

---

<div align="center">

**Made with ❤️ by Group 7**

</div>
