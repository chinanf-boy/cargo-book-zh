## Publishing on crates.io

一旦你有一个你想与世界分享的图书馆,就该发布它了[crates.io]!发布包是指上载特定版本以进行托管[crates.io].

发布箱子时要小心,因为发布是**常驻**.永远不会覆盖该版本,并且无法删除代码.但是,可以发布的版本数量没有限制.

### Before your first publish

首先,你需要一个帐户[crates.io]获取API令牌.为此,[访问主页][crates.io]并通过GitHub帐户登录(现在需要).在此之后,访问你的[帐号设定](https://crates.io/me)页面并运行`cargo login`命令指定.

```console
$ cargo login abcdefghijklmnopqrstuvwxyz012345
```

此命令将通知Cargo您的API令牌并将其存储在您的本地`~/.cargo/credentials`(以前是`~/.cargo/config`).请注意,此令牌是一个**秘密**不应与其他任何人分享.如果因任何原因泄漏,您应立即重新生成.

### Before publishing a new crate

请记住箱子上的名字[crates.io]以先到先得的方式分配.一旦获得箱子名称,它就不能用于另一个箱子.

#### Packaging a crate

下一步是将您的包装箱打包成可上传的格式[crates.io].为此,我们将使用`cargo package`子命令.这将把我们的整个包装箱全部打包成一个`*.crate`文件中`target/package`目录.

```console
$ cargo package
```

作为一个额外的奖励,`*.crate`将独立于当前源树进行验证.之后`*.crate`已创建,已解压缩`target/package`然后从头开始构建,以确保构建成功的所有必要文件.可以使用.禁用此行为`--no-verify`旗.

现在是时候看看了`*.crate`文件,以确保您不会意外地打包2GB视频资产,或用于代码生成,集成测试或基准测试的大型数据文件.目前有10MB的上传大小限制`*.crate`文件.所以,如果大小`tests`和`benches`目录及其依赖项最多可达几MB,您可以将它们保存在包中;否则,最好排除它们.

在打包时,Cargo会自动忽略版本控制系统忽略的文件,但是如果要指定要忽略的额外文件集,则可以使用`exclude`清单中的关键:

```toml
[package]
# ...
exclude = [
    "public/assets/*",
    "videos/*",
]
```

这个数组中每个元素的语法是什么[锈琅/水珠](https://github.com/rust-lang/glob)接受.如果您宁愿使用白名单而不是黑名单,Cargo也支持`include`key,如果设置,则覆盖`exclude`键:

```toml
[package]
# ...
include = [
    "**/*.rs",
    "Cargo.toml",
]
```

### Uploading the crate

现在我们已经有了`*.crate`文件准备好了,可以上传到[crates.io]随着`cargo publish`命令.就是这样,你现在已经发布了你的第一个箱子!

```console
$ cargo publish
```

如果你想跳过`cargo package`一步,`cargo publish`如果找不到副本,子命令将自动打包本地包.

一定要看看[您可以指定的元数据](reference/manifest.md#package-metadata)确保您的箱子更容易被发现!

### Publishing a new version of an existing crate

要发布新版本,请更改`version`您指定的值`Cargo.toml`表现.记住[semver规则](reference/manifest.md#the-version-field).然后可选择运行`cargo package`如果你想检查一下`*.crate`发布前的新版本文件,然后运行`cargo publish`上传新版本.

### Managing a crates.io-based crate

包装箱的管理主要通过命令行完成`cargo`工具而不是[crates.io]网络界面.为此,有一些子命令来管理包.

#### `cargo yank`

在您发布实际上因某种原因而最终被破坏的箱子版本(语法错误,忘记包含文件等)时,可能会出现这种情况.对于诸如此类的情况,Cargo支持一个箱子版本的"猛拉".

```console
$ cargo yank --vers 1.0.1
$ cargo yank --vers 1.0.1 --undo
```

一个猛拉**才不是**删除任何代码.例如,此功能不用于删除意外上传的机密.如果发生这种情况,您必须立即重置这些秘密.

一个被淘汰的版本的语义是不能为该版本创建新的依赖项,但所有现有的依赖项继续有效.其中一个主要目标[crates.io]是作为永久存档的板条箱,不会随着时间的推移而改变,允许删除版本将违背这一目标.基本上是一个猛拉意味着所有包装都带有`Cargo.lock`任何未来都不会破裂`Cargo.lock`生成的文件不会列出抽出的版本.

#### `cargo owner`

箱子通常由不止一个人开发,或者主要维护者可能会随着时间而改变!箱子的所有者是唯一允许发布新版本箱子的人,但是所有者可以指定其他所有者.

```console
$ cargo owner --add my-buddy
$ cargo owner --remove my-buddy
$ cargo owner --add github:rust-lang:owners
$ cargo owner --remove github:rust-lang:owners
```

给这些命令的所有者ID必须是GitHub用户名或GitHub团队.

如果给出了用户名`--add`,该用户成为"命名"所有者,拥有该箱子的完全权利.除了能够发布或抽出箱子的版本,他们还能够添加或删除所有者,*包含*做的主人*他们*所有者.毋庸置疑,你不应该让那些你不完全信任命名所有者的人.要成为命名所有者,用户必须已登录[crates.io]先前.

如果给团队名称`--add`,该团队成为"团队"所有者,对板条箱的权利受到限制.虽然他们有权发布或放弃箱子的版本,但他们*不要*能够添加或删除所有者.除了更方便管理所有者群体之外,团队还可以更安全地防止所有者变得恶意.

团队的语法是目前的`github:org:team`(见上面的例子).要将团队添加为所有者,必须是该团队的成员.删除作为所有者的团队不适用此类限制.

### GitHub permissions

团队成员资格不是GitHub提供的简单公共访问权限,并且在使用它时可能会遇到以下消息:

> 您似乎无权从GitHub查询必要的属性来完成此请求.您可能需要重新进行身份验证[crates.io]授予阅读GitHub组织成员资格的权限.去吧<https://crates.io/login>

这基本上是"你试图查询团队,并且五个级别的成员访问控制中的一个拒绝了这个"的全能.这并不夸张.GitHub对团队访问控制的支持是企业级.

最可能的原因是您在添加此功能之前最后一次登录.我们最初要求*没有*GitHub在对用户进行身份验证时的权限,因为我们实际上并没有将用户的令牌用于登录以外的任何其他内容.但是,为了代表您查询团队成员资格,我们现在需要[该`read:org`范围][oauth-scopes].

您可以自由地拒绝我们这个范围,并且在团队介绍之前所做的一切都将继续有效.但是,您永远无法将团队添加为所有者,或者将团队作为团队所有者发布.如果您尝试这样做,您将收到上述错误.如果你试图发布一个你根本不拥有的箱子,你可能也会看到这个错误,但恰好有一个团队.

如果你改变主意,或者只是不确定是否[crates.io]有足够的许可,你可以随时去<https://crates.io/login>,如果,将提示您获得许可[crates.io]没有它想要的所有范围.

查询GitHub的另一个障碍是组织可能会主动拒绝第三方访问.要检查这一点,您可以访问:

```
https://github.com/organizations/:org/settings/oauth_application_policy
```

哪里`:org`是组织的名称(例如rust-lang).你可能会看到类似的东西:

![Organization Access Control](images/org-level-acl.png)

你可以选择明确删除的地方[crates.io]从您组织的黑名单,或只需按"删除限制"按钮,允许所有第三方应用程序访问此数据.

或者,何时[crates.io]请求了`read:org`范围,您可以明确列入白名单[crates.io]通过按其名称旁边的"授予访问权限"按钮查询有问题的组织:

![Authentication Access Control](images/auth-level-acl.png)

[crates.io]: https://crates.io/

[oauth-scopes]: https://developer.github.com/apps/building-oauth-apps/understanding-scopes-for-oauth-apps/
