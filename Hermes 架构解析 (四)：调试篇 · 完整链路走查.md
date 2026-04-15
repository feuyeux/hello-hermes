# Hermes 架构解析 (四)：调试篇 · 完整链路走查 (v2026.4.8)

示例命令：

```bash
python hermes_cli/main.py chat --quiet -q "Summarize the repository structure in 5 bullets"
```

## 说明

> 这个命令**不会进入** `cli.py::HermesCLI::chat`，而是直接走 `cli.py::<module>::main` 的 quiet one-shot 分支，再直接调用 `run_agent.py::AIAgent::run_conversation`

- 格式统一为：`文件名::类名::方法名 方法的作用 ...`
- 无类方法统一写成 `::<module>::`
- 只给和 `状态 / 上下文 / 会话 / 记忆` 直接相关的方法打标签
- 本文以这个**具体命令**的真实路径为主，条件分支会单独标注 `条件：...`

## 1. 启动链

1. `hermes_cli/main.py::<module>::_apply_profile_override` 预解析 `--profile/-p` 并在其他模块导入前设置 `HERMES_HOME` 【**状态**】
2. `hermes_constants.py::<module>::get_default_hermes_root` 解析 Hermes 默认根目录；仅当命令行未显式传 profile 时读取 `active_profile` 使用
3. `hermes_cli/profiles.py::<module>::resolve_profile_env` 把 profile 名解析为具体的 `HERMES_HOME` 路径 【**状态**】
4. `hermes_cli/env_loader.py::<module>::load_hermes_dotenv` 先加载 `~/.hermes/.env`，再加载项目 `.env`，刷新运行时环境变量 【**状态**】
5. `hermes_logging.py::<module>::setup_logging` 初始化 CLI 文件日志输出
6. `hermes_cli/config.py::<module>::load_config` 预加载配置；仅用于早期网络参数判断 【**状态**】
7. `hermes_constants.py::<module>::apply_ipv4_preference` 在配置要求时提前启用 IPv4 优先
8. `hermes_cli/main.py::<module>::main` 构建 argparse、解析命令、分发子命令

## 2. CLI 分发链

1. `hermes_cli/main.py::<module>::main` 创建顶层 parser 与 `chat` 子命令 parser
2. `hermes_cli/config.py::<module>::get_container_exec_info` 判断当前是否需要路由进容器环境执行
3. `hermes_cli/main.py::<module>::_coalesce_session_name_args` 合并 `--resume/--continue` 后未加引号的多词会话名 【**会话**】
4. `hermes_cli/main.py::<module>::cmd_chat` 处理 `chat` 子命令参数并转交给 `cli.py`
5. `hermes_cli/main.py::<module>::_has_any_provider_configured` 检查当前是否至少存在一个可用推理 provider 【**状态**】
6. `hermes_cli/config.py::<module>::load_config` 读取模型/provider 配置 【**状态**】
7. `hermes_cli/config.py::<module>::get_env_path` 定位 `.env` 路径
8. `hermes_cli/auth.py::<module>::get_auth_status` 检查 provider 登录/认证状态 【**状态**】
9. `hermes_cli/config.py::<module>::get_hermes_home` 获取 Hermes 数据目录 【**状态**】
10. `hermes_cli/banner.py::<module>::prefetch_update_check` 后台预取升级信息
11. `tools/skills_sync.py::<module>::sync_skills` 同步内置 skills 索引
12. `cli.py::<module>::main` 进入 CLI 真实执行入口

## 3. `cli.py` 的 quiet one-shot 路径

