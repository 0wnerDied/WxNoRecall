# NoRecall for macOS WeChat

这是一个用于本地研究与验证的 macOS 微信防撤回补丁项目。补丁完成后，在受支持版本中，对方撤回消息时原消息会尽量保留在本地聊天窗口内，并显示一条更清晰的本地提示。

> 仅用于本地学习、逆向研究、调试验证。请先阅读：[DISCLAIMER.md](./DISCLAIMER.md)。
>
> 微信版本更新后行为可能变化，需要重新测试。


## 效果

- 对方撤回后，原消息尽量保留在本地记录中；
- 微信内仍显示系统提示行；
- 提示行会被替换为本地拦截提示；
- 自己撤回自己的消息时尽量保持微信原生行为；
- 不额外弹外部通知；
- 默认关闭调试日志；
- 工具会尽量避免破坏小程序相关组件签名。

## 环境要求

- macOS；
- 已安装官方微信；
- Xcode Command Line Tools：

  ```bash
  xcode-select --install
  ```

- 系统自带或可用的：

  ```text
  cc
  python3
  codesign
  otool
  install_name_tool
  ```

## 使用方法

### Patch 系统微信

```bash
./NoRecall
```

如果系统应用目录需要管理员权限，工具会自动通过 `sudo` 重新执行。

### Patch 指定副本

推荐先复制一份官方微信到项目目录测试：

```bash
ditto --rsrc --extattr /Applications/WeChat.app ./WeChat.test.app
./NoRecall ./WeChat.test.app
open -na ./WeChat.test.app
```

## 验证

### 1. 验证签名整体可用

```bash
codesign --verify --deep --strict --verbose=1 /Applications/WeChat.app
```

成功时一般会看到：

```text
valid on disk
satisfies its Designated Requirement
```

### 2. 验证小程序核心组件仍是官方签名

```bash
codesign -dvvv /Applications/WeChat.app/Contents/MacOS/WeChatAppEx.app 2>&1 | grep -E 'Authority=|TeamIdentifier='
```

应保留官方签名链，而不是被整体重签成 ad-hoc。

### 3. 验证功能

使用两个测试账号或一台移动端设备配合测试：

1. 启动 patch 后的微信；
2. 让另一端发送一条测试消息；
3. 让另一端撤回；
4. 观察本地聊天窗口中原消息和提示行是否符合预期。

## 日志

默认不产生日志。

## 恢复 / 卸载

最干净的恢复方式是重新安装官方微信。

工具会尽量保存必要的本地回滚材料，但 patch 后应用包和部分组件的签名状态已经变化。若要完全恢复官方签名状态，仍建议直接重装官方微信。

## 注意事项

- 微信更新后需要重新 patch；
- 不要对整个应用包做递归重签，否则可能破坏小程序/扩展组件；
- 不保证适配所有微信版本；
- 请先在副本中验证，再决定是否 patch 系统应用。

## 免责声明

本项目仅用于本地学习、逆向研究、调试验证。运行工具会修改本地应用包内容和代码签名状态，可能导致功能异常、登录状态失效、数据异常、更新失败或第三方服务条款风险。

项目按“现状”提供，不承诺稳定性、兼容性或持续可用性。使用、修改、分发或部署本项目造成的任何后果由使用者自行承担。

完整免责声明见：[DISCLAIMER.md](./DISCLAIMER.md)。
