---
layout: post
title:  "FlowDroid-解析应用的Manifest"
tags:   StaticAnalysis AndroidManifest.xml
date:   2019-12-28 10:10:35 +0800
categories: [FlowDroid]
---



本文的主要内容是初识Flowdroid，并使用Flowdroid能够分析Manifest文件。

## Flowdroid

Flowdroid主页：[点我，我就是主页](https://blogs.uni-paderborn.de/sse/tools/flowdroid/)

FlowDroid是一款使用Java实现的针对Android的静态污点分析框架，可以对Android程序进行静态分析，相比于Soot来说，操作使用起来更简单，也避免了一些操作出现重复造轮子的情况。并且因为Android程序有组件、生命周期、异步等等的情况，使用Soot来解析的话，就需要自己再重复的处理这些，但是Flowdroid针对android的优化，自己写wrapper来模拟生命周期，构建完整的CFG。

## 添加依赖

Flowdroid所有release的jar包以及其他资源在此处下载：[点我](https://github.com/secure-software-engineering/FlowDroid/releases)

后续的学习内容需要导入以下jar包：

![image-20201019155229778](https://raw.githubusercontent.com/ARP2019/ImageUpload/master/img/2020-10-19/image-20201019155229778.png)

#### 导入过程

因为本jar包没有在maven仓库中，所以需要本地导入，导入过程如下（仅供参考）：

1：新建与src同级目录，并将jar包按此结构复制到对应版本文件夹中

![image-20201019160456771](https://raw.githubusercontent.com/ARP2019/ImageUpload/master/img/2020-10-19/image-20201019160456771.png)

2: maven-metadata-local.xml

让maven可以找到我们以这样方式导入的jar包

```xml
<?xml version="1.0" encoding="UTF-8"?>
<metadata>
    <groupId>ca.mcgill.sable</groupId>
    <artifactId>soot-infoflow</artifactId>
    <versioning>
        <release>2.8</release>
        <versions>
            <version>2.7.1</version>
            <version>2.8</version>
        </versions>
        <lastUpdated>20201015</lastUpdated>
    </versioning>
</metadata>
```

3：pom.xml

```xml
    <dependencies>
        <dependency>
            <groupId>ca.mcgill.sable</groupId>
            <artifactId>soot-infoflow</artifactId>
            <version>2.7.1</version>
        </dependency>
    </dependencies>

    <repositories>
        <repository>
            <id>local-deps</id>
            <url>file://${project.basedir}/lib/</url>
        </repository>
    </repositories>
```



## 解析Manifest？
代码很简单:
  1: 指定apk文件的路径。
  2: 实例化 ```ProcessManifest```
  3: getPermissions()：获取APP开发过程中使用到的权限
  4: targetSdkVersion：获取targetSdkVersion

```java
/**解析AndroidManift.xml*/
public void analysisManifest(String apkPath) throws IOException, XmlPullParserException
{
    //实例化processManifest
    ProcessManifest proManifest = new ProcessManifest(apkPath);
    //获取targetSdkVersion
    System.out.println(proManifest.targetSdkVersion());
    //获取应用在Manifest文件中声明的权限（也包括库声明的，最后一起打包了）
    System.out.println(proManifest.getPermissions());
}
```

## 后续有何用？
先看图

  ![](https://raw.githubusercontent.com/ARP2019/ImageUpload/master/img/markdown-img-paste-20191227212748655.png)

获取到的权限：
  - 这里获取到的权限没有区分是否为危险权限，但是此处的权限可以有助于获取应用自身的API-Permission Mapping

targetSdkVersion
  - 这个是APK开发过程中设置的一个重要的属性

## processManifest
本类主要是处理Manifest的，就是Android最后打包出来的整体配置文件内容,此处的打包即应用引用的library和自身的Manifest所合并出来的内容。
下图就是一个apk最后打包好的AndroidManifest的模样：
![](https://raw.githubusercontent.com/ARP2019/ImageUpload/master/img/markdown-img-paste-20191228090353864.png)

- processManfest.targetSdkVersion
```java
public int targetSdkVersion() {
    List<AXmlNode> usesSdk = this.manifest.getChildrenWithTag("uses-sdk");
    //获取uses-sdk标签
    if (usesSdk != null && !usesSdk.isEmpty()) {
        AXmlAttribute<?> attr = ((AXmlNode)usesSdk.get(0)).getAttribute("targetSdkVersion");
        //获取targetSDKVersion标签内容
        if (attr == null) {
            return -1;
        } else {
            return attr.getValue() instanceof Integer ? (Integer)attr.getValue() : Integer.parseInt("" + attr.getValue());
        }
    } else {
        return -1;
    }
}
```
- processManfest.getPermissions
```java
public Set<String> getPermissions() {
    List<AXmlNode> usesPerms = this.manifest.getChildrenWithTag("uses-permission");
    Set<String> permissions = new HashSet();
    Iterator var3 = usesPerms.iterator();

    while(true) {
        while(var3.hasNext()) {
            AXmlNode perm = (AXmlNode)var3.next();
            AXmlAttribute<?> attr = perm.getAttribute("name");
            if (attr != null) {
                permissions.add((String)attr.getValue());
            } else {
                Iterator var6 = perm.getAttributes().values().iterator();

                while(var6.hasNext()) {
                    AXmlAttribute<?> a = (AXmlAttribute)var6.next();
                    if (a.getType() == 3 && a.getName().isEmpty()) {
                        permissions.add((String)a.getValue());
                    }
                }
            }
        }

        return permissions;
    }
}
```

除此之外，此类也提供了许多方法，可以对Manifest文件进行查询以及修改。