1. `cli.py::<module>::main` 处理 `query = query or q`
2. `hermes_cli/tools_config.py::<module>::_get_platform_tools` 在未显式传 `--toolsets` 时解析 `cli` 平台默认 toolsets
3. `cli.py::<module>::_parse_skills_argument` 解析 `--skills` 参数；本命令未传 skills，通常返回空
4. `cli.py::HermesCLI::__init__` 创建 CLI 实例并初始化会话号、配置、控制台、SQLite session store 【**状态**】【**会话**】
5. `hermes_state.py::SessionDB::__init__` 打开 `state.db` 并准备会话库 【**状态**】【**会话**】
6. `hermes_state.py::SessionDB::_init_schema` 初始化/迁移 SQLite schema 【**状态**】【**会话**】
7. `cli.py::<module>::_collect_query_images` 收集 `--image` 和 query 中的本地图像；本命令未传 image，返回空
8. `cli.py::HermesCLI::_ensure_runtime_credentials` 解析当前 provider/base_url/api_key/model 组合 【**状态**】
9. `hermes_cli/runtime_provider.py::<module>::resolve_runtime_provider` 统一解析运行时 provider 凭据和访问模式 【**状态**】
10. `hermes_cli/runtime_provider.py::<module>::resolve_requested_provider` 决定此次请求实际要走哪个 provider 【**状态**】
11. `hermes_cli/runtime_provider.py::<module>::_get_model_config` 读取 `model` 配置并补齐默认/本地模型信息 【**状态**】
12. `hermes_cli/auth.py::<module>::resolve_provider` 把 `auto/custom/...` 解析成真实 provider 标识 【**状态**】
13. `hermes_cli/runtime_provider.py::<module>::_resolve_explicit_runtime` 处理显式传入的 `api_key/base_url` 覆盖 【**状态**】
14. `agent/credential_pool.py::<module>::load_pool` 条件：provider 启用了 credential pool 时装载凭据池 【**状态**】
15. `cli.py::HermesCLI::_normalize_model_for_provider` 修正 provider 与 model 的匹配关系 【**状态**】
16. `cli.py::HermesCLI::_resolve_turn_agent_config` 解析本轮是否启用 smart routing / service tier 覆盖 【**状态**】
17. `agent/smart_model_routing.py::<module>::resolve_turn_route` 为当前 prompt 选择 primary route 或 cheap route 【**状态**】
18. `agent/smart_model_routing.py::<module>::choose_cheap_model_route` 按 prompt 复杂度判断是否降级到 cheap model；本命令是否命中取决于配置
19. `cli.py::HermesCLI::_init_agent` 初始化 `AIAgent`，并在必要时恢复会话历史 【**状态**】【**会话**】
20. `cli.py::HermesCLI::_ensure_runtime_credentials` `_init_agent` 内再次兜底刷新 provider 解析 【**状态**】
21. `run_agent.py::AIAgent::__init__` 创建 agent、本地状态、客户端、工具集合、压缩器、持久化组件 【**状态**】【**会话**】【**记忆**】
22. `cli.py::<module>::main` quiet 分支直接调用 `cli.agent.run_conversation(...)`

## 4. `AIAgent.__init__` 初始化链

