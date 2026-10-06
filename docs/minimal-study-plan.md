# 最小验证计划：虚拟执行会话的音频资源约束

状态：研究假设，尚未验证漏洞。日期：2026-10-06。

## 1. 研究问题与假设

在持续活动的 Android ComputerControl 虚拟会话中，系统配置的静音输入限制是否始终约束目标 App 的实际音频输入？候选假设是：目标 App 的公开设备选择偏好可能与会话资源策略存在优先级冲突。源码分支仅提供调查线索，不证明运行时可达性、真实音频获取、受影响产品或新颖性。

## 2. 威胁模型

研究者控制自己编写的测试 App，在自有或明确授权的测试设备上开展实验。目标 App 是普通应用 UID，事先有正常麦克风授权且满足系统录音资格，没有 root、shell 或特权音频权限。会话测试工具与目标 App 使用不同身份；测试工具的系统集成能力不得传递给目标 App。保持用户同意、会话策略、Context、前台服务资格及 UI 状态可核对。

不涉及第三方真实用户、绕过用户授权、隐蔽采集、远程传输音频或商业固件受影响的推断。

## 3. 范围

首轮只检查一种资源、一个经验证的系统构建、一套测试 App 和同一会话内的对照。不扩展到相机、撤权后行为、多模型或商业产品比较。当前工作仅为计划与工程准备；真实设备后端、录音及环境变更尚未开展。

## 4. 阶段计划与门槛

以下是工作量估计，不是预约或完成时间承诺。系统镜像、设备、平台测试接口的取得时间不包含在内。

| 阶段 | 工作量估计 | 输出 | 进入下一阶段的条件 |
| --- | --- | --- | --- |
| P0 可行性检查 | 0.5–1 个工作日 | 设备/构建/组件与权限清单 | 确认真实 ComputerControl、虚拟设备及静音机制可用；否则停止并记录阻塞 |
| P1 环境与记录 | 0.5 个工作日 | 固定版本配置、日志结构、恢复检查 | 离线记录与恢复测试通过，不将模拟数据作为实机证据 |
| P2 基线 | 0.5 个工作日 | 正常录音资格对照与会话默认静音证据 | 确认输入来源和检测方法有效，静音不是设备输入关闭造成 |
| P3 最小 A/B/C 对照 | 0.5–1 个工作日 | 同一会话内三阶段时间线和证据清单 | 会话和静音工作线程持续有效，仅设备偏好变化 |
| P4 重复与裁决 | 0.5 个工作日 | 至少三组独立重复、失败分析、下一步 | 证据完整后人工复核；不完整则 inconclusive |

P0 依赖未解决时不得把后续阶段排成已确定日期，也不得将普通虚拟显示或自制静音替代真实隔离机制。

## 5. 最小实验设计

A：默认设备选择。B：候选显式输入偏好。C：恢复默认。App、UID、权限、Context、会话、静音策略、录音资格及前台服务状态保持一致。

实际设备阶段仅在确认授权和隐私边界后开展，使用研究者可控的非语音测试信号，不保留旁人语音。新测试信号必须在采集开始后生成，排除缓存与回放。先验证检测方法的正/负对照，再开展候选路径比较。

## 6. 成功标准与否决标准

支持候选假设需要同时证明：
1. 同一个真实 ComputerControl 会话持续活动。
2. 静音策略和注入线程持续运行，而非仅注册成功。
3. 目标 App 未进入物理前台、未被接管、未获新增授权或特权。
4. 活动采集期间的实际输入路由指向物理输入，而非仅 API 接受偏好。
5. B 阶段出现新生成的受控外部测试信号。
6. 匹配的 A/C 阶段没有该信号，并能排除 Context、录音资格、输入关闭等混杂因素。

设备枚举、权限 granted、偏好 API 返回 true、合成日志或手工填写的 JSON 均不能证明成功。

结果分类：supported（完整证据经人工复核支持这一构建上的假设）、falsified-for-tested-path（有效对照下该具体路径被强制阻断）、inconclusive（缺组件、会话失败、证据不足或未测试）。单个失败不否定整个研究领域；模拟器结果不证明真实手机 HAL 行为；无硬件不等于假设被否决。

## 7. 工程连续性约定

Git 保存经过审阅的代码、计划和脱敏摘要；配置记录系统版本、实验参数和协议版本。每轮分配唯一 run ID，清单记录代码提交、配置摘要、时间、阶段、产物哈希及证据来源。事件日志追加写入，检查点原子更新。

中断恢复前检查 Git 状态、配置哈希、已有产物完整性、进程身份与存活状态。无法确认的进程不得视为仍在采集。真实会话失效后新建 attempt，不跨会话拼接 A/B/C。保留旧失败记录；不要覆写失败为成功。

每轮状态固定包含：已验证结论、失败原因、未解决问题、下一步、阻塞与证据链接。当前未完成的代码不能作为可靠恢复工具使用。

## 8. 公开仓库边界

可发布实验过程和脱敏进度日志。不得提交凭据、设备个人信息、原始录音、私人聊天或未经验证的受影响产品声明。原始证据需另行审查后才决定是否公开。当前不发布未完成实验代码。

## 9. 已核对的一手来源

- [ComputerControl 官方说明](https://developer.android.com/ai/computer-control)
- [VirtualDeviceManager](https://developer.android.com/reference/android/companion/virtual/VirtualDeviceManager)
- [SessionImpl，固定源码版本](https://android.googlesource.com/platform/frameworks/base/+/94b4c163b7dfe5ce3607f7bb8456f9573f7de57d/services/companion/java/com/android/server/companion/virtual/computercontrol/ComputerControlSessionImpl.java)
- [静音注入器，固定源码版本](https://android.googlesource.com/platform/frameworks/base/+/94b4c163b7dfe5ce3607f7bb8456f9573f7de57d/services/companion/java/com/android/server/companion/virtual/computercontrol/ComputerControlAudioInjector.java)
- [AudioPolicyManager，固定源码版本](https://android.googlesource.com/platform/frameworks/av/+/475269ec44acec82792be6b01fb8e16357d4d2c8/services/audiopolicy/managerdefault/AudioPolicyManager.cpp)
- [ServiceUtilities，固定源码版本](https://android.googlesource.com/platform/frameworks/av/+/475269ec44acec82792be6b01fb8e16357d4d2c8/media/utils/ServiceUtilities.cpp)

这些源码属于 Android 17.0.0_r1。普通公共 SDK/通用模拟器不自动满足 ComputerControl 测试条件。需要核对平台组件、测试集成和目标 App 运行资格，且始终保留正常录音权限检查。
