# AGENTS.md

## 开发环境规范

当前是 windows powershell 环境，注意命令规范：

在执行任何脚本、命令或处理环境交互时，所有 CLI 命令和脚本执行必须严格遵守以下 PowerShell 规则：

1. 统一 Shell 环境：所有命令必须使用 PowerShell (推荐 pwsh 即 PowerShell 7 以上)。禁止使用 Windows 旧版 cmd.exe 或默认 powershell.exe。
2. 转义字符约束：Bash 与 PowerShell 的转义规则不同，在编写跨平台脚本时，禁止混用 Shell 语法。
3. 严格的错误处理：所有脚本首部必须包含 $ErrorActionPreference = 'Stop'，确保遇到错误时立即中断并报错，避免静默失败。
4. 编码规范：脚本文件生成与读写必须强制指定 -Encoding UTF8，防止中文字符或特殊符号乱码。
5. 管道与对象优先：在处理数据解析时，优先使用 PowerShell 的对象管道特性（如 Select-Object, Where-Object），避免过度依赖传统的文本截取。
6. 在 Windows 执行 shell 命令时优先使用 PowerShell 7：
    - 如果外层 shell 也是 PowerShell，传给 -Command 的脚本优先用单引号包住，避免 $变量 被外层提前展开。
    - 不要在 PowerShell 字符串里用 Bash 风格的 \" 转义双引号；PowerShell 不按 Bash 规则处理它。
    - rg 正则包含 |、(、)、\ 等字符时，用单引号包住 pattern，避免 | 被解析成 PowerShell 管道。
    - 需要在 -Command 脚本中使用 $p/$i/$lines 等变量时，必须确保这些 $ 没有被外层 PowerShell展开：优先使用外层单引号，或用反引号转义 $。
    - 尽量避免嵌套多层引号；复杂命令拆成更简单的单个命令执行。

## 项目开发准则

1. 不需要进行代码测试，除非明确要求
2. 代码编写完，不要编写或者修改文档，除非明确要求
3. python 使用 uv 管理环境，并且尽量相关下载包到当前文件夹下
4. 代码中间产物，或者临时文件放到 ./temp 文件夹下
5. 必要时更新 Makefile 文件
6. 给出必要的 log，格式为：[YYYY-MM-DD HH:mm:ss +08:00] [级别] [模块] - 具体信息，持久化保存到 ./log/ 目录下，按日期分文件保存，文件命名格式：log_YYYY-MM-DD.log
7. 所有目录和包名都使用单数，然后注意符合优秀的命名规范
8. 注释规范：
   - 8.1 非必要不写注释，最好的注释就是把代码写得像人话一样，然后把注释删掉，不要写废话
   - 8.2 代码中必要的注释，使用英文，且尽量简短。“高价值”场景：业务背景与“坑” (The Why)，正则表达式的解释，复杂的算法逻辑
   - 8.3 标准化标签，TODO，FIXME，HACK，NOTE
   - 8.4 函数/方法 (Functions & Methods)的 docstring
     - 简单 Type Hints + 一句话总结。 不要为显而易见的参数写注释
     - 如果函数的参数有特殊限制，或者返回了复杂的结构，使用 Google 风格的 docstring
   - 8.5 行注释位置：放在代码正上方 (Above)
9. 问题尽量使用优雅的方案解决，而不是反复尝试，有时候可以搜索业界优秀的sota是怎么解决的，参考别的解决方案
10. 对于常量，使用大写字母加下划线的方式命名，并且尽量放到文件开头部分，尤其是后续可能需要用户调整的参数
11. 对于给出的文件路径需要给出 [file_name](file_path:column_number) 的链接，方便直接跳转到对应的文件位置

## 工程原则（KISS / YAGNI / DRY / SOLID）

- KISS：优先最小可行改动，避免不必要的复杂性。
- YAGNI：只实现用户当前明确需要的内容，拒绝过度设计。
- DRY：抽取重复逻辑，但不为“未来复用”而提前抽象。
- SOLID：保持职责单一、接口小而清晰、依赖抽象而非具体实现。

---

### Before acting

- If the request is ambiguous, state assumptions or ask — don't silently
  pick one reading and build it.

### When editing existing code

- Change only what the request requires. Don't refactor or restyle working
  code you weren't asked to touch. Match the existing style.

### Design Rules (strict)

Before changing code, check the rules below. If a change would violate one,
stop and explain the smaller redesign first.

Do not fix a banned smell by changing its shape: bool → enum/options,
checks → wrappers, flag/switch → Strategy, pass-through layer → facade/adapter.

1. **Names must disambiguate.** Banned defaults: `data`, `info`, `result`,
   `handler`, `manager`, `process`, `utils`, `helper`, `do_*`, `*_impl`.
   Rename to describe the specific thing/action.

2. **Validate once at edges; trust invariants inside.** Do not scatter
   defensive checks across trusted internal boundaries. No repeated
   `if x is None: return` / `if (!ptr) return -1;`. If the same check
   appears 3+ times, redesign the boundary.

3. **Comments document contracts, invariants, rationale, constraints, and
   rejected alternatives.** Do not narrate code or compensate for bad
   names/boundaries.

4. **No mode/flag parameter for a special case.** No bool, enum, string mode,
   or options bag to switch behavior. If variation is real, use separate
   operations owned by the right abstraction.

5. **Right owner, complete operation.** Put complexity where the decision,
   invariant, or external dependency lives. Expose complete operations, not
   caller-managed steps. Add no API/layer unless it hides caller knowledge,
   enforces an invariant, or adapts an external dependency. Do not stuff
   unrelated behavior together just to keep the API small.

### Stop signals (redesign, don't push through)

- One change spreads across many files → wrong owner or duplicated
  knowledge, not more patches.
- Naming gets hard, or a comment is explaining around an awkward interface
  → suspect the abstraction boundary before adding more words.

---

### Engineering Principles and Design Discipline

Use these as default design instincts unless the repository, user request, or established framework conventions point elsewhere.

KISS: Prefer the smallest clear solution that solves the current problem.

YAGNI: Do not add abstractions, options, or future-facing design without a real current need.

DRY: Remove meaningful duplication, but do not introduce abstractions that obscure intent.

SOLID: Keep responsibilities focused and public interfaces small, clear, and stable.

Prefer specific names over generic defaults such as data, info, result, manager, helper, utils, do_*, or*_impl. Follow existing project conventions when they are deliberate or framework-driven.

Validate at system boundaries and preserve clear internal invariants. Avoid scattering repeated defensive checks through trusted internal code unless the boundary is genuinely unclear.

Comments in production code should explain why: intent, constraints, trade-offs, rejected alternatives, or non-obvious external requirements. Avoid comments that merely paraphrase the code.

For learning material, diagrams, walkthroughs, and debugging explanations, explain both what happens and why it is designed that way. Make the distinction clear so beginners can understand the flow and remember the reason behind it.

Avoid boolean or flag parameters that create hidden modes. Prefer separate concepts, methods, or types when the distinction represents real behaviour.

Keep interfaces narrow and implementations allowed to be substantial. Do not add pass-through layers unless they hide complexity, enforce invariants, or adapt an external dependency.
