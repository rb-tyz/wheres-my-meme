# 批量选中（1.1.0）

Android 和 Windows 都可从搜索结果中选择多张图片，显示已选数量，并全选、清空或退出选择。全选覆盖本次搜索的所有结果；Windows 翻页保留选择。重新搜索会清空选择，非法正则和空查询也会清空旧结果。

Android 点击 **批量选择** 或长按结果进入选择模式，点击图片选中或取消，再点击 **分享所选**。单张仍使用系统单图片分享，多张使用系统多图片分享；接收应用得到每个原图 URI 的临时读取权限。屏幕旋转和返回应用后重新检查媒体版本，保留仍有效的选择。进程重新启动时清空选择。

Windows 点击 **批量选择** 后，点击结果选中或取消；**复制所选到剪贴板** 按搜索结果顺序提供多张独立图片。单张提供图片像素及 PNG；多张提供内嵌每张 PNG 的富文本和原图文件列表。复制同时提供每张一行的绝对路径，供纯文本输入框使用。接收软件选择它支持的格式。图片数据使用动图首帧，原文件列表保留 GIF 等文件的动画。预览的单张图片复制使用相同格式，另有仅复制路径的按钮。

应用在后台检查所选图片是否仍可读、版本是否匹配。一项失效时，本次分享或复制失败；Windows 保留旧剪贴板，不复制剩余部分。选择逻辑只保存版本标识；复制时逐张解码，准备完全部内容后发布。富文本和路径内容限额为 64 MiB，超出时提示减少选择；单张图片仍受已有 3200 万像素解码上限约束。系统或接收软件可能限制一次接收的图片数，可以减少选择后再次发送。

## 自动验证

- Android 纯逻辑测试：80 项通过，包括新增的空选择、去重、按结果排序、版本变化、清空重选、9000 条结果和随机选择变化测试。
- 发布版 APK 构建、v2/v3 签名与合并权限检查通过；版本 `1.1.0`，versionCode `2`，与 1.0.0 使用相同签名证书。Android lint 无错误，11 项提示涉及固定依赖版本和界面文字国际化。
- 桌面逻辑与 Qt 界面测试：Linux 隔离 Python 3.12 环境，146 项通过。core/storage/images/selection/clipboard 总语句覆盖率 98%，selection 与 clipboard 模块 100%。
- 桌面界面测试使用真实 Qt 点击及 Ctrl+V，覆盖独立图片粘贴、换行路径回退、PNG 透明度、图片顺序、跨页选择、全选全部结果、退出后预览、原图哈希不变、文件列表、删除/变化/权限/解码/编码失败、大小限额、重复点击、旧回调和关闭窗口。
- Android 多图片 intent 仪器测试覆盖所有 URI 的 ClipData、单张与多张 action、Parcel 往返和仅临时读权限。测试 APK 已编译；本次未连接设备执行这些测试。
- Linux offscreen 离线自验：27 项检查通过，退出码为 0；包含真实中文 OCR、预览、显式 PNG、单图和多图 Ctrl+V、独立图片资源、纯文本路径回退、文件列表、清空选择和原图哈希检查。自验结束后清除临时图片的剪贴板数据。

可重复运行：

```bash
./gradlew :core:test :app:lintRelease :app:assembleRelease :app:assembleDebugAndroidTest

PYTHONDONTWRITEBYTECODE=1 PYTHONPATH=desktop QT_QPA_PLATFORM=offscreen \
  python -m pytest desktop/tests -p no:cacheprovider \
  --cov=memeocr.core --cov=memeocr.storage --cov=memeocr.images --cov=memeocr.selection \
  --cov=memeocr.clipboard

PYTHONDONTWRITEBYTECODE=1 PYTHONPATH=desktop QT_QPA_PLATFORM=offscreen \
  python -m memeocr --self-test /tmp/memeocr-verification/source-selftest.json
```

Windows 原生构建使用 `scripts/build-windows.ps1`。冻结程序自验读取注册的 PNG 和 HTML Format 原生数据，检查图片可解码及 HTML 中每张图片独立存在；另通过 `CF_HDROP` 和 `DragQueryFileW` 读取实际文件数量与路径。Linux offscreen 自验验证 Qt 行为。

