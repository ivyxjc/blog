---
author: ivyxjc
date: 2018-12-08
title: 将gradle项目部署到Maven Central中
category: 效率
tags: [maven]
keywords:
description: 本文主要介绍如何配置Kafka认证
toc: true
---

本文主要介绍如何一个由gradle构建的项目部署到Maven Central.
<!--nomore-->

网上大部分都是介绍如何将由maven构建的项目部署到Maven Central。与Gradle相关的比较少。

## 申请账号
前往 [sonatype](https://issues.sonatype.org/)申请账号。

申请完，Create Issue。

按照[这个模板](https://issues.sonatype.org/browse/OSSRH-44681)填。

这一块比较简单，网上教程也比较多。

Create Issue结束后，官方会需要你证明你拥有相对应的domain。

证明有以下3个途径： 

1. Add a TXT record to your DNS referencing this JIRA ticket: OSSRH-44681 (Fastest)
2. Setup a redirect to your Github page (if it does not already exist)
3. Send an email to central@sonatype.com referencing this issue from a ... email address

证明完毕之后，你就可以发布包了。

你就可以做下面几件事了：

1. Deploy snapshot artifacts into repository https://oss.sonatype.org/content/repositories/snapshots
2. Deploy release artifacts into the staging repository https://oss.sonatype.org/service/local/staging/deploy/maven2
3. Promote staged artifacts into repository 'Releases'
4. Download snapshot and release artifacts from group https://oss.sonatype.org/content/groups/public
5. Download snapshot, release and staged artifacts from staging group https://oss.sonatype.org/content/groups/staging


## 构建Gradle

下面主要内容基于 
[官方英文教程](https://central.sonatype.org/pages/gradle.html)，加上一些个人构建时候的一些收获。

### build.gralde 文件修改

#### 引入plugin

```groovy
apply plugin: 'maven'
apply plugin: 'signing'
```

```groovy
task javadocJar(type: Jar) {
    classifier = 'javadoc'
    from javadoc
}

task sourcesJar(type: Jar) {
    classifier = 'sources'
    from sourceSets.main.allSource
}

artifacts {
    archives javadocJar, sourcesJar
}

```

#### 引入UploadArchives task

引入`UploadArchives`这个task，记住更改里面的个人相关信息。

其中有`ossrhUsername`和`ossrhPassword`这两个变量是定义在`gradle.properties`中的。

```groovy
uploadArchives {
    repositories {
        mavenDeployer {
            beforeDeployment { MavenDeployment deployment -> signing.signPom(deployment) }

            repository(url: "https://oss.sonatype.org/service/local/staging/deploy/maven2/") {
                authentication(userName: ossrhUsername, password: ossrhPassword)
            }

            snapshotRepository(url: "https://oss.sonatype.org/content/repositories/snapshots/") {
                authentication(userName: ossrhUsername, password: ossrhPassword)
            }

            pom.project {
                name 'Example Application'
                packaging 'jar'
                // optionally artifactId can be defined here 
                description 'A application used as an example on how to set up 
                pushing its components to the Central Repository . '
                url 'http://www.example.com/example-application'

                scm {
                    connection 'scm:git:git@github.com:username/project.git'
                    developerConnection 'scm:git:git@github.com:username/project.git'
                    url 'https://github.com/username/project'
                }

                licenses {
                    license {
                        name 'The Apache License, Version 2.0'
                        url 'http://www.apache.org/licenses/LICENSE-2.0.txt'
                    }
                }

                developers {
                    developer {
                        id 'manfred'
                        name 'Manfred Moser'
                        email 'manfred@sonatype.com'
                    }
                }
            }
        }
    }
}
```

#### 编写gradle.properties
主要是将一些认证信息填在这里。（这些信息不要加入到版本管理中)。
```properties
以下3个信息怎么来下一章节来讲
signing.keyId=YourKeyId
signing.password=YourPublicKeyPassword
signing.secretKeyRingFile=PathToYourKeyRingFile

ossrhUsername=your-jira-id  你在sonatype申请的账号的用户名
ossrhPassword=your-jira-password  你在sonatype申请的账号的密码
```


## 生成GPG加密信息

windows中可以安装gpg4win来生成相关信息。但是我个人在windows10中并没有能够打开。
所以我使用了WSL来生成相关信息。如果你的系统是Linux也可以。

1. 执行`gpg --gen-key`, 按照提示的信息填入密码，用户名等信息，这些信息记录下来。这里填入的密码就是上面`gradle.properties`中的`signing.password`。
2. 执行`gpg --list-keys`, 可以看到
```
/root/.gnupg/pubring.gpg
pub   2048R/B98765 2018-12-08
uid                  
sub 2048R/A123456 
```
3. 第一行便是对应的公钥文件位置，`pug`后面的`B98765`便是public key Id，这个id也就是上面`gradle.properties`中的`signing.keyId`
4. 执行` gpg --keyserver hkp://pool.sks-keyservers.net --recv-keys B98765`将公钥发送到`hkp://pool.sks-keyservers.net`。
5. 记录下`/root/.gnupg/`中`secring.png`的位置，这个位置便是上面`gradle.properties`中的`signing.secretKeyRingFile`的值。
   
## 发布过程

当上述步骤全部完成时，可以直接执行`gradle uploadArchives`。


### 发布Snapshot版本
如果你的版本是`snapshot`的，你可以直接在`https://oss.sonatype.org/content/repositories/snapshots`中看到你的包。

### 发布Release版本
如果你的版本是`release`版本。
登录`https://oss.sonatype.org/#welcome`,选择`Staging Repositories`，然后在右边用`groupId`去搜索。
这样会找到你的项目。选中你的项目close然后confirm。过一会再来寻找一次该构建，点击`Release`在Confirm。过一会就应该能在`https://oss.sonatype.org/content/groups/public`中看到你的项目了。