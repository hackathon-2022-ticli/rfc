# RFC: TiKV CLI in Rust （WIP)

By *go.unwrap()* 团队

| [![wfxr](https://avatars1.githubusercontent.com/u/6105425?s=72)](https://github.com/wfxr) | [![Xuyuanp](https://avatars.githubusercontent.com/u/2245664?s=72)](https://github.com/xuyuanp) | [![Bowser1704](https://avatars.githubusercontent.com/u/43539191?s=72)](https://github.com/Bowser1704) |
| :---:                                                                                     | :---:                                                                                          | :---:                                                                                                 |
| [wfxr](https://github.com/wfxr)                                                           | [Xuyuanp](https://github.com/xuyuanp)                                                          | [Bowser1704](https://github.com/Bowser1704)                                                           |

## 项目介绍

用 rust 为 TiKV 实现一个 modern cli。

## 背景&动机

CLI 对数据库管理人员和开发者来说都是非常重要的工具，它可以方便用户编写脚本和排查问题，也能帮助用户快速上手试用产品。
我们熟知的 MySQL, PostgreSQL, Redis 等数据库都有很好用的 CLI，
但目前 TiKV 官方并没有类似定位的工具，READEME 中只有 SDK 代码调用的示例，
所以为 TiKV 实现一个好用的 CLI 应该是一个很有意义的事情。

Github 上目前有 2 个相关的项目：
[shafreeck/tikv-cli](https://github.com/shafreeck/tikv-cli) 和 [c4pt0r/tcli](https://github.com/c4pt0r/tcli)。
tikv-cli 功能比较简陋，已经不再维护；tcli 是 [dongxu](https://github.com/c4pt0r) 用 go 实现的，功能比较完备，
也在持续维护当中，不过使用体验上和 [mycli](https://github.com/dbcli/mycli) 等优秀的 cli 还有一定的差距。
考虑到 TiKV 本身的 rust 属性，以及 rust 在 CLI 开发中的绝佳体验，TiKV 的 CLI 没有理由不用 rust 来实现。
所以我们计划以 tcli 和 redis-cli 为蓝本，打造一款更好用的 TiKV CLI。

## 项目设计

为 TiKV 提供一个由 rust 编写的命令行工具 `ticli`。用户可以通过 `cargo` 来安装：

```sh
cargo install ticli
```

或者使用其他系统包管理器：

```sh
# Archlinux
pacman -S ticli

# Debian / Ubuntu
apt install ticli

# Rocky Linux / Fedora
dnf install ticli

# macOS
brew install ticli
```

### Command line

用户可以使用 `ticli` 执行基础的增删改查等命令：

- `ticli GET <key>`
- `ticli SET <key> <val>`
- `ticli DEL <key>`
- `ticli INCR <key>`
- `ticli DESC <key>`
- `ticli STRLEN <key>`
- `ticli SCAN <prefix>`
- `ticli COUNT <prefix>`
- `ticli LOAD <csv>`
- `ticli PING`
- ...

以 `SCAN` 命令为例，`ticli` 使用 ASCII 表格排版，可以正确对齐 CJK 字符和 emoji 字符，
并且能通过参数设置不同的表格样式：

![ticli-scan-1](./assets/ticli-scan-1.jpeg)

除了 ASCII 表格，`ticli` 也支持 `csv` 等其他格式的导出：

![ticli-scan-2](./assets/ticli-scan-2.jpeg)

命令执行结果通常以 human readable 的形式展示出来，不过当我们编写脚本的时候，只输出原始结果更方便解析，
所以 `ticli` 只会在检测到 `stdout` 是 `tty` 的时候进行格式化并展示命令耗时等额外的信息：

![ticli-incr-1](./assets/ticli-incr-1.jpeg)

`ticli` 也支持通过管道顺序执行预先定义好的命令序列：

![ticli-cmd-seq](./assets/ticli-cmd-seq.jpeg)

`ticli` 提供详细的 help 文档，完善的错误处理，
以及包括 `bash`, `zsh`, `fish`, `elvish` 和 `powershell` 在内的各种类型的 shell 补全。
得益于 rust 优秀的语言特性和丰富的 CLI 生态，很多功能都可以在编译阶段自动生成，不需要额外进行开发和维护，
而且和代码注释始终保持一致。

### Interactive shell

用命令方式运行 `ticli` 对脚本编写和测试来说非常有用，不过更多的时候我们会用到交互模式。
交互模式会启动一个用于 REPL 的 shell，同样可以执行上面介绍的所有命令：

![ticli-repl-1](./assets/ticli-repl-1.jpeg)

`ticli` 支持使用 TAB 键按命令名称前缀进行补全：

![ticli-repl-2](./assets/ticli-repl-2.jpeg)

在交互模式下，`ticli` 可以记录用户输入的命令，默认保存在 `HOME` 目录下的 `.ticli_history` 文件中，
用户可以通过 `TICLI_HISTFILE` 环境变量指定其他路径。
除了提供 bash 风格的 `Ctrl-R` 历史命令搜索，`ticli` 还会支持体验更好的 fish 风格补全：

![ticli-repl-3](./assets/ticli-repl-3.jpeg)

### Keybindings
Interactive 模式下，`ticli` 提供 Unix/Emacs 风格的键绑（以及 Vi 模式？）：

| Keystroke             | Action                                                                                           |
| --------------------- | ------------------------------------------------------------------------------------------------ |
| Home                  | Move cursor to the beginning of line                                                             |
| End                   | Move cursor to end of line                                                                       |
| Left                  | Move cursor one character left                                                                   |
| Right                 | Move cursor one character right                                                                  |
| Ctrl-A, Home          | Move cursor to the beginning of line                                                             |
| Ctrl-B, Left          | Move cursor one character left                                                                   |
| Ctrl-C                | Interrupt/Cancel edition                                                                         |
| Ctrl-D                | (if line _is_ empty) End of File                                                                 |
| Ctrl-D, Del           | (if line is _not_ empty) Delete character under cursor                                           |
| Ctrl-E, End           | Move cursor to end of line                                                                       |
| Ctrl-F, Right         | Move cursor one character right                                                                  |
| Ctrl-H, Backspace     | Delete character before cursor                                                                   |
| Ctrl-I, Tab           | Next completion                                                                                  |
| Ctrl-J, Ctrl-M, Enter | Finish the line entry                                                                            |
| Ctrl-K                | Delete from cursor to end of line                                                                |
| Ctrl-L                | Clear screen                                                                                     |
| Ctrl-N, Down          | Next match from history                                                                          |
| Ctrl-P, Up            | Previous match from history                                                                      |
| Ctrl-R                | Reverse Search history (Ctrl-S forward, Ctrl-G cancel)                                           |
| Ctrl-T                | Transpose previous character with current character                                              |
| Ctrl-U                | Delete from start of line to cursor                                                              |
| Ctrl-V                | Insert any special character without performing its associated action (#65)                      |
| Ctrl-W                | Delete word leading up to cursor (using white space as a word boundary)                          |
| Ctrl-X Ctrl-U         | Undo                                                                                             |
| Ctrl-Y                | Paste from Yank buffer                                                                           |
| Ctrl-Y                | Paste from Yank buffer (Meta-Y to paste next yank instead)                                       |
| Ctrl-Z                | Suspend (Unix only)                                                                              |
| Ctrl-\_               | Undo                                                                                             |
| Meta-<                | Move to first entry in history                                                                   |
| Meta->                | Move to last entry in history                                                                    |
| Meta-B, Alt-Left      | Move cursor to previous word                                                                     |
| Meta-C                | Capitalize the current word                                                                      |
| Meta-D                | Delete forwards one word                                                                         |
| Meta-F, Alt-Right     | Move cursor to next word                                                                         |
| Meta-L                | Lower-case the next word                                                                         |
| Meta-T                | Transpose words                                                                                  |
| Meta-U                | Upper-case the next word                                                                         |
| Meta-Y                | See Ctrl-Y                                                                                       |
| Meta-Backspace        | Kill from the start of the current word, or, if between words, to the start of the previous word |
| Meta-0, 1, ..., -     | Specify the digit to the argument. `–` starts a negative argument.                               |
