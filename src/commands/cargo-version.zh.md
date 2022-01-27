# cargo-version(1)

## NAME

cargo-version - 显示版本信息

## SYNOPSIS

`cargo version` [_options_]

## DESCRIPTION

显示 Cargo 的版本。

## OPTIONS

<dl>

<dt class="option-term" id="option-cargo-version--v"><a class="option-anchor" href="#option-cargo-version--v"></a><code>-v</code></dt>
<dt class="option-term" id="option-cargo-version---verbose"><a class="option-anchor" href="#option-cargo-version---verbose"></a><code>--verbose</code></dt>
<dd class="option-desc">显示额外的版本信息</dd>

</dl>

## EXAMPLES

1.  显示版本：

```
cargo version
```

2.  该版本也可通过以下标志获得：

```
cargo --version
cargo -V
```

3.  显示额外的版本信息：

```
cargo -Vv
```

## SEE ALSO

[cargo(1)](cargo.zh.md)
