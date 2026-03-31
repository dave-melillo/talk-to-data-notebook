# Talk To Data - Jupyter Notebook Edition

**Learn AI-powered natural language to SQL through interactive cells**

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/dave-melillo/talk-to-data-notebook/blob/main/talk-to-data-notebook.ipynb)

## What Is This?

A simplified, educational implementation of [Talk To Data](https://github.com/dave-melillo/talk-to-data) designed for:
- **Learning:** Each cell = one concept
- **Teaching:** Perfect for books, tutorials, workshops
- **Debugging:** See exactly what the AI generates at each step
- **Portability:** Runs in Google Colab, Jupyter, VS Code—no setup required

## Why This Exists

The production Talk To Data repo has FastAPI backends, Streamlit UIs, Docker orchestration, and deployment complexity. That's great for production, but it obscures the core concepts for learners.

This notebook strips away the infrastructure and presents the pure algorithmic components in discrete, teachable cells.

## Quick Start

### Option 1: Google Colab (Easiest)

1. Click the "Open In Colab" badge above
2. Go to **Secrets** (key icon in left sidebar)
3. Add secret: `OPENAI_API_KEY` with your API key
4. Run all cells (Runtime → Run all)

### Option 2: Local Jupyter

```bash
# Clone the repo
git clone https://github.com/dave-melillo/talk-to-data-notebook.git
cd talk-to-data-notebook

# Install dependencies
pip install -r requirements.txt

# Set your API key
export OPENAI_API_KEY="your-key-here"

# Start Jupyter
jupyter notebook talk-to-data-notebook.ipynb
```

### Option 3: VS Code

1. Install the [Jupyter extension](https://marketplace.visualstudio.com/items?itemName=ms-toolsai.jupyter)
2. Open `talk-to-data-notebook.ipynb` in VS Code
3. Set `OPENAI_API_KEY` in your environment
4. Run cells interactively

## What You'll Learn

### 1. Data Ingestion
Load CSV files or connect to databases using pandas. Display sample rows for verification.

### 2. Data Semantic Generation (AI)
Use an LLM to analyze DataFrame schema and generate structured metadata (YAML) describing:
- Table descriptions
- Column types and meanings
- Relationships between tables
- Sample values

### 3. Business Semantic Definition
Provide domain-specific terminology:
- **Glossary:** Define terms like "churned_customer"
- **KPIs:** Formulas like `SUM(order_total)`
- **Rules:** Domain-specific constraints

### 4. Reference Queries (Few-Shot Learning)
Provide example question-SQL pairs to guide the LLM's output:
```
Q: "Who are my top 10 customers?"
SQL: SELECT customer_id, name, lifetime_value FROM customers ORDER BY lifetime_value DESC LIMIT 10
```

### 5. Query Engine (AI)
Combine data semantic + business semantic + reference queries into LLM context. Generate executable SQL from natural language.

### 6. Execution & Results
Execute generated SQL against in-memory SQLite database. Display results as pandas DataFrame.

## Example Questions

Try asking:
- "Who are my top 10 customers by lifetime value?"
- "What were the total sales in March 2024?"
- "Which products have low stock?"
- "What are the best-selling products by quantity?"
- "How many active customers do we have?"

## Sample Datasets

Included in `data/`:
- **customers.csv** — 100 customer records with demographics and lifetime value
- **orders.csv** — 500 order records with timestamps and totals
- **products.csv** — 50 product records with inventory and pricing

## How It Works

```
CSV File → pandas DataFrame → Schema Analysis
                                    ↓
                            LLM (Data Semantic)
                                    ↓
User Question + Data Semantic + Biz Semantic + References
                                    ↓
                            LLM (SQL Generation)
                                    ↓
                        SQL → SQLite → Results DataFrame
```

## Requirements

- **Python:** 3.9+
- **Dependencies:**
  - `pandas>=2.0.0`
  - `openai>=1.0.0`
  - `pyyaml>=6.0`
- **API Key:** OpenAI (or modify for Anthropic/Gemini)

## LLM Provider Options

**Default:** OpenAI GPT-4o

**To use Anthropic Claude:**
```python
from anthropic import Anthropic
client = Anthropic(api_key=api_key)

response = client.messages.create(
    model="claude-3-5-sonnet-20241022",
    max_tokens=1024,
    messages=[{"role": "user", "content": prompt}]
)
sql = response.content[0].text
```

**To use local models (Ollama):**
```python
import requests
response = requests.post('http://localhost:11434/api/generate', json={
    'model': 'codellama',
    'prompt': prompt
})
sql = response.json()['response']
```

## Cost Estimates

**OpenAI GPT-4o:**
- Data semantic generation: ~$0.01 per dataset
- SQL generation: ~$0.001 per question
- **Total for 100 questions:** ~$0.10–$0.20

**Tips to reduce costs:**
- Cache data semantics (only generate once)
- Use GPT-3.5 for simpler queries
- Switch to local models (Ollama, LM Studio)

## Educational Use

This notebook is designed for:
- **Technical books** — Include as a chapter or appendix
- **Workshops** — Step-through each cell with learners
- **Self-study** — Read markdown cells, run code, experiment
- **University courses** — Demonstrate LLM applications in data science

## Limitations (By Design)

This is **not** a production system. It lacks:
- ❌ Web UI
- ❌ Multi-user support
- ❌ Authentication
- ❌ Production databases (Postgres, MySQL)
- ❌ Query optimization
- ❌ Cost tracking

For production features, see [dave-melillo/talk-to-data](https://github.com/dave-melillo/talk-to-data).

## Extending This Notebook

**Add visualization:**
```python
import seaborn as sns
results = query("What are the top 10 products by revenue?")
sns.barplot(data=results, x='product_name', y='total_revenue')
```

**Add query validation:**
```python
def validate_sql(sql):
    forbidden = ['DROP', 'DELETE', 'UPDATE', 'INSERT', 'ALTER']
    if any(kw in sql.upper() for kw in forbidden):
        raise ValueError("Unsafe SQL detected!")
    return sql
```

**Add query caching:**
```python
query_cache = {}

def ask_question(question):
    if question in query_cache:
        return query_cache[question]
    
    sql = generate_sql(question)
    query_cache[question] = sql
    return sql
```

## Troubleshooting

### "API key not found"
**Solution:** Set `OPENAI_API_KEY` in Colab Secrets or environment variables.

### "ModuleNotFoundError: No module named 'openai'"
**Solution:** Run the first cell to install dependencies:
```python
!pip install -q openai pandas pyyaml
```

### "SQL Error: no such table"
**Solution:** Re-run Cell 2 (Data Ingestion) to create in-memory database.

### "Generated SQL is incorrect"
**Solutions:**
- Add more reference queries in Cell 5
- Improve business semantic definitions in Cell 4
- Use `show_prompt=True` to debug LLM input

## Contributing

Found a bug? Have a suggestion? Open an issue or PR!

This is an educational project—contributions that improve clarity and pedagogy are especially welcome.

## License

MIT License - see [LICENSE](LICENSE) for details.

## Related Projects

- **Production version:** [dave-melillo/talk-to-data](https://github.com/dave-melillo/talk-to-data)
- **Streamlit app:** (coming soon)
- **API version:** (coming soon)

## Acknowledgments

Inspired by:
- [Andrej Karpathy's educational repos](https://github.com/karpathy)
- [Fast.ai](https://www.fast.ai/) — Notebooks as teaching tools
- [Google Colab](https://colab.research.google.com/) — Democratizing access to compute

---

**Questions?** Open an issue or reach out on [Twitter/X](https://x.com/davemelillo).

**Built with ❤️ for learning and teaching.**
