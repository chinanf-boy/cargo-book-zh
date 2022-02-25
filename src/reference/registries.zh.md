## Registries

Cargo安装箱子并从“注册表”中获取依赖项。默认注册表是[crates.io].注册表包含一个“索引”，其中包含可用箱子的可搜索列表。注册表还可以提供web API，以支持直接从Cargo发布新箱子。

> 注意：如果您对镜像或出售现有注册表感兴趣，请查看[Source Replacement].

### Using an Alternate Registry

要使用注册表而不是[crates.io]，必须将注册表的名称和索引URL添加到[`.cargo/config.toml` file][config]这个`registries`表中的每个注册表都有一个键，例如：

```toml
[registries]
my-registry = { index = "https://my-intranet:8080/git/index" }
```

这个`index`键应该是指向带有注册表索引的git存储库的URL。然后，箱子可以通过指定`registry`键和中该依赖项项中注册表名称的值`Cargo.toml`:

```toml
# Sample Cargo.toml
[package]
name = "my-project"
version = "0.1.0"

[dependencies]
other-crate = { version = "1.0", registry = "my-registry" }
```

与大多数配置值一样，可以使用环境变量而不是配置文件指定索引。例如，设置以下环境变量将完成与定义配置文件相同的任务：

```ignore
CARGO_REGISTRIES_MY_REGISTRY_INDEX=https://my-intranet:8080/git/index
```

> 注：[crates.io]不接受依赖其他登记处箱子的包装。

### Publishing to an Alternate Registry

如果注册表支持web API访问，那么包可以从Cargo直接发布到注册表。Cargo的几个命令，比如[`cargo publish`]坐一会儿`--registry`命令行标志，指示要使用哪个注册表。例如，要在当前目录中发布包，请执行以下操作：

1.  `cargo login --registry=my-registry`

    这只需要做一次。您必须输入从注册表网站检索到的机密API令牌。或者，令牌可以直接传递给服务器`publish`指挥`--token`命令行标志或带有注册表名称的环境变量，例如`CARGO_REGISTRIES_MY_REGISTRY_TOKEN`.

2.  `cargo publish --registry=my-registry`

而不是总是通过考试`--registry`命令行选项，可以在中设置默认注册表[`.cargo/config.toml`][config]和`registry.default`钥匙

设定`package.publish`输入`Cargo.toml`清单限制允许将包发布到哪些注册表。这有助于防止意外地将封闭源代码包发布到[crates.io]。该值可以是注册表名列表，例如：

```toml
[package]
# ...
publish = ["my-registry"]
```

这个`publish`价值也可能是`false`限制所有发布，这与空列表相同。

保存的身份验证信息[`cargo login`]存储在`credentials.toml`Cargo主目录中的文件（默认）`$HOME/.cargo`)。每个注册表都有一个单独的表，例如：

```toml
[registries.my-registry]
token = "854DvwSlUwEHtIo3kWy6x7UCPKHfzCmy"
```

### Running a Registry

一个最小的注册表可以通过一个包含索引的git存储库和一个包含压缩文件的服务器来实现`.crate`创建的文件[`cargo package`]。用户将无法使用Cargo向其发布，但对于封闭环境而言，这可能就足够了。

支持发布的功能齐全的注册表还需要一个符合Cargo使用的API的web API服务。下面介绍了web API。

商业和社区项目可用于建立和运行注册表。查看<https://github.com/rust-lang/cargo/wiki/Third-party-registries>查看可用内容的列表。

### Index Format

下面定义了索引的格式。偶尔会添加新功能，只有从引入它们的Cargo版本开始才能理解这些功能。旧版本的Cargo可能无法使用使用新功能的软件包。然而，旧包装的格式不应该改变，所以旧版本的Cargo应该能够使用它们。

索引存储在git存储库中，这样Cargo就可以高效地获取索引的增量更新。存储库的根目录中有一个名为`config.json`其中包含Cargo用于访问注册表的JSON信息。这是一个例子，说明[crates.io]配置文件如下所示：

```javascript
{
    "dl": "https://crates.io/api/v1/crates",
    "api": "https://crates.io"
}
```

关键是：

-   `dl`：这是下载索引中列出的箱子的URL。该值可能具有以下标记，这些标记将替换为相应的值：

    -   `{crate}`：箱子的名称。
    -   `{version}`：箱子版本。
    -   `{prefix}`：根据箱子名称计算的目录前缀。例如，一个名为`cargo`前缀为`ca/rg`.详情见下文。
    -   `{lowerprefix}`：的小写变体`{prefix}`.
    -   `{sha256-checksum}`：箱子的sha256校验和。

    如果不存在任何标记，则值`/{crate}/{version}/download`被附加到结尾。

-   `api`：这是web API的基本URL。此键是可选的，但如果未指定，则可以使用以下命令：[`cargo publish`]行不通。下面介绍web API。