1. `run_agent.py::<module>::_install_safe_stdio` 用安全包装器接管 stdout/stderr，防止 broken pipe 异常
2. `hermes_cli/model_normalize.py::<module>::normalize_model_for_provider` 规范化 model 名称和 provider 约束 【**状态**】
3. `run_agent.py::AIAgent::_is_direct_openai_url` 判断当前 base_url 是否是原生 OpenAI URL 【**状态**】
4. `run_agent.py::AIAgent::_model_requires_responses_api` 判断当前模型是否必须改走 Responses API 【**状态**】
5. `run_agent.py::AIAgent::_create_openai_client` 条件：`chat_completions/codex_responses` 路径下创建 OpenAI 兼容 client 【**状态**】
6. `agent/anthropic_adapter.py::<module>::build_anthropic_client` 条件：`anthropic_messages` 路径下创建 Anthropic client 【**状态**】
7. `model_tools.py::<module>::get_tool_definitions` 根据启用 toolsets 构造本次 session 可用工具 schema 【**状态**】【**上下文**】
8. `model_tools.py::<module>::_discover_tools` 导入并注册内置工具
9. `tools/mcp_tool.py::<module>::discover_mcp_tools` 发现配置中的 MCP 工具
10. `hermes_cli/plugins.py::<module>::discover_plugins` 发现插件工具
11. `tools/registry.py::ToolRegistry::get_definitions` 从 registry 取出通过可用性校验的工具 schema 【**状态**】
12. `model_tools.py::<module>::check_toolset_requirements` 检查 toolset 依赖项是否齐备
13. `tools/checkpoint_manager.py::CheckpointManager::__init__` 初始化文件回滚快照管理器 【**状态**】
14. `tools/todo_tool.py::TodoStore::__init__` 初始化内存中的 todo store 【**状态**】
15. `hermes_state.py::SessionDB::create_session` 创建本次 CLI session 记录 【**会话**】【**状态**】
16. `hermes_state.py::SessionDB::_execute_write` 执行 `create_session` 的 SQLite 写事务 【**会话**】【**状态**】
17. `tools/memory_tool.py::MemoryStore::__init__` 条件：开启 memory 时初始化记忆存储 【**记忆**】【**状态**】
18. `tools/memory_tool.py::MemoryStore::load_from_disk` 条件：从 `MEMORY.md/USER.md` 载入本地记忆 【**记忆**】【**状态**】
19. `agent/subdirectory_hints.py::SubdirectoryHintTracker::__init__` 初始化目录上下文提示跟踪器 【**上下文**】【**状态**】
20. `agent/context_compressor.py::ContextCompressor::__init__` 初始化上下文压缩器和阈值 【**上下文**】【**状态**】
21. `plugins/context_engine/__init__.py::<module>::load_context_engine` 条件：开启 context engine 时装载上下文引擎 【**上下文**】【**状态**】
22. `agent/memory_manager.py::_MemoryManager::initialize_all` 条件：初始化外部 memory provider 插件 【**记忆**】【**状态**】

## 5. `run_conversation` 固定主干

1. `run_agent.py::AIAgent::run_conversation` 执行一次完整的用户请求主循环 【**状态**】【**上下文**】【**会话**】【**记忆**】
2. `run_agent.py::<module>::_install_safe_stdio` 再次确保当前线程标准输出安全
3. `hermes_logging.py::<module>::set_session_context` 把当前日志线程绑定到本次 `session_id` 【**会话**】【**状态**】
4. `run_agent.py::AIAgent::_restore_primary_runtime` 如果上一轮发生 fallback，则先恢复主 provider/runtime 【**状态**】
5. `run_agent.py::<module>::_sanitize_surrogates` 清洗用户输入中的非法 surrogate 字符 【**状态**】
6. `run_agent.py::IterationBudget::__init__` 为本轮建立新的迭代预算状态 【**状态**】
7. `run_agent.py::AIAgent::_cleanup_dead_connections` 清理上轮遗留的坏连接；`anthropic_messages` 下通常跳过 【**状态**】
8. `run_agent.py::AIAgent::_replay_compression_warning` 重放此前的上下文压缩告警 【**上下文**】【**状态**】
9. `run_agent.py::AIAgent::_hydrate_todo_store` 条件：从历史消息恢复 todo 状态 【**状态**】【**会话**】
10. `run_agent.py::AIAgent::_build_system_prompt` 第一次请求时组装稳定 system prompt 【**上下文**】【**会话**】【**记忆**】
11. `hermes_cli/plugins.py::<module>::invoke_hook` 事件：`on_session_start`，允许插件扩展会话启动 【**会话**】【**状态**】
12. `hermes_state.py::SessionDB::update_system_prompt` 把组装后的 system prompt 快照写入 session 行 【**会话**】【**上下文**】【**状态**】
13. `hermes_state.py::SessionDB::_execute_write` 执行 `update_system_prompt` 的 SQLite 写事务 【**会话**】【**状态**】
14. `agent/model_metadata.py::<module>::estimate_request_tokens_rough` 预估当前消息+工具 schema 的总 token 数 【**上下文**】【**状态**】
15. `run_agent.py::AIAgent::_compress_context` 条件：历史已超过压缩阈值时先做 preflight compression 【**上下文**】【**会话**】【**记忆**】【**状态**】
16. `hermes_cli/plugins.py::<module>::invoke_hook` 事件：`pre_llm_call`，允许插件把额外上下文注入到本轮 user message 【**上下文**】【**会话**】
17. `agent/memory_manager.py::_MemoryManager::prefetch_all` 条件：外部记忆插件预取与本轮 query 相关的记忆片段 【**记忆**】【**上下文**】
18. `run_agent.py::AIAgent::clear_interrupt` 清理本轮开始前的中断状态 【**状态**】

