# Jenkins

<div style="clear: both; overflow: hidden;">
  <img :src="$withBase('/kongfu.png')" width="110" alt="kongfu" style="float: left;" />
    <h4>构建伟大，无所不能。</h4>
    <p>Jenkins 是开源 CI&CD 软件领导者， 提供超过 1000 个插件来支持构建、部署、自动化， 满足任何项目的需要。</p>   
</div>

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

## 上传到 OSS

::: tip 什么是 OSS？

对象存储服务（Object Storage Service，OSS）是一种海量、安全、低成本、高可靠的云存储服务，适合存放任意类型的文件。容量和处理能力弹性扩展，多种存储类型供选择，全面优化存储成本。
:::

目前我们使用的是阿里云的 OSS 服务，当 CI 构建完毕，产出成品后，可以传输到 OSS 上，然后运维会通过脚本自动拉取部署。

以下是 Jenkinsfile 的配置，同样是使用 Docker 镜像：

```js
stage('OSS Upload') {
    agent {
        docker { 
            image 'registry.baidubce.com/workplus/agent-oss:1.7.0'
            reuseNode true
        }
    }
    steps {
        sh "ossutil64 cp -e $OSS_ENDPOINT -i $OSS_ACCESS_KEY_ID_USR -k $OSS_ACCESS_KEY_SECRET_USR -r -f ./product.zip oss://w6s-fe/project-name/product.zip"
    }
}
```

因为上传需要密钥，所以还需要在环境变量中添加以下变量值：

```js
environment {
    OSS_ACCESS_KEY_ID = credentials('oss-access-key-id')
    OSS_ACCESS_KEY_SECRET = credentials('oss-access-key-secret')
}
```

而`OSS_ENDPOINT`无需添加，因为已添加到全局环境变量。


## 成品命名规范

为了可以更好地管理成品，请按照以下约定进行设置：

1. 以`${project-name}.${branch-name}.zip`作为成品的统一命名方式；
2. 以`oss://w6s-fe/${project-name}/`作为成品的存放目录。

> `project-name` 以 package.json 里的 name 为准，也就是需要遵循 npm 的包命名规范。