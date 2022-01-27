# Git Authentication

Cargo在使用git依赖项和注册表时支持某些形式的身份验证。本附录包含一些信息，用于以与Cargo兼容的方式设置git身份验证。

如果需要其他身份验证方法，则[`net.git-fetch-with-cli`]配置值可以设置为使Cargo执行`git`用于处理获取远程存储库的可执行文件，而不是使用内置支持。这可以通过`CARGO_NET_GIT_FETCH_WITH_CLI=true`环境变量。

## HTTPS authentication

HTTPS身份验证需要[`credential.helper`]机械装置有多个凭证帮助程序，您可以指定要在全局git配置文件中使用的凭证帮助程序。

```ini
# ~/.gitconfig

[credential]
helper = store
```

Cargo不要求输入密码，因此对于大多数助手，在运行Cargo之前，您需要为助手提供初始用户名/密码。一种方法是跑步`git clone`输入用户名/密码。

> **提示：**<br>MACOS用户可能想考虑使用OXKEY链助手。<br>Windows用户可能想考虑使用[GCM]帮手
>
> **注：**Windows用户需要确保`sh`shell在您的`PATH`。这通常在Git for Windows安装中提供。

## SSH authentication

SSH身份验证需要`ssh-agent`正在运行以获取SSH密钥。确保设置了适当的环境变量(`SSH_AUTH_SOCK`在大多数类似Unix的系统上），并且添加了正确的密钥（使用`ssh-add`).

Windows可以使用Pageant（部分功能）[PuTTY])或者`ssh-agent`.使用`ssh-agent`，Cargo需要使用作为Windows一部分分发的OpenSSH，因为Cargo不支持MinGW或Cygwin使用的模拟Unix域套接字。有关Windows安装的更多信息，请访问[Microsoft
installation documentation]这一页[key management]有关于如何开始的说明`ssh-agent`以及添加密钥。

> **注：**Cargo不支持git的简写SSH URL，比如`git@example.com:user/repo.git`.使用完整的SSH URL，如`ssh://git@example.com/user/repo.git`.
>
> **注：**SSH配置文件（如OpenSSH）`~/.ssh/config`)Cargo的内置SSH库不使用。应该使用更高级的需求[`net.git-fetch-with-cli`].

[`credential.helper`]: https://git-scm.com/book/en/v2/Git-Tools-Credential-Storage

[`net.git-fetch-with-cli`]: ../reference/config.md#netgit-fetch-with-cli

[gcm]: https://github.com/microsoft/Git-Credential-Manager-Core/

[putty]: https://www.chiark.greenend.org.uk/~sgtatham/putty/

[microsoft installation documentation]: https://docs.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse

[key management]: https://docs.microsoft.com/en-us/windows-server/administration/openssh/openssh_keymanagement
