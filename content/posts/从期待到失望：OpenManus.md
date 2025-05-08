---
title: "从期待到失望：OpenManus"
date: 2025-05-08T11:27:36+08:00
draft: false
categories:
  - 项目分析
---

前面一段时间，Manus这个AI工具火得一塌糊涂，但问题是它得靠邀请码才能用，这可把不少想尝鲜的朋友给急坏了。不过别担心，看看下面这个项目。

# 项目简介

根据`OpenManus`项目的介绍，`OpenManus`是2位核心作者和来自`MetaGPT`团队的3位成员用了3个小时完成了第一版的开发。

在`Manus`的无脑宣传下，很多人包括我在内对`Manus`充满了期待。在没有`Manus`邀请码的情况下，这个期待自然就转移到`OpenManus`上来了。

# 实战体验

使用`OpenManus`只需要配置`config/config.toml`中的`llm`、`browser`、`search`就能使用了。

```toml
# Global LLM configuration
[llm] 
api_type = 'Openai'
model = ""                         # The LLM model to use
base_url = ""                      # API endpoint URL
api_key = ""                       # Your API key
max_tokens = 4096                  # Maximum number of tokens in the response
temperature = 0.0                  # Controls randomness

[browser]
# Whether to run browser in headless mode (default: false)
headless = false
# Disable browser security features (default: true)
disable_security = true

# Optional configuration, Proxy settings for the browser
[browser.proxy]
server = "http://127.0.0.1:7890"
username = ""
password = ""

# Optional configuration, Search settings.
[search]
# Search engine for agent to use. Default is "Google", can be set to "Baidu" or "DuckDuckGo" or "Bing".
engine = "Google"
# Language code for search results. Options: "en" (English), "zh" (Chinese), etc.
lang = "zh"
# Country code for search results. Options: "us" (United States), "cn" (China), etc.
country = "cn"
```

启动运行

```
uv run main.py
```

输入我的prompt是: `2025年5月10日到5月11日，从成都市区出发，输出一份成都周边游计划，最后结果保存位markdown格式`，下面是`OpenManus`的输出

