
---
title: macbook (silicon) 运行kettle（data-integration）
tags:
---
macbook   silicon 运行kettle（data-integration）

kettle在windows及intel电脑运行正常，但是在silicon芯片的macos上执行是提示：不支持arm64。搜索 发现解决方案： https://medium.com/@gabriella.mayang/how-to-install-and-open-pentaho-data-integration-on-macbook-m2-apple-silicon-4081c2875a02



# Step 1: Download JDK and JRE

I tried JDK from Oracle but it failed. Now the only JDK that I am using and proven successful in opening PDI in my M2 is the one from **Eclipse Temurin OpenJDK.**

1. Click here to download https://adoptium.net/temurin/releases/?variant=openjdk8&os=mac
2. Choose JDK .pkg

#  Step 2: Download MacOSX X86 JAR

1. Click here to download https://mega.nz/file/VFhgnS6I#s4zUK81rXGdXcpJMuzUvDOHoixZmAl4jyXd0yMU11F0
2. Move the file to ‘libswt/osx64'

就是把下载的文件放到 data-integration/libswt/osx64 目录下。

# Step 4: Create New Terminal Profile for Intel

注：这里是苹果原生的terminal，不是iterm。

1. Open Terminal > Settings
2. Click the ‘**+’** button to add new profile and name it as “Rosetta Intel”. You can choose another name but make sure the new profile name indicates the profile usage is for Intel-based apps
3. Go to ‘Shell’ tab and paste this in the ‘Run Command’ field:

```
env /usr/bin/arch -x86_64 /bin/zsh --login
```

Uncheck ‘Run inside shell’. Uncheck the checkbox would prevent running the shell twice, which could bloat your environment variables since ~/.zshrc gets run twice

### 最后 JAVA_HOME

因为我用了多版本的jdk，所以需要单独设置一个java环境

```
export JAVA_HOME=/Library/Java/JavaVirtualMachines/temurin-8.jdk/Contents/Home

```

然后就成功了。