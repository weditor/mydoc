# 编译tensorflow C++ 库

## 下载源码

源码直接到[tensorflow的github地址](https://github.com/tensorflow/tensorflow)下载。  
如果想编译python库直接按照官方文档就可以。但是一般没必要自己编译，因为官方和第三方都提供tensorflow的python二进制安装版本。同时官方还提供了Java/Go/C的版本.  

## 安装bazel

下载源码后，需要安装[bazel](https://bazel.build/)，一个类似于maven的构建工具，因为tensorflow是通过bazel构建。bazel是单文件的可执行文件，所以可以到[bazel的github](https://github.com/bazelbuild/bazel/releases)下载后直接解压出来，加入到PATH环境变量就能用。唯一需要注意的是bazel需要以来jdk8.  
jdk8安装命令：

```shell
sudo apt-get install openjdk-8-jdk
```

如果对安装过程还不明白，还可以参照[官网安装教程]。另外附上ubuntu16通过apt安装的命令。github国内下载可能比较慢. apt会快很多。

```shell
echo "deb [arch=amd64] http://storage.googleapis.com/bazel-apt stable jdk1.8" | sudo tee /etc/apt/sources.list.d/bazel.list
curl https://bazel.build/bazel-release.pub.gpg | sudo apt-key add -
sudo apt-get update && sudo apt-get install bazel
```

## 编译tensorflow C++

无论编译tensorflow的什么版本，都需要执行configure

```shell
./configure
```

一路enter过去，默认是编译cpu版本。  

官网虽然提供了C++的api文档，但是一直找不到怎么编译C++的方法。所以恶补了一下bazel--[bazel tutorial](https://docs.bazel.build/versions/master/tutorial/cpp.html) , tensorflow/BUILD文件中 name字段为libtensorflow_cc.so的那个编译目标就是了。

```shell
bazel build //tensorflow:libtensorflow_cc.so
```

### .so文件

编译比较耗时,完成后bazel-bin/tensorflow/libtensorflow_cc.so就是最终的动态链接库。

### 头文件

把tensorflow和third_party目录拷贝到include文件。
还有bazel-bin/external，bazel-genfiles/external目录。

### example

.so和头文件都有了之后，就可以把tensorflow/cc/tutorials/example_trainer.cc直接拿来编译了。  
这是我自己的makefile，我并没有把include拷贝到系统的include，而是放到了工程目录下。

```makefile
INCLUDE=-I./ \
	-I./external/protobuf/src/ \
	-I./external/eigen_archive 

trainer: example_trainer.cc
	g++ example_trainer.cc -o trainer ${INCLUDE} -std=c++14 -ltensorflow_cc
```
