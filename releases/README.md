# P0 媒体层探针 · 无影云冒烟测试

产物：`superphone-p0-audio-probe.apk`
- 最低系统：Android 10（minSdk 29），targetSdk 29 / compileSdk 33
- 权限：CAMERA（危险权限，需运行时/手动授权）
- 内置：媒体层三件套探针（Camera2 开摄 / AudioRecord 读 MIC / AudioTrack 播 440Hz）
- 自动跑：广播 `com.superphone.poc.RUN_ALL` 一键触发全部用例，logcat 标签 `P0PROBE`

## 1. 取产物

- 方式 A（git）：`git pull` 后取 `releases/superphone-p0-audio-probe.apk`
- 方式 B（原始文件）：仓库文件直链下载

## 2. 装到无影云实例

- 控制台「应用安装 / 上传 APK」直接上传；或走 ADB：
  1. 在无影控制台把本机 `adbkey.pub` 配进实例（否则 `adb connect` 会 `failed to authenticate`）
  2. `adb connect <实例地址>`
  3. `adb install superphone-p0-audio-probe.apk`
- 云端 Android 12+ 需手动授权相机权限：
  `adb shell pm grant com.superphone.poc android.permission.CAMERA`

## 3. 跑探针

- 在 App 内点「开摄像头 / 反向播放」按钮；或一键全跑：
  `adb shell am broadcast -a com.superphone.poc.RUN_ALL`
- 看结果：`adb logcat -s "P0PROBE:*"`

## 4. 拿到真实采集（推荐）

- 在无影策略里「启用摄像头重定向」，并在客户端设备（例如你的 OPPO）给无影 App 授予相机 / 麦克风权限。
- 此时云端探针读到的就是**客户端真实摄像头 / 麦克风**，且客户端无需 root（重定向由无影客户端在媒体框架层完成）。

## 5. 预期结果

| 用例 | 无重定向（虚拟设备） | 开重定向（真实设备） |
|---|---|---|
| Camera2 开摄 | 可能开到虚拟 cam / 测试图案 | 真实前/后摄首帧 |
| AudioRecord(MIC) | 峰值 ≈ 0–7（静音） | 说话时峰值 > 1000 |
| AudioTrack 播放 | 经虚拟声卡到客户端喇叭 | 同左，可闻 440Hz |
| VOICE_DOWNLINK / UPLINK / CALL | state = 0（死门坐实） | 同左 |

## 6. 说明

- 本 APK 验证的是「媒体层探针代码 + 真实/虚拟采集通路」，并非完整远控栈；采集与推流由无影客户端/框架完成。
- 阶段 0 收口建议：先在实体 OPPO 直装本 APK（`adb install` + 开 USB 调试）拿到真实 MIC/相机数据，再结合无影验证云端渲染路径。
