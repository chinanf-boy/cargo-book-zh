# SemVer Compatibility

本章详细介绍了对于新版本的软件包，通常认为什么是兼容的或破坏性的SemVer更改。看到了吗[SemVer compatibility]本节详细介绍了什么是SemVer，以及Cargo如何使用它来确保库的兼容性。

这些只是*指导方针*，而不是所有项目都必须遵守的硬性规定。这个[Change categories]第节详细介绍了本指南如何对变更的级别和严重性进行分类。本指南的大部分内容侧重于将导致`cargo`和`rustc`未能建造出以前有用的东西。几乎每一个变更都会带来一些风险，即它会对运行时行为产生负面影响，在这些情况下，项目维护人员通常会判断它是否是一个SemVer不兼容的变更。

另见[rust-semverver]，这是一个尝试以编程方式检查兼容性规则的实验工具。

[change categories]: #change-categories

[rust-semverver]: https://github.com/rust-dev-tools/rust-semverver

[semver compatibility]: resolver.md#semver-compatibility

## Change categories

以下列出的所有政策均按变化程度分类：

-   **重大变化**：需要进行大幅度碰撞的更改。
-   **小变化**：只需要轻微碰撞的更改。
-   **可能是零钱**一些项目可能会考虑重大变化，而其他项目则考虑较小。

“可能突破”类别包括具有以下特征的更改：*潜在的*在更新过程中中断，但不一定会导致中断。应仔细考虑这些变化的影响。具体性质将取决于变更和项目维护人员的原则。

一些项目可能会选择只在一个小的更改上增加补丁号。建议遵循SemVer规范，并且只在补丁版本中应用错误修复。然而，一个bug修复可能需要一个标记为“微小更改”的API更改，并且不应该影响兼容性。本指南没有对如何对待每个“微小变化”采取立场，因为微小变化和补丁变化之间的区别是取决于变化性质的惯例。

有些更改被标记为“微小”，尽管它们有破坏构建的潜在风险。这适用于可能性极低的情况，并且潜在的破坏性代码不太可能用惯用语言编写，或者特别不鼓励使用。

本指南使用术语“主要”和“次要”，假设这与“1.0.0”或更高版本有关。以“0.y.z”开头的初始开发版本可以将“y”中的更改视为主要版本，“z”中的更改视为次要版本。“0.0.z”版本总是主要的变化。这是因为Cargo使用的约定是，只有最左边的非零组件中的更改才被视为不兼容。