下载端点应发送`.crate`请求的包的文件。Cargo支持https、http和文件URL、http重定向、HTTP1和HTTP2。TLS支持的具体细节取决于Cargo运行的平台、Cargo的版本以及编译方式。

索引存储库的其余部分为每个包包含一个文件，其中文件名是包的小写名称。每个版本的软件包在文件中都有单独的一行。这些文件被组织在一层目录中：

-   名称为1个字符的包被放置在名为`1`.
-   名为2个字符的包被放置在名为`2`.
-   具有3个字符名称的包被放置在目录中`3/{first-character}`哪里`{first-character}`是包名称的第一个字符。
-   所有其他包都存储在名为`{first-two}/{second-two}`其中，顶层目录是包名的前两个字符，下一个子目录是包名的第三个和第四个字符。例如`cargo`将存储在名为`ca/rg/cargo`.

> 注意：虽然索引文件名是小写的，但包含包名的字段`Cargo.toml`索引JSON数据区分大小写，可能包含大小写字符。

上面的目录名是根据转换成小写的包名计算的；它由标记表示`{lowerprefix}`.当使用原始包名而不进行大小写转换时，生成的目录名由标记表示`{prefix}`.例如，包`MyCrate`会有一个`{prefix}`属于`My/Cr`还有`{lowerprefix}`属于`my/cr`.一般来说，使用`{prefix}`推荐超过`{lowerprefix}`，但每种选择都有利弊。使用`{prefix}`在不区分大小写的文件系统上，会导致（无害但不美观的）目录别名。例如`crate`和`CrateTwo`有`{prefix}`价值观`cr/at`和`Cr/at`; 这些在Unix机器上是不同的，但在Windows上是同一目录的别名。使用大小写规范化的目录可以避免别名，但在区分大小写的文件系统上，更难支持缺少大小写的旧版本的Cargo`{prefix}`/`{lowerprefix}`。例如，nginx重写规则可以轻松构建`{prefix}`但无法将大小写转换为构造`{lowerprefix}`.

注册中心应考虑对包索引中添加的包名称执行限制。Cargo本身允许使用任何名称[alphanumeric], `-`或`_`角色。[crates.io]施加了自身的限制，包括：

-   仅允许使用ASCII字符。
-   只有字母数字，`-`和`_`角色。
-   第一个字符必须按字母顺序排列。
-   不区分大小写的碰撞检测。
-   防止分歧`-` vs `_`.
-   在特定长度下（最大64）。
-   拒绝保留名称，例如Windows特殊文件名，如“nul”。

