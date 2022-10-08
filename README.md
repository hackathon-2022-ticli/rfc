# RFC: tcli in rust

By [TBD] 团队

| [![wfxr](https://avatars1.githubusercontent.com/u/6105425?s=72)](https://github.com/wfxr) | [![Xuyuanp](https://avatars.githubusercontent.com/u/2245664?s=72)](https://github.com/xuyuanp) | [![Bowser1704](https://avatars.githubusercontent.com/u/43539191?s=72)](https://github.com/Bowser1704) |
| :---:                                                                                     | :---:                                                                                          | :---:                                                                                                 |
| [wfxr](https://github.com/wfxr)                                                           | [Xuyuanp](https://github.com/xuyuanp)                                                          | [Bowser1704](https://github.com/Bowser1704)                                                           |

## 项目介绍

用 rust 为 TiKV 实现一个 modern cli。

## 背景&动机

CLI 对数据库管理者和开发者来说都是非常重要的一个工具，它可以方便用户编写脚本和排查问题，也能帮助用户快速上手试用产品。
我们熟知的 MySQL, PostgreSQL, Redis 等数据库都有很好用的 CLI，
但目前 TiKV 官方并没有类似定位的工具，READEME 中只有 SDK 代码调用的示例，
所以我们觉得为 TiKV 实现一个好用的 CLI 应该是一个很有意义的事情。

有了这个想法之后，我们在 Github 找到了 2 个类似的项目：
[shafreeck/tikv-cli](https://github.com/shafreeck/tikv-cli) 和 [c4pt0r/tcli](https://github.com/c4pt0r/tcli)。
前面一个功能比较简陋，已经不怎么维护了；后面一个是 [dongxu](https://github.com/c4pt0r/tcli) 用 go 实现的，功能比较完备，
也在持续维护当中，不过使用体验上和 [mycli](https://github.com/dbcli/mycli) 等优秀的 cli 还有一定的差距。
所以我们计划以 tcli 为蓝本，借助 rust 繁荣的 cli 生态，打造一款更好用的 TiKV cli，朝着 `ultimate CLI tool for TiKV` 
的目标更进一步。

## 项目设计

WIP

### 改进方向

- 完善的 CI/CD
- 多平台 Release（Debian / Archlinux / Homebrew / Windows）
- Self upgrade ?
- cli 命令补全
- 历史命令补全
- 改进输出样式（可配置）
- 支持更多的命令？
- JSON value 支持？
- CJK / emoji 支持
- RawKV 模式
- 管道友好
- Ascii Art？
- 关键字高亮？
- Watch 模式（能比直接使用 `watch` 命令更香吗？）
- 更丰富的元数据展示？
