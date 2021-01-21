# Jenkins

> 构建伟大，无所不能。

Jenkins 是开源 CI&CD 软件领导者， 提供超过 1000 个插件来支持构建、部署、自动化， 满足任何项目的需要。

## 多分支流水线

在 Jenkins 上，可以创建多种类型的任务，例如 自由风格、GitHub Flow 等。

而根据我司产品及项目的业务情况，我们统一采取`多分支流水线`的任务类型，而流水线即`Pipeline`，通过 Jenkinsfile 进行描述，它将会落到任何一个需要持续构建或发布的分支上。

当前，通过 w6s-cli 创建的项目，默认会初始化一个 Jenkinsfile 文件，并且已制定基本的项目依赖安装及构建任务。

```jenkinsfile
pipeline {
    agent { docker 'node:10' }
    stages {
        stage('build') {
            steps {
                sh 'npm --version'
            }
        }
    }
}
```

`Jenkinsfile`的详细编码方法，请查看[官网教程](https://www.jenkins.io/zh/doc/pipeline/tour/hello-world/)。