注册中心应考虑合并类似的限制，并考虑安全性的影响，例如[IDN homograph
attacks](https://en.wikipedia.org/wiki/IDN_homograph_attack)以及其他方面的担忧[UTR36](https://www.unicode.org/reports/tr36/)和[UTS39](https://www.unicode.org/reports/tr39/).

包文件中的每一行都包含一个JSON对象，该对象描述包的发布版本。下面是一个漂亮的打印示例，带有解释条目格式的注释。

```javascript
{
    // The name of the package.
    // This must only contain alphanumeric, `-`, or `_` characters.
    "name": "foo",
    // The version of the package this row is describing.
    // This must be a valid version number according to the Semantic
    // Versioning 2.0.0 spec at https://semver.org/.
    "vers": "0.1.0",
    // Array of direct dependencies of the package.
    "deps": [
        {
            // Name of the dependency.
            // If the dependency is renamed from the original package name,
            // this is the new name. The original package name is stored in
            // the `package` field.
            "name": "rand",
            // The SemVer requirement for this dependency.
            // This must be a valid version requirement defined at
            // https://doc.rust-lang.org/cargo/reference/specifying-dependencies.html.
            "req": "^0.6",
            // Array of features (as strings) enabled for this dependency.
            "features": ["i128_support"],
            // Boolean of whether or not this is an optional dependency.
            "optional": false,
            // Boolean of whether or not default features are enabled.
            "default_features": true,
            // The target platform for the dependency.
            // null if not a target dependency.
            // Otherwise, a string such as "cfg(windows)".
            "target": null,
            // The dependency kind.
            // "dev", "build", or "normal".
            // Note: this is a required field, but a small number of entries
            // exist in the crates.io index with either a missing or null
            // `kind` field due to implementation bugs.
            "kind": "normal",
            // The URL of the index of the registry where this dependency is
            // from as a string. If not specified or null, it is assumed the
            // dependency is in the current registry.
            "registry": null,
            // If the dependency is renamed, this is a string of the actual
            // package name. If not specified or null, this dependency is not
            // renamed.
            "package": null,
        }
    ],
    // A SHA256 checksum of the `.crate` file.
    "cksum": "d867001db0e2b6e0496f9fac96930e2d42233ecd3ca0413e0753d4c7695d289c",
    // Set of features defined for the package.
    // Each feature maps to an array of features or dependencies it enables.
    "features": {
        "extras": ["rand/simd_support"]
    },
    // Boolean of whether or not this version has been yanked.
    "yanked": false,
    // The `links` string value from the package's manifest, or null if not
    // specified. This field is optional and defaults to null.
    "links": null
}
```

除了`yanked`其值可能随时更改的字段。

### Web API

注册表可以在中定义的位置托管web API`config.json`支持下面列出的任何操作。

Cargo包括`Authorization`需要身份验证的请求的标头。标头值是API标记。如果令牌无效，服务器应使用403响应代码进行响应。用户需要访问注册中心的网站以获取代币，Cargo可以使用[`cargo login`]命令，或通过在命令行上传递令牌。

响应使用200响应代码表示成功和错误。Cargo查看JSON响应以确定是成功还是失败。故障响应有一个JSON对象，其结构如下：

```javascript
{
    // Array of errors to display to the user.
    "errors": [
        {
            // The error message as a string.
            "detail": "error message text"
        }
    ]
}
```

服务器还可以使用404响应代码进行响应，以指示未找到请求的资源（例如，未知的箱子名称）。但是，使用200响应和`errors`对象允许注册表在需要时提供更详细的错误消息。

为了向后兼容，服务器应该忽略任何意外的查询参数或JSON字段。如果缺少JSON字段，则应假定该字段为空。端点的版本由`v1`路径的组件，Cargo负责处理将来需要的向后兼容性回退。

Cargo为所有请求设置以下标题：

-   `Content-Type`: `application/json`
-   `Accept`: `application/json`
-   `User-Agent`：Cargo版本，例如`cargo 1.32.0 (8610973aa
    2019-01-02)`。用户可在配置值中对此进行修改。添加到1.29中。

#### Publish

-   终点：`/api/v1/crates/new`
-   方法：放
-   授权：包括

发布端点用于发布箱子的新版本。服务器应验证箱子，使其可供下载，并将其添加到索引中。

Cargo发送的数据主体是：

-   JSON数据长度的32位无符号小整数。
-   作为JSON对象的包元数据。
-   数据长度的32位无符号小整数`.crate`文件
-   这个`.crate`文件

下面是JSON对象的注释示例。关于美国实施的一些限制的一些注释[crates.io]仅用于说明可能进行的验证类型的一些建议，不应被视为限制的详尽列表[crates.io]强加。

```javascript
{
    // The name of the package.
    "name": "foo",
    // The version of the package being published.
    "vers": "0.1.0",
    // Array of direct dependencies of the package.
    "deps": [
        {
            // Name of the dependency.
            // If the dependency is renamed from the original package name,
            // this is the original name. The new package name is stored in
            // the `explicit_name_in_toml` field.
            "name": "rand",
            // The semver requirement for this dependency.
            "version_req": "^0.6",
            // Array of features (as strings) enabled for this dependency.
            "features": ["i128_support"],
            // Boolean of whether or not this is an optional dependency.
            "optional": false,
            // Boolean of whether or not default features are enabled.
            "default_features": true,
            // The target platform for the dependency.
            // null if not a target dependency.
            // Otherwise, a string such as "cfg(windows)".
            "target": null,
            // The dependency kind.
            // "dev", "build", or "normal".
            "kind": "normal",
            // The URL of the index of the registry where this dependency is
            // from as a string. If not specified or null, it is assumed the
            // dependency is in the current registry.
            "registry": null,
            // If the dependency is renamed, this is a string of the new
            // package name. If not specified or null, this dependency is not
            // renamed.
            "explicit_name_in_toml": null,
        }
    ],
    // Set of features defined for the package.
    // Each feature maps to an array of features or dependencies it enables.
    // Cargo does not impose limitations on feature names, but crates.io
    // requires alphanumeric ASCII, `_` or `-` characters.
    "features": {
        "extras": ["rand/simd_support"]
    },
    // List of strings of the authors.
    // May be empty.
    "authors": ["Alice <a@example.com>"],
    // Description field from the manifest.
    // May be null. crates.io requires at least some content.
    "description": null,
    // String of the URL to the website for this package's documentation.
    // May be null.
    "documentation": null,
    // String of the URL to the website for this package's home page.
    // May be null.
    "homepage": null,
    // String of the content of the README file.
    // May be null.
    "readme": null,
    // String of a relative path to a README file in the crate.
    // May be null.
    "readme_file": null,
    // Array of strings of keywords for the package.
    "keywords": [],
    // Array of strings of categories for the package.
    "categories": [],
    // String of the license for the package.
    // May be null. crates.io requires either `license` or `license_file` to be set.
    "license": null,
    // String of a relative path to a license file in the crate.
    // May be null.
    "license_file": null,
    // String of the URL to the website for the source repository of this package.
    // May be null.
    "repository": null,
    // Optional object of "status" badges. Each value is an object of
    // arbitrary string to string mappings.
    // crates.io has special interpretation of the format of the badges.
    "badges": {
        "travis-ci": {
            "branch": "master",
            "repository": "rust-lang/cargo"
        }
    },
    // The `links` string value from the package's manifest, or null if not
    // specified. This field is optional and defaults to null.
    "links": null
}
```

成功的响应包括JSON对象：

```javascript
{
    // Optional object of warnings to display to the user.
    "warnings": {
        // Array of strings of categories that are invalid and ignored.
        "invalid_categories": [],
        // Array of strings of badge names that are invalid and ignored.
        "invalid_badges": [],
        // Array of strings of arbitrary warnings to display to the user.
        "other": []
    }
}
```

#### Yank

-   终点：`/api/v1/crates/{crate_name}/{version}/yank`
-   方法：删除
-   授权：包括

yank端点将设置`yank`指定箱子版本的字段`true`在索引中。

成功的响应包括JSON对象：

```javascript
{
    // Indicates the delete succeeded, always true.
    "ok": true,
}
```

#### Unyank

-   终点：`/api/v1/crates/{crate_name}/{version}/unyank`
-   方法：放
-   授权：包括

unyank端点将设置`yank`指定箱子版本的字段`false`在索引中。

成功的响应包括JSON对象：

```javascript
{
    // Indicates the delete succeeded, always true.
    "ok": true,
}
```

#### Owners

Cargo没有用户和所有者的固有概念，但它确实提供了`owner`协助管理有权控制箱子的人员的命令。由注册中心决定用户和所有者的具体处理方式。看到了吗[publishing documentation]想了解一下[crates.io]通过GitHub用户和团队处理所有者。

##### Owners: List

-   终点：`/api/v1/crates/{crate_name}/owners`
-   方法：获取
-   授权：包括

owners端点返回箱子所有者的列表。

成功的响应包括JSON对象：

```javascript
{
    // Array of owners of the crate.
    "users": [
        {
            // Unique unsigned 32-bit integer of the owner.
            "id": 70,
            // The unique username of the owner.
            "login": "github:rust-lang:core",
            // Name of the owner.
            // This is optional and may be null.
            "name": "Core",
        }
    ]
}
```

##### Owners: Add

-   终点：`/api/v1/crates/{crate_name}/owners`
-   方法：放
-   授权：包括

PUT请求将向注册表发送向箱子添加新所有者的请求。如何处理请求取决于注册表。例如[crates.io]向用户发送一个邀请，用户在添加之前必须接受该邀请。

请求应包括以下JSON对象：

```javascript
{
    // Array of `login` strings of owners to add.
    "users": ["login_name"]
}
```

成功的响应包括JSON对象：

```javascript
{
    // Indicates the add succeeded, always true.
    "ok": true,
    // A string to be displayed to the user.
    "msg": "user ehuss has been invited to be an owner of crate cargo"
}
```

##### Owners: Remove

-   终点：`/api/v1/crates/{crate_name}/owners`
-   方法：删除
-   授权：包括

删除请求将从箱子中删除所有者。请求应包括以下JSON对象：

```javascript
{
    // Array of `login` strings of owners to remove.
    "users": ["login_name"]
}
```

成功的响应包括JSON对象：

```javascript
{
    // Indicates the remove succeeded, always true.
    "ok": true
}
```

#### Search

-   终点：`/api/v1/crates`
-   方法：获取
-   查询参数：
    -   `q`：搜索查询字符串。
    -   `per_page`：结果数，默认为10，最大为100。

搜索请求将使用服务器上定义的条件搜索箱子。

成功的响应包括JSON对象：

```javascript
{
    // Array of results.
    "crates": [
        {
            // Name of the crate.
            "name": "rand",
            // The highest version available.
            "max_version": "0.6.1",
            // Textual description of the crate.
            "description": "Random number generators and other randomness functionality.\n",
        }
    ],
    "meta": {
        // Total number of results available on the server.
        "total": 119
    }
}
```

#### Login

-   终点：`/me`

“登录”端点不是实际的API请求。它的存在完全是为了[`cargo login`]命令显示URL，以指示用户在web浏览器中访问以登录并检索API令牌。

[source replacement]: source-replacement.md

[`cargo login`]: ../commands/cargo-login.md

[`cargo package`]: ../commands/cargo-package.md

[`cargo publish`]: ../commands/cargo-publish.md

[alphanumeric]: ../../std/primitive.char.html#method.is_alphanumeric

[config]: config.md

[crates.io]: https://crates.io/

[publishing documentation]: publishing.md#cargo-owner
