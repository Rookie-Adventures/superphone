# P0 本地音频 PoC（闸门一）

> **本目录是全项目当前唯一应该投入资源的地方。**
> 在 P0 出结论之前，不建议铺开 10 人团队和 31 项任务。

## 为什么先做这个

软件版能否成立，取决于一个尚未验证的问题：**普通手机装 APK（尽量 No Root）到底能不能提取并注入通话音频**。

- 能 → 语音版（S2）可卖，全线铺开
- 不能 → 降级到只卖控制版（S1），A 阶段 8 项任务照样商业化，止损

这是一道**闸门**，不是一个任务。闸门不打开，后面所有投入都是在赌。

## 边界（严格，来自设计草稿 §7）

| 做 | 不做 |
|---|---|
| 只测 **B 端本地**能否调通三个 API 取/注 PCM | 不涉及 A 端 |
| 记录机型、系统、SoC、SIM、VoLTE、延迟、稳定性 | 不涉及服务器、P2P/Relay |
| 落本地文件验证可听 | 不引入跨境链路 |

**P0 通过 ≠ 跨境通话可用。** 跨境是独立的 P3 战场（任务 C7）。

## 待测 API（三个，逐个实测）

```java
TelephonyManager.isPstnCallAudioInterceptable()          // 是否可拦截
TelephonyManager.getCallDownlinkExtractionAudioRecord()  // 下行提取（对方声音）
TelephonyManager.getCallUplinkInjectionAudioTrack()      // 上行注入（我方声音）
```

对每个 API 必须记录：是否存在、是否抛 `SecurityException` / `UnsupportedOperationException`、返回的 `AudioRecord` / `AudioTrack` 是否真正成功读到/写出数据。

## 执行顺序

| 序 | 任务 | 产出 | 负责 | 周期 |
|---|---|---|---|---|
| B1 | 机型矩阵与测试设备准备 | [机型矩阵.md](./机型矩阵.md) 填满 | E10 QA | 3-5 天 |
| B2 | PoC 探针工具与埋点 | 可复现探针工程 | E03 | 1 周 |
| B3 | 下行提取 PoC | 至少 1 款机型取到清晰可辨的下行 PCM | E03 | 1-2 周 |
| B4 | 上行注入 PoC | 至少 1 款机型注入成功且对方可听 | E03 | 1-2 周 |
| B5 | No Root 路径验证 | 权限边界实测结论 | E03 | 1-2 周 |

**B1 无依赖，今天就能开始**（买设备、列清单），且可与 A 阶段并行。这是目前唯一能提前消化的时间。

## No Root 待验路径

按优先级尝试，每条都要给出**权限边界实测结论**：

1. 普通 APK 直接调用（最理想）
2. ADB 授权（`adb shell` 一次性授权）
3. Wireless Debugging（Android 11+，免连线）
4. Shizuku（借助已有高权限 App 中转）
5. Embedded ADB / 系统签名（最后手段，等于放弃 No Root）

## 完成判据

- [ ] 机型矩阵至少覆盖 6 款真机，横跨 Android 10–15
- [ ] B3 与 B4 各有**至少 1 款机型**成功
- [ ] 所有结论已录入 [事实分级清单](./事实分级清单.md)，并标注等级
- [ ] No Root 路径给出明确结论（可行 / 需 ADB / 必须 Root）

任一判据不满足 → 触发降级评估，走设计草稿 §9 的"降级 SKU"预案。
