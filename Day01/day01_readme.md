# Day 01 : Introduction to Agents

Today’s whitepaper introduces Al agents. It presents a taxonomy of agent capabilities, emphasizes the need for an Agent Ops discipline for reliability and governance, and discusses the importance of agent interoperability and security through identity and constrained policies.

In today's codelabs, you'll be building your first AI agent and your first multi-agent system, using Agent Development Kit (ADK), powered by Gemini, and giving it the ability to use Google Search to answer questions with up-to-date information. In the second codelab, the focus will be on multi-agent systems, where you'll learn how to create teams of specialized agents and explore different architectural patterns.

#### Day 1 Assignments

1. Complete the Unit 1 – “Introduction to Agents”:

 - Listen to the [summary podcast episode](https://www.youtube.com/watch?v=zTxvGzpfF-g) for this unit, created by [NotebookLM](https://notebooklm.google.com/?pli=1).
 - To complement the podcast, read the [“Introduction to Agents” whitepaper](https://www.kaggle.com/whitepaper-introduction-to-agents)
 - Complete these codelabs on Kaggle:
    - [Build](https://www.kaggle.com/code/kaggle5daysofai/day-1a-from-prompt-to-action) your first agent using Gemini and ADK.
    - [Build](https://www.kaggle.com/code/kaggle5daysofai/day-1b-agent-architectures) your first multi-agent systems using ADK.
    - Make sure you [phone verify](https://www.kaggle.com/settings) your Kaggle account before starting, it's necessary for the codelabs.
    - We also have a [troubleshooting guide](https://www.kaggle.com/code/kaggle5daysofai/day-0-troubleshooting-and-faqs) for the codelabs. Be sure to check there for solutions to common problems.
    - Want to have an [interactive conversation](https://support.google.com/notebooklm/?visit_id=638985882264394645-2014075471&rd=2&topic=14272891#topic=16164070)? Try adding the whitepapers to [NotebookLM](https://notebooklm.google.com).


### Key-Takeways :

#### Lab

**1. Build a Simple Agent**

Defined the Agent() method, configues it's property to instruct what to do and how to operate:
1. name: name of the agent
2. model: The specific LLM that will power the agent's reasoning. We used "gemini-2.5-flash-lite".
3. description: description of the agent which will help us identify it
4. instruction: The agent's guiding prompt. This tells the agent what its goal is and how to behave.
5. tools: A list of [tools](https://google.github.io/adk-docs/tools/) that the agent can use. To start with, we used only one tool which is google_search tool, which lets it find up-to-date information online.

```
root_agent = Agent(
    name="helpful_assistant",
    model=Gemini(
        model="gemini-2.5-flash-lite",
        retry_options=retry_config
    ),
    description="A simple agent that can answer general questions.",
    instruction="You are a helpful assistant. Use Google Search for current info or if unsure.",
    tools=[google_search],
)
```

Run the agent using a runner which is the central component within ADK that acts as the orchestrator. Then, run the .run_debug() method to send our prompt and get an answer:

```
runner = InMemoryRunner(agent=root_agent)
# ask any question
response = await runner.run_debug(
    "What is Agent Development Kit from Google? What languages is the SDK available in?"
)
```
2. Multi-Agents

Architecture: Single Agent vs Multi-Agent Team
![Architecture: Single Agent vs Multi-Agent Team](./images/single_vs_multi-agent_archi.png)


Types of multi-agent:
1. Sequential Agent
2. Parallel Agent
3. Loop Agent
4. LLM Orchestrator

![Types of multi-agent](./images/types_of_multi-agents.png)

Example to explain the multi-agent implementation:<br>
Describe the sub-agents research agent and summarizer agent same as decsribing simple agent explained above. <br>
```
# Research Agent: Its job is to use the google_search tool and present findings.
research_agent = Agent(
    name="ResearchAgent",
    model=Gemini(
        model="gemini-2.5-flash-lite",
        retry_options=retry_config
    ),
    instruction="""You are a specialized research agent. Your only job is to use the
    google_search tool to find 2-3 pieces of relevant information on the given topic and present the findings with citations.""",
    tools=[google_search],
    output_key="research_findings",  # The result of this agent will be stored in the session state with this key.
)

# Summarizer Agent: Its job is to summarize the text it receives.
summarizer_agent = Agent(
    name="SummarizerAgent",
    model=Gemini(
        model="gemini-2.5-flash-lite",
        retry_options=retry_config
    ),
    # The instruction is modified to request a bulleted list for a clear output format.
    instruction="""Read the provided research findings: {research_findings}
Create a concise summary as a bulleted list with 3-5 key points.""",
    output_key="final_summary",
)


```
Describe the root agent that orchestrates the workflow by calling the sub-agents.

```
 Root Coordinator: Orchestrates the workflow by calling the sub-agents as tools.
root_agent = Agent(
    name="ResearchCoordinator",
    model=Gemini(
        model="gemini-2.5-flash-lite",
        retry_options=retry_config
    ),
    # This instruction tells the root agent HOW to use its tools (which are the other agents).
    instruction="""You are a research coordinator. Your goal is to answer the user's query by orchestrating a workflow.
1. First, you MUST call the `ResearchAgent` tool to find relevant information on the topic provided by the user.
2. Next, after receiving the research findings, you MUST call the `SummarizerAgent` tool to create a concise summary.
3. Finally, present the final summary clearly to the user as your response.""",
    # We wrap the sub-agents in `AgentTool` to make them callable tools for the root agent.
    tools=[AgentTool(research_agent), AgentTool(summarizer_agent)],
)

```
Run the agent and call the agent

```
runner = InMemoryRunner(agent=root_agent)
response = await runner.run_debug(
    "What are the latest advancements in quantum computing and what do they mean for AI?"
)
```

#### Whitepaper: 
An AI Agent is fundamentally defined as a combination of models, tools, an orchestration layer, and runtime services that utilizes an LLM in a continuous loop to achieve a goal.
The architecture comprises three essential components:

**1. Model**<br>
The core LM that serves as the central reasoning engine to process information, evaluate options, and make decisions. Selection depends on the business problem, prioritizing superior reasoning and reliable tool use over generic benchmarks. Robust systems often employ model routing (a "team of specialists") to use powerful models for complex tasks and faster, cost-effective models for high-volume, simpler tasks
<br><br>
**2. Tools**<br>
Mechanisms that connect the agent's reasoning to the outside world, enabling it to act beyond text generation. Tools include:
   - Retrieving Information: Utilizing Retrieval-Augmented Generation (RAG) systems (like vector databases) and Natural Language to SQL (NL2SQL) to ground the agent in reality and reduce hallucinations.
   - Executing Actions: Wrapping existing APIs or code functions to perform actions like sending emails or updating records. This includes Human in the Loop (HITL) tools to pause workflows and request confirmation for critical decisions.
   - Function Calling: The process by which the agent reliably selects and uses tools, often relying on structured contracts like the OpenAPI specification or protocols like the Model Context Protocol (MCP)
<br><br>
**3. Orchestration layer**<br>
The governing process that runs the agent’s operational loop. It manages planning, memory (state), and reasoning strategy execution (using frameworks like Chain-of-Thought or ReAct). This layer manages the agent’s memory, including short-term memory (session history) and long-term memory (often implemented as a specialized RAG system)
<br><br>
The Agent operates on a continuous, cyclical Problem-Solving Process to achieve its objectives:
<br><br>
========> Think ===========> Act ===========> Observe  
<br><br>
1. It Gets the Mission
2. Scans the Scene (gathers context)
3. Thinks It Through (devises a multi-step plan)
4. Takes Action (invokes a tool), and
5. Observes and Iterates