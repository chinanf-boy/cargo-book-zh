
## 包ID规格

### 包ID规格

Cargo的子命令经常需要引用依赖关系图中的特定包来进行各种操作,例如更新,清理,构建等.为了解决这个问题,Cargo支持包ID规范.规范是一个字符串,用于唯一地引用包图形中的一个包.

#### 规范语法

包ID规范的形式语法是:

```notrust
pkgid := pkgname
       | [ proto "://" ] hostname-and-path [ "#" ( pkgname | semver ) ]
pkgname := name [ ":" semver ]

proto := "http" | "git" | ...
```

这里,括号表示内容是可选的.

#### 示例规格

这些都可以是对包的引用`foo`版`1.2.3`来自注册表`crates.io`

|                        pkgid |  名称 |   版   |           网址           |
| ---------------------------: | :-: | :---: | :--------------------: |
|                        `foo` | foo |   \*  |           \*           |
|                  `foo:1.2.3` | foo | 1.2.3 |           \*           |
|              `crates.io/foo` | foo |   \*  |   \*://crates.io/foo   |
|        `crates.io/foo#1.2.3` | foo | 1.2.3 |   \*://crates.io/foo   |
|    `crates.io/bar#foo:1.2.3` | foo | 1.2.3 |   \*://crates.io/bar   |
| `http://crates.io/foo#1.2.3` | foo | 1.2.3 | <http://crates.io/foo> |

#### 规格简洁

这样做的目的是启用简洁和详尽的语法来引用依赖图中的包.不明确的引用可以指代一个或多个包.如果可以使用相同的规范引用多个包,则大多数命令会生成错误.
