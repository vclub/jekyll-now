---
layout: post
title: Jenkins pipeline 解析Json
tags: [Jenkins]
---

如果要将数据转换成JSON,需要引用

```import groovy.json.JsonOutput```

[内容参考](https://stackoverflow.com/questions/44707265/jenkins-pipeline-groovy-json-parsing)


Jenkins要安装 Pipeline Utility Steps 插件

 [使用说明地址](https://github.com/jenkinsci/pipeline-utility-steps-plugin/blob/master/docs/STEPS.md)

### Configuration Files

* readProperties - Read java properties from files in the workspace or text. (help)
* readManifest - Read a Jar Manifest. (help)
* readYaml - Read YAML from files in the workspace or text. (help)
* writeYaml - Write a YAML file from an object. (help)
* readJSON - Read JSON from files in the workspace or text. (help)
* writeJSON - Write a JSON object to a files in the workspace. (help)