```
INFO     [browser_use] BrowserUse logging setup complete with level info
INFO     [root] Anonymized telemetry enabled. See https://docs.browser-use.com/development/telemetry for more information.
2025-05-08 16:46:06.393 | INFO     | app.agent.manus:initialize_mcp_servers:71 - Connected to MCP server server1 at http://localhost:8000/sse
Enter your prompt: 2025年5月10日到5月11日，从成都市区出发，输出一份成都周边游计划，最后结果保存位markdown格式
2025-05-08 16:46:07.786 | INFO     | app.agent.base:update_memory:73 - Adding message to memory: user 2025年5月10日到5月11日，从成都市区出发，输出一份成都周边游计划，最后结果保存位markdown格式
2025-05-08 16:46:07.786 | INFO     | app.agent.base:run:104 - Executing step 1/20
2025-05-08 16:46:24.971 | INFO     | app.llm:update_token_count:56 - Token usage: Input=1916, Completion=491, Cumulative Input=1916, Cumulative Completion=491, Total=2407, Cumulative Total=2407
2025-05-08 16:46:24.972 | INFO     | app.agent.toolcall:think:93 - ✨ Manus's thoughts: 


2025-05-08 16:46:24.972 | INFO     | app.agent.toolcall:think:94 - 🛠️ Manus selected 1 tools to use
2025-05-08 16:46:24.972 | INFO     | app.agent.toolcall:think:98 - 🧰 Tools being prepared: ['browser_use']
2025-05-08 16:46:24.972 | INFO     | app.agent.toolcall:think:101 - 🔧 Tool arguments:  {"action": "web_search", "query": "成都周边两日游推荐 2025"}
2025-05-08 16:46:24.972 | INFO     | app.agent.toolcall:execute_tool:169 - 🔧 Activating tool: 'browser_use'...
2025-05-08 16:46:27.087 | INFO     | app.tool.web_search:_try_all_engines:245 - 🔎 Attempting search with Google...
2025-05-08 16:46:58.936 | INFO     | app.agent.toolcall:act:140 - 🎯 Tool 'browser_use' completed its mission! Result: Observed output of cmd `browser_use` executed:
Error: Browser action 'web_search' failed: Page.goto: Timeout 30000ms exceeded.
Call log:
  - navigating to "https://hk.trip.com/travel-guide/destination/chengdu-104/", waiting until "load"

2025-05-08 16:46:58.937 | INFO     | app.agent.base:run:104 - Executing step 2/20
2025-05-08 16:47:38.161 | INFO     | app.llm:update_token_count:56 - Token usage: Input=2110, Completion=709, Cumulative Input=4026, Cumulative Completion=1200, Total=2819, Cumulative Total=5226
2025-05-08 16:47:38.161 | INFO     | app.agent.toolcall:think:93 - ✨ Manus's thoughts: 


2025-05-08 16:47:38.161 | INFO     | app.agent.toolcall:think:94 - 🛠️ Manus selected 1 tools to use
2025-05-08 16:47:38.161 | INFO     | app.agent.toolcall:think:98 - 🧰 Tools being prepared: ['browser_use']
2025-05-08 16:47:38.162 | INFO     | app.agent.toolcall:think:101 - 🔧 Tool arguments:  {"action": "go_to_url", "url": "https://www.ctrip.com"}
2025-05-08 16:47:38.162 | INFO     | app.agent.toolcall:execute_tool:169 - 🔧 Activating tool: 'browser_use'...
2025-05-08 16:47:42.261 | INFO     | app.agent.toolcall:act:140 - 🎯 Tool 'browser_use' completed its mission! Result: Observed output of cmd `browser_use` executed:
Navigated to https://www.ctrip.com
2025-05-08 16:47:42.262 | INFO     | app.agent.base:run:104 - Executing step 3/20
2025-05-08 16:48:00.168 | INFO     | app.llm:update_token_count:56 - Token usage: Input=2256, Completion=494, Cumulative Input=6282, Cumulative Completion=1694, Total=2750, Cumulative Total=7976
2025-05-08 16:48:00.168 | INFO     | app.agent.toolcall:think:93 - ✨ Manus's thoughts: 


2025-05-08 16:48:00.169 | INFO     | app.agent.toolcall:think:94 - 🛠️ Manus selected 1 tools to use
2025-05-08 16:48:00.169 | INFO     | app.agent.toolcall:think:98 - 🧰 Tools being prepared: ['browser_use']
2025-05-08 16:48:00.169 | INFO     | app.agent.toolcall:think:101 - 🔧 Tool arguments:  {"action": "web_search", "query": "成都周边两日游推荐 2025"}
2025-05-08 16:48:00.169 | INFO     | app.agent.toolcall:execute_tool:169 - 🔧 Activating tool: 'browser_use'...
2025-05-08 16:48:00.169 | INFO     | app.tool.web_search:_try_all_engines:245 - 🔎 Attempting search with Google...
2025-05-08 16:48:06.630 | INFO     | app.agent.toolcall:act:140 - 🎯 Tool 'browser_use' completed its mission! Result: Observed output of cmd `browser_use` executed:
Search results for '成都周边两日游推荐 2025':

1. 成都自由行旅遊攻略（2025）：附近熱門好去處、人氣景點、行程規劃
   URL: https://hk.trip.com/travel-guide/destination/chengdu-104/
   Description:  想探索成都但不知從何入手？Trip.com 全面的旅遊指南可以助您探索成都。輕鬆搜尋附近餐廳、景點和娛樂，為您提供詳細的資訊和準確位置。另外更可以獲取即時交通資訊 ... 
   Content: 成都自由行旅遊攻略（2025）：附近熱門好去處、人氣景點、行程規劃、住宿、美食、交通、天氣等等——重要資訊一文睇清（更新於2025年5月）｜Trip.com https://hk.trip.com/travel-guide/destination/chengdu-104/  App 聯絡我們 HKD 搜尋訂單 住宿 機票 高鐵 / 火車票 租車／接送 租車 機場接送 門票 / 體驗 門票 / 體驗 eSIM & SIM 機票+酒店 旅遊攻略 旅遊地圖 禮品卡 旅程策劃師 郵輪 保險 Trip.com 獎賞計劃 優惠 聯絡我們 客戶服務支援 服務保障 更多服務資訊 關於 關於 Trip.com 傳媒中心 加入我們 條款及細則 私隱條款 關於 Trip.com Group 其他服務 投資者關係 Trip.com 獎賞 分銷夥伴合作計劃 刊登我的住宿 全部酒店 安全性 付款方式 我們的合作夥伴 Copyright © 2025 Trip.com Travel Singapore Pte. Ltd. 版權所有。 網站營運者：Trip.com Travel Singapore Pte. Ltd. 由Ctrip.com (Hong Kong) Limited持有香港旅行代理商牌照編號 352367
2025-05-08 16:48:06.630 | INFO     | app.agent.base:run:104 - Executing step 4/20
2025-05-08 16:48:46.256 | INFO     | app.llm:update_token_count:56 - Token usage: Input=2866, Completion=477, Cumulative Input=9148, Cumulative Completion=2171, Total=3343, Cumulative Total=11319
2025-05-08 16:48:46.256 | INFO     | app.agent.toolcall:think:93 - ✨ Manus's thoughts: 


2025-05-08 16:48:46.256 | INFO     | app.agent.toolcall:think:94 - 🛠️ Manus selected 1 tools to use
2025-05-08 16:48:46.256 | INFO     | app.agent.toolcall:think:98 - 🧰 Tools being prepared: ['browser_use']
2025-05-08 16:48:46.257 | INFO     | app.agent.toolcall:think:101 - 🔧 Tool arguments:  {"action": "click_element", "index": 1}
2025-05-08 16:48:46.257 | INFO     | app.agent.toolcall:execute_tool:169 - 🔧 Activating tool: 'browser_use'...
2025-05-08 16:48:46.257 | INFO     | app.agent.toolcall:act:140 - 🎯 Tool 'browser_use' completed its mission! Result: Observed output of cmd `browser_use` executed:
Error: Browser action 'click_element' failed: 1
2025-05-08 16:48:46.257 | INFO     | app.agent.base:run:104 - Executing step 5/20
2025-05-08 16:49:31.367 | INFO     | app.llm:update_token_count:56 - Token usage: Input=3009, Completion=426, Cumulative Input=12157, Cumulative Completion=2597, Total=3435, Cumulative Total=14754
2025-05-08 16:49:31.367 | INFO     | app.agent.toolcall:think:93 - ✨ Manus's thoughts: 


2025-05-08 16:49:31.367 | INFO     | app.agent.toolcall:think:94 - 🛠️ Manus selected 1 tools to use
2025-05-08 16:49:31.367 | INFO     | app.agent.toolcall:think:98 - 🧰 Tools being prepared: ['browser_use']
2025-05-08 16:49:31.368 | INFO     | app.agent.toolcall:think:101 - 🔧 Tool arguments:  {"action": "go_to_url", "url": "https://hk.trip.com/travel-guide/destination/chengdu-104/"}
2025-05-08 16:49:31.368 | INFO     | app.agent.toolcall:execute_tool:169 - 🔧 Activating tool: 'browser_use'...
2025-05-08 16:49:35.645 | INFO     | app.agent.toolcall:act:140 - 🎯 Tool 'browser_use' completed its mission! Result: Observed output of cmd `browser_use` executed:
Navigated to https://hk.trip.com/travel-guide/destination/chengdu-104/
2025-05-08 16:49:35.645 | INFO     | app.agent.base:run:104 - Executing step 6/20
2025-05-08 16:50:45.262 | INFO     | app.llm:update_token_count:56 - Token usage: Input=3178, Completion=481, Cumulative Input=15335, Cumulative Completion=3078, Total=3659, Cumulative Total=18413
2025-05-08 16:50:45.262 | INFO     | app.agent.toolcall:think:93 - ✨ Manus's thoughts: 


2025-05-08 16:50:45.262 | INFO     | app.agent.toolcall:think:94 - 🛠️ Manus selected 1 tools to use
2025-05-08 16:50:45.262 | INFO     | app.agent.toolcall:think:98 - 🧰 Tools being prepared: ['browser_use']
2025-05-08 16:50:45.262 | INFO     | app.agent.toolcall:think:101 - 🔧 Tool arguments:  {"action": "extract_content", "goal": "提取成都周边两日游推荐行程信息，包括景点、美食、交通建议，输出结构化数据"}
2025-05-08 16:50:45.262 | INFO     | app.agent.toolcall:execute_tool:169 - 🔧 Activating tool: 'browser_use'...
2025-05-08 16:50:45.855 | ERROR    | app.llm:ask_tool:138 - Unexpected error in ask_tool: Error code: 400 - {'code': 20015, 'message': 'Value error, The `required` option for tool_choice is not yet supported.', 'data': None}
2025-05-08 16:50:45.855 | INFO     | app.agent.toolcall:act:140 - 🎯 Tool 'browser_use' completed its mission! Result: Observed output of cmd `browser_use` executed:
Error: Browser action 'extract_content' failed: Error code: 400 - {'code': 20015, 'message': 'Value error, The `required` option for tool_choice is not yet supported.', 'data': None}
2025-05-08 16:50:45.855 | INFO     | app.agent.base:run:104 - Executing step 7/20
2025-05-08 16:55:44.972 | INFO     | app.llm:update_token_count:56 - Token usage: Input=3382, Completion=1264, Cumulative Input=18717, Cumulative Completion=4342, Total=4646, Cumulative Total=23059
2025-05-08 16:55:44.973 | INFO     | app.agent.toolcall:think:93 - ✨ Manus's thoughts: 


2025-05-08 16:55:44.973 | INFO     | app.agent.toolcall:think:94 - 🛠️ Manus selected 1 tools to use
2025-05-08 16:55:44.973 | INFO     | app.agent.toolcall:think:98 - 🧰 Tools being prepared: ['str_replace_editor']
2025-05-08 16:55:44.973 | INFO     | app.agent.toolcall:think:101 - 🔧 Tool arguments:  {"command": "create", "path": "/Users/zengqiang/codespace/everest/OpenManus/workspace/成都周边游计划.md", "file_text": "```markdown\n# 成都周边两日游计划（2025年5月10日-11日）\n\n## 行程概览\n- **日期**：2025年5月10日-11日\n- **出发地**：成都市区\n- **主题**：自然风光+文化体验\n\n## 详细行程\n\n### 第一天：九寨沟一日游\n- **上午**：成都市区出发，乘坐高铁前往九寨沟（约3.5小时）\n- **下午**：游览五花海、长海等核心景点（建议游览5小时）\n- **晚上**：入住九寨沟景区附近酒店，体验藏式美食\n\n### 第二天：乐山大佛+峨眉山\n- **上午**：前往乐山大佛景区，参观凌云寺、佛像（约2小时）\n- **下午**：乘景区车前往峨眉山，登金顶观日出（需提前预约）\n- **晚上**：返回成都市区，品尝地道川菜\n\n## 交通建议\n- **高铁**：成都东站至九寨沟站（约400元/人）\n- **当地交通**：景区观光车+包车（九寨沟至黄龙景区可选）\n\n## 美食推荐\n- **九寨沟**：藏族糌粑、酥油茶\n- **乐山**：跷脚牛肉、甜皮鸭\n- **成都**：麻辣烫、钟水饺\n\n## 注意事项\n- 5月气候多变，需备防风外套\n- 九寨沟景区需提前3天预约门票\n- 峨眉山金顶需凌晨起床，建议携带手电筒\n\n```"}
2025-05-08 16:55:44.973 | INFO     | app.agent.toolcall:execute_tool:169 - 🔧 Activating tool: 'str_replace_editor'...
2025-05-08 16:55:44.975 | INFO     | app.agent.toolcall:act:140 - 🎯 Tool 'str_replace_editor' completed its mission! Result: Observed output of cmd `str_replace_editor` executed:
File created successfully at: /Users/zengqiang/codespace/everest/OpenManus/workspace/成都周边游计划.md
2025-05-08 16:55:44.975 | INFO     | app.agent.base:run:104 - Executing step 8/20
2025-05-08 16:56:35.605 | INFO     | app.llm:update_token_count:56 - Token usage: Input=3964, Completion=322, Cumulative Input=22681, Cumulative Completion=4664, Total=4286, Cumulative Total=27345
2025-05-08 16:56:35.605 | INFO     | app.agent.toolcall:think:93 - ✨ Manus's thoughts: 

