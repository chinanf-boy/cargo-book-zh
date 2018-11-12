
## 持续集成

### 特拉维斯CI

要在Travis CI上测试您的项目,这里有一个示例`.travis.yml`文件:

```
language: rust
rust:
  - stable
  - beta
  - nightly
matrix:
  allow_failures:
    - rust: nightly
```

这将测试所有三个发布渠道,但每晚的任何破坏都不会使整体构建失败.请看看[Travis CI Rust文档](https://docs.travis-ci.com/user/languages/rust/)了解更多信息.
