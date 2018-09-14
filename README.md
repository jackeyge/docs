# jenkins-openshift-django

本文通过一个django demo程序，演示Jenkins结合openshift的CI/CD流程，是通过在源码外构建pipeline的方式，jenkins的pipeline调用Openshift的S2I、BC、DC等。

## 流程架构图

## 步骤

### 1、制作Jenkins slave镜像

由于此项目使用的是Python语言，过程中增加了单元测试，因此在openshift[官方基础镜像](https://github.com/openshift/jenkins/tree/master/slave-base)上做了修改，安装了一些需要的模块，通过[修改后的Dockerfile创建](./file/jenkins/slave-base/Dockerfile)Jenkins slave镜像。

### 2、启动Jenkins

- 通过openshift catalog 里的Jenkins模板启动Jenkins

- 登录Jenkins，安装Cobertura Plugin插件，用来展示Python代码测试覆盖率报表

- 在openshift导入Jenkins slave configmap [yaml文件](file/jenkins/slave-base/openshift-jenkins-slave-pod-template.yaml),确保使用第一步创建的镜像

  ![](images/jenkins-slave-cm.png)

  

  

### 3、导入template

### 4、导入Jenkins pipeline

### 5、配置githubwebhook

### 6、测试流程

