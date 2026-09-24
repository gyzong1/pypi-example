# PyPI + JFrog CLI GitHub Actions 示例

基于测试项目 [gyzong1/pypi-example](https://github.com/gyzong1/pypi-example)，用 JFrog CLI 完成：

1. **构建并上传**至 Artifactory
2. **搜集并发布** Build Info
3. **扫描** Build（Xray）

示例工作流文件：`.github/workflows/pypi.yml`

## 使用方法

将 `pypi.yml` 复制到目标仓库：

```text
.github/workflows/pypi.yml
```

### Github 仓库配置


| 类型       | 名称                | 说明                                           |
| -------- | ----------------- | -------------------------------------------- |
| Variable | `JF_URL`          | JFrog Platform URL，如 `https://acme.jfrog.io` |
| Secret   | `JF_ACCESS_TOKEN` | 需具备 Deploy / Build Info / Xray Scan 权限       |


### Artifactory 仓库配置

流水线通过 `PYPI_REPO_RESOLVE` / `PYPI_REPO_DEPLOY` 指向 Pypi 仓库（默认使用同一个 virtual）。请先在 Artifactory 中创建以下 **Pypi** 类型仓库：


| 类型      | 示例名称                        | 说明                                                               |
| ------- | --------------------------- | ---------------------------------------------------------------- |
| Local   | `guoyz-github-pypi-local`   | 存放本流水线发布的 Python 包                                                |
| Remote  | `guoyz-github-pypi-remote`  | 代理 PyPI，URL 为 `https://files.pythonhosted.org`                  |
| Virtual | `guoyz-github-pypi-virtual` | 聚合上述 local + remote；**Default Deployment Repository** 指向对应 local |


说明：

流水线中使用了 build-scan, 需将相关仓库和 build 加入 **Xray Indexed Resources**，以便 `jf build-scan` 可扫描依赖与制品。

### 可调环境变量


| 变量                       | 默认值                         | 说明                          |
| ------------------------ | --------------------------- | --------------------------- |
| `JFROG_CLI_BUILD_NAME`   | `guoyz-github-pypi-example` | Build 名称                    |
| `JFROG_CLI_BUILD_NUMBER` | `${{ github.run_number }}`  | Build 编号                    |
| `JFROG_CLI_MODULE`       | `jfrog-python-example`      | Build Info module（对应包名）     |
| `PYPI_REPO_RESOLVE`      | `guoyz-github-pypi-virtual` | 解析仓库                        |
| `PYPI_REPO_DEPLOY`       | `guoyz-github-pypi-virtual` | 部署仓库                        |


## 核心 JFrog CLI 步骤


| 步骤            | 命令                                                                  |
| ------------- | ------------------------------------------------------------------- |
| 配置 pip 仓库     | `jf pip-config`                                                     |
| 安装依赖并记录       | `jf pip install -r requirements.txt --build-name/--build-number`    |
| 打包            | `python -m build`                                                   |
| 发布包并记录        | `jf twine upload dist/* --build-name/--build-number`                |
| 搜集环境信息        | `jf rt build-collect-env`                                           |
| 搜集 Git 信息     | `jf rt build-add-git`                                               |
| 发布 Build Info | `jf rt build-publish`                                               |
| 扫描 Build      | `jf build-scan`                                                     |


与 [pypi-example README](https://github.com/gyzong1/pypi-example) 的差异：上传改用官方推荐的 `jf twine upload`（替代 `python setup.py sdist bdist_wheel` + `jf rt u dist/`），效果等价且与 `jf pip-config` 的 deploy 仓库配置一致。

## 参考链接

- [安装 JFrog CLI](https://docs.jfrog.com/integrations/docs/download-and-install-the-jfrog-cli)
- [JFrog CLI 快速开始](https://docs.jfrog.com/integrations/docs/jfrog-cli-quick-start)
- [JFrog CLI 文档总览](https://docs.jfrog.com/integrations/docs/jfrog-cli)
- [jf pip 命令说明](https://docs.jfrog.com/artifactory/docs/jf-pip)
- [jf twine 命令说明](https://docs.jfrog.com/artifactory/docs/jf-twine)