旅游计划文档已成功保存至路径：`/Users/zengqiang/codespace/everest/OpenManus/workspace/成都周边游计划.md`。文件内容包含：

```markdown
# 成都周边两日游计划（2025年5月10日-11日）
...
```

是否需要调整行程细节（如更换景点、优化交通方式等）？或需要我协助完成其他任务？
2025-05-08 16:56:35.605 | INFO     | app.agent.toolcall:think:94 - 🛠️ Manus selected 0 tools to use
2025-05-08 16:56:35.605 | INFO     | app.agent.base:run:104 - Executing step 9/20
2025-05-08 16:56:43.556 | INFO     | app.llm:update_token_count:56 - Token usage: Input=4140, Completion=451, Cumulative Input=26821, Cumulative Completion=5115, Total=4591, Cumulative Total=31936
2025-05-08 16:56:43.556 | INFO     | app.agent.toolcall:think:93 - ✨ Manus's thoughts: 

旅游计划文档已成功保存至路径：`/Users/zengqiang/codespace/everest/OpenManus/workspace/成都周边游计划.md`。文件内容包含：

```markdown
# 成都周边两日游计划（2025年5月10日-11日）
...
```

是否需要调整行程细节（如更换景点、优化交通方式等）？或需要我协助完成其他任务？
2025-05-08 16:56:43.556 | INFO     | app.agent.toolcall:think:94 - 🛠️ Manus selected 0 tools to use
2025-05-08 16:56:43.556 | INFO     | app.agent.base:run:104 - Executing step 10/20
2025-05-08 16:56:51.487 | INFO     | app.llm:update_token_count:56 - Token usage: Input=4316, Completion=427, Cumulative Input=31137, Cumulative Completion=5542, Total=4743, Cumulative Total=36679
2025-05-08 16:56:51.487 | INFO     | app.agent.toolcall:think:93 - ✨ Manus's thoughts: 


