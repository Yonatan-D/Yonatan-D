## 1.提交标签时触发脚本

```yaml
# .gitlab-ci.yml

stages:
  - release

release-to-linux:
  stage: release
  rules:
    # 只有发布标签时才触发
    - if: $CI_COMMIT_TAG != null
  script:
    - echo "Building in Linux..."
    # 执行 linux 环境下的构建脚本
    - bash release_linux.sh $CI_COMMIT_TAG
  tags:
    - linux

release-to-windows:
  stage: release
  rules:
    # 只有发布标签时才触发
    - if: $CI_COMMIT_TAG != null
  script:
    - echo "Building in Windows..."
    # 执行 windows 环境下的构建脚本
    - bash release_win.sh $CI_COMMIT_TAG
  tags:
    - windows
```

## 2.缓存 yarn，避免每次构建

yarn cache: https://classic.yarnpkg.com/en/docs/install-ci/

使用 node 镜像

```yaml
# .gitlab-ci.yml
image: node:9.11.1

before_script:
  - yarn install --cache-folder .yarn

test:
  stage: test
  cache:
    paths:
    - node_modules/
    - .yarn
```

如果使用的是没有安装 yarn 的 docker 镜像，需要安装 yarn

```yaml
# .gitlab-ci.yml
image: does-not-have-yarn

before_script:
  # Install yarn
  - curl -o- -L https://yarnpkg.com/install.sh | bash
  # Add yarn to the path
  - export PATH="$HOME/.yarn/bin:$HOME/.config/yarn/global/node_modules/.bin:$PATH"
```