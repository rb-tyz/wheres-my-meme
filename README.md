# Where's My Meme

A fully local app for finding saved memes on Android and Windows. It reads an album or folder, recognizes and saves the text in each image, and lets you search for memes by that text. No server is needed.

[简体中文](README-zh_cn.md) · [Windows EXE](https://github.com/rb-tyz/wheres-my-meme/releases/download/v1.1.0/WhereIsMyMeme-1.1.0-windows-x64.exe) · [Android APK](https://github.com/rb-tyz/wheres-my-meme/releases/download/v1.1.0/MemeOCR-1.1.0.apk) · [Windows instructions](docs/windows.md)

- **Fully local**: Both downloads include a Chinese OCR model. Recognition, caching and search work without an Internet connection or image uploads.
- **Batch processing**: Saved results are reused, with bounded image decoding and thumbnail caching.
- **Batch selection**: Select several search results, share them on Android or copy separate images to the clipboard on Windows. Selection persists across Windows result pages.
- **No server to set up**: Install the Android APK or open the portable Windows executable.

## Background

**Who doesn't love a good meme?**

I've collected about 9,000 memes so far. I'll be in a chat and think of the perfect one, the kind that would absolutely win the conversation, only to give up because I can't find it in my gallery.

Then Astra came to the rescue and helped me write this app. And by "helped," I mean did the coding while I supplied the vibes.

## Release and compatibility

Version 1.1.0 adds batch selection on Android and Windows. The links above point to the published packages. See [batch selection and verification](docs/bulk-selection.md).

Windows 1.1.0 is a portable executable for Windows 10/11 x64. A release or a port of existing features does not change the application version. It includes Python, Qt, the Chinese OCR models and the required runtime libraries. You do not need to install Python or download a model. Its first launch extracts runtime files into the system temporary directory.

Android 1.1.0 is a signed, non-debuggable APK, approximately 44.2 MiB. It supports Android 8.0 / API 26 and later.

The Android app has been tested on an Android 11 AOSP emulator without Google Play services and with networking disabled, and on a Huawei Mate 60 Pro. Windows build and verification details are in [docs/windows.md](docs/windows.md).

## Use on Windows

1. Download [WhereIsMyMeme-1.1.0-windows-x64.exe](https://github.com/rb-tyz/wheres-my-meme/releases/download/v1.1.0/WhereIsMyMeme-1.1.0-windows-x64.exe) from Releases and open it by double-clicking it. No installer or administrator access is needed.
2. On **识别相册**, click **选择文件夹**. Enable **包含子文件夹** if your memes are stored in nested folders.
3. Choose a batch size, then click **开始识别本批**. The default is 1,000 images; the range is 1–10,000. **停止本批** saves the current image before stopping. Start again to continue, or use **重试本相册失败项** to retry failures.
4. On **搜索图片**, enter text and click **查找图片**. Results span all registered folders. **正则模式** supports the same RE2 syntax described below.
5. Click a result to preview it. **复制到剪贴板** copies image pixels and PNG data for pasting into another app; a plain text field receives the absolute file path. **复制图片所在路径** copies only that path.
6. Click **批量选择**, then click the images you want. **全选结果** selects all search results across pages; **清空选择** clears them. **复制所选到剪贴板** provides separate images in rich text, the original files for apps that accept them, and one absolute path per line for plain text fields. Images remain separate. **退出选择** returns to preview mode. A new search clears the selection.

Original images are read-only. The cache is stored in `%LOCALAPPDATA%\WhereIsMyMeme\cache.sqlite3`. Completed images, including those with no text, are skipped until their size or modification time changes. Closing the window waits for active work to finish; each completed result is already saved.

Windows uses RapidOCR's bundled Chinese PP-OCRv4 models; Android uses ML Kit. Their recognition results may differ. Animated images use the first frame. Copying the path preserves access to the original animated file.

## Build the Windows executable

Use Python 3.12 x64 on Windows and PowerShell. The first build needs an Internet connection for the pinned dependencies.

```powershell
./scripts/build-windows.ps1 -OutputDir "$env:TEMP/wheres-my-meme-windows"
```

The script rejects output directories inside the checkout. Its virtual environment, caches, work files, executable, checksums and test reports are written under `OutputDir`. It runs the unit/UI tests, packages a single executable, then tests that executable with real Chinese OCR and the native Windows clipboard. GitHub Actions uses the same script on `windows-2022`.

The desktop source is in `desktop/`. [Windows verification](docs/windows.md) records the tested scope. [Third-party notices](desktop/THIRD_PARTY_NOTICES.txt) and original dependency licenses are also included in the executable.

## Use on Android

1. Copy [MemeOCR-1.1.0.apk](https://github.com/rb-tyz/wheres-my-meme/releases/download/v1.1.0/MemeOCR-1.1.0.apk) to the phone and open it with the system package installer. Allow installation from the file-opening app if the system asks.
2. Open **Meme 文字搜索** and grant photo access. The app only lists photos it is permitted to read. Notification permission allows progress to appear in the notification area.
3. Select a local album. Start with a small batch such as 100 images to evaluate your own pictures. The default batch size is 1,000; accepted values are 1–10,000.
4. Tap **开始识别本批**. Each result is saved before progress advances. **停止本批** lets the current image finish and saves it.
5. Start another batch to continue. Completed, unchanged images—including images with no text—are skipped. Use **重试本相册失败项** to retry failed images in the selected album.
6. Open **搜索图片**, enter text and tap **查找图片**. Tap a result to preview the original and open Android's share sheet.
7. Tap **批量选择** or long-press a result to start selecting images. Tap images to select or deselect them, use **全选结果** or **清空选择**, then tap **分享所选** to open the system share sheet with all selected originals. A new search clears the selection. Rotation and returning from the share sheet keep selections whose image versions are still accessible; restarting the process clears them.

Search spans all recognized albums that are still accessible. Images that were deleted, changed or are no longer authorized are excluded from search.

<table align="center">
  <tr>
    <th align="center">Text search: 猫猫 (cats)</th>
    <th align="center">Regex search: 狗|猫 (dog or cat)</th>
  </tr>
  <tr>
    <td align="center" width="50%"><a href="pics/image.png"><img src="pics/image.png" alt="Text search for 猫猫 (cats), showing matching memes" width="280"></a></td>
    <td align="center" width="50%"><a href="pics/image-1.png"><img src="pics/image-1.png" alt="Regex search for 狗|猫, showing memes containing either word" width="280"></a></td>
  </tr>
</table>

## Search modes

Ordinary search matches a substring after normalizing full-width ASCII, letter case and repeated whitespace. It cannot find text-free images by what they depict or correct OCR mistakes.

Regular-expression mode matches the original OCR text and is case-sensitive. Examples:

| Expression | Meaning |
| --- | --- |
| 摸鱼\|放假 | Either phrase |
| [0-9]{4} | Four consecutive digits |
| (?s)晚安.*明天 | 晚安 followed by 明天, allowing line breaks in between |

Regular expressions use RE2/J to avoid catastrophic backtracking. Lookaround and backreferences are unsupported and produce an error message.

## Storage, privacy and limitations

- Original images are opened read-only. The app does not rename, move, rewrite or upload them.
- The release APK does not request INTERNET, WRITE_EXTERNAL_STORAGE or MANAGE_EXTERNAL_STORAGE. OCR does not require a first-run model download.
- OCR results, errors and metadata stay in the app's private SQLite database. Automatic app-data backups are disabled. Uninstalling the app or clearing its data removes the recognition cache.
- Cache identity uses media URI, file size and modification time. A changed version is processed again. Moved/copied images can be processed again; content-hash deduplication is not included.
- Animated images are recognized from their first frame. Preview is static; sharing sends the original file.
- OCR can struggle with blurry text, small print and stylized lettering. In one test image, “今天也要开心” was read as “今天也妻开心”, so an exact search for a long sentence may miss a match. Try a shorter phrase, though words the OCR missed still won't be searchable.
- A foreground service reports progress, but the OS may stop it. Saved results survive process termination; reopen the app and start the next batch. Automatic restart after force-stop/reboot is not provided.
- Initial recognition time and accuracy for 9,000 real memes have not been measured. The 9,000-record automated checks cover scheduling and search, not an OCR throughput benchmark.

## Build

Use JDK 17, Android SDK platform 35, build-tools 34.0.0, and an Internet connection for the first dependency download. Gradle 8.9 and its checksum are pinned in the wrapper; AGP 8.7.3 and Kotlin 2.0.21 are pinned in the project.

    export JAVA_HOME=/path/to/jdk17
    export ANDROID_HOME=/path/to/android-sdk
    ./scripts/build-apk.sh

The script runs unit tests and release lint, builds and signs the APK, verifies its signature and permissions, and writes a SHA-256 checksum to artifacts/SHA256SUMS. The first run creates a private signing key in .signing/. Preserve that directory securely for future updates; it is excluded from version control and must not be distributed with the APK. Losing the key prevents in-place updates signed with the same identity.

An existing Gradle executable can be supplied through GRADLE_BIN. GRADLE_USER_HOME can point to an isolated cache. No global Java or system configuration changes are required.

## Tests and code

    ./gradlew :core:test :app:lintDebug :app:assembleDebug
    ./gradlew :app:connectedDebugAndroidTest

Run the device tests on an emulator or a dedicated test device. They use synthetic images and temporary database records. The Chinese OCR integration check verifies that the offline model runs and recognizes specific words; it does not require a perfect transcription. Known errors are documented in the verification record.

- core/: platform-independent batch selection, processing loop, status, normalization and search.
- app/: native Android UI, read-only MediaStore access, bounded decoding, SQLite and foreground OCR service.
- app/src/androidTest/: actual decoder, SQLite and bundled OCR tests.
- scripts/build-apk.sh: repeatable release and signing workflow.
- docs/verification.md: commands, results, known OCR error, screenshots and remaining device checks.

The app uses [ML Kit's bundled Chinese recognizer](https://developers.google.com/ml-kit/vision/text-recognition/v2/android), [Android MediaStore](https://developer.android.com/training/data-storage/shared/media) and [RE2/J](https://github.com/google/re2j).

## License

This project's code is available under the [MIT License](LICENSE). Third-party dependencies and the memes shown in screenshots retain their respective licenses and copyrights.
