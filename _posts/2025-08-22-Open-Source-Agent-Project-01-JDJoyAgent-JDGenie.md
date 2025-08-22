---
layout: mypost
title: Open-Source Agent Project 01-JDJoyAgent-JDGenie
categories: [Open-Source Project]
date: 2025-08-22 00:05:00 +0000
---

# 后端整体架构

- `genie-backend\src\main\java\com\jd\genie\config\GenieController`
- 从`GenieController.java`的AutoAgent函数开始，创建`AgentContex`t，包括request id、tool list、agent type等内容，然后执行handler函数

### 怎么路由到具体Agent？

- 严格说只有两个Agent，分别是
    - Plan & Execute Agent，即开启了DeepThink的链路
    - ReAct Agent，即未开启DeepThink的链路
- 前端是否点击DeepThink按钮，决定了`agentContext`里的`agentType`，传入后端进行路由
- `GenieController AutoAgent`函数执行了getHandler函数
    
    ```python
     AgentHandlerService handler = agentHandlerFactory.getHandler(agentContext, request);
    ```
    
- 然后`AgentHandlerFactory`通过`handlerMap`里所有handler执行`support()`函数来进行路由
    
    ```python
    public AgentHandlerService getHandler(AgentContext context, AgentRequest request) {
            if (Objects.isNull(context) || Objects.isNull(request)) {
                return null;
            }
    
            // 方法1：通过supports方法匹配
            for (AgentHandlerService handler : handlerMap.values()) {
                if (handler.support(context, request)) {
                    return handler;
                }
            }
    
            return null;
        }
    ```
    
- handler的support函数具体是看agentContext（由前端是否点击DeepThink按钮决定）里的agentType（int型数字）是否等于当前agent的agentType值
    
    ```python
    @Override
        public Boolean support(AgentContext agentContext, AgentRequest request) {
            return AgentType.PLAN_SOLVE.getValue().equals(request.getAgentType());
        }
    }
    ```
    
    ```python
    public enum AgentType {
        COMPREHENSIVE(1),
        WORKFLOW(2),
        PLAN_SOLVE(3),
        ROUTER(4),
        REACT(5);
    ```