2025-05-08 16:56:51.487 | INFO     | app.agent.toolcall:think:94 - 🛠️ Manus selected 1 tools to use
2025-05-08 16:56:51.487 | INFO     | app.agent.toolcall:think:98 - 🧰 Tools being prepared: ['terminate']
2025-05-08 16:56:51.487 | INFO     | app.agent.toolcall:think:101 - 🔧 Tool arguments:  {"status": "success"}
2025-05-08 16:56:51.487 | INFO     | app.agent.toolcall:execute_tool:169 - 🔧 Activating tool: 'terminate'...
2025-05-08 16:56:51.487 | INFO     | app.agent.toolcall:act:140 - 🎯 Tool 'terminate' completed its mission! Result: Observed output of cmd `terminate` executed:
The interaction has been completed with status: success
```

这个是`OpenManus`给我输出的markdown文件内通

```markdown
# 成都周边两日游计划（2025年5月10日-11日）

## 行程概览
- **日期**：2025年5月10日-11日
- **出发地**：成都市区
- **主题**：自然风光+文化体验

## 详细行程

### 第一天：九寨沟一日游
- **上午**：成都市区出发，乘坐高铁前往九寨沟（约3.5小时）
- **下午**：游览五花海、长海等核心景点（建议游览5小时）
- **晚上**：入住九寨沟景区附近酒店，体验藏式美食