初版记录：2026-10-03，[Windows 构建 37034868142](https://github.com/rb-tyz/wheres-my-meme/actions/runs/37034868142) 在 `windows-2022` 验证提交 `eebe682243d3a192c3020dc355dcc7ec5fb7e22c`：114 项测试通过，4 项 POSIX 专用权限测试跳过，覆盖率 96%，selection 模块 100%。打包后的 `1.1.0` EXE 使用 Windows 平台插件，26 项自验全部通过，包括单图片数据、路径文本、多图文件列表、原生 `CF_HDROP` 文件数量和路径、关闭线程和原图哈希不变。该版本随后根据用户的粘贴反馈修正。

初版 `WhereIsMyMeme-1.1.0-windows-x64.exe`：144,665,588 字节，SHA-256 `33bde76be619940d1aaf4815df373b13842adce9019de73d863f12545398ea62`。GitHub ZIP 摘要 `7fd317458cc15e297a60579db3db756fdcaf3ccec724826965101474e17e26ff`、ZIP CRC、包内 EXE 校验和和 PE x64 格式曾核对通过。

图片粘贴修正的 [Windows 构建 37045918302](https://github.com/rb-tyz/wheres-my-meme/actions/runs/37045918302) 验证提交 `bab64d7`：142 项测试通过、4 项平台相关测试跳过，clipboard 模块覆盖率 100%；冻结 EXE 的 38 项自验通过。该包的 SHA-256 为 `36ae997e44a857ee938ecd27017cf00c8e53e8974ca5f44e2dc331ab395770eb`。

随后在桌面共享显示会话 `:0` 进行 Wine 跨程序测试：PNG 已传到桌面，但接收端未得到标准 `text/html`，只看到 Windows 的 HTML Format，因此富文本粘贴退回路径。提交 `62e3c57` 补充 UTF-8 的注册 `text/html` 格式；46 项相关测试通过。

最新交付包来自 [Windows 构建 37048994666](https://github.com/rb-tyz/wheres-my-meme/actions/runs/37048994666)，源码为 `62e3c57cbe774126c6a95bcda129eef91c8fd01e`：142 项测试通过、4 项平台相关测试跳过，clipboard 模块覆盖率 100%，冻结 EXE 的 38 项原生自验通过。下载后已核对 GitHub ZIP 摘要、ZIP CRC、包内 EXE 校验和及 PE x64 格式。交付文件为 `artifacts/WhereIsMyMeme-1.1.0-windows-x64.exe`，144,927,694 字节，SHA-256 `47e22b825ce04c41ed1796bf518537293bc1a9b622deb26ee6c75624117764c0`。

同一交付 EXE 在隔离 Wine 11.0 环境及桌面共享显示 `:0` 复验，40 项自验通过，退出码为 0。独立 Linux Qt 接收程序实际执行 Ctrl+V：单张可解码为 900×280 的图片，多张得到两张独立的 900×280 图片，纯文本框按行得到完整路径；接收端可读取标准 `text/html`。原图哈希不变，测试窗口已关闭。

新增纯选择逻辑的测试行数超过实现行数。原生 Android 控件与生命周期使用编译检查和真机验收，Qt 控件与 Windows 剪贴板使用上述界面测试和冻结程序自验。

## 真实环境验收

2026-10-03，用户反馈 Android 1.1.0 已通过手机验收。用户随后反馈 Windows 的“复制到剪贴板”功能已在微信聊天框、Microsoft Office Word、Microsoft Edge 地址栏测试成功，并要求提交 PR。本次未新增 Android 模拟器验证。其他使用检查可参考以下过程：

1. 搜索后选择 2–10 张图片，确认已选数量和逐张取消。Android 打开系统分享菜单，Windows 在常用聊天软件中粘贴。
2. Windows 切换结果页再返回，确认选择保留；在不同页选图，再使用全选和清空。
3. Android 旋转屏幕、取消分享或从接收应用返回，确认查询、选择模式和仍有效的选择保留。编辑搜索词但尚未点击查找时，恢复后的结果仍对应上一次查询。
4. 选择图片后重新搜索、输入非法正则，确认旧选择清空。选择后移动或修改一张测试图片，确认本次操作提示失效。
5. 检查单张预览及原有分享、复制图片、复制路径仍可使用；确认缓存升级后仍可继续搜索。

功能已通过 PR #2 合并，并发布 [v1.1.0 release](https://github.com/rb-tyz/wheres-my-meme/releases/tag/v1.1.0)。tag 指向合并提交 `e5b11ce`，附件提供 Android APK、Windows EXE 和两个安装包的 SHA-256 校验和。

## 执行记录

只读平台调查：`ses_f02b2cd0cffem9upyHbfmW5I4r`。环境准备：`ses_f02ad4e67ffeXD2RaodoluMVU5`，提供独立临时 JDK 17、SDK 35、build-tools 34.0.0 和已校验的 Gradle 8.9。初版实现会话 `ses_f02ad4e6effe9xQlRbXUjPL3Mw` 的选择模型由主代理重写；停止已获服务端确认，主代理随后接管实现和验证。

源码复审会话 `ses_f0296640bffema0Tlni2Ir1kmF` 只读检查两端选择、生命周期、批量操作和测试，未发现可证实的代码问题。复审未运行测试；Android 界面恢复和接收应用兼容性仍由设备验收。

CI 获取会话 `ses_f0285a213ffeGIRY4VHj4IsiXd` 核对构建状态，但未在时限内完成大文件下载；服务端确认已停止。主代理另行下载原生验证记录，检查截图，并使用分段下载获取 EXE。
