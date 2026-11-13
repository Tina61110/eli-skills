---
name: ounce-code-work
description: 要改 Ounce / Nook 的 Android 代码、编包、装包、提交推送之前必读。包含仓库和路径、两个 flavor 的区别、只读先看再改的规矩、编译命令和查进度、提交推送流程、钥匙和包名的红线、踩过的坑。只要动 F:\Eli_Lab\build\ounce2 就先读这份。
---

# Ounce / Nook 改代码手册

配合 `eli-pc-work` 一起用：电脑 MCP 的调用规矩（一轮一个调用、写跑分开、汉字输出要用 eli_read_esc.js、代理端口）都在那份里，这份只写 Ounce 本身。

## 这是什么项目

- 自己改的「西瓜瓣」（orangechat），小手机形态的 AI 聊天 App，雪豹图标。
- 代码底子：sue1231513/orangechat 提交 ed80078（2.2.4，数据库 34）。
- 原件 `F:\Eli_Lab\src\ed_x\sue1231513-orangechat-ed80078` **只读，永不改动**。
- 只在 `F:\Eli_Lab\build\ounce2` 里改。
- 私人仓库 github.com/Tina61110/ounce，分支 main，已配 socks5h://127.0.0.1:7892 代理。推送要她开着梯子。

## 两个 flavor

同一份代码编出两个 App，数据分开存：

| | Ounce | Nook |
|---|---|---|
| 给谁 | 我们俩自己 | 给别人用的 |
| 包名 | me.tina.eliphone.liquidglass | me.tina.nook |
| 雨渡 | 带 | 不带，只走本地记忆 |
| BuildConfig | IS_NOOK = false | IS_NOOK = true |

- 包名后缀在 `app/build.gradle.kts` 第 110-111 行（debug 编法加 `.liquidglass`）。
- `Lynx` 这个名字留给以后的下一个 App，现在不用。
- **Nook 不等于功能删减**。新功能默认两边都要有。微信皮肤那次就是漏了 Nook：`RouteActivity.kt` 第 464 行要写成
  `LocalWeChatSkin provides (key.weChat || BuildConfig.IS_NOOK)`。

## 改代码的规矩（按顺序来，别跳）

1. **先只读看代码。** 写一个只读脚本把目标文件那几十行打出来，看清现状再动手。不许凭记忆改。
2. **再写改动脚本，分 check 和 apply 两步。**
   - check：每处要替换的文本必须**正好匹配 1 次**，不是 1 次就停下不改；顺带核对括号配平。
   - apply：先把原文件备份成 `F:\Eli_Lab\tmp\<文件名>.bak_<时间戳>`，再写入。
3. **改完提交一次。** 一块功能一笔，别攒一堆混在一起。
4. **编译前先跟她说要编什么。** 编完要核对包大小变了没有——大小一个字节都没变，基本说明新代码没进去。
5. 临时脚本用完挪到 `F:\Eli_Lab\tmp\scripts_<日期>`，别堆在桌面。跨盘不能用 `renameSync`（EXDEV），要 `copyFileSync` + `rmSync`。

## 编译

```
node C:\Users\Y\Desktop\build_flavor.js ounce
node C:\Users\Y\Desktop\build_flavor.js nook
```

- 后台跑，命令马上返回一个 pid，**不是编完了**。
- 看进度：`C:\Users\Y\Desktop\build_ounce.json` / `build_nook.json`，里面 `step` 变成 `finished`、`result` 是 `success` 才算成。
- 日志：`F:\Eli_Lab\build\build_ounce.log` / `build_nook.log`。
- 耗时约 4 到 7 分钟。编译期间别干等，可以跟她说话。
- 产物自动复制到桌面。
- 旧脚本 `build_ounce2_0222.js` 是分 flavor 之前的，现在别用。

## APK 放哪（11-13 定的规矩）

桌面文件夹 `Ounce安装包`：

