# 中文翻译参与说明

## 本地运行网站

### 安装 Hugo

#### MacOS

##### 方法一：使用 brew 安装

```
brew install hugo
```

##### 方法二：直接下载二进制文件使用

在 [Hugo 的 release 页面](https://github.com/gohugoio/hugo/releases)下载最新安装包，如 0.62.0 版本：

```
wget https://github.com/gohugoio/hugo/releases/download/v0.62.0/hugo_0.62.0_macOS-64bit.tar.gz
```

#### Linux

##### 源码编译安装（需要 Golang 1.12 及以上版本）

```
git clone https://github.com/gohugoio/hugo.git
cd hugo
# 注意一定需要设置 GO111MODULE 变量为 on
GO111MODULE=on go install --tags extended
```

#### 其他平台安装方式参考：

- https://gohugo.io/getting-started/installing/

### 运行 

```
hugo server -D
```

默认绑定地址是 `127.0.0.1`，如果想要修改为 `0.0.0.0`，需要加上参数：`--bind="0.0.0.0"`

### 访问

浏览器访问地址 `http://localhost:1313` 即可。

## 翻译流程

1. 创建分支
2. 修改要翻译的文件
3. 提交代码
4. 提交 PR
5. 合并到 master
