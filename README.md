## 我们一起用AndroidStudio写一个自定义的Gradle插件吧

- 注：这篇文章注重实际操作，插图丰富，易于理解上手。 对于理论知识的解释我自己也一头雾水，所以跟着我做就好啦。我也慢慢在学嘛

我把这个整个过程分为三部

1. 创建项目和插件，
2. 完善插件内容和配置，
3. 打包发布并且正确引用自定义插件

### 一.创建项目和插件

我们写一个gradle插件是为了用在android项目中，我们先创建一个android项目起名字叫做DemoPlugin

![](https://github.com/katherineqj/MIAOMIAO/blob/master/1.png)

  那创建好项目好之后就要开始写插件了 ，可能会想是不是要去new一个xxxPlugin啊，可是我们会发现AndroidStudio中并没有这样后缀的项目正确的做法是 我们创建一个library类型的module 。那下面我的这个插件起名字叫kathiePlugin

![这里写图片描述](https://github.com/katherineqj/MIAOMIAO/blob/master/2.png)

![这里写图片描述](https://github.com/katherineqj/MIAOMIAO/blob/master/3.png)
![这里写图片描述](https://github.com/katherineqj/MIAOMIAO/blob/master/4.png)



- 点击完finish之后。来看看是不是创建了这样一个插件呢，他的目录结构是图中红框的部分， 

![这里写图片描述](https://github.com/katherineqj/MIAOMIAO/blob/master/5.png)

- 下一步我们要做的是删除kathiePlugin默认创建的部分文件 ，只留下部分的文件，其余全部删除了

![这里写图片描述](https://github.com/katherineqj/MIAOMIAO/blob/master/6.png)

那我们删了一些还要创建一些目标文件夹和文件

首先我们要在`src/main/`这个路径下创建两个文件夹

- 一个是`groovy/com/kathie/plugin`
- 另外一个是`resources/META-INF/gradle-plugins`

注意⚠️ 这里的**目录名字**千万不能写错 。不然运行的时候就会找不到

![这里写图片描述](https://github.com/katherineqj/MIAOMIAO/blob/master/7.png)

基础的创建工作大概就完成了 下来就进入到完善插件内容和配置信息这部分

### 二. 完善插件内容和配置信息

- 因为我们插件是用groovy语言写的（不用怕
  我个人感觉和Java很像，而且不用会可以完成这篇教程，如果有深入研究的兴趣，可以自己找资料去看看）所以我们要把gradle改变一下成支持groovy的以及我们还要支持打包所以要用到maven。

把插件下的gradle中的内容全部删除，将下面这段代码复制进去

```groovy
apply plugin: 'groovy'

dependencies {
    compile gradleApi()
    compile localGroovy()
}

repositories {
    mavenCentral()
}
```

![这里写图片描述](https://github.com/katherineqj/MIAOMIAO/blob/master/8.png)

- 下一步要在`com/kathie/plugin`新建一个`xxx.groovy`的文件，我们找不到后缀为groovy的创建步骤
  这样就可以了（new->file->myPlugin.groovy ） 这个groovy名字可以随便起嘛？当然可以 不过这个名字后面有用到
  我这里的groovy叫做`myPlugin.groovy`

我们这个groovy文件的目标是成为gradle的 需要实现`org.gradle.api.Plugin`的接口 ，`apply()`方法中是插件执行的逻辑，我这个插件只是输出了 五句相同的代码，代码如下

```groovy
package com.kathie.plugin

import org.gradle.api.Plugin
import org.gradle.api.Project


class myPlugin implements Plugin<Project>{

    @Override
    void apply(Project project) {

        println "-----------this is kethie plugin!----------"
        println "-----------this is kethie plugin!----------"
        println "-----------this is kethie plugin!----------"
        println "-----------this is kethie plugin!----------"
        println "-----------this is kethie plugin!----------"


    }
}
```

![这里写图片描述](https://github.com/katherineqj/MIAOMIAO/blob/master/9.png)



我们创建了groovy 下来就要声明了

- 声明文件在目录`resources/META-INF/gradle-plugins`下，我们新建一个`xxx.properties`的文件
  注意这个文件的名字是之后我们引用插件（apply）所需要的名字
- 这个文件的内容就是指明我们自定义的类

`implementation-class=自定义的类的路径 `

![这里写图片描述](https://github.com/katherineqj/MIAOMIAO/blob/master/10.png)

### 三.打包发布并且正确引用自定义插件

- 前面我们已经自定义好了插件，下一步我们就要进行打包的过程了，我们需要在自定义插件的build.gradle里面指定打包的路径，版本号等参数信息代码指定好了之后我们需要编译一下
  **点击右侧边栏 里面插件的build**，代码如下

```groovy
apply plugin: 'groovy'
apply plugin: 'maven'

dependencies {
    compile gradleApi()
    compile localGroovy()
}

repositories {
    mavenCentral()
}
uploadArchives {
    repositories {
        mavenDeployer {
            repository(url: uri('repo'))
            //设置插件的GAV参数
            pom.groupId = 'com.kathie.myplugin'
            pom.artifactId = 'myPlugin'
            pom.version = 1.0
        }
    }
}

```

![这里写图片描述](https://github.com/katherineqj/MIAOMIAO/blob/master/11.png)

我们只是配置好了打包路径等等，还没有看到出现repo这个路径呢 那是因为我们还没有开始打包，还是右侧边栏的kathieplugin这个task下 在upload下面有一个**uploadArchives** 点击他就会开始打包了
![这里写图片描述](https://github.com/katherineqj/MIAOMIAO/blob/master/12.png)

- 他build完我们就可以看到在同级目录下出现了repo文件夹，我们点开就可以看到我们打包好的插件了

![这里写图片描述](https://github.com/katherineqj/MIAOMIAO/blob/master/13.png)
至此 打包就完成啦。下一步就是我们怎么引用他了

我们需要在两个`build.gradle`中进行操作

- 一个是app下的`build.gradle` 我们需要将插件apply进去
  apply后面的名字是我们之前说过的`xxx.properties`文件的名字

```groovy
apply plugin: 'kathie-plugin'

```

![这里写图片描述](https://github.com/katherineqj/MIAOMIAO/blob/master/14.png)

- 还有一个是在project下的`build.gradle` ，我们需要指定插件的地址和版本之类的信息 这个跟的地址是repo的地址

```groovy
 maven { 
         url uri("/Users/katherine/AndroidStudioProjects/DemoPlugin/kathieplugin/repo")
        }
        
 dependencies {
        classpath 'com.kathie.myplugin:myPlugin:1.0'
    }

```

![这里写图片描述](https://github.com/katherineqj/MIAOMIAO/blob/master/15.png)

 配置完了之后 我们重新编译一下 如果没有错误的话就可以在下面的gradleConsole中看到我们apply里面执行的逻辑啦
![这里写图片描述](https://github.com/katherineqj/MIAOMIAO/blob/master/16.png)
 好啦 这样我们就为gradle写好了一个自定义的插件了。

有了这个基础 如果我们有一些在编译期间需要完成的需求就可以在插件中完成了。



- 讲给自己：

```
有时间多学习，很厉害了吗？都学会了吗？sdk写的很溜了吗？那你想别人干什么啊？干好自己的事情吧先
```