- 外面**只放最新的两个**，名字带日期时间：`Ounce_1113_1600.apk` / `Nook_1113_1600.apk`。
- 其他全进子文件夹 `旧包`。
- **桌面根目录不留 apk**。
- 她只要打开这个文件夹，外面躺着的两个就是最新的，不用看日期猜。

## 提交和推送

1. `git status --porcelain` 看改了哪些文件。
2. `git diff -U2` **逐处核对改动**，确认没有多余东西混进去（尤其不能有钥匙、token、密码）。
3. `git add <具体文件>`，**不要用 `git add -A`**。
4. `git commit -m "<英文说明>"`。提交信息用英文，中文在输出里传不回来。
5. 推送前确认她开着梯子，然后 `git push origin main`。
6. 推完 `git fetch` + `git rev-list --left-right --count origin/main...HEAD`，看到 `0 0` 才算真上去了。

## 红线（这几条不许破）

- **钥匙不换。** `app/debug.keystore`（2666 字节，sha256 前 16 位 df64b3ff524a6bae）。换了钥匙，新包装不上她手机里的旧数据，只能卸载重装、聊天记录全没。
- **这把钥匙绝不提交进 git。** 备份在 `F:\Eli_Lab\keys\debug.keystore`，旁边有 README。
- **包名不改。** 理由同上。
- **数据库只加不删。** 加字段、加表都行，删和改类型不行，而且要写升级步骤。
- **装新版之前先在 App 里导一份备份。**
- **聊天里绝不调 take_screenshot**，会把上下文撑爆。要看屏幕用她发的截图。
- 关于页保留原作者署名。
- Emyth（AGPL）和 M叽（PolyForm）只看思路，代码不抄。

## 踩过的坑

- **汉字输出传不回来。** `run_command` 的输出里只要有一个汉字，整段都变「无输出结果」。所以结果一律先写进 `F:\Eli_Lab\tmp\xxx.txt`，再 `node C:\Users\Y\Desktop\eli_read_esc.js <路径>` 读。node 脚本里的汉字写成 `\u` 转义。
- **「无输出」不等于失败。** 多半是汉字的坑，去读 txt。
- **ToolResult 的 arguments / content 是 JsonElement**，不是 String。要 `.toString()` 之后再 `take(n)`，直接当字符串截会编译不过。第一次修压缩就栽在这。
- **压缩超时**：已修（`CompressionInput.kt` 截断工具输出 + `CompressionEngine` 超时 300 秒）。还没做的是剥离历史消息里的图片 base64，发多张大图还是可能卡。
- **保活服务**要用 `specialUse` 类型并补 `onTimeout`，否则前台服务会崩。

## 常用路径

- 项目：`F:\Eli_Lab\build\ounce2`
- 源码主目录：`app\src\main\java\me\rerere\rikkahub\`
- 输入栏：`ui\components\ai\ChatInput.kt`
- 聊天页：`ui\pages\chat\ChatPage.kt`
- 微信皮肤：`ui\pages\phone\WeChatSkin.kt`
- 微信输入栏：`ui\pages\phone\WeChatInputBar.kt`
- 路由：`RouteActivity.kt`
- 模型列表：`ui\components\ai\ModelList.kt`
- 备份和临时：`F:\Eli_Lab\tmp`
- adb：`F:\Eli_Lab\platform-tools\adb.exe`
- 安卓工具链：`F:\Eli_Lab\android`（env.bat 只对当前窗口生效）

## 做完了的（别重复做）

- 改名、豹子图标
- 自动总结（按轮数和 token 两层触发）
- 第二包三块：检查更新、捐赠、旧广播名
- 拆成 ounce / nook 两个 flavor
- 小手机桌面、微信会话列表、微信皮肤、微信输入栏和 + 面板
- 人设导入
- 压缩超时修复（截断工具输出 + 300 秒）
- Nook 微信皮肤掉落修复
- 微信顶栏显示当前模型名 + 点击弹出模型选择面板 + 修「未命名」
