---
tip: translate by openai@2023-06-20 08:08:40
...

---

## title: About These Documents

These documents are generated from [reStructuredText](http://docutils.sourceforge.net/rst.html) sources by [Sphinx](http://sphinx-doc.org/), a document processor specifically written for the Python documentation.

> 这些文件是由[reStructuredText](http://docutils.sourceforge.net/rst.html)源和[Sphinx](http://sphinx-doc.org/)生成的，Sphinx 是专门为 Python 文档编写的文档处理器。

# Building The Documentation

To build the keepalived documentation, you will need to have a recent version of Sphinx installed on your system. Alternatively, you could use a python virtualenv.

> 要构建 keepalived 文档，您需要在系统中安装最新版本的 Sphinx。或者，您可以使用 python 虚拟环境。

From the root of the repository clone, run the following command to build the documentation in HTML format:

> 从仓库克隆的根目录中，运行以下命令以 HTML 格式构建文档：

```
cd keepalived-docs
make html
```

For PDF, you will also need `docutils` and various `texlive-*` packages for converting reStructuredText to LaTex and finally to PDF:

> 要将 reStructuredText 转换为 LaTex，然后再转换为 PDF，您还需要`docutils`和各种`texlive-*`软件包：

```
pip install docutils
cd keepalived-docs
make latexpdf
```

Alternatively, you can use the `sphinx-build` command that comes with the Sphinx package:

> 另外，您可以使用随 Sphinx 包一起提供的`sphinx-build`命令：

```
cd keepalived-docs
sphinx-build -b html . build/html
```

::: todo

make latexpdf needs pdflatex provided by texlive-latex on RHEL6 and texlive-latex-bin-bin on Fedora21

> 使用 RHEL6 上的 texlive-latex 提供的 pdflatex 和 Fedora21 上的 texlive-latex-bin-bin 来制作 latexpdf。
> :::

::: todo

make linkcheck to check for broken links

> 进行链接检查以检查断开的链接
> :::
