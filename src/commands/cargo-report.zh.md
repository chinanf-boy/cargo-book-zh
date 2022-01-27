# cargo-report(1)

## NAME

Cargo报告-生成和显示各种报告

## SYNOPSIS

`cargo report` *类型* [_options_]

### DESCRIPTION

显示给定项目的报告*类型*-目前只有`future-incompat`支持

## OPTIONS

<dl>

<dt class="option-term" id="option-cargo-report---id"><a class="option-anchor" href="#option-cargo-report---id"></a><code>--id</code> <em>id</em></dt>
<dd class="option-desc">Show the report with the specified Cargo-generated id</dd>

<dt class="option-term" id="option-cargo-report--p"><a class="option-anchor" href="#option-cargo-report--p"></a><code>-p</code> <em>spec</em>...</dt>
<dt class="option-term" id="option-cargo-report---package"><a class="option-anchor" href="#option-cargo-report---package"></a><code>--package</code> <em>spec</em>...</dt>
<dd class="option-desc">Only display a report for the specified package</dd>

</dl>

## EXAMPLES

1.  显示最新的未来不匹配报告：

    Cargo报告未来不匹配

2.  显示特定软件包的最新未来不兼容报告：

    Cargo报告未来不匹配——包装我的部门：0.0.1

## SEE ALSO

[Future incompat report](../reference/future-incompat-report.html)

[cargo(1)](cargo.zh.md)
