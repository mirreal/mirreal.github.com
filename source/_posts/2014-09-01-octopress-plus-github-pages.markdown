---
layout: post
title: "Octopress + Github Pages"
date: 2014-09-01 15:04:01 +0800
comments: true
categories:
---

## Octopress + Github Pages = a new blog

[document](http://octopress.org/docs/setup)

Enviroment: Ubuntu 12.00

### 1.Installing Ruby With RVM

首先，由于 `octopress` 使用 `ruby`，版本要高于 `1.9.3`，系统自带的版本可能并不满足，在这里通过 `RVM` 来安装高版本的 `ruby`。

如果系统已经有了合适的 `ruby` 版本，这一步可以跳过。

接下来详细说明。

#### Install RVM

```sh
curl -L https://get.rvm.io | bash -s stable --ruby
```

<!-- more -->

Then run:

```sh
source ~/.rvm/scripts/rvm
```

to start RVM

#### Install Ruby 1.9.3

```sh
rvm install 1.9.3
rvm use 1.9.3
rvm rubygems latest
```

### 2.Octopress Setup

#### Get octopress

```sh
git clone git://github.com/imathis/octopress.git octopress
cd octopress
```

#### Install dependencies.

```sh
gem install bundler
bundle install
```

如果 `gem` 太慢，可以将源设置成国内淘宝的，直接修改当前的 `Gemfile`

```sh
source "https://ruby.taobao.org"
```

#### Install the default Octopress theme.

```sh
rake install
```

### 3.Deploying to Github Pages

下面我们就把静态文件发到 `github` 上

#### With Github User/Organization pages

Create a new Github repository and name the repository with the format `username.github.io`

在 `github` 上新建一个仓库，名字格式必须是 `username.github.io`，比如 `mirreal.github.io`，如果要直接通过 `https://mirreal.github.io` 访问的话。

另外一种通过路径的方式，具体参考 `github pages` 文档。

```sh
rake setup_github_pages
```

Next run:

```
rake generate
rake deploy
```
