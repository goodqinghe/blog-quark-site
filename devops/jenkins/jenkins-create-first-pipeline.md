---
title: Jenkins 创建第一个流水线
---

# Jenkins 创建第一个流水线

这篇记录从零创建一个可重复执行的 Jenkins Pipeline。目标不是堆复杂配置，而是先让构建、测试和结果反馈形成一条稳定链路。

## 创建 Pipeline 任务

在 Jenkins 首页选择“新建任务”，输入任务名称，选择 **Pipeline**，然后确认创建。

建议先使用“Pipeline script”直接粘贴一段最小脚本，验证 Jenkins Agent、工具链与日志是否正常；脚本稳定后再迁移到代码仓库中的 `Jenkinsfile`。

## 最小流水线

```groovy
pipeline {
  agent any

  stages {
    stage('检出代码') {
      steps {
        checkout scm
      }
    }

    stage('构建') {
      steps {
        sh 'echo "开始构建"'
      }
    }

    stage('测试') {
      steps {
        sh 'echo "开始测试"'
      }
    }
  }
}
```

## 配置代码来源

如果任务连接 Git 仓库，在“Pipeline”设置中选择 **Pipeline script from SCM**：

1. SCM 选择 Git；
2. 填写仓库地址和凭据；
3. 分支先填 `main`；
4. Script Path 填 `Jenkinsfile`。

这样，每次构建都使用代码仓库中同一次提交对应的流水线定义。

## 第一次执行时要看什么

- Agent 是否成功分配；
- Git 凭据和仓库地址是否有效；
- 构建环境中是否有需要的 JDK、Node.js、Docker 等工具；
- Console Output 是否能清楚显示失败阶段。

先让流程足够短、日志足够清晰。部署、制品上传和通知，等构建与测试稳定后再依次加入。