## 6. 主循环中每轮 API 调用前的固定链

1. `tools/checkpoint_manager.py::CheckpointManager::new_turn` 重置本轮 checkpoint 去重状态 【**状态**】
2. `run_agent.py::AIAgent::_touch_activity` 更新 agent 当前活动状态和时间戳 【**状态**】
3. `run_agent.py::IterationBudget::consume` 消耗一次 LLM 调用预算 【**状态**】
4. `run_agent.py::AIAgent::_sanitize_api_messages` 清理 orphan tool result、非法 role、坏 tool_call 配对 【**上下文**】【**状态**】
5. `agent/memory_manager.py::<module>::build_memory_context_block` 把外部记忆预取结果包成注入到 user message 的上下文块 【**记忆**】【**上下文**】
6. `agent/prompt_caching.py::<module>::apply_anthropic_cache_control` 条件：Claude/OpenRouter/Anthropic 路径下设置 prompt cache breakpoint 【**上下文**】【**状态**】
7. `agent/model_metadata.py::<module>::estimate_messages_tokens_rough` 估算送入 API 的消息体 token 数 【**上下文**】【**状态**】
8. `run_agent.py::AIAgent::_build_api_kwargs` 把内部消息结构转换成 provider 专属请求体 【**上下文**】【**状态**】
9. `hermes_cli/plugins.py::<module>::invoke_hook` 事件：`pre_api_request`，允许插件观察请求发送前状态 【**状态**】【**会话**】
10. `run_agent.py::AIAgent::_interruptible_streaming_api_call` 默认走流式调用路径，并包裹超时/中断/重试逻辑 【**状态**】
11. `run_agent.py::AIAgent::_interruptible_api_call` 条件：流式不可用或被禁用时走非流式调用 【**状态**】

## 7. `_build_system_prompt` 下钻

1. `run_agent.py::AIAgent::_build_system_prompt` 按固定层次拼接 identity、memory、skills、project context、timestamp、platform hints 【**上下文**】【**会话**】【**记忆**】
2. `agent/prompt_builder.py::<module>::load_soul_md` 读取 `HERMES_HOME/SOUL.md` 作为 agent identity 【**上下文**】
3. `agent/prompt_builder.py::<module>::build_nous_subscription_prompt` 条件：按可用工具生成 Nous 订阅提示
4. `tools/memory_tool.py::MemoryStore::format_for_system_prompt` 条件：把本地 memory/user profile 格式化进 system prompt 【**记忆**】【**上下文**】
5. `agent/memory_manager.py::_MemoryManager::build_system_prompt` 条件：把外部 memory provider 的 system prompt 块加入 prompt 【**记忆**】【**上下文**】
6. `agent/prompt_builder.py::<module>::build_skills_system_prompt` 条件：当 `skills_list/skill_view/skill_manage` 可用时构造 skills 提示 【**上下文**】
7. `agent/prompt_builder.py::<module>::build_context_files_prompt` 装载项目级上下文文件 【**上下文**】
8. `agent/prompt_builder.py::<module>::_load_hermes_md` 条件：优先查找 `.hermes.md/HERMES.md` 【**上下文**】
9. `agent/prompt_builder.py::<module>::_load_agents_md` 条件：若无 `.hermes.md`，读取仓库根 `AGENTS.md` 【**上下文**】
10. `agent/prompt_builder.py::<module>::_load_claude_md` 条件：若无前两者，读取 `CLAUDE.md` 【**上下文**】
11. `agent/prompt_builder.py::<module>::_load_cursorrules` 条件：若无前三者，读取 `.cursorrules` / `.cursor/rules/*.mdc` 【**上下文**】
12. `hermes_time.py::<module>::now` 生成固定会话起始时间戳 【**会话**】【**状态**】
13. `agent/prompt_builder.py::<module>::build_environment_hints` 追加 WSL/Termux/平台运行环境提示 【**上下文**】

