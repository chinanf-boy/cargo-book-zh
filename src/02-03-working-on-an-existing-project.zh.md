
## 在现有的货运项目上工作

如果您下载使用Cargo的现有项目,那么它很容易上手.

首先,从某个地方获取项目.在这个例子中,我们将使用`rand`从GitHub上的存储库克隆:

```shell
$ git clone https://github.com/rust-lang-nursery/rand.git
$ cd rand
```

要建立,使用`cargo build`:

```shell
$ cargo build
   Compiling rand v0.1.0 (file:///path/to/project/rand)
```

这将获取所有依赖项,然后与项目一起构建它们.
