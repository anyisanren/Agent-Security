# Android 实验环境选型（2026-10-06）

结论：优先调查官方 Cuttlefish 预构建镜像与官方 ComputerControl CTS；暂不购买手机。尚未验证任何可直接运行本实验的具体镜像。本文仅包含公开来源的资料研究，不包含已运行实验的证据。

## 1. 官方测试入口确实存在

Android17r1 源码包含 ComputerControl 扩展库、VirtualDeviceManager 平台应用，以及 CtsVirtualDevicesComputerControlTests 模块。官方 Setup 测试检查 feature、VDM 服务、secondary-display 支持、扩展共享库、VDM 包及扩展实例。测试规则会跳过不支持的设备，因此 skipped 不等于通过。

- [扩展库](https://android.googlesource.com/platform/frameworks/base/+/android-17.0.0_r1/libs/computercontrol/Android.bp)
- [VDM 平台应用](https://android.googlesource.com/platform/frameworks/base/+/android-17.0.0_r1/packages/VirtualDeviceManager/Android.bp)
- [CTS 模块](https://android.googlesource.com/platform/cts/+/android-17.0.0_r1/tests/tests/virtualdevice/computercontrol/Android.bp)
- [Setup 测试](https://android.googlesource.com/platform/cts/+/android-17.0.0_r1/tests/tests/virtualdevice/computercontrol/src/android/virtualdevice/cts/computercontrol/ComputerControlSetupTest.kt)
- [Skip 规则](https://android.googlesource.com/platform/cts/+/android-17.0.0_r1/tests/tests/virtualdevice/computercontrol/src/android/virtualdevice/cts/computercontrol/ComputerControlRule.kt)

产品配置仅在 RELEASE_PACKAGE_COMPUTER_CONTROL=true 时打包 VirtualDeviceManager。未核实任何具体预构建镜像中该值及所有依赖是否有效。

[产品打包条件](https://android.googlesource.com/platform/build/+/android-17.0.0_r1/target/product/media_system.mk)

## 2. 候选排序

### A. 官方 Cuttlefish 预构建：第一候选，兼容性待证

官方文档提供 aosp-android-latest-release 分支，目标 aosp_cf_x86_64_only_phone-userdebug（另有 ARM64 目标）。镜像与 cvd-host_package.tar.gz 应来自同一构建。latest 会移动，实际研究需固定构建号、API、指纹及校验值。

官方运行路径需要支持 KVM 的 Linux 主机。这里尚未确认可用研究主机和具体镜像，不应直接开始音频实验。

[入门与要求](https://source.android.com/docs/devices/cuttlefish/get-started)，[官方 CI](https://ci.android.com/builds/branches/aosp-android-latest-release/grid)

### B. 源码匹配的 AOSP Cuttlefish：高成本后备

源码目标存在，但完整编译的官方主机要求包括64GB内存与400GB空闲磁盘。只有预构建组件检查不满足、且合法平台集成路径明确时，再评估编译投入。

[构建要求](https://source.android.com/docs/setup/start/requirements)，[Android17r1 Cuttlefish 产品](https://android.googlesource.com/device/google/cuttlefish/+/android-17.0.0_r1/vsoc_x86_64_only/phone/aosp_cf.mk)

### C. 已有 Pixel10/10 Pro/10 Pro XL：物理验证候选

Google 消费者屏幕自动化列表包含这些机型，但受地区、账号及分批推出影响。消费者功能可用，不证明研究者自己的目标 App 或独立测试工具满足平台政策。Android17 更新支持范围更广，也不能自动推导 ComputerControl 可用。

[消费者支持范围](https://support.google.com/pixelphone/answer/16940971?hl=en)，[Android17 获取](https://developer.android.com/about/versions/17/get)

### D. Samsung 已有支持机型：后续 OEM 比较

消费者支持列表包含 Galaxy S26 系列、Z Flip8/Fold8。未验证公开研究镜像或独立平台测试集成路线，不建议仅为该研究购机。

### E. 普通 AVD/GSI：不作为默认主线

官方 Android17 AVD/GSI 可获取，但尚未验证包含本实验所需 ComputerControl 集成。模拟器麦克风默认关闭，宿主输入与物理手机音频路径不同；GSI 还引入刷机与硬件兼容成本。

[模拟器控制说明](https://developer.android.com/studio/run/emulator-extended-controls)，[GSI 限制](https://developer.android.com/about/versions/17/gsi-release-notes)

## 3. 最小下一步

1. 固定一个官方 Cuttlefish 构建，并核对产物清单和组件条件。
2. 在明确授权、满足官方要求的主机上，验证官方 Setup 检查确实执行而非跳过。
3. 确认真实会话与目标 App 普通身份，再讨论音频实验。
4. 缺组件则记录环境不适用，不能当成研究假设被否决。

官方 CTS 下载提供 Android17 R2 ARM/x86 套件，但本轮未检查压缩包，未确认二进制套件包含上述模块。源码存在与预构建可用需分开验证。

[CTS 下载](https://source.android.com/docs/compatibility/cts/downloads)

## 4. 结论边界

Cuttlefish 适合框架机制研究，不能独立证明真实手机物理麦克风暴露或商业固件影响。需要分别验证平台组件、合法会话资格与最终物理数据路径。

[官方 Cuttlefish 定位](https://source.android.com/docs/devices/cuttlefish)，[ComputerControl 前置条件](https://developer.android.com/reference/android/companion/virtual/VirtualDeviceManager)

未购买设备、下载大型镜像、刷机、安装或执行实验。本文不是实验成功记录，也不改变现有未完成代码的状态。
