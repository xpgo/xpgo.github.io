---
layout: post
title: Boyuan Private Maven Repository!
author: hookszhang
---

这是一个完整的 mvnrepository.com 代理，你可以用此代替官方版本，并且可以下载 Boyuan 的 Java 组件

- Nexus 首页: [http://180.167.77.60:30001](http://180.167.77.60:30001)
- Maven Public: http://180.167.77.60:30001/repository/maven-public

## Maven

#### 在 POM 中配置 Nexus 仓库

```xml
<project>
    ...
    <repositories>
        <repository>
            <id>nexus</id>
            <name>Nexus</name>
            <url>http://180.167.77.60:30001/repository/maven-public</url>
            <releases><enabled>true</enabled></releases>
            <snapshots><enabled>true</enabled></snapshots>
        </repository>
    </repositories>

    <pluginRepositories>
        <pluginRepository>
            <id>nexus</id>
            <name>Nexus</name>
            <url>http://180.167.77.60:30001/repository/maven-public</url>
            <releases><enabled>true</enabled></releases>
            <snapshots><enabled>true</enabled></snapshots>
        </pluginRepository>
    </pluginRepositories>
    ...
</project>
```

这样的配置只对当前的Maven项目生效，在实际应用中，我们往往想要通过一次配置就能让本机所有的Maven项目都使用自己的Maven私服。这个时候可能会想到setting.xml文件，该文件中的配置对所有本机Maven项目有效，但是settings.xml并不支持直接配置repositories和pluginRepositories。所幸Maven还提供了Profile机制，能让用户将仓库配置放到setting.xml中的Profile中。

#### 在 settings.xml 中配置 Nexus 仓库

```xml
<settings>
    ...
    <profiles>
        <profile>
            <id>nexus</id>
            <repositories>
                <repository>
                    <id>nexus</id>
                    <name>Nexus</name>
                    <url>http://180.167.77.60:30001/repository/maven-public</url>
                    <releases><enabled>true</enabled></releases>
                    <snapshots><enabled>true</enabled></snapshots>
                </repository>
            </repositories>

            <pluginRepositories>
                <pluginRepository>
                    <id>nexus</id>
                    <name>Nexus</name>
                    <url>http://180.167.77.60:30001/repository/maven-public</url>
                    <releases><enabled>true</enabled></releases>
                    <snapshots><enabled>true</enabled></snapshots>
                </pluginRepository>
            </pluginRepositories>
        </profile>
    </profiles>
    <activeProfiles>
        <activeProfile>nexus</activeProfile>
    </activeProfiles>
    ...
</settings>
```

这样的配置已经能让本机所有的Maven项目从Nexus私服下载构件，但Maven除了从Nexus下载构件之外，还会不时地访问中央仓库central，我们希望的是所有Maven下载请求都仅仅通过Nexus，以全面发挥私服的作用。这就需要借助于Maven镜像配置了。可以创建一个匹配任何仓库的镜像，镜像的地址为私服，这样，Maven对任何仓库的构件下载请求都会转到私服中。

#### 配置镜像让 Maven 只使用私服

```xml
<settings>
    ...
    <mirrors>
        <mirror>
            <id>nexus</id>
            <mirrorOf>*</mirrorOf>
            <url>http://180.167.77.60:30001/repository/maven-public</url>
        </mirror>
    </mirrors>
    <profiles>
        <profile>
            <id>nexus</id>
            <repositories>
                <repository>
                    <id>central</id>
                    <url>http://central</url>
                    <releases><enabled>true</enabled></releases>
                    <snapshots><enabled>true</enabled></snapshots>
                </repository>
            </repositories>

            <pluginRepositories>
                <pluginRepository>
                    <id>central</id>
                    <url>http://central</url>
                    <releases><enabled>true</enabled></releases>
                    <snapshots><enabled>true</enabled></snapshots>
                </pluginRepository>
            </pluginRepositories>
        </profile>
    </profiles>
    <activeProfiles>
        <activeProfile>nexus</activeProfile>
    </activeProfiles>
    ...
</settings>
```

## Gradle

一个简单的办法，修改项目根目录下的build.gradle，将jcenter()或者mavenCentral()替换掉即可：

```
allprojects {
    repositories {
        maven{ url 'http://180.167.77.60:30001/repository/maven-public/'}
    }
}
```

但是架不住项目多，难不成每个都改一遍么？ 自然是有省事的办法，将下面这段Copy到名为init.gradle文件中，并保存到 USER_HOME/.gradle/文件夹下即可。

```
allprojects{
    repositories {
        def REPOSITORY_URL = 'http://180.167.77.60:30001/repository/maven-public/'
        all { ArtifactRepository repo ->
            if(repo instanceof MavenArtifactRepository){
                def url = repo.url.toString()
                if (url.startsWith('https://repo1.maven.org/maven2') || url.startsWith('https://jcenter.bintray.com/')) {
                    project.logger.lifecycle "Repository ${repo.url} replaced by $REPOSITORY_URL."
                    remove repo
                }
            }
        }
        maven {
            url REPOSITORY_URL
        }
    }
}
```

init.gradle文件其实是Gradle的初始化脚本(Initialization Scripts)，也是运行时的全局配置。
更详细的介绍请参阅 [http://gradle.org/docs/current/userguide/init_scripts.html](http://gradle.org/docs/current/userguide/init_scripts.html)
