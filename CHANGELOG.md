# CHANGELOG

本仓库为 [`szvone/vmqApk`](https://github.com/szvone/vmqApk)（V免签 安卓监控端）的**维护分支**，
在原版基础上修复新版支付宝/微信通知解析问题，并移除已失效的 2019 旧工具链依赖。

## [v1.1.0] - 2026-09-01

### 修复：新版支付宝通知解析（核心，真实支付不推送/金额错误）

- **通知文案从 content 挪到 title**：新版支付宝把「你已成功收款0.06元」放进了通知**标题(title)**，
  内容(content)只剩「已转入余额…」。原版只解析 content 字段 → 关键词匹配不到 → 永不推送。
  现改为 **title + content 双字段**合并判断（支付宝/微信均加固）。
- **金额提取被广告数字误导**：新版内容字段混入推广文案（如「经营码仅0.38%收钱费率」），
  原「取内容最后一个数字」逻辑会误取 `0.38` 而非实际收款金额。
  新增 `findMoney()`：**优先匹配「X元」模式**（如 `0.06元`），匹配不到才回退旧逻辑。
- 微信收款通知同样双字段加固（保留 title 判断，补充「收款」关键词）。

### 兼容性修复（脱离 2019 旧工具链）

- 移除 support/appcompat 依赖（jcenter 已关停，依赖死掉）：
  `AppCompatActivity`→`Activity`、`ActivityCompat`→原生 API、`NotificationCompat`→`Notification`。
- 删除 `Canvas.save(int)` 调用（API 26 已移除，该段 save/restore 无实际绘制）。
- Manifest 增加 `usesCleartextTraffic="true"`（Android 9+ 默认禁止明文 HTTP，监控端需直连 http 服务端）。
- Gradle 仓库 `jcenter` → `google`/`mavenCentral`；AGP `2.3.2` → `3.5.4`；compileSdk 26 → 28
  （minSdk/targetSdk 保持原值不变）。

### 构建说明（2026-09-01 实测）

- 本机 gradle 6.x/8.x native 库系统性加载失败，改用**手工构建**：
  `aapt2 → javac → d8 → apksigner`（build-tools 28.0.3），产物 `vmq-fixed3.apk`（约 485KB）。
- 签名密钥为新生成，**与原版签名不同**：升级安装需先卸载旧版（会丢失 App 内配置，重装后重新填写
  host / 通讯密钥，并重新开启通知监听权限）。
- 修复后已在真实环境验证：支付宝 0.06 元收款 → 通知解析出 0.06 → appPush → 服务端回调 → 支付页跳转全链路通过。