-   API兼容性
    -   项目
        -   [Major: renaming/moving/removing any public items](#item-remove)
        -   [Minor: adding new public items](#item-new)
    -   结构体
        -   [Major: adding a private struct field when all current fields are public](#struct-add-private-field-when-public)
        -   [Major: adding a public field when no private field exists](#struct-add-public-field-when-no-private)
        -   [Minor: adding or removing private fields when at least one already exists](#struct-private-fields-with-private)
        -   [Minor: going from a tuple struct with all private fields (with at least one field) to a normal struct, or vice versa](#struct-tuple-normal-with-private)
    -   枚举类型
        -   [Major: adding new enum variants (without `non_exhaustive`)](#enum-variant-new)
        -   [Major: adding new fields to an enum variant](#enum-fields-new)
    -   特点
        -   [Major: adding a non-defaulted trait item](#trait-new-item-no-default)
        -   [Major: any change to trait item signatures](#trait-item-signature)
        -   [Possibly-breaking: adding a defaulted trait item](#trait-new-default-item)
        -   [Major: adding a trait item that makes the trait non-object safe](#trait-object-safety)
        -   [Major: adding a type parameter without a default](#trait-new-parameter-no-default)
        -   [Minor: adding a defaulted trait type parameter](#trait-new-parameter-default)
    -   启动位置
        -   [Possibly-breaking change: adding any inherent items](#impl-item-new)
    -   仿制药
        -   [Major: tightening generic bounds](#generic-bounds-tighten)
        -   [Minor: loosening generic bounds](#generic-bounds-loosen)
        -   [Minor: adding defaulted type parameters](#generic-new-default)
        -   [Minor: generalizing a type to use generics (with identical types)](#generic-generalize-identical)
        -   [Major: generalizing a type to use generics (with possibly different types)](#generic-generalize-different)
        -   [Minor: changing a generic type to a more generic type](#generic-more-generic)
    -   功能
        -   [Major: adding/removing function parameters](#fn-change-arity)
        -   [Possibly-breaking: introducing a new function type parameter](#fn-generic-new)
        -   [Minor: generalizing a function to use generics (supporting original type)](#fn-generalize-compatible)
        -   [Major: generalizing a function to use generics with type mismatch](#fn-generalize-mismatch)
    -   属性
        -   [Major: switching from `no_std` support to requiring `std`](#attr-no-std-to-std)
-   工具和环境兼容性
    -   [Possibly-breaking: changing the minimum version of Rust required](#env-new-rust)
    -   [Possibly-breaking: changing the platform and environment requirements](#env-change-requirements)
    -   Cargo
        -   [Minor: adding a new Cargo feature](#cargo-feature-add)
        -   [Major: removing a Cargo feature](#cargo-feature-remove)
        -   [Major: removing a feature from a feature list if that changes functionality or public items](#cargo-feature-remove-another)
        -   [Possibly-breaking: removing an optional dependency](#cargo-remove-opt-dep)
        -   [Minor: changing dependency features](#cargo-change-dep-feature)
        -   [Minor: adding dependencies](#cargo-dep-add)
-   [Application compatibility](#application-compatibility)

## API compatibility

下面的所有示例都包含三个部分：原始代码、修改后的代码，以及可能出现在另一个项目中的代码用法示例。在一个小的改变中，示例用法应该成功地使用之前和之后的版本构建。

<a id="item-remove"></a>

### Major: renaming/moving/removing any public items

没有公开曝光的[item][items]将导致该项的任何使用无法编译。

```rust,ignore
// MAJOR CHANGE

///////////////////////////////////////////////////////////
// Before
pub fn foo() {}

///////////////////////////////////////////////////////////
// After
// ... item has been removed

///////////////////////////////////////////////////////////
// Example usage that will break.
fn main() {
    updated_crate::foo(); // Error: cannot find function `foo`
}
```

这包括添加任何类型的[`cfg` attribute]它可以根据以下内容更改可用的项目或行为：[conditional compilation].

缓解策略：

-   将要删除的项目标记为[deprecated]，然后在以后的版本中删除它们。
-   将重命名的项目标记为[deprecated]，并使用[`pub use`]要重新导出为旧名称的项。

<a id="item-new"></a>

### Minor: adding new public items

添加新的、公开的[items]这是一个小变化。

```rust,ignore
// MINOR CHANGE

///////////////////////////////////////////////////////////
// Before
// ... absence of item

///////////////////////////////////////////////////////////
// After
pub fn foo() {}

///////////////////////////////////////////////////////////
// Example use of the library that will safely work.
// `foo` is not used since it didn't previously exist.
```

请注意，在某些罕见的情况下，这可能是一个错误**零钱**由于全球进口。例如，如果您添加了一个新特性，而一个项目使用了一个全局导入，将该特性引入范围，并且新特性引入了一个关联项，该项与它所实现的任何类型都冲突，这可能会由于不明确而导致编译时错误。例子：

```rust,ignore
// Breaking change example

///////////////////////////////////////////////////////////
// Before
// ... absence of trait

///////////////////////////////////////////////////////////
// After
pub trait NewTrait {
    fn foo(&self) {}
}

impl NewTrait for i32 {}

///////////////////////////////////////////////////////////
// Example usage that will break.
use updated_crate::*;

pub trait LocalTrait {
    fn foo(&self) {}
}

impl LocalTrait for i32 {}

fn main() {
    123i32.foo(); // Error:  multiple applicable items in scope
}
```

这不被认为是一个重大变化，因为传统上，glob进口是一个已知的兼容性风险。Glob应避免从外部板条箱进口物品。

<a id="struct-add-private-field-when-public"></a>

### Major: adding a private struct field when all current fields are public

当一个私有字段被添加到一个以前拥有所有公共字段的结构中时，这将破坏任何试图用[struct literal].

```rust,ignore
// MAJOR CHANGE

///////////////////////////////////////////////////////////
// Before
pub struct Foo {
    pub f1: i32,
}

///////////////////////////////////////////////////////////
// After
pub struct Foo {
    pub f1: i32,
    f2: i32,
}

///////////////////////////////////////////////////////////
// Example usage that will break.
fn main() {
    let x = updated_crate::Foo { f1: 123 }; // Error: cannot construct `Foo`
}
```

缓解策略：

-   不要向所有公共字段结构添加新字段。
-   将结构标记为[`#[non_exhaustive]`][non_exhaustive]当第一次引入struct以防止用户使用struct literal语法时，而是提供构造函数方法和/或[Default]实施

<a id="struct-add-public-field-when-no-private"></a>

### Major: adding a public field when no private field exists

当一个公共字段被添加到一个包含所有公共字段的结构中时，这将破坏任何试图用公共字段构造它的代码[struct literal].

```rust,ignore
// MAJOR CHANGE

///////////////////////////////////////////////////////////
// Before
pub struct Foo {
    pub f1: i32,
}

///////////////////////////////////////////////////////////
// After
pub struct Foo {
    pub f1: i32,
    pub f2: i32,
}

///////////////////////////////////////////////////////////
// Example usage that will break.
fn main() {
    let x = updated_crate::Foo { f1: 123 }; // Error: missing field `f2`
}
```

缓解策略：

-   不要向所有公共字段结构添加新字段。
-   将结构标记为[`#[non_exhaustive]`][non_exhaustive]当第一次引入struct以防止用户使用struct literal语法时，而是提供构造函数方法和/或[Default]实施

<a id="struct-private-fields-with-private"></a>

### Minor: adding or removing private fields when at least one already exists

当结构已经至少有一个私有字段时，可以安全地从结构中添加或删除私有字段。

```rust,ignore
// MINOR CHANGE

///////////////////////////////////////////////////////////
// Before
#[derive(Default)]
pub struct Foo {
    f1: i32,
}

///////////////////////////////////////////////////////////
// After
#[derive(Default)]
pub struct Foo {
    f2: f64,
}

///////////////////////////////////////////////////////////
// Example use of the library that will safely work.
fn main() {
    // Cannot access private fields.
    let x = updated_crate::Foo::default();
}
```

这是安全的，因为现有代码不能使用[struct literal]去构建它，也不完全匹配它的内容。

请注意，对于元组结构，这是一个**重大变化**如果元组包含公共字段，并且添加或删除私有字段会更改任何公共字段的索引。

```rust,ignore
// MAJOR CHANGE

///////////////////////////////////////////////////////////
// Before
#[derive(Default)]
pub struct Foo(pub i32, i32);

///////////////////////////////////////////////////////////
// After
#[derive(Default)]
pub struct Foo(f64, pub i32, i32);

///////////////////////////////////////////////////////////
// Example usage that will break.
fn main() {
    let x = updated_crate::Foo::default();
    let y = x.0; // Error: is private
}
```

<a id="struct-tuple-normal-with-private"></a>

### Minor: going from a tuple struct with all private fields (with at least one field) to a normal struct, or vice versa

如果所有字段都是私有的，那么将元组结构更改为普通结构（反之亦然）是安全的。

```rust,ignore
// MINOR CHANGE

///////////////////////////////////////////////////////////
// Before
#[derive(Default)]
pub struct Foo(i32);

///////////////////////////////////////////////////////////
// After
#[derive(Default)]
pub struct Foo {
    f1: i32,
}

///////////////////////////////////////////////////////////
// Example use of the library that will safely work.
fn main() {
    // Cannot access private fields.
    let x = updated_crate::Foo::default();
}
```

这是安全的，因为现有代码不能使用[struct literal]构建它，也不匹配它的内容。

<a id="enum-variant-new"></a>

### Major: adding new enum variants (without `non_exhaustive`)

如果枚举不使用[`#[non_exhaustive]`][non_exhaustive]属性

```rust,ignore
// MAJOR CHANGE

///////////////////////////////////////////////////////////
// Before
pub enum E {
    Variant1,
}

///////////////////////////////////////////////////////////
// After
pub enum E {
    Variant1,
    Variant2,
}

///////////////////////////////////////////////////////////
// Example usage that will break.
fn main() {
    use updated_crate::E;
    let x = E::Variant1;
    match x { // Error: `Variant2` not covered
        E::Variant1 => {}
    }
}
```

缓解策略：

-   引入枚举时，将其标记为[`#[non_exhaustive]`][non_exhaustive]强制用户使用[wildcard patterns]捕捉新的变种。

<a id="enum-fields-new"></a>

### Major: adding new fields to an enum variant

向枚举变量添加新字段是一个突破性的改变，因为所有字段都是公共的，构造函数和匹配项将无法编译。

```rust,ignore
// MAJOR CHANGE

///////////////////////////////////////////////////////////
// Before
pub enum E {
    Variant1 { f1: i32 },
}

///////////////////////////////////////////////////////////
// After
pub enum E {
    Variant1 { f1: i32, f2: i32 },
}

///////////////////////////////////////////////////////////
// Example usage that will break.
fn main() {
    use updated_crate::E;
    let x = E::Variant1 { f1: 1 }; // Error: missing f2
    match x {
        E::Variant1 { f1 } => {} // Error: missing f2
    }
}
```

缓解策略：

-   引入枚举时，将变量标记为[`non_exhaustive`][non_exhaustive]因此，如果没有通配符，就无法构造或匹配它。
    ```rust,ignore,skip
    pub enum E {
        #[non_exhaustive]
        Variant1{f1: i32}
    }
    ```
-   引入枚举时，使用显式结构作为值，这样可以控制字段的可见性。
    ```rust,ignore,skip
    pub struct Foo {
       f1: i32,
       f2: i32,
    }
    pub enum E {
        Variant1(Foo)
    }
    ```

<a id="trait-new-item-no-default"></a>

### Major: adding a non-defaulted trait item

将非默认项添加到特征中是一个突破性的改变。这将打破这种特性的任何实现者。

```rust,ignore
// MAJOR CHANGE

///////////////////////////////////////////////////////////
// Before
pub trait Trait {}

///////////////////////////////////////////////////////////
// After
pub trait Trait {
    fn foo(&self);
}

///////////////////////////////////////////////////////////
// Example usage that will break.
use updated_crate::Trait;
struct Foo;

impl Trait for Foo {}  // Error: not all trait items implemented
```

缓解策略：

-   始终为新关联的特征项提供默认实现或值。
-   在介绍这种特质时，使用[sealed trait]防止板条箱外的用户实现该特性的技术。

<a id="trait-item-signature"></a>

### Major: any change to trait item signatures

对特征项签名进行任何更改都是一个突破性的改变。这可能会打破这种特质的外部实现者。

```rust,ignore
// MAJOR CHANGE

///////////////////////////////////////////////////////////
// Before
pub trait Trait {
    fn f(&self, x: i32) {}
}

///////////////////////////////////////////////////////////
// After
pub trait Trait {
    // For sealed traits or normal functions, this would be a minor change
    // because generalizing with generics strictly expands the possible uses.
    // But in this case, trait implementations must use the same signature.
    fn f<V>(&self, x: V) {}
}

///////////////////////////////////////////////////////////
// Example usage that will break.
use updated_crate::Trait;
struct Foo;

impl Trait for Foo {
    fn f(&self, x: i32) {}  // Error: trait declaration has 1 type parameter
}
```

缓解策略：

-   引入带有默认实现的新项，以覆盖新功能，而不是修改现有项。
-   在介绍这种特质时，使用[sealed trait]防止板条箱外的用户实现该特性的技术。

<a id="trait-new-default-item"></a>

### Possibly-breaking: adding a defaulted trait item

添加默认特征项通常是安全的。然而，这有时会导致编译错误。例如，如果另一个特征中存在同名的方法，这可能会导致歧义。

```rust,ignore
// Breaking change example

///////////////////////////////////////////////////////////
// Before
pub trait Trait {}

///////////////////////////////////////////////////////////
// After
pub trait Trait {
    fn foo(&self) {}
}

///////////////////////////////////////////////////////////
// Example usage that will break.
use updated_crate::Trait;
struct Foo;

trait LocalTrait {
    fn foo(&self) {}
}

impl Trait for Foo {}
impl LocalTrait for Foo {}

fn main() {
    let x = Foo;
    x.foo(); // Error: multiple applicable items in scope
}
```

请注意，这种模糊性确实存在*不*存在的名称冲突[inherent
implementations]，因为他们优先于特质项目。

看见[trait-object-safety](#trait-object-safety)在添加特性项时要考虑的特殊情况。

缓解策略：

-   一些项目可能会认为这种破损是可以接受的，尤其是如果新项目名称不太可能与任何现有代码冲突的话。仔细选择名称以避免这些冲突。此外，要求下游用户添加[disambiguation syntax]更新依赖项时选择正确的函数。

<a id="trait-object-safety"></a>

### Major: adding a trait item that makes the trait non-object safe

这是一个突破性的改变，添加一个特性项，改变特性为不存在[object safe].

```rust,ignore
// MAJOR CHANGE

///////////////////////////////////////////////////////////
// Before
pub trait Trait {}

///////////////////////////////////////////////////////////
// After
pub trait Trait {
    // An associated const makes the trait not object-safe.
    const CONST: i32 = 123;
}

///////////////////////////////////////////////////////////
// Example usage that will break.
use updated_crate::Trait;
struct Foo;

impl Trait for Foo {}

fn main() {
    let obj: Box<dyn Trait> = Box::new(Foo); // Error: cannot be made into an object
}
```

反过来做是安全的（把一个非对象安全的特征变成一个安全的特征）。

<a id="trait-new-parameter-no-default"></a>

### Major: adding a type parameter without a default

在没有默认特性的情况下添加类型参数是一个突破性的改变。

```rust,ignore
// MAJOR CHANGE

///////////////////////////////////////////////////////////
// Before
pub trait Trait {}

///////////////////////////////////////////////////////////
// After
pub trait Trait<T> {}

///////////////////////////////////////////////////////////
// Example usage that will break.
use updated_crate::Trait;
struct Foo;

impl Trait for Foo {}  // Error: missing generics
```

缓解策略：

-   看见[adding a defaulted trait type parameter](#trait-new-parameter-default).

<a id="trait-new-parameter-default"></a>

### Minor: adding a defaulted trait type parameter

向trait添加类型参数是安全的，只要它有默认值。外部实现者将使用默认值，而无需指定参数。

```rust,ignore
// MINOR CHANGE

///////////////////////////////////////////////////////////
// Before
pub trait Trait {}

///////////////////////////////////////////////////////////
// After
pub trait Trait<T = i32> {}

///////////////////////////////////////////////////////////
// Example use of the library that will safely work.
use updated_crate::Trait;
struct Foo;

impl Trait for Foo {}
```

<a id="impl-item-new"></a>

### Possibly-breaking change: adding any inherent items

通常，向实现中添加固有项应该是安全的，因为固有项优先于特性项。然而，在某些情况下，如果名称与具有不同签名的实现特征项相同，冲突可能会导致问题。

```rust,ignore
// Breaking change example

///////////////////////////////////////////////////////////
// Before
pub struct Foo;

///////////////////////////////////////////////////////////
// After
pub struct Foo;

impl Foo {
    pub fn foo(&self) {}
}

///////////////////////////////////////////////////////////
// Example usage that will break.
use updated_crate::Foo;

trait Trait {
    fn foo(&self, x: i32) {}
}

impl Trait for Foo {}

fn main() {
    let x = Foo;
    x.foo(1); // Error: this function takes 0 arguments
}
```

请注意，如果签名匹配，则不会出现编译时错误，但运行时行为可能会发生无声更改（因为它现在正在执行不同的函数）。

缓解策略：

-   一些项目可能会认为这种破损是可以接受的，尤其是如果新项目名称不太可能与任何现有代码冲突的话。仔细选择名称以避免这些冲突。此外，要求下游用户添加[disambiguation syntax]更新依赖项时选择正确的函数。

<a id="generic-bounds-tighten"></a>

### Major: tightening generic bounds

收紧类型的泛型界限是一个突破性的改变，因为这会打破用户对宽松界限的期望。

```rust,ignore
// MAJOR CHANGE

///////////////////////////////////////////////////////////
// Before
pub struct Foo<A> {
    pub f1: A,
}

///////////////////////////////////////////////////////////
// After
pub struct Foo<A: Eq> {
    pub f1: A,
}

///////////////////////////////////////////////////////////
// Example usage that will break.
use updated_crate::Foo;

fn main() {
    let s = Foo { f1: 1.23 }; // Error: the trait bound `{float}: Eq` is not satisfied
}
```

<a id="generic-bounds-loosen"></a>

### Minor: loosening generic bounds

放松类型的泛型界限是安全的，因为它只扩展了允许的范围。

```rust,ignore
// MINOR CHANGE

///////////////////////////////////////////////////////////
// Before
pub struct Foo<A: Clone> {
    pub f1: A,
}

///////////////////////////////////////////////////////////
// After
pub struct Foo<A> {
    pub f1: A,
}

///////////////////////////////////////////////////////////
// Example use of the library that will safely work.
use updated_crate::Foo;

fn main() {
    let s = Foo { f1: 123 };
}
```

<a id="generic-new-default"></a>

### Minor: adding defaulted type parameters

向类型添加类型参数是安全的，只要它有默认值。所有现有引用都将使用默认值，而无需指定参数。

```rust,ignore
// MINOR CHANGE

///////////////////////////////////////////////////////////
// Before
#[derive(Default)]
pub struct Foo {}

///////////////////////////////////////////////////////////
// After
#[derive(Default)]
pub struct Foo<A = i32> {
    f1: A,
}

///////////////////////////////////////////////////////////
// Example use of the library that will safely work.
use updated_crate::Foo;

fn main() {
    let s: Foo = Default::default();
}
```

<a id="generic-generalize-identical"></a>

### Minor: generalizing a type to use generics (with identical types)

结构或枚举字段可以从具体类型更改为泛型类型参数，前提是更改会导致所有现有用例的类型相同。例如，允许进行以下更改：

```rust,ignore
// MINOR CHANGE

///////////////////////////////////////////////////////////
// Before
pub struct Foo(pub u8);

///////////////////////////////////////////////////////////
// After
pub struct Foo<T = u8>(pub T);

///////////////////////////////////////////////////////////
// Example use of the library that will safely work.
use updated_crate::Foo;

fn main() {
    let s: Foo = Foo(123);
}
```

因为现有的`Foo`是的缩写`Foo<u8>`这会产生相同的字段类型。

<a id="generic-generalize-different"></a>

### Major: generalizing a type to use generics (with possibly different types)

如果类型可以更改，将结构或枚举字段从具体类型更改为泛型类型参数可能会中断。

```rust,ignore
// MAJOR CHANGE

///////////////////////////////////////////////////////////
// Before
pub struct Foo<T = u8>(pub T, pub u8);

///////////////////////////////////////////////////////////
// After
pub struct Foo<T = u8>(pub T, pub T);

///////////////////////////////////////////////////////////
// Example usage that will break.
use updated_crate::Foo;

fn main() {
    let s: Foo<f32> = Foo(3.14, 123); // Error: mismatched types
}
```

<a id="generic-more-generic"></a>

### Minor: changing a generic type to a more generic type

将泛型类型更改为更通用的类型是安全的。例如，下面添加了一个默认为原始类型的泛型参数，这是安全的，因为所有现有用户将对这两个字段使用相同的类型，不需要指定默认参数。

```rust,ignore
// MINOR CHANGE

///////////////////////////////////////////////////////////
// Before
pub struct Foo<T>(pub T, pub T);

///////////////////////////////////////////////////////////
// After
pub struct Foo<T, U = T>(pub T, pub U);

///////////////////////////////////////////////////////////
// Example use of the library that will safely work.
use updated_crate::Foo;

fn main() {
    let s: Foo<f32> = Foo(1.0, 2.0);
}
```

<a id="fn-change-arity"></a>

### Major: adding/removing function parameters

改变函数的算术性是一个突破性的改变。

```rust,ignore
// MAJOR CHANGE

///////////////////////////////////////////////////////////
// Before
pub fn foo() {}

///////////////////////////////////////////////////////////
// After
pub fn foo(x: i32) {}

///////////////////////////////////////////////////////////
// Example usage that will break.
fn main() {
    updated_crate::foo(); // Error: this function takes 1 argument
}
```

缓解策略：

-   引入具有新签名的新函数，并可能[deprecate][deprecated]老的那个。
-   介绍采用struct参数的函数，其中struct是使用builder模式构建的。这允许将来将新字段添加到结构中。

<a id="fn-generic-new"></a>

### Possibly-breaking: introducing a new function type parameter

通常，添加非默认类型参数是安全的，但在某些情况下，这可能是一个突破性的更改：

```rust,ignore
// Breaking change example

///////////////////////////////////////////////////////////
// Before
pub fn foo<T>() {}

///////////////////////////////////////////////////////////
// After
pub fn foo<T, U>() {}

///////////////////////////////////////////////////////////
// Example usage that will break.
use updated_crate::foo;

fn main() {
    foo::<u8>(); // Error: this function takes 2 generic arguments but 1 generic argument was supplied
}
```

然而，这种显式调用非常罕见（通常可以用其他方式编写），因此这种中断通常是可以接受的。我们应该考虑使用显式类型参数调用相关函数的可能性。

<a id="fn-generalize-compatible"></a>

### Minor: generalizing a function to use generics (supporting original type)

函数的参数类型或其返回值可以是*广义的*使用泛型，包括引入新的类型参数，只要可以将其实例化为原始类型。例如，允许进行以下更改：

```rust,ignore
// MINOR CHANGE

///////////////////////////////////////////////////////////
// Before
pub fn foo(x: u8) -> u8 {
    x
}
pub fn bar<T: Iterator<Item = u8>>(t: T) {}

///////////////////////////////////////////////////////////
// After
use std::ops::Add;
pub fn foo<T: Add>(x: T) -> T {
    x
}
pub fn bar<T: IntoIterator<Item = u8>>(t: T) {}

///////////////////////////////////////////////////////////
// Example use of the library that will safely work.
use updated_crate::{bar, foo};

fn main() {
    foo(1);
    bar(vec![1, 2, 3].into_iter());
}
```

因为所有现有的使用都是新签名的实例。

也许有些令人惊讶的是，鉴于每个特征都实现了自身，一般化也适用于特征对象：

```rust,ignore
// MINOR CHANGE

///////////////////////////////////////////////////////////
// Before
pub trait Trait {}
pub fn foo(t: &dyn Trait) {}

///////////////////////////////////////////////////////////
// After
pub trait Trait {}
pub fn foo<T: Trait + ?Sized>(t: &T) {}

///////////////////////////////////////////////////////////
// Example use of the library that will safely work.
use updated_crate::{foo, Trait};

struct Foo;
impl Trait for Foo {}

fn main() {
    let obj = Foo;
    foo(&obj);
}
```

（使用`?Sized`是必要的；否则您无法恢复原始签名。）

以这种方式引入泛型可能会导致类型推断失败。这通常很少见，对于某些项目来说可能是可以接受的，因为这可以通过附加类型注释来修复。

```rust,ignore
// Breaking change example

///////////////////////////////////////////////////////////
// Before
pub fn foo() -> i32 {
    0
}

///////////////////////////////////////////////////////////
// After
pub fn foo<T: Default>() -> T {
    Default::default()
}

///////////////////////////////////////////////////////////
// Example usage that will break.
use updated_crate::foo;

fn main() {
    let x = foo(); // Error: type annotations needed
}
```

<a id="fn-generalize-mismatch"></a>

### Major: generalizing a function to use generics with type mismatch

如果泛型类型限制或更改了以前允许的类型，则更改函数参数或返回类型是一种突破性的更改。例如，以下内容添加了现有代码可能无法满足的通用约束：

```rust,ignore
// MAJOR CHANGE

///////////////////////////////////////////////////////////
// Before
pub fn foo(x: Vec<u8>) {}

///////////////////////////////////////////////////////////
// After
pub fn foo<T: Copy + IntoIterator<Item = u8>>(x: T) {}

///////////////////////////////////////////////////////////
// Example usage that will break.
use updated_crate::foo;

fn main() {
    foo(vec![1, 2, 3]); // Error: `Copy` is not implemented for `Vec<u8>`
}
```

<a id="attr-no-std-to-std"></a>

### Major: switching from `no_std` support to requiring `std`

如果您的库特别支持[`no_std`]环境，这是一个突破性的变化，使新的版本需要`std`.

```rust,ignore,skip
// MAJOR CHANGE

///////////////////////////////////////////////////////////
// Before
#![no_std]
pub fn foo() {}

///////////////////////////////////////////////////////////
// After
pub fn foo() {
    std::time::SystemTime::now();
}

///////////////////////////////////////////////////////////
// Example usage that will break.
// This will fail to link for no_std targets because they don't have a `std` crate.
#![no_std]
use updated_crate::foo;

fn example() {
    foo();
}
```

缓解策略：

-   避免这种情况的一个常见习惯用法是包含`std` [Cargo feature]这可以选择性地启用`std`支持，当该功能关闭时，该库可以在`no_std`环境

## Tooling and environment compatibility

<a id="env-new-rust"></a>

### Possibly-breaking: changing the minimum version of Rust required

在新版本的Rust中引入新功能可以打破使用旧版本Rust的项目。这还包括在新发布的Cargo中使用新功能，以及要求在之前在stable上工作的板条箱中只使用夜间功能。

出于各种原因，一些项目选择在小版本中允许这种情况。更新到更新版本的Rust通常相对容易。Rust还有一个快速的6周发布周期，一些项目将在一个发布窗口内提供兼容性（例如当前的稳定版本加上N个以前的版本）。请记住，一些大型项目可能无法快速更新其生锈工具链。

缓解策略：

-   使用[Cargo features]让新功能选择加入。
-   为旧版本提供一个大的支持窗口。
-   如果可能，请复制新标准库项的源，以便可以继续使用旧版本，但利用新功能。
-   提供一个较旧的次要版本的单独分支，可以接收重要错误修复的后台端口。
-   留心这件事[`[cfg(version(..))]`][cfg-version]和[`#[cfg(accessible(..))]`][cfg-accessible]为新功能提供选择加入机制的功能。这些频道目前不稳定，只能在夜间频道使用。

<a id="env-change-requirements"></a>

### Possibly-breaking: changing the platform and environment requirements

对于一个库所运行的环境，有各种各样的假设，例如主机平台、操作系统版本、可用服务、文件系统支持等。如果您发布一个新版本，限制以前支持的内容，那么这可能是一个突破性的改变，例如，需要更新版本的操作系统。这些更改可能很难跟踪，因为您可能并不总是知道更改是否在未自动测试的环境中中断。

一些项目可能认为这种破坏是可以接受的，尤其是如果大多数用户不太可能破坏，或者项目没有支持所有环境的资源。另一个值得注意的情况是，当供应商停止支持某些硬件或操作系统时，项目可能认为停止支持也是合理的。

缓解策略：

-   记录您特别支持的平台和环境。
-   在CI中的各种环境中测试代码。

### Cargo

<a id="cargo-feature-add"></a>

#### Minor: adding a new Cargo feature

添加新的内容通常是安全的[Cargo features]。如果该功能引入了新的更改，导致出现突破性的更改，这可能会给具有更严格向后兼容性需求的项目带来困难。在这种情况下，避免将该功能添加到“默认”列表中，并可能记录启用该功能的后果。

```toml
# MINOR CHANGE

###########################################################
# Before
[features]
# ..empty

###########################################################
# After
[features]
std = []
```

<a id="cargo-feature-remove"></a>

#### Major: removing a Cargo feature

这通常是一个突破性的改变删除[Cargo features]。这将导致启用该功能的任何项目出错。

```toml
# MAJOR CHANGE

###########################################################
# Before
[features]
logging = []

###########################################################
# After
[dependencies]
# ..logging removed
```

缓解策略：

-   清楚地记录你的特征。如果存在内部或实验性功能，请将其标记为该功能，以便用户了解该功能的状态。
-   保留旧功能`Cargo.toml`，但在其他方面删除其功能。记录该功能已被弃用，并在未来的主要SemVer版本中删除它。

<a id="cargo-feature-remove-another"></a>

#### Major: removing a feature from a feature list if that changes functionality or public items

如果从另一个功能中删除一个功能，如果现有用户希望通过该功能可以使用该功能，那么这可能会中断现有用户。

```toml
# Breaking change example

###########################################################
# Before
[features]
default = ["std"]
std = []

###########################################################
# After
[features]
default = []  # This may cause packages to fail if they are expecting std to be enabled.
std = []
```

<a id="cargo-remove-opt-dep"></a>

#### Possibly-breaking: removing an optional dependency

删除可选依赖项可能会中断使用库的项目，因为另一个项目可能通过[Cargo features].

```toml
# Breaking change example

###########################################################
# Before
[dependencies]
curl = { version = "0.4.31", optional = true }

###########################################################
# After
[dependencies]
# ..curl removed
```

缓解策略：

-   清楚地记录你的特征。如果可选的依赖项不包含在文档的特征列表中，那么您可能会认为更改无文档条目是安全的。
-   保留可选依赖项，只是不要在库中使用它。
-   将可选依赖项替换为[Cargo feature]这不起任何作用，并记录它已被弃用。
-   使用支持可选依赖项的高级功能，并将其记录为支持扩展功能的首选方式。例如，如果您的库对“networking”之类的东西有可选支持，请创建一个通用功能名“networking”，以启用实现“networking”所需的可选依赖项。然后记录“联网”功能。

<a id="cargo-change-dep-feature"></a>

#### Minor: changing dependency features

更改依赖项上的功能通常是安全的，只要该功能不引入破坏性的更改。

```toml
# MINOR CHANGE

###########################################################
# Before
[dependencies]
rand = { version = "0.7.3", features = ["small_rng"] }


###########################################################
# After
[dependencies]
rand = "0.7.3"
```

<a id="cargo-dep-add"></a>

#### Minor: adding dependencies

添加新的依赖项通常是安全的，只要新的依赖项不会引入导致破坏性更改的新需求。例如，添加一个新的依赖项是一个重大的改变，该依赖项需要在以前在stable上工作的项目中每晚工作一次。

```toml
# MINOR CHANGE

###########################################################
# Before
[dependencies]
# ..empty

###########################################################
# After
[dependencies]
log = "0.4.11"
```

## Application compatibility

Cargo项目还可能包括具有自己接口（例如CLI接口、操作系统级交互等）的可执行二进制文件。因为它们是Cargo包装的一部分，所以它们通常使用和共享与包装相同的版本。在对应用程序进行更改时，您需要决定是否以及如何与用户签订SemVer合同。一个应用程序的潜在破坏性和兼容性更改太多，无法列出，因此我们鼓励您使用[SemVer]规范来指导您决定如何对应用程序应用版本控制，或者至少记录您的承诺。

[`cfg` attribute]: ../../reference/conditional-compilation.md#the-cfg-attribute

[`no_std`]: ../../reference/names/preludes.html#the-no_std-attribute

[`pub use`]: ../../reference/items/use-declarations.html

[cargo feature]: features.md

[cargo features]: features.md

[cfg-accessible]: https://github.com/rust-lang/rust/issues/64797

[cfg-version]: https://github.com/rust-lang/rust/issues/64796

[conditional compilation]: ../../reference/conditional-compilation.md

[default]: ../../std/default/trait.Default.html

[deprecated]: ../../reference/attributes/diagnostics.html#the-deprecated-attribute

[disambiguation syntax]: ../../reference/expressions/call-expr.html#disambiguating-function-calls

[inherent implementations]: ../../reference/items/implementations.html#inherent-implementations

[items]: ../../reference/items.html

[non_exhaustive]: ../../reference/attributes/type_system.html#the-non_exhaustive-attribute

[object safe]: ../../reference/items/traits.html#object-safety

[rust-feature]: https://doc.rust-lang.org/nightly/unstable-book/

[sealed trait]: https://rust-lang.github.io/api-guidelines/future-proofing.html#sealed-traits-protect-against-downstream-implementations-c-sealed

[semver]: https://semver.org/

[struct literal]: ../../reference/expressions/struct-expr.html

[wildcard patterns]: ../../reference/patterns.html#wildcard-pattern
