---
title: Jenkins流水线Jenkinsfile编写
tags: [软件开发, 系统运维, Java, Jenkins]
---

## 前言

Jenkinsfile文件描述Jenkins项目构建的步骤，以达到将项目构建的配置描述转移到文件中的目的。

## Jenkinsfile例子

```groovy
pipeline {  
    agent any  
    // 指定使用的JDK和Maven，关联的是全局工具配置里面的名称
    tools {  
        jdk 'JDK'  
        maven 'Maven'  
    }
    // 指定构建使用的参数  
    parameters {  
        gitParameter branch: '',  
            branchFilter: '.*',  
            defaultValue: 'origin/test',  
            description: '分支或标签',  
            name: 'branch',  
            quickFilterEnabled: false,  
            selectedValue: 'NONE',  
            sortMode: 'NONE',  
            tagFilter: '*',  
            type: 'GitParameterDefinition'  
        choice choices: ['odm-test', 'odm-dev', 'odm-prod'],  
            description: '''部署到开发环境，选择odm-dev  
部署到测试环境，选择odm-test  
部署到生产环境，选择odm-prod''',  
            name: 'deploy'  
    }
    // 流水线分为5个阶段  
    stages {  
        stage('pull') {  
            // 拉取代码阶段
            steps {  
                checkout poll: false,  
                    scm: scmGit(branches: [[name: '${branch}']],  
                        extensions: [],  
                        userRemoteConfigs: [[credentialsId: 'ac63e72e-96c4-4870-a5bb-0864d699e703',  
                            url: 'http://192.168.2.220/gitlab-instance-f8b3317d/odm.git']])  
            }  
        }  
        stage('pre steps') {  
            // 构建前准备阶段
            steps {  
                // 使用Publish Over SSH插件，构建参数deploy指定要发送的服务器，变量名称与系统配置中的服务器名称关联
                // 这里不传输文件，只执行shell脚本，清理上一次部署留下的文件
                sshPublisher(publishers: [sshPublisherDesc(configName: "${params.deploy}",  
                    transfers: [sshTransfer(cleanRemote: false,  
                        excludes: '',  
                        execCommand: '''rm -rf /opt/odm  
mkdir -p /opt/odm''',  
                        execTimeout: 120000,  
                        flatten: false,  
                        makeEmptyDirs: false,  
                        noDefaultExcludes: false,  
                        patternSeparator: '[, ]+',  
                        remoteDirectory: '',  
                        remoteDirectorySDF: false,  
                        removePrefix: '',  
                        sourceFiles: '')],  
                    usePromotionTimestamp: false,  
                    useWorkspaceInPromotion: false,  
                    verbose: false)])  
            }  
        }  
        stage('build') {  
            // 构建阶段
            steps {  
                // 执行shell命令，使用Maven构建打包
                sh "mvn clean package"  
            }  
        }  
        stage('post steps') {  
            // 构建后处理阶段
            steps {  
                // 同样使用Publish Over SSH插件，将Maven构建后生成的jar包发送到服务器指定路径，并修改jar包名称
                sshPublisher(publishers: [sshPublisherDesc(configName: "${params.deploy}",  
                    transfers: [sshTransfer(cleanRemote: false,  
                        excludes: '',  
                        execCommand: 'mv /opt/odm/*.jar /opt/odm/odm.jar',  
                        execTimeout: 120000,  
                        flatten: false,  
                        makeEmptyDirs: false,  
                        noDefaultExcludes: false,  
                        patternSeparator: '[, ]+',  
                        remoteDirectory: '/opt/odm',  
                        remoteDirectorySDF: false,  
                        removePrefix: 'target',  
                        sourceFiles: 'target/*.jar')],  
                    usePromotionTimestamp: false,  
                    useWorkspaceInPromotion: false,  
                    verbose: false)])  
                // 从Maven构建内容中找到Dockerfile，发送到服务器指定路径
                sshPublisher(publishers: [sshPublisherDesc(configName: "${params.deploy}",  
                    transfers: [sshTransfer(cleanRemote: false,  
                        excludes: '',  
                        execCommand: '',  
                        execTimeout: 120000,  
                        flatten: false,  
                        makeEmptyDirs: false,  
                        noDefaultExcludes: false,  
                        patternSeparator: '[, ]+',  
                        remoteDirectory: '/opt/odm',  
                        remoteDirectorySDF: false,  
                        removePrefix: 'target/classes',  
                        sourceFiles: 'target/classes/Dockerfile')],  
                    usePromotionTimestamp: false,  
                    useWorkspaceInPromotion: false,  
                    verbose: false)])  
                // 从Maven构建内容中找到需要执行的shell脚本，发送到服务器指定路径，设置可执行权限，并执行
                sshPublisher(publishers: [sshPublisherDesc(configName: "${params.deploy}",  
                    transfers: [sshTransfer(cleanRemote: false,  
                        excludes: '',  
                        execCommand: '''chmod +x /opt/odm/*.sh  
/opt/odm/clean.sh  
/opt/odm/run.sh''',  
                        execTimeout: 120000,  
                        flatten: false,  
                        makeEmptyDirs: false,  
                        noDefaultExcludes: false,  
                        patternSeparator: '[, ]+',  
                        remoteDirectory: '/opt/odm',  
                        remoteDirectorySDF: false,  
                        removePrefix: 'target/classes/shell',  
                        sourceFiles: 'target/classes/shell/*.sh')],  
                    usePromotionTimestamp: false,  
                    useWorkspaceInPromotion: false,  
                    verbose: false)])  
                // 清理Jenkins工作空间
                cleanWs()  
            }  
        }  
    }  
}
```

## 流水线语法生成

新增一个流水线

![](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/202307201707099.png)

点击流水线语法辅助生成Jenkinsfile

![](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/202307201706563.png)

上文例子中使用到的流水线语法生成

Declarative Directive Generator
* tools: Tools

片段生成器
* sh: Shell Script
* checkout: Check out from version control
* sshPublisher: Send build artifacts SSH
* cleanWs: Delete workspace when build is done


## 参数化构建

流水线语法生成

Declarative Directive Generator
* parameters: Parameters

### Git参数

安装插件 Git Parameter Plug-In

```
parameters {
  gitParameter branch: '', branchFilter: '.*', defaultValue: 'origin/test', description: '分支或标签', name: 'branch', quickFilterEnabled: false, selectedValue: 'NONE', sortMode: 'NONE', tagFilter: '*', type: 'GitParameterDefinition'
}
```

```
checkout poll: false, scm: scmGit(branches: [[name: '${branch}']], extensions: [], userRemoteConfigs: [[credentialsId: 'ac63e72e-96c4-4870-a5bb-0864d699e703', url: 'http://192.168.2.220/gitlab-instance-f8b3317d/dms.git']])
```

### 选项参数

```
parameters {
  choice choices: ['odm-test', 'odm-dev', 'odm-prod'], description: '''部署到开发环境，选择odm-dev
部署到测试环境，选择odm-test
部署到生产环境，选择odm-prod''', name: 'deploy'
}
```

参数插值使用时，需要在双引号内

```
sshPublisher(publishers: [sshPublisherDesc(configName: "${params.deploy}", transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '''rm -rf /opt/odm
mkdir -p /opt/odm''', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '', remoteDirectorySDF: false, removePrefix: '', sourceFiles: '')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
```