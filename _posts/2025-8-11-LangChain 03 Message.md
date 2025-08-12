---
layout: mypost
title: LangChain 03：Message解析
categories: [LangChain, Framework, Agent]
---
## Table of Contents
- [Message](#message)
- [ChatPromptTemplate](#chatprompttemplate)
- [使用方法](#使用方法)
- [注意事项](#注意事项)

# Message

* message的核心作用：字符串str到传递给的LLM信息的转换
* 什么时候使用System Message 什么时候使用 HumanMessage？
* Message的底层，最终怎么结合role给llm？

# Prompt
## ChatPromptTemplate


# 花括号解析问题

# 原来的（会触发模板解析）
executor_prompt_tpl = ChatPromptTemplate.from_messages([
    ("system", executor_prompt.format(past_steps=past_steps_str, current_step=current_plan_str)),
    ("placeholder", "{messages}")
])
# 花括号解析报错

```python
LangChain 03：Message解析
```
