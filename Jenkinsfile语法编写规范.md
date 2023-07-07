---
attachments: [Clipboard_2023-07-05-10-37-05.png, Clipboard_2023-07-05-10-39-13.png]
title: Jenkinsfile语法编写规范
created: '2023-07-05T01:49:45.773Z'
modified: '2023-07-05T03:27:41.107Z'
---

# Jenkinsfile语法编写规范
## 1. @Library(库名) _
该指令用于加载共享库，_符号用于加载，不可省略。

## 2. env.JOB_NAME
env.JOB_NAME是jenkins中的内置的全局变量。
该全局变量除了JOB_NAME以外，还有BRANCH_NAME,BRANCH_IS_PIMARY,CHANGE_ID,CHANGE_URL,CHANGE_TITLE等。
全局变量|作用
-------|----
BRANCH_NAME|用于在多分支项目中，设置为正在构建的分支的名称
BRANCH_IS_PRIMARY|如果SCM源报告正在构建的分支是主分支，则该值设置为true，如果没有设置，可能会将多个分支报告为主分支。
JOB_NAME|构建的项目的名称,该方法获取的项目名称包含项目所在路径
JOB_BASE_NAME|不包含文件夹路径的项目的短名称
WORKSPACE|指定给作为工作区的生成的目录的绝对路径。
GIT_URL|远程URL，可以是多个URL地址。
NODE_NAME|如果构建在代理上，则为代理的名称；如果在内置节点上运行，则为“内置”
NODE_LABELS|为节点分配的以空格分隔的标签列表。

## 3.pipeline中的agent介绍
Pipeline 块定义了整个流水线中完成的所有工作。pipeline是jenkins中用于实施和集成持续交付的一套插件。

### 3.1 agent基础介绍
agent指示 Jenkins 为整个流水线分配一个执行器。
agent中的label指令用于为该执行器分配一个标签，标签名为label指令后提供的内容。
例如:
```
pipeline{
  agent any
}
```
agent any告诉Jenkins Master任意可用的agent都可以执行。
agent必须放在pipeline的顶层定义，或者在stages标签中可选定义，如果在stages标签中定义，意味着是在不同阶段中使用不同的agent。

## 4.pipeline中的options介绍
option指令的作用是配置整个pipeline本身的选项，根据选项的内容，可放在pipeline块中或者stage块中。
options指令允许从流水线内部配置特定于流水线的选项。例如提供了buildDiscarder选项，同时也可以由插件提供例如timestamps的选项。

### 4.1 pipeline中的具体参数
一、buildDiscarder
作用域：只能在pipeline块中使用。
buildDiscarder保存最近历史构建记录的数量。当pipeline执行完毕后，会在硬盘上保存记录、发布的内容和构建执行的日志，如果长时间不对这些内容进行清理会占用大量的空间。通过在options中配置该选项可以实现对于空间的自动清理和回收。
例如：
```
pipeline{
  agent{
    label any
  }
  options{
    buildDiscarder(logRotator(numToKeepStr:'3'))
  }
  stages{
    stage('say_hello'){
      steps{
        echo "hello,User"
      }
    }
  }
}
```
我在pipeline的options块中使用了buildDiscarder命令，因此在BuildHistory中只保留了3次历史记录。
![](@attachment/Clipboard_2023-07-05-10-37-05.png)
通过查看Console Ouput中的内容，能够看到在stages中，执行了对应的步骤，输出了hello，world指令
![](@attachment/Clipboard_2023-07-05-10-39-13.png)

二、disableConcurrentBuilds参数
disableConcurrentBuilds参数不允许并行执行Pipeline参数，可以用于防止同时访问共享资源。
因为用的workspace还是同一个，所以同时构建可能会访问共享资源导致冲突。
例如:
```
pipeline{
  agent any
  options{
    disableConcurrentBuilds()
  }
}
```

三、skipStagesAfterUnstable参数
一旦构建状态变得不稳定，那么就跳过该阶段。
例如：
```
  pipeline{
    agent any
    options{
      skipStagesAfterUnstable()
    }
  }
```

四、timeout参数
设定了流水线的超时时间，如果流水线运行时间超过超时时间，jenkins将会中止流水线。
例如:
```
  pipeline{
    angent any
    timeout{
      time:1,
      unit:'HOURS'
    }
  }
```

五、timestamps参数
预谋所有由流水线生成的控制台输出，与该流水线发出的时间一致，例如：
```
  options{
    timestamps()
  }
```

六、lock参数
用来阻止多个构建在同一时间试图使用同一个资源。资源包括节点、代理节点的结合，或者是用于上锁的名字。若指定的资源在系统中没有在全局配置中定义，那么会被自动 的加入到系统中。


