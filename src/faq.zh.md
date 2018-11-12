## Frequently Asked Questions

### Is the plan to use GitHub as a package repository?

不,货物的计划是使用.[克拉西奥.]像NPM或RuuGuMes和NPMJS.org和RuuGeMy.org一起使用.

我们计划永远支持git存储库作为包的来源,因为它们可以用于早期开发和临时补丁,即使人们使用注册表作为包的主要来源.

### Why build crates.io rather than use GitHub as a registry?

我们认为支持多种下载包的方式非常重要,包括从GitHub下载包并将包复制到项目本身.

也就是说,我们认为[克拉西奥.]提供了许多重要的好处,很可能成为人们在货物中下载包裹的主要方式.

对于前例,节点.js[npm][1]露比的[捆扎机][2]支持中央注册中心模型和基于Git的模型,并且大多数包都是通过这些生态系统中的注册中心下载的,其中重要的少数包使用基于git的包.

[1]: https://www.npmjs.org

[2]: https://bundler.io

使中央注册表在其他语言中流行的一些优点包括:

-   **可发现性**. 中央注册表提供了查找现有包的简单位置.结合标记,这也使得注册中心能够提供生态系统范围的信息,例如最流行或最依赖的包的列表.
-   **速度**. 中心注册中心使得可以快速有效地只获取包的元数据,然后只高效地下载已发布的包,而不会出现在存储库中的其他膨胀.这增加了依赖性解析和获取速度的显著提高.当依赖图放大时,下载所有的GIT存储库都会很快下降.还请记住,不是每个人都有高速、低延迟的Internet连接.

### Will Cargo work with C code (or other languages)?

对!

货物处理编制锈码,但我们知道许多锈项目链接C代码.我们也知道,除了编写生锈的语言外,还有几十年的编译工具.

我们的解决方案:货物允许包装[specify a script](03-05-build-scripts.html)(生锈)在调用之前运行`rustc`. 利用RISE实现平台特定的配置,并重构包之间的公共构建功能.

### Can Cargo be used inside of `make` (or `ninja`, or ...)

的确.尽管我们希望Cargo作为在顶层编译Rust项目的独立方式有用,但我们知道有些人会希望从其他构建工具调用Cargo.

我们已将Cargo设计成在这些环境中工作良好,并注意错误代码和机器可读输出模式等事项.在这些方面我们还有一些工作要做,但是在传统脚本上下文中使用Cargo是我们从一开始就设计的,并且将继续优先考虑.

### Does Cargo handle multi-platform projects or cross-compilation?

RITE本身提供了基于平台配置代码段的工具.货物也支持[特定平台依赖关系][target-deps],我们计划支持更多的每个平台配置`Cargo.toml`未来.

[target-deps]: 03-02-manifest.html#the-dependencies-section

从长远来看,我们正在寻找如何方便地使用货物交叉编译项目的方法.

### Does Cargo support environments, like `production` or `test`?

我们通过使用环境来支持环境.[轮廓][profile]支持:

[profile]: 03-02-manifest.html#the-profile-sections

-   环境特定标志(类)`-g --opt-level=0`为了发展和`--opt-level=3`用于生产).
-   环境特定依赖性(类)`hamcrest`对于测试断言).
-   特定环境`#[cfg]`
-   一`cargo test`命令

### Does Cargo work on Windows?

对!

所有提交的货物都需要通过Windows上的本地测试套件.但是,如果你发现一个Windows问题,我们认为它是一个bug,所以[请提出一个问题][3].

[3]: https://github.com/rust-lang/cargo/issues

### Why do binaries have `Cargo.lock` in version control, but not libraries?

A的目的`Cargo.lock`是在成功建造时描述世界的状态.然后,它用于通过确保编译完全相同的依赖项来提供跨构建项目的任何机器的确定性构建.

从依赖链(二进制文件)的末尾的应用程序和项目中,这个属性是最可取的.因此,建议所有二进制文件都在其内部进行检查.`Cargo.lock`.

对于图书馆来说,情况有些不同.图书馆不仅是图书馆开发人员使用的图书馆,也是图书馆的下游用户使用的图书馆.依赖图书馆的用户不会检查图书馆`Cargo.lock`(即使它存在).这正是因为图书馆应该**不**对库的所有用户进行确定性地重新编译.

如果一个库最终被多个依赖项传递使用,那么很可能只需要该库的一个副本(基于semver兼容性).如果所有的库都要检查`Cargo.lock`然后,将使用多个库副本,甚至可能会出现版本冲突.

换句话说,库为它们的依赖性指定了Sever要求,但不能看到完整的图片.只有二进制文件之类的最终产品才有完整的图片来决定应该使用什么版本的依赖项.

### Can libraries use `*` as a version for their dependencies?

**截至2016年1月22日,[克拉西奥.]拒绝通配符依赖约束的所有包(不只是库).**

当图书馆*可以*严格来说,他们不应该这样做.版本要求`*`说"这将适用于所有版本",这是永远不会是真的.库应该总是指定它们使用的范围,即使它是一般的"每1×.y版本".

### Why `Cargo.toml`?

作为与货物最频繁的交互之一,为什么配置文件被命名为`Cargo.toml`不时出现.领先的资本——`C`被选中以确保清单在目录列表中与其他类似的配置文件分组.排序文件通常在小写字母之前放置大写字母,确保文件像`Makefile`和`Cargo.toml`放在一起.拖尾`.toml`选择强调文件是在[TOML configuration
format](https://github.com/toml-lang/toml).

货物不允许其他名称,如`cargo.toml`或`Cargofile`强调如何容易识别货物仓库.许多可能的名字的选择在历史上导致混乱,其中一个案件被处理,但其他人被意外地遗忘.

[crates.io]: https://crates.io/

### How can Cargo work offline?

货物通常用于网络访问有限或没有网络访问的情况,如飞机、CI环境或嵌入大型生产部署中.当Cargo试图从网络获取资源时,用户常常感到惊讶,因此频繁出现Cargo离线工作的请求.

除非被告知这样做,否则它的心脏不会试图访问网络.也就是说,如果没有来自.es.io、git存储库或其他网络位置的板条箱,则Cargo永远不会尝试进行网络连接.因此,如果货物试图接触网络,那么这是因为它需要获取所需的资源.

货物对缓存信息也很有攻击性,以减少网络活动的数量.例如,它将保证`cargo build`(或等效)运行到完成,然后是下一个.`cargo build`保证不接触网络,只要`Cargo.toml`在此期间还没有被修改.网络的这种回避归结为`Cargo.lock`现有的和在锁文件中反映的板条箱的填充缓存.如果缺少这些组件中的任何一个,则它们需要构建成功并必须远程获取.

至于锈迹,1.110货物可以看到新的旗帜,`--frozen`这是它不应该接触网络的断言.当通过时,如果货物尝试网络请求,货物将立即返回错误.错误应该包括关于为什么首先进行网络请求以帮助调试的上下文信息.注意这个标志*不改变货物的行为*它只是断言Cargo不应该触摸网络,因为前面的命令已经被运行以确保网络活动不应该是必要的.

有关版本管理的详细信息,请参阅文档[源置换][replace].

[replace]: 03-08-source-replacement.html
