<div align="center">

<img src="./android/app/src/main/res/ic_launcher.png" alt="logo" width="128" height="128" />

<h1>南哪课表 APP</h1>

**为 不止 NJUer 打造的 Android/iOS 平台课程表,使用 Flutter 编写**

</div>

## 下载/Download

[南哪课表官网](https://nju.app)

[安卓下载地址](https://mirror.nju.edu.cn/download/%E5%8D%97%E5%93%AA%E8%AF%BE%E8%A1%A8)

[iOS 下载地址](https://apps.apple.com/cn/app/%E5%8D%97%E5%93%AA%E8%AF%BE%E8%A1%A8/id1511705694)

[南哪课表 FAQ](https://idealclover.top/archives/606/)

<!--more-->

## 功能/Features

当前适配学校：南京大学、东南大学、上海交通大学、中国人民大学

更多学校适配欢迎联系作者或自行 PR~

1. Android & iOS 全平台支持
2. 支持南京大学统一认证登录，本科生&研究生均支持
3. 一键导入南大教务处，自动获取当前周
4. 支持免修不免考，同一时段允许多门课程
5. 支持不同学期多个课表管理与切换
6. 更多样式，自由添加背景图片

## 如何支持更多学校

南哪课表解析课程的主要原理是通过打开页面，并通过 JS 脚本获取页面中的课程信息，然后解析出课程的时间、地点、教师等信息。

涉及到的主要文件：

> [api/schoolList.json](api/schoolList.json): 学校列表，包含学校名称、登录 URL、课程表 URL 等信息，在其中添加需要适配学校的相关信息
>
> [api/tools/njubksjw2.js](api/tools/njubksjw2.js): 当前南京大学使用的示例课程表解析脚本，你可以参考该文件，根据学校的课程表页面，获取课程信息并解析出时间、地点、教师等信息
>
> [api/tools/seubksxk.js](api/tools/parseCourse.js): 当前东南大学使用的示例课程表解析脚本，你可以参考该文件，在 JS 脚本中进行 http 请求，根据学校的课程表页面，获取课程信息并解析出时间、地点、教师等信息

需要返回的数据格式（返回下面 json 格式数据的 encodeURIComponent 结果）：

```
{
    "name": "当前学期名称",
    "courses": [
        {
            "name": "课程名称",
            "classroom": "课程地点",
            "class_number": "课程编号",
            "teacher": "教师名称",
            "test_time": "考试时间",
            "test_location": "考试地点",
            "link":"课程详情链接，没有传null",
            "week": [1,2,3], // 课程第几周上，是一个数组
            "week_time": 1, // 课程周几上
            "start_time": 1, // 课程开始节数，从1开始,
            "time_count": 2, // 课程持续节数，即结束节数-开始结束，注意如果是3-4节则为1，不需要额外进行加一操作
            "import_type": 1, // 固定填1，作为自动导入方式
            "info": "课程详情，没有传null",
            "data": null
        }
    ]
}

```

## 部署相关

三端使用 `.fvmrc` 固定的 Flutter `3.47.7+ohos-0.0.1-pre`（Dart 3.13.5）。扫码依赖使用 `third_party/mobile_scanner` 中带鸿蒙适配的 7.1.4 源码，并延后 Android 方向传感器访问至实际扫码；来源与补丁见该目录的 `NJU_PATCHES.md`。iOS 使用苹果 Vision，可在 Apple 芯片模拟器原生构建。

同时使用带鸿蒙适配的 WebView 4.13.0、sqflite 2.4.2 和 fluttertoast 9.0.0，清除旧 Android v1 注册接口的依赖。系统分享插件暂保留原实现，并将仅移除 v1 注册代码的兼容补丁放在 `third_party/share_extend`（来源与变更见该目录的 `NJU_PATCHES.md`），无需修改本机 Pub 缓存。

1. 在项目根目录执行 `source tool/setup_ohos_env.sh` 加载开发环境。需要国内 Flutter 下载镜像时，可先设置 `export USE_CN_FLUTTER_MIRROR=1`。
2. 设置 `export PUB_HOSTED_URL=https://pub.flutter-io.cn`，与锁文件中的包源保持一致。使用 Python 3.12 或更高版本运行 `bash tool/ci/bootstrap.sh`，脚本会检出固定版本的 SDK 和依赖、应用项目补丁并恢复锁定的包版本。
3. 如需构建鸿蒙版，配置 DevEco SDK 后执行 `.fvm/flutter_sdk/bin/flutter config --enable-ohos`。使用 `.fvm/flutter_sdk/bin/flutter doctor -v` 检查目标平台的工具链。

Android 构建使用 JDK 21、Android SDK 37、AGP 9.1.1 和 Gradle 9.5.0。`targetSdkVersion` 沿用 33，最低支持 Android 7.0（API 24）。iOS 的 UMCommon / UMDevice 固定到带 ARM64 模拟器切片的官方 XCFramework 版本。4.1.0 起主应用最低支持 iOS 15，小组件仍需 iOS 17；真机与模拟器部署下限一致，支持 Xcode 27 构建。

### 使用项目 SDK 构建

GitHub Actions 已提供 Android / iOS / HarmonyOS 三端构建，支持无签名校验和手动签名打包；触发方式、产物与 Secrets 配置见 [三端 CI 说明](tool/ci/README.md)。

三端均使用项目固定的 SDK，避免终端切回旧版全局 Flutter：

```bash
.fvm/flutter_sdk/bin/flutter pub get
.fvm/flutter_sdk/bin/flutter build apk --release
.fvm/flutter_sdk/bin/flutter run -d <iOS模拟器UUID>
.fvm/flutter_sdk/bin/flutter build hap --debug
```

Apple 芯片 Mac 的 iOS 验证使用指定模拟器的 ARM64 构建。当前 OHOS SDK 的 macOS Release `gen_snapshot` 仍仅提供 x86_64，需通过 Rosetta 运行。若本机尚未安装，可按[苹果官方说明](https://support.apple.com/zh-cn/102527)执行 `softwareupdate --install-rosetta` 并接受许可，再运行 `.fvm/flutter_sdk/bin/flutter build hap --release`。Rosetta 用于 Mac 上的构建工具，输出的鸿蒙应用仍为 ARM64。

鸿蒙入口通过引擎参数 `--no-enable-merged-platform-ui-thread` 保留独立 UI 线程，避免 Flutter 3.35 Release 在平台线程挂载页面时栈溢出、停留白屏。该设置放在项目的 `EntryAbility.getFlutterShellArgs()` 中。
