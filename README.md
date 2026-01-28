# 🔍 Deep Research Reflection Agent with LangGraph

<p align="center">
  <img src="https://img.shields.io/badge/LangGraph-000000?style=for-the-badge&logo=chainlink&logoColor=white" />
  <img src="https://img.shields.io/badge/OpenAI_GPT--4o--mini-412991?style=for-the-badge&logo=openai&logoColor=white" />
  <img src="https://img.shields.io/badge/Tavily_Search-00A67E?style=for-the-badge&logo=google-cloud&logoColor=white" />
  <img src="https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white" />
</p>

## 🌟 Overview
This project implements a **Reflexion Framework** agent designed for deep research. Unlike standard LLMs that may hallucinate or provide outdated info, this agent **critiques its own answers**, identifies missing evidence, and executes **external web searches** to refine its output.

### 🔄 The Reflection Loop
The agent follows a cyclical, iterative workflow:
1.  **Responder**: Generates an initial evidence-based answer.
2.  **Self-Critique**: Identifies knowledge gaps and generates targeted search queries.
3.  **Tool Execution**: Uses the **Tavily Search API** to gather real-time data.
4.  **Revisor**: Rewrites the answer, incorporating new evidence and numerical citations.

## 🛠️ Tech Stack & Concepts
* **Framework**: [LangGraph](https://python.langchain.com/docs/langgraph) for cyclical workflows.
* **External Integration**: [Tavily Search API](https://tavily.com/) for real-time web research.
* **Data Validation**: [Pydantic](https://docs.pydantic.dev/) for structured output and tool binding.
* **Persona Engineering**: Implements a nuanced expert persona (Dr. Paul Saladino) to provide evidence-based nutritional advice.

## 🚀 Features
* ✅ **Iterative Refinement**: Automatically loops up to 4 times to maximize accuracy.
* ✅ **Evidence-Based**: Enforces the use of numerical citations and peer-reviewed research links.
* ✅ **Structured Thinking**: Uses specialized `Responder` and `Revisor` nodes to separate draft generation from fact-checking.

## 📂 Quick Start
1. Clone the repository.
2. Install dependencies: `pip install langgraph langchain-openai langchain-community`.
3. Set your `TAVILY_API_KEY` environment variable.
4. Run the Jupyter Notebook to watch the agent research and refine its answers in real-time!

---
*Created as part of the IBM Skills Network Advanced AI Agents series.*
