# 构建

https://github.com/envoyproxy/envoy/blob/master/bazel/README.md

## 安装Bazel

### linux用二进制包安装

参考官方安装文档：

https://docs.bazel.build/versions/master/install-ubuntu.html

主要命令:

```bash
sudo apt-get install pkg-config zip g++ zlib1g-dev unzip python

## 下载 bazel-0.14.0-installer-linux-x86_64.sh
chmod +x bazel-0.14.0-installer-linux-x86_64.sh
./bazel-0.14.0-installer-linux-x86_64.sh --user

## 修改~/.bashrc文件，增加内容
export PATH="$PATH:$HOME/bin"

## 重新source
source ~/.bashrc
## 检查版本
bazel version
```

### mac brew安装

```bash
brew install bazel
brew install cmake
```



## 环境和依赖

https://www.envoyproxy.io/docs/envoy/latest/install/building.html#requirements

- GCC 5+ (for C++14 support).

- These [pre-built](https://github.com/envoyproxy/envoy/blob/master//ci/build_container/build_recipes) third party dependencies.

  这些依赖，可以在本地envoy目录中找到对应的sh文件，然后`sudo sh **.sh` 进行安装即可

- These [Bazel native](https://github.com/envoyproxy/envoy/blob/master/bazel/repository_locations.bzl) dependencies.