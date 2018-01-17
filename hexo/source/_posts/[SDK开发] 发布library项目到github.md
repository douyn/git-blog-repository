---
title: 给library module添加远程依赖
date: 2018年1月17日15:23:16
---

# 给library module添加远程依赖

1. 创建library module项目
	
    File --> NEW --> New Module --> Library Module

2. 修改library module下的build.gradle文件

	1. 在library module项目根目录下创建xx.gradle文件

             // 1.maven-插件
            apply plugin: 'maven'

            // 2.maven-信息
            ext {// ext is a gradle closure allowing the declaration of global properties
                PUBLISH_GROUP_ID = 'com.flame'
                PUBLISH_ARTIFACT_ID = 'mySDK'
                PUBLISH_VERSION = android.defaultConfig.versionName
            }

            // 3.maven-输出路径
            uploadArchives {
                repositories.mavenDeployer {
                    //这里就是最后输出地址，在自己电脑上新建个文件夹，把文件夹路径粘贴在此
                    //注意”file://“ + 路径，有三个斜杠，别漏了
                    repository(url: "file:///Users/flame/Documents/sourceTree/mysdk")

                    pom.project {
                        groupId project.PUBLISH_GROUP_ID
                        artifactId project.PUBLISH_ARTIFACT_ID
                        version project.PUBLISH_VERSION
                    }
                }
            }

            //以下代码会生成jar包源文件，如果是不开源码，请不要输入这段
            //aar包内包含注释
            task androidSourcesJar(type: Jar) {
                classifier = 'sources'
                from android.sourceSets.main.java.sourceFiles
            }

            artifacts {
                archives androidSourcesJar
            }
        
	2. 在build.gradle中apply这个gradle配置

			apply from xx.gradle	
        
	3. 在gradle菜单中执行xx.gradle中的编译方法
    
    
3. 创建并上传生成的sdk项目到github

	使用git命令上传到github目录
    
4. 在应用项目中调用

	在app module的build.gradle根节点下添加repositories节点
    	repositories {
            jcenter()
            maven{url "https://github.com/flameandroid/mysdk/raw/master"}
        }
        
     在dependencies中添加compile
     
     	 compile('com.flame:mySDK:1.0')