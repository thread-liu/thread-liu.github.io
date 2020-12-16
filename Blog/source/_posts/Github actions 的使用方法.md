---
title: Github actions 的使用流程
index_img: /img/github.png
date: 2020-11-07 23:35:57
tags: [actions, ci]
---

## 什么是 CI （CONTINUOUS INTEGRATION）

在持续集成环境中，开发人员将会频繁的提交代码到主干。这些新提交在最终合并到主线之前，都需要通过编译和[自动化](http://www.ttlsa.com/auto/)测试流进行验证。这样做是基于之前

持续集成过程中很重视自动化测试验证结果，以保障所有的提交在合并主线之后的质量问题，对可能出现的一些问题进行预警

## GitHub Actions

> GitHub Actions makes it easy to automate all your software workflows, now with world-class CI/CD. Build, test, and deploy your code right from GitHub. Make code reviews, branch management, and issue triaging work the way you want.

## 使用方法

### 创建工作流程

- 从 GitHub 上的仓库，在 `.github/workflow` 目录中创建一个名为 `superlinter.yml` 的新文件。 更多信息请参阅“[创建新文件](https://docs.github.com/cn/free-pro-team@latest/github/managing-files-in-a-repository/creating-new-files)”。

- 将以下 YAML 内容复制到 `superlinter.yml` 文件中。 **注：** 如果您的默认分支不是 `main`，请更新 `DEFAULT_BRANCH` 的值以匹配您仓库的默认分支名称。

  ```yaml
  name: Super-Linter
  
  # Run this workflow every time a new commit pushed to your repository
  on: push
  
  jobs:
    # Set the job key. The key is displayed as the job name
    # when a job name is not provided
    super-lint:
      # Name the Job
      name: Lint code base
      # Set the type of machine to run on
      runs-on: ubuntu-latest
  
      steps:
        # Checks out a copy of your repository on the ubuntu-latest machine
        - name: Checkout code
          uses: actions/checkout@v2
  
        # Runs the Super-Linter action
        - name: Run Super-Linter
          uses: github/super-linter@v3
          env:
            DEFAULT_BRANCH: main
            GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  ```

- 在仓库中提交工作流程文件会触发 `push` 事件并运行工作流程。

### 查看工作流程

- 在 GitHub 上，导航到仓库的主页面。

- 在仓库名称下，单击 **Actions（操作）**。

  ![主仓库导航中的操作选项卡](https://docs.github.com/assets/images/help/repository/actions-tab.png)

- 在左侧边栏中，单击您想要查看的工作流程。

  ![左侧边栏中的工作流程列表](https://docs.github.com/assets/images/help/repository/superlinter-workflow-sidebar.png)

- 从工作流程运行列表中，单击要查看的运行的名称。

  ![工作流程运行的名称](https://docs.github.com/assets/images/help/repository/superlinter-run-name.png)

- 在左侧边栏中，单击 **Lint code base（Lint 代码库）**作业。

  ![Lint 代码库作业](https://docs.github.com/assets/images/help/repository/superlinter-lint-code-base-job.png)

- 任何失败的步骤都会自动展开以显示结果。‘

  ![Super linter 工作流程结果](https://docs.github.com/assets/images/help/repository/super-linter-workflow-results-updated.png)

## 高阶用法

### 定时

```yaml
# Runs at 16:00 UTC on the 1st of every month
schedule:
  - cron:  '0 16 1 * *'
```

### env

```
env:
  SERVER: production
```

- 在工作流程中使用 env：

  ```yaml
  steps:
    - name: Hello world
      run: echo Hello world $FIRST_NAME $middle_name $Last_Name!
      env:
        FIRST_NAME: Mona
        middle_name: The
        Last_Name: Octocat
  ```

- 您也可以使用 `GITHUB_ENV`  设置工作流程中的以下步骤可以使用的环境变量

  ```yaml
  steps:
    - name: Hello world1
      run: echo  "TEST_ENV=true" >> $GITHUB_ENV
    - name: Hello world2
      run: echo $TEST_ENV
  ```

### 矩阵

#### 使用多个 Python

```yaml
strategy:
  matrix:
    python: [3.5, 3.6, 3.7, 3.8]
steps:
  - uses: actions/setup-node@v1
    with:
      python-version: ${{ matrix.python }}
```

#### 使用多个操作系统

```yaml
runs-on: ${{ matrix.os }}
strategy:
  matrix:
    os: [ubuntu-16.04, ubuntu-18.04]
    python: [3.5, 3.6, 3.7, 3.8]
steps:
  - uses: actions/setup-node@v1
    with:
      python-version: ${{ matrix.python }}
```

#### 在矩阵中使用环境变量

Example: [RT-Thread action](https://github.com/RT-Thread/rt-thread/blob/master/.github/workflows/action.yml)

```yaml
    strategy:
    # 设置为 true 时，如果任何 matrix 作业失败，GitHub 将取消所有进行中的作业。 默认值：true
      fail-fast: false
      matrix:
       legs:
         - {RTT_BSP: "CME_M7", RTT_TOOL_CHAIN: "sourcery-arm"}
         - {RTT_BSP: "apollo2", RTT_TOOL_CHAIN: "sourcery-arm"}
         - {RTT_BSP: "asm9260t", RTT_TOOL_CHAIN: "sourcery-arm"}
         - {RTT_BSP: "at91sam9260", RTT_TOOL_CHAIN: "sourcery-arm"} 
         - {RTT_BSP: "allwinner_tina", RTT_TOOL_CHAIN: "sourcery-arm"} 
         - {RTT_BSP: "efm32", RTT_TOOL_CHAIN: "sourcery-arm"} 
         - {RTT_BSP: "gd32e230k-start", RTT_TOOL_CHAIN: "sourcery-arm"} 
         - {RTT_BSP: "gd32303e-eval", RTT_TOOL_CHAIN: "sourcery-arm"}         
         - {RTT_BSP: "gd32450z-eval", RTT_TOOL_CHAIN: "sourcery-arm"}
         - {RTT_BSP: "gkipc", RTT_TOOL_CHAIN: "sourcery-arm"}
         - {RTT_BSP: "imx6sx/cortex-a9", RTT_TOOL_CHAIN: "sourcery-arm"} 
         - {RTT_BSP: "imxrt/imxrt1052-atk-commander", RTT_TOOL_CHAIN: "sourcery-arm"}  
         - {RTT_BSP: "imxrt/imxrt1052-fire-pro", RTT_TOOL_CHAIN: "sourcery-arm"}
         - {RTT_BSP: "imxrt/imxrt1052-nxp-evk", RTT_TOOL_CHAIN: "sourcery-arm"}
         - {RTT_BSP: "lm3s8962", RTT_TOOL_CHAIN: "sourcery-arm"}
         - {RTT_BSP: "lm3s9b9x", RTT_TOOL_CHAIN: "sourcery-arm"}
    steps:
      - uses: actions/checkout@v2
      - name: Set up Python
        uses: actions/setup-python@master
        with:
          python-version: 3.8
          
      - name: Bsp Scons Compile
        if: ${{ success() }}
        shell: bash
        env:
          RTT_BSP: ${{ matrix.legs.RTT_BSP }}
          RTT_TOOL_CHAIN: ${{ matrix.legs.RTT_TOOL_CHAIN }}
        run: |
          scons -C bsp/$RTT_BSP
```

### if 语法

Example: [RT-ThreadStudio action](https://github.com/RT-Thread-Studio/sdk-index/blob/master/.github/workflows/action.yml)

可以使用 `if` 条件阻止作业在条件得到满足之前运行。 可以使用任何支持上下文和表达式来创建条件

#### 主分支触发

检查触发工作流程的分支本仓库并且上一个步骤成功时才会执行，否则跳过该步骤：

```yaml
- name: Generate-Import-Compile
  shell: bash
  if: ${{ github.ref != 'refs/heads/master' && success() }}
  run: |
    cd ${{ github.workspace }}
```

#### 合并时触发

```
if: github.event.pull_request.merged
```

## Marketplace

你可以在 Github 的 Marketplace 发现很多 Actions, 将这些已经写好的 action 添加到你的 CI 文件中，会减少开发者的工作量

#### Checkout V2

[链接](https://github.com/actions/checkout)

> This action checks-out your repository under `$GITHUB_WORKSPACE`, so your workflow can access it.

```yaml
- uses: actions/checkout@v2
```

#### setup-python V2

[链接](https://github.com/actions/setup-python)

> This action sets up a Python environment for

```yaml
- name: Set up Python
  uses: actions/setup-python@master
  with:
    python-version: 3.8
```

#### Upload-Artifact v2

[链接](https://github.com/actions/upload-artifact)

> This uploads artifacts from your workflow allowing you to share data between jobs and store data once a workflow is complete.

```yaml
- name: Generate txt
  run: echo "hellowold" > test.txt
- name: Upload Results
  if: ${{ github.ref != 'refs/heads/master' && success() }}
  uses: actions/upload-artifact@v2
  continue-on-error: True
  with:
    name: check-report
    path: ${{ github.workspace }}/test.txt
```

#### file-existence-action

[链接](https://github.com/andstor/file-existence-action/)

> This is a GitHub Action to check for existence of files. It can be used for conditionally running workflow steps based on file(s) existence.

```yaml
name: "File existence check"

on: [push, pull_request]

jobs:
  file_existence:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v1

      - name: Check file existence
        id: check_files
        uses: andstor/file-existence-action@v1
        with:
          files: "package.json, LICENSE, README.md"

      - name: File exists
        if: steps.check_files.outputs.files_exists == 'true'
        # Only runs if all of the files exists
        run: echo All files exists!
```

#### Close Stale Issues and PRs

[链接](https://github.com/actions/stale)

> Warns and then closes issues and PRs that have had no activity for a specified amount of time.

```yaml
name: "Close stale issues"
on:
  schedule:
  - cron: "30 1 * * *"

jobs:
  stale:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/stale@v3
      with:
        repo-token: ${{ secrets.GITHUB_TOKEN }}
        stale-issue-message: 'Message to comment on stale issues. If none provided, will not mark issues stale'
        stale-pr-message: 'Message to comment on stale PRs. If none provided, will not mark PRs stale'
```