## 8. 真实请求发送分支

1. `run_agent.py::AIAgent::_build_api_kwargs` 根据 `api_mode` 组装 `chat_completions / codex_responses / anthropic_messages` 三类请求体 【**上下文**】【**状态**】
2. `run_agent.py::AIAgent::_prepare_anthropic_messages_for_api` 条件：Anthropic path 下重写消息结构 【**上下文**】
3. `agent/anthropic_adapter.py::<module>::build_anthropic_kwargs` 条件：生成 Anthropic Messages API 参数
4. `run_agent.py::AIAgent::_chat_messages_to_responses_input` 条件：Codex/Responses path 下把 chat messages 转为 Responses input items 【**上下文**】
5. `run_agent.py::AIAgent::_preflight_codex_api_kwargs` 条件：Codex path 下做 Responses request 预校验 【**状态**】
6. `run_agent.py::AIAgent::_interruptible_streaming_api_call` 实际发起流式请求并负责重试、超时、中断、流式 delta 聚合 【**状态**】
7. `run_agent.py::AIAgent::_interruptible_streaming_api_call._call_chat_completions(内嵌)` 条件：`chat_completions` 路径下调用 `request_client.chat.completions.create(..., stream=True)`
8. `run_agent.py::AIAgent::_interruptible_streaming_api_call._call_anthropic(内嵌)` 条件：`anthropic_messages` 路径下调用 Anthropic stream
9. `run_agent.py::AIAgent::_call_anthropic` 处理 Anthropic 流式事件并回填最终消息对象 【**状态**】
10. `run_agent.py::AIAgent::_fire_first_delta` 首个 token 到达时通知上层显示逻辑 【**状态**】
11. `run_agent.py::AIAgent::_fire_stream_delta` 持续推送可见文本增量 【**状态**】
12. `run_agent.py::AIAgent::_fire_reasoning_delta` 持续推送 reasoning 增量 【**状态**】【**上下文**】
13. `run_agent.py::AIAgent::_fire_tool_gen_started` 流式工具生成开始时通知上层

## 9. API 返回后的固定处理链

1. `run_agent.py::AIAgent::_normalize_codex_response` 条件：Codex path 下把 Responses 返回归一化成 assistant message 【**上下文**】
2. `agent/anthropic_adapter.py::<module>::normalize_anthropic_response` 条件：Anthropic path 下把 content blocks 归一化成 assistant message 【**上下文**】
3. `hermes_cli/plugins.py::<module>::invoke_hook` 事件：`post_api_request`，允许插件观察本次返回结果 【**状态**】【**会话**】
4. `agent/usage_pricing.py::<module>::normalize_usage` 统一不同 provider 的 usage 结构 【**状态**】
5. `agent/context_compressor.py::ContextCompressor::update_from_response` 用真实 usage 更新 prompt/completion token 统计 【**上下文**】【**状态**】
6. `agent/usage_pricing.py::<module>::estimate_usage_cost` 根据 model/provider 估算本次调用费用 【**状态**】
7. `hermes_state.py::SessionDB::update_token_counts` 把 token/cost 增量写回 session 行 【**会话**】【**状态**】
8. `hermes_state.py::SessionDB::_execute_write` 执行 `update_token_counts` 的 SQLite 写事务 【**会话**】【**状态**】

## 10. 无工具调用时的收尾链

