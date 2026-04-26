# NoRecall for macOS WeChat

这是一个用于本地研究与验证的 macOS 微信防撤回补丁项目。补丁完成后，在受支持版本中，对方撤回消息时原消息会尽量保留在本地聊天窗口内，并显示一条更清晰的本地提示。

当前已验证微信 `4.2.0.54`（Apple Silicon）。

> 仅用于本地学习、逆向研究、调试验证。请先阅读：[DISCLAIMER.md](./DISCLAIMER.md)。
>
> 微信版本更新后行为可能变化，需要重新测试。


## 效果

- 对方撤回后，原消息尽量保留在本地记录中；
- 微信内仍显示系统提示行；
- 提示行会被替换为本地拦截提示；
- 提示紧跟被撤回的原消息，左侧摘要继续显示最后一条正常消息；
- 自己撤回自己的消息时正常删除原消息；
- 不额外弹外部通知；
- 默认关闭调试日志；
- 工具会尽量避免破坏小程序相关组件签名。

## 环境要求

- Apple Silicon macOS；
- 已安装官方微信；
- Xcode Command Line Tools：

  ```bash
  xcode-select --install
  ```

- 系统自带或可用的：

  ```text
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

推荐先复制一份官方微信到项目目录测试。验证副本前，请先退出其他位置正在运行的微信：

```bash
ditto --rsrc --extattr /Applications/WeChat.app ./WeChat.test.app
./NoRecall ./WeChat.test.app
open -na ./WeChat.test.app
```

### 允许系统窗口截图

如果系统“截取所选窗口”提示无法捕捉微信窗口，可选择启用：

```bash
./NoRecall --allow-window-capture ./WeChat.test.app
```

该选项默认关闭，只对指定副本生效，安装后需要重启。启用后，具有录屏权限的
其他应用也能捕获普通微信窗口。不带该选项重新运行可恢复原有窗口保护。

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
2. 让另一端依次发送“a”“b”；
3. 让另一端撤回“a”；
4. 确认聊天内显示“a → 提示 → b”，左侧摘要仍显示“b”；
5. 自己撤回一条消息，确认原消息正常删除。

## 日志

默认不产生日志。

## 恢复 / 卸载

恢复时请重新安装官方微信，或用未修改的官方微信副本替换目标应用。

## 注意事项

- 微信更新后需要重新 patch；
- 不要对整个应用包做递归重签，否则可能破坏小程序/扩展组件；
- 不保证适配所有微信版本；
- 请先在副本中验证，再决定是否 patch 系统应用。

## 免责声明

本项目仅用于本地学习、逆向研究、调试验证。运行工具会修改本地应用包内容和代码签名状态，可能导致功能异常、登录状态失效、数据异常、更新失败或第三方服务条款风险。

项目按“现状”提供，不承诺稳定性、兼容性或持续可用性。使用、修改、分发或部署本项目造成的任何后果由使用者自行承担。

完整免责声明见：[DISCLAIMER.md](./DISCLAIMER.md)。