### 第二天：乐山大佛+峨眉山
- **上午**：前往乐山大佛景区，参观凌云寺、佛像（约2小时）
- **下午**：乘景区车前往峨眉山，登金顶观日出（需提前预约）
- **晚上**：返回成都市区，品尝地道川菜

## 交通建议
- **高铁**：成都东站至九寨沟站（约400元/人）
- **当地交通**：景区观光车+包车（九寨沟至黄龙景区可选）

## 美食推荐
- **九寨沟**：藏族糌粑、酥油茶
- **乐山**：跷脚牛肉、甜皮鸭
- **成都**：麻辣烫、钟水饺

## 注意事项
- 5月气候多变，需备防风外套
- 九寨沟景区需提前3天预约门票
- 峨眉山金顶需凌晨起床，建议携带手电筒
```

# 项目分析

最后来看看`OpenManus`是怎么实现的，首先使用`deepwiki`分析一下：https://deepwiki.com/mannaandpoem/OpenManus

总体来看，`OpenManus`按照`ReAct`的思想来实现。提供给llm一些prompt和tool，让给llm决策是否使用tool，然后调用tool，将tool返回的继续添加到对话中，如此循环

```python
class ReActAgent(BaseAgent, ABC):
  @abstractmethod
  async def think(self) -> bool:
      """Process current state and decide next action"""

  @abstractmethod
  async def act(self) -> str:
      """Execute decided actions"""

  async def step(self) -> str:
      """Execute a single step: think and act."""
      should_act = await self.think()
      if not should_act:
          return "Thinking complete - no action needed"
      return await self.act()