1. `run_agent.py::AIAgent::_strip_think_blocks` 移除 `<think>/<REASONING_SCRATCHPAD>` 后得到最终可见回复 【**上下文**】
2. `run_agent.py::AIAgent::_build_assistant_message` 构建最终 assistant message，保留 reasoning/tool_calls/finish_reason 【**上下文**】【**会话**】【**状态**】
3. `run_agent.py::AIAgent::_save_trajectory` 条件：若启用 trajectory 保存，则写入轨迹文件 【**会话**】【**状态**】
4. `run_agent.py::AIAgent::_cleanup_task_resources` 清理本轮 task 的 VM/browser/tool 资源 【**状态**】
5. `run_agent.py::AIAgent::_persist_session` 统一把本轮消息持久化到 JSON log + SQLite 【**会话**】【**状态**】
6. `run_agent.py::AIAgent::_apply_persist_user_message_override` 条件：把 API-only user message 替换回持久化用原始消息 【**会话**】【**状态**】
7. `run_agent.py::AIAgent::_save_session_log` 把消息列表写入 `~/.hermes/sessions/session_<id>.json` 【**会话**】【**状态**】
8. `run_agent.py::AIAgent::_flush_messages_to_session_db` 把尚未 flush 的消息逐条落到 SQLite 【**会话**】【**状态**】
9. `hermes_state.py::SessionDB::ensure_session` 保证 session 行存在，避免 `create_session` 失败后丢消息 【**会话**】【**状态**】
10. `hermes_state.py::SessionDB::_execute_write` 执行 `ensure_session` 的 SQLite 写事务 【**会话**】【**状态**】
11. `hermes_state.py::SessionDB::append_message` 逐条追加消息到 `messages` 表，并维护 `message_count/tool_call_count` 【**会话**】【**状态**】
12. `hermes_state.py::SessionDB::_execute_write` 执行 `append_message` 的 SQLite 写事务 【**会话**】【**状态**】
13. `hermes_cli/plugins.py::<module>::invoke_hook` 事件：`post_llm_call`，允许插件在本轮完成后处理对话结果 【**会话**】
14. `run_agent.py::AIAgent::clear_interrupt` 清理本轮中断状态 【**状态**】
15. `agent/memory_manager.py::_MemoryManager::sync_all` 条件：把这轮 user/assistant 对话同步给外部 memory provider 【**记忆**】【**会话**】
16. `agent/memory_manager.py::_MemoryManager::queue_prefetch_all` 条件：为下一轮 query 异步预取记忆 【**记忆**】【**状态**】
17. `run_agent.py::AIAgent::_spawn_background_review` 条件：触发 memory/skills 的后台 review 【**记忆**】【**状态**】
18. `hermes_cli/plugins.py::<module>::invoke_hook` 事件：`on_session_end`，允许插件在本轮结束时清理会话态 【**会话**】【**状态**】
19. `cli.py::<module>::main` quiet one-shot 分支打印 `response`
20. `cli.py::<module>::main` quiet one-shot 分支打印 `session_id`

## 11. 有工具调用时的分支链

