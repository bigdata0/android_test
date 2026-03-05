```mermaid
graph TD
    %% 样式定义
    classDef user fill:#f9d0c4,stroke:#333,stroke-width:2px;
    classDef mainAgent fill:#bbdefb,stroke:#333,stroke-width:2px;
    classDef router fill:#ffe082,stroke:#333,stroke-width:2px;
    classDef memory fill:#ce93d8,stroke:#333,stroke-width:2px;
    classDef db fill:#c8e6c9,stroke:#333,stroke-width:2px;
    classDef coder fill:#ffccbc,stroke:#333,stroke-width:2px;

    USER((👨‍💻 用户)) ::: user

    subgraph 前台总管体系 (Main Conversational Hub)
        MAIN_AGENT["🗣️ 前台记忆管家<br>(Main Agent)"] ::: mainAgent
        ROUTER{"🔀 隐形内部调度器<br>(Router/Tool Call)"} ::: router
        MEM_TOOLS["🛠️ 记忆与轻量工具<br>(MySQL检索/基础查询)"] ::: memory
        EXTRACTOR["📝 记忆提取器<br>(Extractor Node)"] ::: memory
    end

    subgraph 后台代码车间 (Code Factory Sub-Graph)
        PLANNER["🧠 Planner<br>(生成PRD与调度)"] ::: coder
        CODER["💻 Coder<br>(XML代码生成与探路)"] ::: coder
        EXECUTOR["⚙️ Executor<br>(沙盒执行与日志提取)"] ::: coder
    end

    DB[(MySQL 数据库<br>topic & memory)] ::: db
    WORKSPACE[(本地沙盒<br>Workspace)] ::: db

    %% 核心交互流转
    USER <-->|唯一对话入口<br>自然语言交互| MAIN_AGENT
    MAIN_AGENT -->|意图识别| ROUTER
    
    %% 分支 1：日常对话与考据 (走轻量工具)
    ROUTER -->|需要回忆或剧情考据| MEM_TOOLS
    MEM_TOOLS <-->|查询/写入| DB
    MEM_TOOLS -->|返回上下文| MAIN_AGENT
    
    %% 分支 2：重度工程任务 (走后台车间)
    ROUTER -->|代码开发/多文件工程| PLANNER
    PLANNER -->|下发任务| CODER
    CODER <-->|增删改查文件| WORKSPACE
    CODER -->|提交运行| EXECUTOR
    EXECUTOR -->|报错打回| PLANNER
    EXECUTOR -->|运行成功 (Exit 0) / 熔断| MAIN_AGENT
    
    %% 记忆沉淀机制 (静默运行)
    MAIN_AGENT -.->|触发 N 轮对话阈值| EXTRACTOR
    EXTRACTOR -.->|提炼新记忆入库| DB
```
