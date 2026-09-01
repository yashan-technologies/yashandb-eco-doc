## Contributing to YashanDB Eco Documentation | 贡献指南

[English](#english) | [中文](#中文)

---

<span id="english" name="english"></span>

### English

Thank you for contributing to the YashanDB open-source ecosystem documentation. This guide covers changes to documentation, navigation, images, repository automation, and related project governance.

#### Code of Conduct

Please be respectful, precise, and constructive in issues, pull requests, reviews, and other project discussions. Keep technical disagreements focused on the content and evidence.

#### How to Contribute

##### Reporting an Issue

Every change merged into this repository must be associated with a valid issue in the current repository. Before starting work:

1. Search existing issues to avoid duplicates.
2. Create an issue describing the background, expected result, affected components or documents, scope, and acceptance criteria.
3. Do not use a pull request without an issue for ordinary changes.

An issue does not need to specify an owner or a planned release cycle.

##### Updating Documentation

1. Fork the repository and create a working branch.
2. Update the relevant component documentation, navigation files, indexes, and images together.
3. Run the local checks described below.
4. Open a pull request that follows the title and body requirements.
5. Address review comments and wait for the required checks and approvals before merge.

##### Pull Request Title

The title must use the following format:

```text
<type><#Issue number>: <English summary>
```

Allowed types are `feat`, `fix`, `refactor`, and `docs`. For example:

```text
docs<#2>: Update JDBC driver and dialect package locations
```

The issue number must refer to an existing primary issue in this repository. The summary must describe the actual change and must not use vague titles such as `Update docs` or `Fix issue`.

##### Pull Request Body

The body must include the following sections:

```markdown
Closes #<Issue number>

## Change summary

## Affected components/documents

## Validation result

## Compatibility/publication impact

## Reviewer checklist
```

The body may be written in Chinese. `Closes #<Issue number>` must match the primary issue in the title. Mention related issues separately when needed.

##### Commit Messages

Each commit in a pull request must use this format:

```text
<type><#Issue number>: <summary>
```

Allowed commit types are `fix`, `feat`, `docs`, `test`, and `refactor`. The repository checks pull request titles and commit messages automatically.

#### Development Setup

##### Prerequisites

- Git
- Python 3.12 or a compatible Python version for local document checks
- A Markdown editor that preserves UTF-8 and repository-specific syntax

##### Repository Layout

```text
生态对接/
├── 00生态对接.md       # Section introduction
├── _index.md            # Website navigation order (optional)
├── SQL工具/             # SQL tools
├── ORM框架/             # ORM frameworks
├── 数据集成/             # Data integration
└── 其他/                # Other integrations
```

The repository root also contains `README.md`, this guide, and the documentation-check workflows under `.github/workflows/`.

##### Running Checks

The pull request workflow checks document format, local image references, Markdown links, `_index.md`, and component indexes through `opendoccheck`. When the checker repository is checked out next to this repository, run:

```bash
python opendoccheck/main.py check yashandb-eco-doc
```

Run the same command before opening a pull request and fix every non-zero result. GitHub Actions also validates the pull request title and all commit message headlines.

#### Documentation Conventions

- Write standard Markdown and preserve UTF-8 text.
- Each directory may contain a required `00` introduction file, an optional `_index.md`, an optional `image/` directory, and document files.
- `_index.md` controls website navigation. `filename` lists child directories or files separated by commas, and `enName` provides the corresponding English names in the same order.
- When adding, removing, or renaming a component, update the component documentation, the README component index, and the related `_index.md` in the same pull request.
- Store images used by a directory in that directory's `image/` folder. Use PNG files and relative paths such as `./image/example.png`.
- Verify that every local image and Markdown link is accessible.
- Preserve website extensions such as `::: tabs`; do not rewrite them as ordinary Markdown.

#### Pull Request and Review Guidelines

- Keep each pull request focused on one primary issue.
- Include all affected documents, indexes, examples, and images in the same change.
- Update documentation when a behavior, integration procedure, or public navigation entry changes.
- Do not push directly to `main`.
- Use Squash Merge after required checks and approvals pass.
- New commits require the existing approvals to be confirmed again.
- Emergency changes still require an issue, pull request, review, and validation.

| Role | Responsibility |
| --- | --- |
| Pull request author | Ensure complete content, executable examples, synchronized indexes, and resolved CI/review feedback |
| Primary maintainer | Own final content quality and merge confirmation |
| Co-maintainer | Maintain conventions, review content, and support merge confirmation |
| Merger | Merge with Squash Merge after CI and approvals pass, and confirm issue closure |

#### Publication Rules

- Review publication readiness every four weeks according to the documentation release cycle.
- Compare the latest release tag with `main`; do not create a tag or publish an empty version when there are no documentation changes.
- Create a tag only after the required checks on `main` pass. A tag must point to the validated commit and must never be moved, deleted, or reused.
- GitHub tags define open-source documentation release boundaries; they are not website display versions.
- Confirm that the target commit and tag have synchronized to GitLab before publication.
- The website displays the logical `main` label and serves content from the latest successful release tag.
- If website publication fails, keep the previous successful tag and do not move the GitHub tag.

Severe technical, security, legal, or usability defects may justify an emergency publication outside the four-week cycle. The complete issue, pull request, review, and validation process still applies.

#### Pending Decisions

Contributors should not decide the following items until the review conclusions are documented:

1. The exact release tag format and version-number expression.
2. The first website publication process, including entry point, executor, steps, verification, and rollback procedure.

---

<span id="中文" name="中文"></span>

### 中文

感谢你参与 YashanDB 开源生态对接文档的维护。本指南适用于文档、导航、图片、仓库自动化和相关治理规则的变更。

#### 行为准则

请在 Issue、Pull Request、代码评审和其他项目讨论中保持尊重、准确和建设性。技术分歧应围绕内容和证据展开。

#### 如何贡献

##### 提交 Issue

所有合入本仓库的变更都必须关联当前仓库中的有效 Issue。开始修改前：

1. 搜索已有 Issue，避免重复提交。
2. 创建 Issue，说明背景、预期表现、涉及的组件或文档、改动范围和验收标准。
3. 普通变更不得使用没有 Issue 的 PR 直接合入。

Issue 不要求填写第一责任人或计划发布周期。

##### 修改文档

1. Fork 本仓库并创建工作分支。
2. 同步修改相关组件文档、导航文件、索引和图片。
3. 按照下文运行本地检查。
4. 按照标题和正文要求提交 PR。
5. 处理评审意见，等待必需检查和审批通过后合入。

##### PR 标题

标题必须使用以下格式：

```text
<type><#Issue号>: <English summary>
```

允许的类型为 `feat`、`fix`、`refactor` 和 `docs`。示例：

```text
docs<#2>: Update JDBC driver and dialect package locations
```

Issue 号必须是当前仓库中存在的主要 Issue。摘要必须描述实际变化，不得使用 `Update docs`、`Fix issue` 等无法识别改动范围的标题。

##### PR 正文

PR 正文必须包含以下章节：

```markdown
Closes #<Issue号>

## Change summary

## Affected components/documents

## Validation result

## Compatibility/publication impact

## Reviewer checklist
```

正文可以使用中文。`Closes #<Issue号>` 必须与标题中的主要 Issue 一致；其他相关 Issue 可以在正文中单独列出。

##### Commit Message

PR 中的每个提交都必须使用以下格式：

```text
<type><#Issue号>: <summary>
```

允许的提交类型为 `fix`、`feat`、`docs`、`test` 和 `refactor`。仓库会自动检查 PR 标题和提交信息。

#### 开发环境

##### 前提条件

- Git
- Python 3.12 或兼容版本，用于运行本地文档检查
- 能够保留 UTF-8 和仓库扩展语法的 Markdown 编辑器

##### 仓库结构

```text
生态对接/
├── 00生态对接.md       # 目录介绍
├── _index.md            # 官网导航顺序（可选）
├── SQL工具/             # SQL 工具
├── ORM框架/             # ORM 框架
├── 数据集成/             # 数据集成
└── 其他/                # 其他生态对接
```

仓库根目录还包含 `README.md`、本指南以及 `.github/workflows/` 下的文档检查工作流。

##### 运行检查

PR 工作流通过 `opendoccheck` 检查文档格式、本地图片引用、Markdown 链接、`_index.md` 和组件索引。在将检查器仓库与当前仓库并列检出后，可运行：

```bash
python opendoccheck/main.py check yashandb-eco-doc
```

提交 PR 前请运行同样的命令，并修复所有非零结果。GitHub Actions 还会检查 PR 标题和所有提交信息的标题行。

#### 文档规范

- 使用通用 Markdown 格式并保留 UTF-8 编码。
- 每级目录可以包含必选的 `00` 前言文件、可选的 `_index.md`、可选的 `image/` 目录及文档文件。
- `_index.md` 控制官网导航顺序；`filename` 使用逗号分隔的子目录或文件名，`enName` 与其逐项对应。
- 新增、删除或重命名组件时，必须在同一个 PR 中同步更新组件文档、README 组件索引和相关 `_index.md`。
- 当前目录使用的图片放在该目录的 `image/` 下，使用 PNG 格式，并通过相对路径引用，例如 `./image/example.png`。
- 提交前确认所有本地图片和 Markdown 链接可访问。
- 保留官网文档扩展语法，例如 `::: tabs`；不要将其改写为普通 Markdown。

#### PR 与评审规范

- 每个 PR 只处理一个主要 Issue，并保持改动聚焦。
- 在同一个 PR 中包含所有受影响的文档、索引、示例和图片。
- 当行为、接入步骤或公开导航入口变化时，必须同步更新文档。
- 禁止直接推送到 `main` 分支。
- 必需检查和审批通过后使用 Squash Merge 合入。
- PR 新增提交后，原有批准需要重新确认。
- 紧急变更仍必须补齐 Issue、PR、评审和校验流程。

| 角色 | 职责 |
| --- | --- |
| PR 作者 | 保证内容完整、示例可执行、索引同步，并处理 CI 和评审意见 |
| 第一责任人 | 负责最终内容质量和合入确认 |
| 协同维护人 | 参与规范维护、内容评审和合入确认 |
| 合入人 | 在 CI 和审批满足要求后执行 Squash Merge，并确认 Issue 自动关闭 |

#### 发布规则

- 每四周按照文档发布周期检查一次发布条件。
- 比较最新发布 Tag 与 `main`；没有文档变更时不创建 Tag，也不发布空版本。
- 确认 `main` 的必需检查全部通过后再创建 Tag。Tag 必须指向已经通过校验的提交，发布后不得移动、删除或复用。
- GitHub Tag 是开源文档的发布边界，不是官网展示版本。
- 发布前确认目标提交和 Tag 已同步到 GitLab。
- 官网始终展示逻辑标签 `main`，内容来自最新一次成功发布的 Tag。
- 官网发布失败时保留上一个成功 Tag 的内容，不移动 GitHub Tag。

已发布内容存在严重技术错误、安全风险、法律风险或导致用户无法操作的问题时，可以在四周周期外紧急发布。紧急发布仍需完整的 Issue、PR、评审和校验流程。

#### 待确认事项

以下两项在评审结论明确前不由贡献者自行决定：

1. 发布 Tag 格式的准确命名规则及版本号表达方式。
2. 官网首次发布流程，包括发布入口、执行人、操作步骤、验证方式和失败回退方式。
