---
layout: post
title: GitLab CI简单设置
permalink: /gitlab_ci_setting/
---

## 配置Runner

Runner可以理解为是跑sonar分析的机器，我配在了某机器上。关键是安装gitlab-runner命令。可看[参考](https://blog.csdn.net/magicpenta/article/details/106880267)的3.3。其中安装gitlab-runner办公网机器会因为网络问题没法下载，我们可以参考某机器上ip的/etc/yum.repos.d/runner_gitlab-runner.repo，加一个yum源即可成功下载安装。

## 配置.gitlab-ci.yml

```yaml
.sonar_scan_template:
  stage: SONAR_CODE_CHECK
  tags:
    - sonar-scanner
  script:
    - export
    - git clone -b test --single-branch https://git.***.com/***/devops/cicd-python.git
    - cd cicd-python && python3 sonarqube_scan.py --service-name $CI_PROJECT_NAME
stages:
  - SONAR_CODE_CHECK


test_branch_merges:
  extends: .sonar_scan_template
  only:
    variables:
      # 合并的目标分支是test
      - $CI_MERGE_REQUEST_TARGET_BRANCH_NAME == "test"
    refs:
      - merge_requests
  except:
    variables:
      - $CI_MERGE_REQUEST_SOURCE_BRANCH_NAME == "master"

# sonar_scan_4_test_pushes:
#   extends: .sonar_scan_template
#   only:
#     variables:
#       # 提交的分支为test
#       - $CI_COMMIT_REF_NAME == "test"
#     refs:
#       - pushes
      
# sonar_scan_4_master_pushes:
#   extends: .sonar_scan_template
#   only:
#     variables:
#       # 提交的分支为master
#       - $CI_COMMIT_REF_NAME == "master"
#     refs:
#       - pushes

master_branch_merges:
  extends: .sonar_scan_template
  only:
    variables:
      # 合并的目标分支是master
      - $CI_MERGE_REQUEST_TARGET_BRANCH_NAME == "master"
    refs:
      - merge_requests
  except:
    variables:
      - $CI_MERGE_REQUEST_SOURCE_BRANCH_NAME == "test"
```

其基本内容很容易懂。无非就是only和except，其中特别注意的是每个only或except下面的“-”后带的表达式，是以“或”的关系，而variables和refs是“与”的关系。

如下段代码，是名为test_branch_merges的job，顾名思义就是test分支合并时候产生的job，其执行的条件是，only情况
$CI_MERGE_REQUEST_TARGET_BRANCH_NAME == "test"（合并请求时目标分支为test） AND merge_requests（合并请求） AND except情况
$CI_MERGE_REQUEST_SOURCE_BRANCH_NAME == "master"（合并请求时目标分支为master）。翻译白话就是：当发起合并请求是，目标分支为test并且源分支不为master的时候，就触发test_branch_merges任务。

```yaml
test_branch_merges:
  extends: .sonar_scan_template
  only:
    variables:
      # 合并的目标分支是test
      - $CI_MERGE_REQUEST_TARGET_BRANCH_NAME == "test"
    refs:
      - merge_requests
  except:
    variables:
      - $CI_MERGE_REQUEST_SOURCE_BRANCH_NAME == "master"
```

extends字段其实就是把公用代码抽取成.sonar_scan_template，几个job能通过extends复用。

```yaml
.sonar_scan_template:
  stage: SONAR_CODE_CHECK
  tags:
    - sonar-scanner
  script:
    - export
    - git clone -b test --single-branch https://git.***.com/***/devops/cicd-python.git
    - cd cicd-python && python3 sonarqube_scan.py --service-name $CI_PROJECT_NAME
```

**在这里关键的tags，是用来指定该job要到哪台Runner执行。
具体语法规则可查看[官方文档](http://repositories.compbio.cs.cmu.edu/help/ci/yaml/README.md)**


## .gitlab-ci.yml统一管理

	
因为需要在每个服务都加上这个文件（有没有方便的方式能给每个服务每个分支都加上这个文件的方法，暂时没去查），不可能后期需要变更的时候每个服务的文件都需要改一遍，因此可以通过include字段来引用上面所说的文件，以便于后期维护CI。其代码如下，内容很简单不再赘述：
```yaml
include:
	- project: '***/devops/ci-templates'
	  file: 'sonar-ci-template.yml'
```
