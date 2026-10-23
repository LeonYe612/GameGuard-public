# Repository Guidelines

GameGuard 仓库贡献指南。

GameGuard 是一个 Python 3.11 库和 CLI：它把策划文档转成确定性的游戏测试计划，在沙箱中执行，并生成回归与分诊报告。请保持“计划”和“执行”分离：LLM 负责生成产物，沙箱回放必须保持确定性。

## 五项目协作边界（2026-10-23）

跨项目职责、架构和问题路由以 `/Users/helloppx/GitHub/WFGameAI/docs/technical-docs/2026-10-23-五项目架构与协作说明.md` 为准。

- 本仓是独立的受限 Invariant、确定性沙箱与 Plan/Execute 分离参考实现，负责 `InvariantBundle → TestPlan → Runner → trace/缺陷报告`。
- 当前未找到 GameGuard 被 WFGameAI 运行时直接调用；沙箱、mock Unity 或 Agent 评估结果只证明本仓对应实验，不能写成 WFGameAI 的真实真机或业务判定结果。
- 需要回答业务资产、测试点义务、人工终审或真实设备失败时，查 `/Users/helloppx/GitHub/WFGameAI` 与 `/Users/helloppx/GitHub/playbee-mcp`；本仓只回答规则可计算性、确定性与沙箱执行问题。
- 若未来接入 WFGameAI，先固定调用方向、请求/响应契约、状态真源与失败语义，再实现适配。

## 项目结构与模块

- `gameguard/`：核心包。`agents/` 放 LLM 工作流，`domain/` 放 Pydantic 领域模型，`tools/` 定义 Agent 工具，`testcase/` 负责加载与执行计划，`sandbox/` 提供 PySim、QuestSim 和 Unity 适配器，报告代码在 `reports/`。
- `tests/`：pytest 测试；`evals/`：Agent 效果评估与夹具；`testcases/`：可供评审的 YAML 测试计划；`docs/`：架构和设计资料。
- `artifacts/`：运行时 trace、快照和报告。这里只放产物，不要当源码修改。

## 构建、测试与开发命令

先创建环境，之后的开发命令都在 `gameguard` Conda 环境内运行；Makefile 会检查这一点。

```bash
make env        # 根据 environment.yml 创建环境
make install    # 以可编辑模式安装开发依赖
make test       # 运行 pytest
make lint       # 运行 Ruff
make typecheck  # 运行 mypy --strict
make demo       # 跑“文档 → Bug 报告”端到端示例
make regress    # 比较 v1 与 v2 测试套件
make eval       # 跑 Agent 评估并汇总
```

修改 `gameguard/sandbox/unity/proto/gameguard_v1.proto` 后，用 `make proto` 重新生成 Python gRPC 绑定。`make unity-server` 和 `make test-unity` 用于 Unity mock 通路。

## 代码风格与命名

遵循 PEP 8，行宽上限 100 字符。使用类型标注和 `from __future__ import annotations`；函数、变量用 `snake_case`，类用 `PascalCase`，常量用 `UPPER_SNAKE_CASE`。优先复用现有的 Pydantic 模型和工具抽象。不要手改 `sandbox/unity/generated/*`，应修改 proto 后重新生成。

## 测试规范

使用 pytest：测试文件命名为 `tests/test_*.py`，测试函数命名为 `test_*`。逻辑变更要补针对性测试，并保留固定 seed 的确定性。管线测试应 mock LLM 响应；真实模型评估放在 `evals/`。Unity 传输测试需要加 `@pytest.mark.unity`，依赖不可用时用 `pytest -m "not unity"` 跳过。

## 提交与 Pull Request

提交信息用中文；标题用简短的命令式描述，可选 `feat:`、`refactor:`、`docs:` 等前缀。PR 要说明问题和方案、列出验证命令、关联 Issue；用户可见输出有变化时附截图或报告片段。不要提交 `.env`、API Key、缓存或运行时产物。
