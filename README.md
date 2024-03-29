stc-docs用于发布希姆计算的用户文档。具体来说，源md文件存放在GitHub repo中，并将GitHub repo关联到Read the Docs，文件更新时由Read the Docs自动完成构建和发布。

文档项目使用Sphinx生成，比较重要的文件/文件夹有：

- stc-docs/source文件夹：源md文件以及相关的目录结构、前端配置文件。
- stc-docs/source/index.rst：目录结构文件，将md组织成展示网页时所需的目录结构，遵循reStructuredText语法。
- stc-docs/source/conf.py：前端配置文件，包括HTML主题、静态资源路径、项目信息、显示语言、表格插件等相关配置。
- stc-docs/source/readthedocs.yaml：Read the Docs针对Sphinx项目必须的配置文件，指定了OS版本、Python版本及所需的组件等信息，如果缺失会导致构建失败。
- stc-docs/source/_statics文件夹：静态资源文件，包括图片、css文件等。