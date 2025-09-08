---
layout: mypost
title: Open-Source Agent Project 03-DeerFlow
categories: [Open-Source Project]
date: 2025-09-08 
---
# 整体架构

![image.png](attachment:fb78d1f9-dfd3-4326-9d8a-b603723cb62b:image.png)


## plan & execute

- [ ]  怎么知道自己执行到第几步？
- [ ]  planner高度集中，每次结束都会返回planner

# Node / State / Edge

## State

```python
from src.prompts.planner_model import Plan
from src.rag import Resource

class State(MessagesState):
    """State for the agent system, extends MessagesState with next field."""

    # Runtime Variables
    locale: str = "en-US"
    research_topic: str = ""
    observations: list[str] = []
    resources: list[Resource] = []
    plan_iterations: int = 0
    current_plan: Plan | str = None
    final_report: str = ""
    auto_accepted_plan: bool = False
    enable_background_investigation: bool = True
    background_investigation_results: str = None
```

## planner

- [ ]  怎么设计plan
- [ ]  new plan 和 current plan的关系

## human_feedback

- 如果没有开启对planner生成的plan的auto_accept，则调用LangGraph的interrupt来打断，并要求review the plan
- [ ]  langgraph的interrupt怎么实现的？有没有resume？

```python
auto_accepted_plan = state.get("auto_accepted_plan", False)
    if not auto_accepted_plan:
        feedback = interrupt("Please Review the Plan.")
```

- 支持edit plan：用户可以edit plan，如果edit了，说明用户没有接受plan，则将feedback重新发回去给planner
    - [ ]  这里怎么保证用户edit plan后收到的feedback一定是`startswith("[EDIT_PLAN]")` ？

```python
# if the feedback is not accepted, return the planner node
        if feedback and str(feedback).upper().startswith("[EDIT_PLAN]"):
            return Command(
                update={
                    "messages": [
                        HumanMessage(content=feedback, name="feedback"),
                    ],
                },
                goto="planner",
            )
```

- 如果用户accept了plan，则goto research_team node

## researcher_node

- tools支持web search 和 爬虫

```python
tools = [get_web_search_tool(configurable.max_search_results), crawl_tool]
```

- 支持本地rag检索
- [ ]  什么情况下会开启本地rag检索，具体怎么做？什么情况下state会有resources

```python
retriever_tool = get_retriever_tool(state.get("resources", []))
```

## coder_node

- 只有一个python_repl_tool

```python
async def coder_node(
    state: State, config: RunnableConfig
) -> Command[Literal["research_team"]]:
    """Coder node that do code analysis."""
    logger.info("Coder node is coding.")
    return await _setup_and_execute_agent_step(
        state,
        config,
        "coder",
        [python_repl_tool],
    )
```

# Tools

## 封装

- 装饰器封装支持其中过程
- 怎么使tool调用期间有日志？

## web search tool

- 控制一下domain 调用TAVILY

```python
def get_web_search_tool(max_search_results: int):
    search_config = get_search_config()

    if SELECTED_SEARCH_ENGINE == SearchEngine.TAVILY.value:
        # Only get and apply include/exclude domains for Tavily
        include_domains: Optional[List[str]] = search_config.get("include_domains", [])
        exclude_domains: Optional[List[str]] = search_config.get("exclude_domains", [])

        logger.info(
            f"Tavily search configuration loaded: include_domains={include_domains}, exclude_domains={exclude_domains}"
        )

        return LoggedTavilySearch(
            name="web_search",
            max_results=max_search_results,
            include_raw_content=True,
            include_images=True,
            include_image_descriptions=True,
            include_domains=include_domains,
            exclude_domains=exclude_domains,
        )
```

## python repl tool

## retriver tool

# RAG

- rag内容列表在configuration里

```python
@dataclass(kw_only=True)
class Configuration:
    """The configurable fields."""

    resources: list[Resource] = field(
        default_factory=list
    )  # Resources to be used for the research
    max_plan_iterations: int = 1  # Maximum number of plan iterations
    max_step_num: int = 3  # Maximum number of steps in a plan
    max_search_results: int = 3  # Maximum number of search results
    mcp_settings: dict = None  # MCP settings, including dynamic loaded tools
    report_style: str = ReportStyle.ACADEMIC.value  # Report style
    enable_deep_thinking: bool = False  # Whether to enable deep thinking
```

- coordinator把resources从configuration放入state中

```python
return Command(
        update={
            "messages": messages,
            "locale": locale,
            "research_topic": research_topic,
            "resources": configurable.resources,
        },
        goto=goto,
    )
```

# set up and execute agent

- 处理MCP：有多个MCP servers，每个MCP servers又有多个tools
- 每个MCP server有支持的agent，需要确认当前agent是否在这个server的支持的agent列表里、
- 收集该 server 的核心信息，存储在字典里，包括传输类型，参数，url等
- `enabled_tools[tool_name] = server_name` 表示某个MCP tool是由哪个 server 提供的。

```python
# Extract MCP server configuration for this agent type
    if configurable.mcp_settings:
        for server_name, server_config in configurable.mcp_settings["servers"].items():
            if (
                server_config["enabled_tools"]
                and agent_type in server_config["add_to_agents"]
            ):
                mcp_servers[server_name] = {
                    k: v
                    for k, v in server_config.items()
                    if k in ("transport", "command", "args", "url", "env", "headers")
                }
                for tool_name in server_config["enabled_tools"]:
                    enabled_tools[tool_name] = server_name
```

## _execute_agent_step

- agent具体执行Plan里的steps的过程

# Memory

- [ ]  安装PostgreSQL
    - [ ]  为什么是PostgreSQL
- [ ]  用checkpoint还是sql？
- [ ]  

## 短期memory

## 长期memory

# Prompt Enhancer

# RAG