1. `run_agent.py::AIAgent::_repair_tool_call` 条件：模型生成了错误工具名时尝试自动修复 【**状态**】
2. `run_agent.py::AIAgent::_cap_delegate_task_calls` 限制单轮 `delegate_task` 数量，避免并发失控 【**状态**】
3. `run_agent.py::AIAgent::_deduplicate_tool_calls` 去重本轮 tool calls，避免模型重复执行同一调用 【**状态**】
4. `run_agent.py::AIAgent::_build_assistant_message` 把带 tool_calls 的 assistant turn 规范化入消息流 【**上下文**】【**会话**】【**状态**】
5. `run_agent.py::AIAgent::_emit_interim_assistant_message` 把中间 assistant turn 通知给上层 UI / callback
6. `run_agent.py::AIAgent::_execute_tool_calls` 按批次执行当前 assistant 提交的工具调用 【**状态**】
7. `run_agent.py::<module>::_should_parallelize_tool_batch` 判断当前工具批是否允许并行
8. `run_agent.py::AIAgent::_execute_tool_calls_sequential` 逐个串行执行工具；交互型工具默认走这里
9. `run_agent.py::AIAgent::_execute_tool_calls_concurrent` 并发执行互不冲突的工具批
10. `run_agent.py::AIAgent::_invoke_tool` 对单个工具做 agent 内建分发或 registry 分发
11. `tools/todo_tool.py::<module>::todo_tool` 条件：执行内建 todo 工具 【**状态**】
12. `tools/session_search_tool.py::<module>::session_search` 条件：执行 session_search 【**会话**】【**上下文**】
13. `tools/memory_tool.py::<module>::memory_tool` 条件：执行内建 memory 工具 【**记忆**】【**状态**】
14. `tools/clarify_tool.py::<module>::clarify_tool` 条件：向用户追问补充信息 【**上下文**】【**状态**】
15. `tools/delegate_tool.py::<module>::delegate_task` 条件：创建子代理任务 【**状态**】
16. `agent/memory_manager.py::_MemoryManager::handle_tool_call` 条件：执行外部 memory provider 暴露的工具 【**记忆**】【**状态**】
17. `model_tools.py::<module>::handle_function_call` 执行普通 registry 工具的统一分发入口 【**状态**】
18. `model_tools.py::<module>::coerce_tool_args` 按 schema 把工具参数字符串转换为目标类型 【**状态**】
19. `tools/registry.py::ToolRegistry::dispatch` 把工具调用路由到具体 `tools/*.py` handler
20. `hermes_cli/plugins.py::<module>::invoke_hook` 事件：`post_tool_call`，允许插件观察工具执行结果 【**状态**】【**会话**】
21. `tools/tool_result_storage.py::<module>::maybe_persist_tool_result` 条件：把大型工具结果替换为可复用引用，控制上下文膨胀 【**上下文**】【**状态**】
22. `agent/subdirectory_hints.py::SubdirectoryHintTracker::check_tool_call` 根据工具调用结果附加目录上下文提示 【**上下文**】【**状态**】
23. `tools/tool_result_storage.py::<module>::enforce_turn_budget` 限制单轮工具结果总量，避免消息爆炸 【**上下文**】【**状态**】
24. `run_agent.py::AIAgent::_compress_context` 条件：工具结果把上下文推过阈值时压缩历史 【**上下文**】【**会话**】【**记忆**】【**状态**】
25. `run_agent.py::AIAgent::_save_session_log` 增量写入当前中间状态 【**会话**】【**状态**】
26. `run_agent.py::AIAgent::run_conversation` 回到 `while` 顶部发起下一轮 API 调用 【**状态**】

## 12. 条件分支补充：`SessionDB` / 压缩 / 记忆

1. `run_agent.py::AIAgent::_compress_context` 调用 `ContextCompressor.compress` 生成中段摘要并保留前后关键消息 【**上下文**】【**会话**】【**记忆**】【**状态**】
2. `agent/context_compressor.py::ContextCompressor::compress` 执行真实的上下文压缩算法 【**上下文**】【**状态**】
3. `run_agent.py::AIAgent::_invalidate_system_prompt` 压缩后使缓存的 system prompt 失效，准备重建 【**上下文**】【**状态**】
4. `hermes_state.py::SessionDB::end_session` 条件：压缩时结束旧 session 并切换 lineage 【**会话**】【**状态**】
5. `hermes_state.py::SessionDB::create_session` 条件：压缩后创建新的 session lineage 节点 【**会话**】【**状态**】
6. `hermes_state.py::SessionDB::set_session_title` 条件：给压缩后新 session 续写 lineage title 【**会话**】【**状态**】
7. `tools/file_tools.py::<module>::reset_file_dedup` 条件：压缩后清空文件读取去重缓存 【**上下文**】【**状态**】
8. `agent/memory_manager.py::_MemoryManager::on_memory_write` 条件：内建 memory 工具写入后桥接外部记忆系统 【**记忆**】【**状态**】

## 结

- 无工具调用的完整主链看到 10 为止
- 有工具调用的完整链路在 10 之后继续接 11
- 12 只是压缩 / SessionDB / 记忆相关的补充分支说明，不是主链尾部
