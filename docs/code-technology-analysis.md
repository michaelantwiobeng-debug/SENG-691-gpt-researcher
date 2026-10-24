# GPT Researcher: Agent Code and Technology Choices Analysis

**Project:** SENG 691 Group Project  
**Prepared by:** Michael Obeng Antwi  
**Status:** Phase 1 draft for team review

## 1. System Purpose

GPT Researcher is an open source autonomous research agent that accepts a research question, plans how to investigate it, gathers information from web or document sources, evaluates and summarizes the material, and produces a cited report. Its main value is that it turns a broad research request into a repeatable workflow rather than relying on one prompt and one model response.

The core design follows a **planner, executor, and publisher** pattern:

1. The planner interprets the research request and creates supporting questions.
2. Research or crawler components search multiple sources for each question.
3. Retrieved pages are processed, summarized, and associated with their source URLs.
4. The system filters and combines the research context.
5. The report generator writes the final report and includes citations.

This architecture is appropriate for research because planning, information retrieval, source processing, and report writing have different requirements. Separating them also makes the project easier to extend and test.

## 2. Main Code and Agent Components

The primary Python interface is the `GPTResearcher` class. A typical application creates a researcher object, calls `conduct_research()`, and then calls `write_report()`. Supporting components documented by the project include `ResearchConductor`, which manages research activity, and `ReportGenerator`, which creates the final output.

The system supports two related operating styles:

- **Standard research workflow:** A research agent plans questions, collects sources, builds context, and writes a report.
- **Multiagent workflow:** A LangGraph based team assigns separate roles such as chief editor, researcher, reviewer, reviser, writer, and publisher. This provides specialization and review but increases execution time, model calls, and coordination complexity.

The design is asynchronous so searches and network calls can run without blocking the entire application. Streaming events can be sent to the user interface through WebSockets, allowing users to see research progress before the final report is complete.

## 3. Technology Stack

| Layer | Main technology | Purpose and reason for use |
|---|---|---|
| Core language | Python 3.12+ | Strong AI ecosystem, readable code, asynchronous networking, and broad library support. |
| Agent and model integration | LangChain | Provides common interfaces for language models, prompts, tools, document processing, and model providers. |
| Workflow orchestration | LangGraph | Represents multi-step and multiagent research as a controlled graph with state, branching, and repeatable transitions. |
| Model provider abstraction | LiteLLM and provider packages | Allows the project to use different hosted or local language models without rewriting the complete workflow. |
| API layer | FastAPI | Provides asynchronous REST and WebSocket services with automatic request validation and documentation. |
| Data validation | Pydantic | Defines and validates configuration, request, and response structures. |
| Search and retrieval | Tavily by default, with other retrievers | Supplies current external information. The project supports multiple search providers so it is not tied to one vendor. |
| External data integration | Model Context Protocol (MCP) | Connects the agent to sources such as GitHub repositories, databases, and custom APIs. |
| Frontend | Next.js and React | Provides the research form, configuration controls, live progress, results display, and a production oriented web interface. |
| Communication | REST and WebSockets | REST supports regular requests; WebSockets stream logs and research progress in real time. |
| Deployment | Docker and Docker Compose | Packages dependencies consistently and can start the Python API and web interface as separate services. |
| Testing | pytest and GitHub Actions | Supports automated Python tests and continuous integration. |

## 4. Why These Technology Choices Fit the Project

Python is a practical choice because most language model, web retrieval, document processing, and evaluation libraries provide first class Python support. FastAPI complements it by supporting asynchronous operations, which are important when the agent makes many search, scraping, and model requests.

LangChain reduces the effort required to connect different language models, retrievers, embeddings, and document tools. LangGraph adds explicit workflow control. This matters because a research agent may need to branch into several questions, repeat searches when evidence is weak, pass results to a reviewer, and preserve state between stages.

The configurable model and retriever layers are also good design decisions. Teams can select faster models for simple operations and stronger models for planning or synthesis. They can also change search providers or combine web research with internal sources through MCP. This improves flexibility and reduces dependence on a single vendor.

The Next.js frontend and FastAPI backend create a clear separation between presentation and research logic. This supports independent development and makes the backend usable through the web interface, Python package, REST API, WebSocket connection, or MCP server.

## 5. Strengths

- The workflow separates planning, retrieval, processing, and report generation.
- Search tasks can run in parallel, reducing the time required for broad research.
- Sources are retained so the final report can include citations.
- The system supports different models, search providers, report formats, and research depths.
- MCP support allows research to include private or specialized data sources when properly authorized.
- Docker, tests, and multiple interfaces make the system easier to reproduce and deploy.
- The multiagent option adds role specialization and a review stage for complex reports.

## 6. Limitations and Risks

The quality of the final report still depends on search quality, source credibility, model reasoning, and citation accuracy. A citation may point to a real source while failing to support the exact claim, so users must review high impact reports.

Multiagent research creates additional model calls and coordination overhead. Agents may duplicate searches, pass incomplete context, or reinforce another agent's error. The team should compare the benefit of added agents with execution time and API cost.

The application processes untrusted web content and user supplied research queries. This creates risks involving prompt injection, malicious page content, unsafe HTML rendering, and accidental exposure of API keys or private documents. Secrets must remain in environment variables and must never be committed to GitHub. Outputs displayed in the browser should be sanitized, and tool permissions should follow least privilege.

A public repository issue reported reflected cross site scripting through research task names in certain earlier versions. Our team should inspect the exact version in our fork and test the relevant frontend and WebSocket rendering path before stating whether the issue affects our implementation.

## 7. Phase 1 Assessment

GPT Researcher is a suitable foundation for the group project because it already contains visible agent behavior, a clear planning and execution process, multiple integration points, and a report that can be demonstrated. It also gives the team several realistic areas for enhancement, including source ranking, stale information warnings, stronger citation verification, clearer execution logs, security controls, and comparison of changes between research runs.

For Phase 1, the team should record the exact repository commit or version used, document the selected model and retriever, confirm that a complete research run succeeds, and capture the resulting report and citations. These details will make the technical report and demonstration reproducible.

## References

1. GPT Researcher GitHub repository: https://github.com/assafelovic/gpt-researcher
2. GPT Researcher documentation: https://docs.gptr.dev/
3. GPT Researcher Docker quickstart: https://docs.gptr.dev/docs/gpt-researcher/getting-started/getting-started-with-docker
4. GPT Researcher Python package documentation: https://docs.gptr.dev/docs/gpt-researcher/gptr/pip-package
5. GPT Researcher MCP server documentation: https://docs.gptr.dev/docs/gpt-researcher/mcp-server/getting-started
6. Reported XSS issue for version verification: https://github.com/assafelovic/gpt-researcher/issues/1692
