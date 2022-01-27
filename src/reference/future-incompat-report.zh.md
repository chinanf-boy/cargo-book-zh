### Future incompat report

Cargo检查所有依赖项中的未来不兼容警告。这些警告针对可能在将来成为硬错误的更改，导致依赖关系在未来版本的rustc中停止构建。如果发现任何警告，将显示一个小通知，指示已发现警告，并提供有关如何显示完整报告的说明。

可以使用`cargo report future-incompatibilities
--id ID`命令，或者使用`--future-incompat-report`选项然后，开发人员应该将他们的依赖项更新到问题得到修复的版本，或者与依赖项的开发人员一起帮助解决问题。

此功能可以通过`[future-incompat-report]`部分`.cargo/config`。目前，受支持的选项包括：

```
[future-incompat-report]
frequency = FREQUENCY
```

的支持值`FREQUENCY`你总是`and 'never', which control
whether or not a message is printed out at the end of`Cargo建造`/`Cargo检查。
