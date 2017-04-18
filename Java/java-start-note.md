# Java入门学习笔记

视频教程:   
[Java轻松入门实用教程【柠檬学院】](https://ke.qq.com/course/155221)

## 2017-04-18

- java的工作方式  
  java源文件(.java文件)  ---->  经过java编译器的编译  ---->  字节码文件(.class文件)  ---->  被类装载器装载到java虚拟机(JVM)  ---->  被JVM解释给操作系统  ---->  操作系统执行

- java虚拟机(JVM)的组成  
  指令集、寄存器、堆栈、垃圾处理器、方法区域  

- 类库  
  标准类库、开发者自己的类库

- 包  
  本质上是文件夹的形式，用于组织项目文件

- JRE  
  Java Runtime Environment(java的运行环境)

- JDK  
  Java Development Kit(Java开发组建)

- 搭建JAVA开发环境(win环境)  

  > 1. 安装jdk，默认配置路径：C:\Program Files\Java\jdk1.7.0_67\
  > 2. 配置jdk环境变量(我的电脑---属性---高级系统设置---环境变量---系统变量)  
  >    JAVA_HOME    C:\Program Files\Java\jdk1.7.0_67\    (需要新建)  
  >    PATH    C:\Program Files\Java\jdk1.7.0_67\bin    (该变量已存在，添加新的路径即可，每个路径以;分割)  
  >    CLASSPATH    .    (.表示当前路径，需要新建)  
  > 3. 验证环境搭建成功  
  >    win+r --- cmd --- 键入java(或者java -version) --- 键入javac