## 5.Pipeline中的environment参数
environment指令指定一系列键值对，这些对值将被定义为所有步骤的环境变量或阶段特定步骤

environment{…}, 大括号里面写一些键值对，也就是定义一些变量并赋值，这些变量就是环境变量。

一般情况下， 定义全局环境变量，如果将envirnoment写在stage中，那么将会作为局部环境变量。

## 6.Pipeline中的parameters参数
Pipeline支持的六种参数
参数类型|参数说明
-------|--------
string|字符串类型参数
text|文本类型参数，与字符串的区别在于可以包含多行信息，用于传入较多信息输入
booleanParam|布尔类型参数
choice|类似下拉框或者支持多值的单选参数
file|指定构建过程中所需要的文件
password|考虑到安全的因素，需要通过参数方式传递的密码类型

例子:
```
pipeline{
  agent any
  parameters{
      string(name: 'PERSON', defaultValue: 'Mr Jenkins', description: 'Who should I say hello to?')

      text(name: 'BIOGRAPHY', defaultValue: '', description: 'Enter some information about the person')

      booleanParam(name: 'TOGGLE', defaultValue: true, description: 'Toggle this value')

      choice(name: 'CHOICE', choices: ['One', 'Two', 'Three'], description: 'Pick something')

      password(name: 'PASSWORD', defaultValue: 'SECRET', description: 'Enter a password')
  }
}
```

这里的参数相当于是一种key-value的形式，对于字符串类型数据来说，相当于PERSON变量赋值为Mr Jenkins,对于text数据类型来说，相当于BIOGRAPHY变量赋值为空，对于boolean变量来说，相当于给TOGGLE变量赋值默认为true。
对于password数据类型来说，相当于给PASSWORD变量赋值默认为SECRRT。

## 7.Pipeline中的stages参数
pipeline由多个步骤组成，允许用户构建、测试和部署应用，Jenkins pipeline允许使用简单的方式组合多个步骤，帮助用户实现自动化的构建过程。
stages在pipeline中相当于我们声明了我们需要执行的指令列表。我们在列表中以单独的stage定义我们需要执行的步骤名和对应步骤需要执行的内容。
每一个Step是一个执行单一步骤的过程。当任何一个步骤执行失败，pipeline的执行过程就视为失败。
结构为：
```
  pipeline{
    stages{
      stage('say_hello'){
        steps{
          echo "hello"
        }
      }
      stage('say_bye'){
        steps{
          echo "byebye"
        }
      }
    }
  }
```
当然为了避免执行失败时直接导致pipeline的退出或是单一步骤由于脚本问题或是网络问题长时间没有执行完毕的，可以增加容错设置。通过设置retry、timeout来设置出错重试、超时退出的机制。
例如：
```
pipeline{
  agent any
  stages{
    stage('error_dect'){
      retry(3){
        echo "hello,world"
      }
    }
    stage('timeout_dect'){
      timeout(time:3,unit:'MINUTES'){
        echo "byebye"
      }
    }
  }
}
```

## 7.完成时动作post
当 Pipeline 运行完成时，你可能需要做一些清理工作或者基于 Pipeline 的运行结果执行不同的操作， 这些操作可以放在 post 部分。
例如:
```
pipeline{
  agent any
  post {
        always {
            echo 'This will always run'
        }
        success {
            echo 'This will run only if successful'
        }
        failure {
            echo 'This will run only if failed'
        }
        unstable {
            echo 'This will run only if the run was marked as unstable'
        }
        changed {
            echo 'This will run only if the state of the Pipeline has changed'
            echo 'For example, if the Pipeline was previously failing but is now successful'
        }
    }
}
```
always为总是，每次完成执行后总是会执行always中所需要执行的指令。
同样的success块只有执行成功时才会进行执行。
failure在失败时进行输出。
unstable在不稳定时进行输出。
changed在更改时进行输出。

## 8.补充
### 8.1 credentials 参数使用补充
credentials相当于身份认证鉴权的通行证。当pipeline需要和第三方网站和应用程序进行数据交互、使用基于云的存储系统和服务时，需要jenkins管理员在credentials中就三方交互信息进行配置。

<b>存储在Jenkins中的credentials可以被使用：</b>
·适用于Jenkins的任何地方 (即全局 credentials),
·通过特定的Pipeline项目/项目 (在 处理 credentials 和 使用Jenkinsfile部分了解更多信息),
·由特定的Jenkins用户 (如 Pipeline 项目中创建 Blue Ocean的情况).
例如需要访问svn和github的情况。

