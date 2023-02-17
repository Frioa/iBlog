---
title: Mac下安装Python2
date: 2023-02-17 23:03:07
tags:
- Python
- Mac
- Tech
---

最近项目上需编译老项目，用到Python2。可Mac Monterey默认移除了Python2。实际上Python官方宣布 2020 年 1 月后不再更新维护 Python2，最后版本为[2.7.18](https://www.python.org/downloads/release/python-2718/)。

在2020年4月之前可使用 `brew install python@2` 来安装python2，但实测已失效，报错如下：

<!--more-->


```
❯ brew install python@2 Warning: No available formula with the name 
"python@2". Did you mean ipython, bpython, jython or cython? ==> Searching for 
similarly named formulae... These similarly named formulae were found: ipython 
bpython jython cython To install one of them, run (for example): brew install 
ipython ==> Searching for a previously deleted formula (in the last month)... 
Error: No previously deleted formula found. ==> Searching taps on GitHub... 
Error: No formulae found in taps.

```

只需3步，轻松在Mac上管理Python版本。

1.  执行 `brew install pyenv` ：

```
brew install pyenv
```
2. 第二步安装python
```
pyenv install 2.7.18
```
3. 设置全局默认
```
pyenv global 2.7.18
```
可写入 `.zshrc` 或 `.bash_profile` **中**


```
❯ echo -e 'if command -v pyenv 1>/dev/null 2>&1; then\n  eval "$(pyenv init -)"\nfi' >> ~/.zshrc


❯ echo -e 'if command -v pyenv 1>/dev/null 2>&1; then\n  eval "$(pyenv init -)"\nfi' >> ~/.bash_profile
```

下面需要在 `npm install` 的时候，使用python2进行编译：

```
❯ npm config set python /Users/jerry/.pyenv/versions/2.7.18/bin/python
```
大功告成！