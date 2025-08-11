---
layout: mypost
title: LangChain 01：Runnable
categories: [LangChain, Framework, Agent]
---


- Langchain langgraph的agent总结

基本正确，但有两点细化下就更准了：

- **LangChain v0.3**
    - `create_tool_calling_agent`（以及同类 helper）⇒ 产出的是一个 *agent runnable*：它“决定用哪个 tool 以及参数是什么”（通过工具的 schema 让模型生成 `tool_calls`/`AgentAction`）。它**本身不执行**工具。([LangChain](https://python.langchain.com/api_reference/langchain/agents/langchain.agents.tool_calling_agent.base.create_tool_calling_agent.html?utm_source=chatgpt.com))
    - `AgentExecutor` ⇒ 把“agent + tools”包起来跑循环：读取 agent 的 tool call/参数 → 执行对应 Python 工具 → 把工具输出塞回 scratchpad → 直到 agent 返回 `AgentFinish` 为止。也就是你说的“负责执行 tools 并管理迭代”。([LangChain](https://python.langchain.com/api_reference/langchain/agents/langchain.agents.agent.AgentExecutor.html?utm_source=chatgpt.com))
- **LangGraph**
    - `create_react_agent`（LangGraph 里的预置版）⇒ 直接构建一个**会自己“选工具+执行工具”的图**：图里既有“让模型产出 `tool_calls` 的节点”，也有真正**执行工具**的 `ToolNode`；它会在图内循环直到满足停止条件。所以它**覆盖了从参数生成到执行的全过程**（你这点理解是对的）。([LangChain AI](https://langchain-ai.github.io/langgraph/reference/agents/?utm_source=chatgpt.com))

> 小结
> 
> - LangChain：**决策（agent）** 与 **执行（AgentExecutor）** 是分开的。([LangChain](https://python.langchain.com/api_reference/langchain/agents/langchain.agents.tool_calling_agent.base.create_tool_calling_agent.html?utm_source=chatgpt.com))
> - LangGraph：预置的 `create_react_agent` **在一个图里集成了二者**（决策 + 执行 + 循环控制）。([LangChain AI](https://langchain-ai.github.io/langgraph/reference/agents/?utm_source=chatgpt.com))

（补充：LangChain 自己的 `create_react_agent` 现在也建议用 **LangGraph** 的实现替代做生产场景。([LangChain](https://python.langchain.com/api_reference/langchain/agents/langchain.agents.react.agent.create_react_agent.html?utm_source=chatgpt.com))）
