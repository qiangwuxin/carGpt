# cargpt

- chatbot
  组件、tailwindcss messages
  ai streaming 流式输出 复杂 封装？
  大模型
- 专业领域的chatbot
  RAG 汽车知识库 检索增强生成
  - 知识库（爬虫）
  - 向量数据库 supabase 

## 项目中用到的技术

- RAG 检索增强生成
  - embedding openai embed 向量化 
  - 相似度 cos -> 1 倒序排序
  - 存到spabase数据库 
### package.json
- ai sdk
   build AI-powered applications
   封装LLM的调用
   @ai-sdk/openai
   "@ai-sdk/openai":"^1.3.22",  调用LLM
    "@ai-sdk/react":"1.2.12" hooks api 式一行完成流式输出

- supabase 
  BASS Backend as Service  
  supabase是Postgres 的一个云端数据库 支持 向量数据库

- langchain
  LangChain 是一个用于构建基于大语言模型（LLM）应用的开源框架，提供模块化组件来连接 LLM 与外部数据、工具和记忆，支持链式调用、代理、检索增强生成（RAG）等高级功能。
   "@langchain/community" 社区提供的工具（爬虫）
    "@langchain/core" 核心模块
- dotenv
- puppeteer 无头浏览器
  Puppeteer 是一个由 Google 开发的 Node.js 库，提供高级 API 来通过 DevTools 协议控制无头（或有头）Chrome 或 Chromium 浏览器，常用于网页自动化、爬虫、截图、PDF 生成和端到端测试。
- lucide-react
  Lucide React 是一个轻量、开源的 React 图标库，提供简洁一致的 SVG 图标组件，是 Feather Icons 的社区维护分支，支持 Tree-shaking 和 TypeScript。
- react-markdown 是一个react组件，用于渲染markdown文本


## Next.js
- layout metadata 修改
  这有关于SEO
- "use client" 是 Next.js 13+ 中的指令，用于标记一个 React 组件为客户端组件，使其能在浏览器中使用 useState、useEffect、事件处理等交互逻辑（默认组件是服务端渲染的）。

## tailwindcss 
- max-w-3xl 
  响应式的技巧
  48rem (适配所有设备) 3xl 768px ipad 竖着拿的尺寸 
  移动设备 （phone,pad） width=100%=100vh
  PC端 768px,mx-auto
  Mobile First 移动设备优先
- 在 Tailwind CSS 中，方括号 [] 用于定义任意值类（arbitrary values），允许你临时使用未内置的自定义值，例如 max-h-[80vh] 表示“最大高度为 80% 视口高度”。

- @ai-sdk/react
  hooks 封装chatLLM的功能,方便流式输出。

## typeScript 
- 组件的props 类型定义

## 前端部分的亮点
- @ai-sdk/react 对chatBot 响应式业务的封装 一行代码完成流式输出
  useChat hook 
- raect-markdown ai响应 markdown是主要的格式
  # - 1 【】 （） 解析
- tailwindcss 适配
- react组件划分和ts的类型约束
  shadcn 按需加载、定制性强
- lucide-react 图标库
- useChat 对hooks的理解 响应式业务的封装，一般函数封装的区别
- promprt 模版设计
  - 准确
  - 复用
  - 格式
    - 身份
    - 任务
    - 分区 context,和 question 
  - 返回格式
  - 约束 不回答手机之外的内容
  - 接受一个参数，函数返回，我们的应用就由几个核心的promptTemplate 构成，用心设计

## 后端亮点
- ai streamText 流式输出
- result.toDataStreamResponse（）将 streamText 生成的流式结果转换为一个可被前端消费的 Response 对象，支持以数据流形式传输 AI 输出，实现逐字显示等实时效果。
- 爬虫脚本
  - seed 脚本任务
    npm run seed
    填充知识库 
  - seed.ts 编写这个脚本
    ts文件不可以直接运行
    ts-node + typescript 可以直接运行
    先解析成js，再运行。
- langchain Agent 开发框架
  coze  promptTemplate 记忆MessageMemory Community 
- 正则html替换
- vercel 的AI版图
   - next.js
   - ai-sdk
   - js 的云端运行环境
   - v0 bolt 
     ai-sdk/react 流式输出 -> prompt  -> emdedding 
     网页(wikipidia) -> langchain/community+puppeteer（爬取） ->
     langchain提供的分块机制(chunks? 段落) -> embedding-> supabase 存储 
     -> supabase
- 向量存储
 CREATE TABLE public.chunks (
    id uuid NOT NULL DEFAULT gen_random_uuid(),
    content text null,
    vector extensions.vector null,
    url text null,
    date_updated timestamp without time zone DEFAULT now(),
    CONSTRAINT chunks_pkey PRIMARY KEY (id)
  );
## 遇到的问题
- ai-sdk检索的时候， LLM给了老版本的代码，导致调试出了些问题，mcp解决问题
- ts-node 编译时不支持esm，
   tsconfig.json ts 配置文件
   支持ts-node 编译为commonjs
- rpc 调用
  在supabase 数据库中调用函数 
  ```sql
  create or replace function get_relevant_chunks(
  -- 一个长度为 1536 的“向量”
  query_vector vector(1536),
  -- 只找“相似度”超过这个值的结果
  match_threshold float,
  -- 最多返回多少条结果。
  match_count int
)
returns table (
  id uuid,
  content text,
  url text,
  date_updated timestamp,
  similarity float
)
-- 这个函数执行完后，会返回一个“表格形式”的结果。
language sql stable
-- 说明这个函数是用 SQL 语言写的，并且是“稳定的”
-- 函数内容开始。
as $$
  select
    id,
    content,
    url,
    date_updated,
    -- chunks.vector <=> query_vector 是 pgvector 扩展提供的“距离”计算
    1 - (chunks.vector <=> query_vector) as similarity
  from chunks
  where 1 - (chunks.vector <=> query_vector) > match_threshold
  order by similarity desc
  limit match_count;
  -- 函数内容结束。
$$;
  ```
- 向量的相似度计算
   - mysql 不支持，postgresql 支持，
     <=> 举例计算
   - 1 - >
   - 数据库支持函数
    传参
    指定返回的内容
    构建sql 