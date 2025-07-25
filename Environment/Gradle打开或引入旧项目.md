总结版本



优先看JVM版本问题



部分三方Github上的库需要在setting.gradle.kts加入：
maven { url = uri("[https://jitpack.io"](https://jitpack.io")) }

```groovy
pluginManagement {
    repositories {
        google {
            content {
                includeGroupByegex("com\\.android.*")
                includeGroupByRegex("com\\.google.*")
                includeGroupByRegex("androidx.*")
            }
        }
        maven { url = uri("https://maven.aliyun.com/repository/public") }
        mavenCentral()
        gradlePluginPortal()
    }
}
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        maven { url = uri("https://maven.aliyun.com/repository/public") }
        maven { url = uri("https://jitpack.io") }
        mavenCentral()
    }
}

rootProject.name = "TransitionAnim"
include(":app")

```


