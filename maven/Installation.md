# Installation

Maven 是一个java tool，所以你要先安装java

## 下载[Maven](http://maven.apache.org/download.html)

## 安装Maven

[installation instructions](http://maven.apache.org/install.html)

1. 确保环境变量JAVA_HOME被设置好了，并且指向你的JDK installation
2. 解压缩maven

    ```shell
    unzip apache-maven-3.6.2-bin.zip
    ```

    或者

    ```shell
    tar xzvf apache-maven-3.6.2-bin.tar.gz
    ```

3. 将Maven的bin目录添加到环境变量PATH中
4. 确认是否安装成功

    ```shell
    maven -v
    ```

    结果类似下面的显示：

    ```shell
    Apache Maven 3.6.2 (40f52333136460af0dc0d7232c0dc0bcf0d9e117; 2019-08-27T17:06:16+02:00)
    Maven home: /opt/apache-maven-3.6.2
    Java version: 1.8.0_45, vendor: Oracle Corporation
    Java home: /Library/Java/JavaVirtualMachines/jdk1.8.0_45.jdk/Contents/Home/jre
    Default locale: en_US, platform encoding: UTF-8
    OS name: "mac os x", version: "10.8.5", arch: "x86_64", family: "mac"
    ```
