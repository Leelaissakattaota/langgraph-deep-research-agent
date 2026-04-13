# 🔬 Deep Research Reflexion Agent — LangGraph + Tavily + OpenAI

![Language](https://img.shields.io/badge/Language-Python%203.11-3776AB?style=flat-square&logo=python&logoColor=white)
![Framework](https://img.shields.io/badge/Framework-LangGraph-FF6B35?style=flat-square)
![LLM](https://img.shields.io/badge/LLM-GPT--4.1--nano-412991?style=flat-square&logo=openai&logoColor=white)
![Search](https://img.shields.io/badge/Search-Tavily%20API-2E7D32?style=flat-square)
![Pattern](https://img.shields.io/badge/Pattern-Reflexion%20Loop-0052CC?style=flat-square)
![Validation](https://img.shields.io/badge/Validation-Pydantic-6A0DAD?style=flat-square)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen?style=flat-square)

---

## 📌 Project Overview

This project builds a **Reflexion Agent** — an advanced AI agent that 
doesn't just answer questions, but **critiques its own answers, identifies 
weaknesses, searches for external evidence, and iteratively revises** 
its responses to be more accurate and comprehensive.

The agent embodies **Dr. Paul Saladino** (initial response) and 
**Dr. David Attia** (revision) personas — two distinct evidence-based 
nutritional experts — demonstrating how AI agents can adopt specialized 
domain knowledge while applying rigorous self-improvement cycles.

**Domain:** Agentic AI — Reflexion Pattern with External Tools  
**LLM:** OpenAI GPT-4.1 Nano  
**Search Tool:** Tavily Search API (real-time web search)  
**Framework:** LangGraph (`MessageGraph`)  
**Pattern:** Reflexion (Generate → Critique → Search → Revise → Loop)  

---

## 🏗️ Reflexion Agent Architecture

```
User Question
      │
      ▼
┌─────────────────────────────────────────┐
│         RESPOND Node                    │
│  GPT-4.1 Nano as Dr. Paul Saladino     │
│  Carnivore MD — Animal-based nutrition  │
│                                         │
│  Structured Output (bind_tools):        │
│  AnswerQuestion {                       │
│    answer: str                          │
│    reflection: {missing, superfluous}   │
│    search_queries: List[str]            │
│  }                                      │
└───────────────┬─────────────────────────┘
                │
                ▼
┌─────────────────────────────────────────┐
│       EXECUTE_TOOLS Node                │
│  Tavily Search API (max_results=3)      │
│  Runs each search_query                 │
│  Returns ToolMessage with JSON results  │
└───────────────┬─────────────────────────┘
                │
                ▼
┌─────────────────────────────────────────┐
│         REVISOR Node                    │
│  GPT-4.1 Nano as Dr. David Attia       │
│  Evidence-based, mechanistic approach   │
│                                         │
│  Structured Output (bind_tools):        │
│  ReviseAnswer {                         │
│    answer: str (revised)                │
│    reflection: {missing, superfluous}   │
│    search_queries: List[str]            │
│    references: List[str]  ← New!        │
│  }                                      │
└───────────────┬─────────────────────────┘
                │
         ┌──────▼──────┐
         │ event_loop  │
         │             │
         │ iterations  │
         │ < MAX(4)?   │
         │             │
    Yes  │             │  No
    ─────┘             └─────
    back to                END
 execute_tools
```

---

## 📂 Project Structure

```
langgraph-deep-research-agent/
│
├── lab - Building a Reflexion Agent with External Knowledge In.ipynb
└── README.md
```

---

## 🛠️ Tech Stack

| Component | Technology |
|---|---|
| Agent Framework | LangGraph (`MessageGraph`) |
| LLM | OpenAI GPT-4.1 Nano |
| Search Tool | Tavily Search API |
| Data Validation | Pydantic BaseModel + Field |
| Prompts | ChatPromptTemplate + MessagesPlaceholder |
| Tool Binding | `llm.bind_tools([AnswerQuestion])` |
| LangChain | langchain-openai, langchain-community |

---

## 🧠 Pydantic Structured Output Schemas

```python
class Reflection(BaseModel):
    missing: str     = Field(description="What information is missing")
    superfluous: str = Field(description="What information is unnecessary")

class AnswerQuestion(BaseModel):
    answer: str           = Field(description="Main response to the question")
    reflection: Reflection = Field(description="Self-critique of the answer")
    search_queries: List[str] = Field(description="Queries for additional research")

class ReviseAnswer(AnswerQuestion):
    """Revise your original answer to your question."""
    references: List[str] = Field(description="Citations motivating your updated answer")
```

---

## 🔄 LangGraph Workflow

```python
MAX_ITERATIONS = 4

def event_loop(state: List[BaseMessage]) -> str:
    """Stop after MAX_ITERATIONS tool searches."""
    count_tool_visits = sum(isinstance(item, ToolMessage) for item in state)
    if count_tool_visits >= MAX_ITERATIONS:
        return END
    return "execute_tools"

# Build MessageGraph
graph = MessageGraph()
graph.add_node("respond", initial_chain)       # Dr. Saladino initial answer
graph.add_node("execute_tools", execute_tools)  # Tavily web search
graph.add_node("revisor", revisor_chain)        # Dr. Attia revised answer

# Wire edges
graph.add_edge("respond", "execute_tools")
graph.add_edge("execute_tools", "revisor")
graph.add_conditional_edges("revisor", event_loop)  # Loop or END
graph.set_entry_point("respond")

app = graph.compile()
```

---

## 🔧 Tool Execution

```python
tavily_tool = TavilySearchResults(max_results=3)

def execute_tools(state: List[BaseMessage]) -> List[BaseMessage]:
    last_ai_message = state[-1]
    tool_messages = []
    for tool_call in last_ai_message.tool_calls:
        if tool_call["name"] in ["AnswerQuestion", "ReviseAnswer"]:
            search_queries = tool_call["args"].get("search_queries", [])
            query_results = {}
            for query in search_queries:
                result = tavily_tool.invoke(query)
                query_results[query] = result
            tool_messages.append(ToolMessage(
                content=json.dumps(query_results),
                tool_call_id=tool_call["id"]
            ))
    return tool_messages
```

---

## 👨‍⚕️ Two Expert AI Personas

### 🥩 Responder — Dr. Paul Saladino (Carnivore MD)
- Advocates animal-based nutrition
- Emphasizes organ meats as superfoods
- Challenges plant-based dietary dogma
- Focuses on antinutrients (oxalates, lectins, phytates)
- Generates initial draft + self-critique + search queries

### 🔬 Revisor — Dr. David Attia (Evidence-Based MD)
- Clinical, mechanistic approach
- Requires peer-reviewed citations
- Distinguishes correlation vs causation
- Considers biomarkers (lipid panels, inflammatory markers)
- Addresses metabolic flexibility + insulin sensitivity
- Adds numbered references section

---

## 🚀 How to Run

**Step 1 — Install dependencies:**
```bash
pip install langchain-openai==0.3.10 langchain==0.3.21 openai==1.68.2 langchain-community==0.3.24 langgraph
```

**Step 2 — Set API keys:**
```bash
export OPENAI_API_KEY="your-key"
export TAVILY_API_KEY="your-key"  # Free at app.tavily.com
```

**Step 3 — Run Jupyter Notebook:**
```bash
jupyter notebook "lab - Building a Reflexion Agent with External Knowledge In.ipynb"
```

**Step 4 — Test with a query:**
```python
responses = app.invoke(
    """I'm pre-diabetic and need to lower my blood sugar, and I have heart issues.
    What breakfast foods should I eat and avoid?"""
)
```

---

## 📋 Reflexion Iteration Flow

```
Iteration 1:
  → Respond: Initial draft (Dr. Saladino persona)
  → Critique: Missing studies on glycemic index
  → Search: ["antinutrients blood sugar", "animal protein insulin"]
  → Execute: Tavily fetches 3 results per query
  → Revise: Dr. Attia refines with citations

Iteration 2:
  → Critique: Missing mechanistic data on satiety
  → Search: ["carnivore diet diabetes RCT", ...]
  → Execute: New Tavily searches
  → Revise: Further refined with new evidence

... continues up to MAX_ITERATIONS = 4
```

---

## 🎓 Skills Demonstrated

- LangGraph `MessageGraph` for cyclical agent workflows
- Reflexion pattern — generate, critique, search, revise loop
- `bind_tools()` for Pydantic-enforced structured LLM output
- Inheritance-based Pydantic schemas (`ReviseAnswer` extends `AnswerQuestion`)
- Tavily Search API integration for real-time web research
- Tool message management in LangGraph conversation history
- Conditional loop control with `event_loop()` + MAX_ITERATIONS
- `MessagesPlaceholder` for dynamic conversation history in prompts
- Two distinct AI expert personas via prompt engineering
- Evidence-based answer revision with numbered citations
- `ChatPromptTemplate.partial()` for reusable prompt variants
- Complex healthcare Q&A with iterative evidence gathering

---

## 📜 Certifications

| Certification | Issuer | Platform |
|---|---|---|
| IBM Data Science Professional Certificate | IBM | Coursera |
| IBM Generative AI Professional Certificate | IBM | Coursera |
| IBM Agentic AI with RAG Certificate | IBM | Coursera |
| IBM RAG and Agentic AI Professional Certificate | IBM | Coursera |

---

## 🤝 Connect with Me

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Leela%20A-0077B5?style=flat-square&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/leela-a)
[![Gmail](https://img.shields.io/badge/Gmail-attotaleelaissak@gmail.com-D14836?style=flat-square&logo=gmail&logoColor=white)](mailto:attotaleelaissak@gmail.com)
[![GitHub](https://img.shields.io/badge/GitHub-Leelaissakattaota-181717?style=flat-square&logo=github&logoColor=white)](https://github.com/Leelaissakattaota)