```

再来看看`OpenManus`内置了哪些工具

* ask_human.py: 需要人类提供更多信息的时候，会让人再进行输入
* browser_use_tool.py: 使用浏览器的工具
* file_operators.py：操作文件的工具
* python_execute.py：执行代码的工具
* search：各种搜索引擎的搜索实现
* str_replace_editor.py：内部使用了file_operators.py完成文件的操作
* terminate.py：终止
* web_search.py：使用搜索引擎的能力，内部使用search包中的各种搜索引擎实现

实现`ReAct`的模式主要是 toolcall.py

```python
async def think(self) -> bool:
    if self.next_step_prompt:
            user_msg = Message.user_message(self.next_step_prompt)
            self.messages += [user_msg]
    
    response = await self.llm.ask_tool(
        messages=self.messages,
        system_msgs=(
            [Message.system_message(self.system_prompt)]
            if self.system_prompt
            else None
        ),
        tools=self.available_tools.to_params(),
        tool_choice=self.tool_choices,
    )

    self.tool_calls = tool_calls = (
        response.tool_calls if response and response.tool_calls else []
    )


    assistant_msg = (
        Message.from_tool_calls(
            content=content,
            tool_calls=self.tool_calls,
        )
        if self.tool_calls
        else Message.assistant_message(content)
    )
    self.memory.add_message(assistant_msg)
```

```python
async def act(self) -> str:
    for command in self.tool_calls:
        result = await self.execute_tool(command)
        # Add tool response to memory
        tool_msg = Message.tool_message(
            content=result,
            tool_call_id=command.id,
            name=command.function.name,
        )
        self.memory.add_message(tool_msg)
        results.append(result)
```

最后再看一下`OpenManus`的提示词

```python
SYSTEM_PROMPT = (
    "You are OpenManus, an all-capable AI assistant, aimed at solving any task presented by the user. You have various tools at your disposal that you can call upon to efficiently complete complex requests. Whether it's programming, information retrieval, file processing, web browsing, or human interaction (only for extreme cases), you can handle it all."
    "The initial directory is: {directory}"
)

NEXT_STEP_PROMPT = """Based on user needs, proactively select the most appropriate tool or combination of tools. For complex tasks, you can break down the problem and use different tools step by step to solve it. After using each tool, clearly explain the execution results and suggest the next steps.

If you want to stop the interaction at any point, use the `terminate` tool/function call.
"""
```

到这里基本上`OpenManus`就大概了解了。当然`OpenManus`肯定也不止这些，其还支持`MCP`、目前不稳定的`workflow`等。我就不深入研究了，思想就是上面介绍的`ReAct`。


# 小结

* `OpenManus`运行游很多不确定性，有时可能成功，有时失败，但是失败占大多数。成功的时候，输出的内容也和预期相差很远。
* `OpenManus`项目有很多的Issue，而且开发者看Issue的回复和处理效率也不是很高，可能这都是大部分开源项目的通病吧，最后可能也是一堆shit.
* `OpenManus`可以作为玩具来试试，但是用于实际场景可能还有点差距。不过可以通过`OpenManus`学习一些agent开发思想也不错

一些Issue

* [issues#965](https://github.com/FoundationAgents/OpenManus/issues/965)
* [issues#1028](https://github.com/FoundationAgents/OpenManus/issues/1028)
* [issues#1074](https://github.com/FoundationAgents/OpenManus/issues/1074)
* [issues#1115](https://github.com/FoundationAgents/OpenManus/issues/1115)
