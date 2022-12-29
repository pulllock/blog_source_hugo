---
title: Maven源码学习记录
date: 2020-07-29 22:12:40
categories: Maven
tags: 
- Maven

---

Maven源码学习过程中一些重要的知识点记录。

<!--more-->

# plexus-classworlds

在maven的启动脚本中有如下的代码：

```bash
// ...
CLASSWORLDS_JAR=`echo "${MAVEN_HOME}"/boot/plexus-classworlds-*.jar`
CLASSWORLDS_LAUNCHER=org.codehaus.plexus.classworlds.launcher.Launcher
// ...

exec "$JAVACMD" \
  $MAVEN_OPTS \
  $MAVEN_DEBUG_OPTS \
  -classpath "${CLASSWORLDS_JAR}" \
  "-Dclassworlds.conf=${MAVEN_HOME}/bin/m2.conf" \
  "-Dmaven.home=${MAVEN_HOME}" \
  "-Dlibrary.jansi.path=${MAVEN_HOME}/lib/jansi-native" \
  "-Dmaven.multiModuleProjectDirectory=${MAVEN_PROJECTBASEDIR}" \
  ${CLASSWORLDS_LAUNCHER} "$@"
```

这段代码的意思是使用java命令启动`org.codehaus.plexus.classworlds.launcher.Launcher`，并指定了classpath为maven安装目录下的`/boot/plexus-classworlds-*.jar`，设置了如下几个系统属性：

- `classworlds.conf`：系统属性，plexus-classworlds的配置文件，在maven安装目录的`bin/m2.conf`

- `maven.home`：系统属性，指定了maven主目录

其中maven安装目录下的`boot/`目录下只有一个`plexus-classworlds-2.6.0.jar`文件。maven安装目录下的`bin/m2.conf`文件内容如下：

```
main is org.apache.maven.cli.MavenCli from plexus.core

set maven.conf default ${maven.home}/conf

[plexus.core]
load       ${maven.conf}/logging
optionally ${maven.home}/lib/ext/*.jar
load       ${maven.home}/lib/*.jar
```

这段代码和配置的大概意思是：`plexus-classworlds-2.6.0.jar`中`org.codehaus.plexus.classworlds.launcher.Launcher`类的`main`方法使用`bin/m2.conf`配置文件来启动Maven，这段配置文件的说明如下：

- `main is org.apache.maven.cli.MavenCli from plexus.core`：maven的main方法在`org.apache.maven.cli.MavenCli`中，对应的jar包在配置文件的下面的`[plexus.core]`中指定

- `set maven.conf default ${maven.home}/conf`：设置系统属性，指定maven默认配置文件目录

- `[plexus.core]`中指定了需要加载的文件

plexus-classworlds源码很少，源码解析可以参考：https://github.com/dachengxi/plexus-classworlds

# Maven启动类

在`m2.conf`中给出了启动类：`org.apache.maven.cli.MavenCli`并使用`plexus-classworlds`进行加载启动。