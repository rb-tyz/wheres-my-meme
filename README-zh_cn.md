# Where's My Meme

一款纯本地的 Android 和 Windows meme 搜索工具。读取本地相册或文件夹，识别图片中的文字并保存结果，之后输入文字就能查找对应的 meme，无需部署服务器。

[Windows EXE](https://github.com/rb-tyz/wheres-my-meme/releases/download/v1.1.0/WhereIsMyMeme-1.1.0-windows-x64.exe) · [Android APK](https://github.com/rb-tyz/wheres-my-meme/releases/download/v1.1.0/MemeOCR-1.1.0.apk) · [Windows 使用说明](docs/windows.md) · [English](README.md)

- **纯本地**：两个平台都随程序提供中文 OCR 模型，识别、缓存和搜索无需联网，不上传图片。
- **分批处理**：复用已保存的识别结果，限制图片解码和缩略图缓存的内存占用。
- **批量选中**：选择多张搜索结果，Android 使用系统分享，Windows 将多张独立图片复制到剪贴板；Windows 翻页后保留选择。
- **无需服务器**：手机安装 APK；Windows 打开便携 EXE 即可使用。

## 项目背景

**谁会不喜欢meme呢？**

作为一个收藏了9000张（目前）meme的老吃家，我总是在聊天的时候偶然想到一张非常符合氛围、可以直接杀死比赛的meme，但是苦于没有好的检索手段，总是只能作罢。
此时Astra大人神兵天降，帮 ~~（替）~~ 我写 ~~（vibe）~~ 出了这款工具。

## 当前版本

当前版本为 1.1.0，新增两端的批量选中功能。上方提供已发布的 Android APK 和 Windows EXE。功能和测试范围见 [批量选中说明](docs/bulk-selection.md)。

Windows 1.1.0 是适用于 Windows 10/11 x64 的便携 EXE。发布 release 或移植已有功能不提升应用版本号。程序内置 Python、Qt、中文 OCR 模型和运行库，无需安装 Python，也不需要首次下载模型。启动时会将运行文件解压到系统临时目录。

Android 1.1.0，已签名的非调试版 APK，约 44.2 MiB。支持 Android 8.0 / API 26 及以上版本。

Android 已在没有 Google Play 服务、关闭网络的 Android 11 AOSP 模拟器及华为 Mate 60 Pro 手机中运行验证。Windows 构建和测试情况见 [Windows 测试记录](docs/windows.md)。

## Windows 使用方法

1. 从 Releases 下载 [WhereIsMyMeme-1.1.0-windows-x64.exe](https://github.com/rb-tyz/wheres-my-meme/releases/download/v1.1.0/WhereIsMyMeme-1.1.0-windows-x64.exe)，双击打开，无需安装或管理员权限。
2. 在 **识别相册** 页点击 **选择文件夹**；需要识别下级目录时，勾选 **包含子文件夹**。
3. 设置批量数量，点击 **开始识别本批**。默认 1000 张，范围 1–10000。**停止本批** 会保存当前图片的结果后停止。再次开始会继续处理；失败图片通过 **重试本相册失败项** 重试。
4. 在 **搜索图片** 页输入文字，点击 **查找图片**。搜索覆盖所有已登记的文件夹，**正则模式** 沿用下文的 RE2 语法。
5. 点击结果预览。**复制到剪贴板** 提供图片像素和 PNG 数据，纯文本输入框会粘贴图片的绝对路径；**复制图片所在路径** 只复制路径。
6. 点击 **批量选择**，再点击需要的图片。**全选结果** 选择全部搜索结果，包含其他页；**清空选择** 取消所有选择。**复制所选到剪贴板** 同时提供多张独立图片的富文本、原图文件列表，以及每张一行的绝对路径。接收应用按支持的格式粘贴。**退出选择** 返回预览模式。重新搜索会清空选择。

原图只读，缓存保存在 `%LOCALAPPDATA%\WhereIsMyMeme\cache.sqlite3`。已成功处理的图片（包括没有文字的图片）会跳过，文件大小或修改时间变化后重新识别。退出时等待正在执行的任务结束，每张已完成的结果都已保存。

Windows 使用 RapidOCR 内置的中文 PP-OCRv4 模型，Android 使用 ML Kit，两者识别结果可能不同。动图识别及剪贴板图片数据使用首帧。复制路径可用于找到原始动图。

## 构建 Windows EXE

在 Windows 上准备 Python 3.12 x64 和 PowerShell，首次构建需联网获取固定版本的依赖。

```powershell
./scripts/build-windows.ps1 -OutputDir "$env:TEMP/wheres-my-meme-windows"
```

脚本拒绝将产物写入源码目录。虚拟环境、缓存、中间文件、EXE、校验和及测试报告统一保存在 `OutputDir`。构建流程会运行逻辑和界面测试，再对打包后的 EXE 执行中文 OCR 和 Windows 系统剪贴板自验。GitHub Actions 的 `windows-2022` 构建机使用同一脚本。

桌面源码位于 `desktop/`；测试范围见 [docs/windows.md](docs/windows.md)。程序内也包含 [第三方许可说明](desktop/THIRD_PARTY_NOTICES.txt) 及依赖的原始许可文件。

## Android 安装与使用

1. 将 [MemeOCR-1.1.0.apk](https://github.com/rb-tyz/wheres-my-meme/releases/download/v1.1.0/MemeOCR-1.1.0.apk) 传到手机，使用系统安装器打开。如果系统询问，允许用于打开文件的应用安装此 APK。
2. 打开 **Meme 文字搜索**，授予照片读取权限。应用只显示获准访问的图片。通知权限用于在通知栏展示识别进度。
3. 选择本地相册。第一次建议先处理 100 张，看看自己图库里的识别效果。默认每批 1000 张，可输入 1–10000。
4. 点击 **开始识别本批**。每张识别完成后先保存，再更新进度。点击 **停止本批** 后，会处理并保存当前图片，然后停止。
5. 再次开始时自动跳过已完成且未改变的图片；没有文字也算完成。失败的图片通过 **重试本相册失败项** 单独重试。
6. 切换到 **搜索图片**，输入文字并点击 **查找图片**。点击结果可预览原图，再打开系统分享菜单。
7. 点击 **批量选择**，或长按一张结果，进入选择模式。点击图片可以选中或取消，**全选结果** 和 **清空选择** 操作全部结果；**分享所选** 将所有选中的原图交给系统分享菜单。重新搜索会清空选择。旋转屏幕或从分享应用返回后，会保留仍可访问、版本未变化的选择；重新启动进程会清空选择。

搜索范围为所有已经识别、当前仍获准访问的相册。图片删除、内容发生可检测的变化或访问权限丢失后，旧记录不会继续出现在搜索结果里。

<table align="center">
  <tr>
    <th align="center">文字搜索：猫猫</th>
    <th align="center">正则搜索：狗|猫</th>
  </tr>
  <tr>
    <td align="center" width="50%"><a href="pics/image.png"><img src="pics/image.png" alt="搜索“猫猫”，显示匹配的表情包" width="280"></a></td>
    <td align="center" width="50%"><a href="pics/image-1.png"><img src="pics/image-1.png" alt="使用正则“狗|猫”，查找包含任一文字的表情包" width="280"></a></td>
  </tr>
</table>

## 文字与正则搜索

普通搜索按文字包含关系匹配，忽略英文字母大小写、全角 ASCII 差异，并合并连续空白。搜索不会纠正 OCR 错字，也不能按画面含义查找无文字图片。

勾选 **正则模式** 后，对识别原文匹配，区分大小写。例如：

| 表达式 | 匹配内容 |
| --- | --- |
| 摸鱼\|放假 | 包含任意一个词 |
| [0-9]{4} | 连续四位数字 |
| (?s)晚安.*明天 | 先出现“晚安”，后出现“明天”，中间可以换行 |

正则使用 RE2/J，避免复杂表达式反复回溯导致长时间卡住。不支持前后查找和反向引用；不支持的语法会显示错误提示。

## 图片、缓存与隐私

- 原图只读，不改名、不移动、不写入，也不上传。
- 发布 APK 没有联网权限、外部存储写入权限或“管理所有文件”权限。中文模型已打包，不需要首次使用时联网下载。
- 识别文字、图片元数据和失败原因保存在应用私有 SQLite 数据库中，关闭了应用数据自动备份。卸载或清除应用数据会删除识别缓存。
- 用媒体 URI、文件大小和修改时间判断是否为已处理版本。检测到版本变化会重新识别；移动或复制图片后可能再次处理。首版不做内容哈希去重。
- 动图只识别首帧，预览也是静态图；分享时发送原文件。
- 模糊字、小字和艺术字可能误识别。测试样本中出现过“今天也要开心”被识别为“今天也妻开心”，因此不能保证长句逐字匹配。可以尝试较短的词语，但未识别出的字仍无法检索。
- 前台服务显示进度，但系统仍可能结束任务。已保存结果保留，重新打开后开始下一批即可；不承诺强制结束或重启手机后自动继续。
- 尚未测量 9000 张真实 meme 的全量识别耗时与准确率。自动测试中的 9000 条数据用于检验批次和搜索行为，不代表手机 OCR 的速度。

## 从源码构建

需要 JDK 17、Android SDK platform 35、build-tools 34.0.0。第一次下载依赖需要网络。项目固定使用 Gradle 8.9、AGP 8.7.3、Kotlin 2.0.21，Gradle wrapper 包含下载校验值。

    export JAVA_HOME=/path/to/jdk17
    export ANDROID_HOME=/path/to/android-sdk
    ./scripts/build-apk.sh

脚本运行单元测试和发布版 lint，构建并签名 APK，检查签名和权限，最后生成 artifacts/SHA256SUMS。

首次运行会在 .signing/ 生成签名密钥。以后发布更新需要保留同一密钥，请妥善备份该目录，不要随 APK 分发，也不要提交到版本库。丢失密钥后，新签名的 APK 无法覆盖安装旧版本。

可通过 GRADLE_BIN 指定已有 Gradle，通过 GRADLE_USER_HOME 指定隔离缓存，无须更改全局 Java 或系统配置。

## 测试与目录

    ./gradlew :core:test :app:lintDebug :app:assembleDebug
    ./gradlew :app:connectedDebugAndroidTest

设备测试应使用模拟器或专用测试设备，输入为自制图片和临时数据库记录。中文 OCR 集成测试验证离线模型可调用、可识别指定词语，不要求每个字符完全正确；已知误差保存在测试记录中。

- core/：批次选择、逐张处理循环、状态、文字标准化与搜索。
- app/：Android 原生界面、只读相册访问、有界图片解码、SQLite 和前台识别服务。
- app/src/androidTest/：在 Android 上运行的解码、数据库和 OCR 测试。
- scripts/build-apk.sh：构建、签名与安装包检查。
- docs/verification.md：实际结果、截图、识别误差和真机待验项目。

实现资料：[ML Kit 中文识别](https://developers.google.com/ml-kit/vision/text-recognition/v2/android)、[Android MediaStore](https://developer.android.com/training/data-storage/shared/media)、[RE2/J](https://github.com/google/re2j)。

## 许可证

本项目代码采用 [MIT 许可证](LICENSE)。第三方依赖和截图中的表情包仍受各自的许可证及版权约束。
