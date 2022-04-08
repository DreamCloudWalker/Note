### 打带符号信息的so

build.gradle里带上doNotStrip信息。这样可以让打入包或aar的so带上符号信息。最终发包前，把符号信息去掉。这样即不会增加包大小，也可以用来分析线上崩溃堆栈。

```groovy
android {
  packagingOptions {
    doNotStrip '**/armeabi-v7a/*.so'
    doNotStrip '**/arm64-v8a/*.so'
    doNotStrip '**/x86_64/*.so'
    doNotStrip '**/x86/*.so'
  }
}
```

查看一个so是否带符号信息，可以用命令： nm -D ***.so



### 配置assembleRelease

```	groovy
android {
  signingConfigs {    // 签名的配置
    release {
      storeFile file("test.jks")
      storePassword '123456'
      keyAlias 'test'
      keyPassword '123456'
    }
  }
  buildTypes {
    release {
      minifyEnabled true
      proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
      signingConfig signingConfigs.release
    }
  }
}
```



### 打aar包并上传maven

配置uploadArchives

```groovy
uploadArchives{
    repositories.mavenDeployer{
        println properties
        repository(url: "https://maven.xxx.com/xxx/repositories/releases") {
            authentication(userName: "admin", password: "123456")
        }
        snapshotRepository(url: "https://maven.xxx.com/xxx/repositories/snapshots") {
            authentication(userName: "admin", password: "123456")
        }
        pom.artifactId = libraryName()
        pom.name = 'editor'
        pom.groupId = groupName()
        pom.packaging = 'aar'
        pom.version = versionName()
    }
}

tasks.whenTaskAdded { task ->
    if (task.name == "assembleDebug" || task.name == "assembleRelease") {
        task.mustRunAfter 'cleanOutput'
    }
    if (task.name == 'output') {
        task.mustRunAfter 'assembleRelease'
    }
    if (task.name == "uploadArchives") {	// 执行前先assembleRelease
        task.mustRunAfter 'assembleRelease'
    }
}

def groupName() {
    "com.dengjian.test"
}

def libraryName() {
    "testlib"
}

def versionName() {
    "1.0.0.x"
}
```

后面其他app引用这个aar，可以用

```groovy
api "com.dengjian.test:1.0.0.x@aar"
```