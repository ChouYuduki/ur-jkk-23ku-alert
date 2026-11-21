# UR / JKK 东京23区 空房监视

[English](README.md) | **简体中文** | [日本語](README.ja.md)

监视 **UR賃貸** 和 **JKK東京（東京都住宅供給公社）先着順空き家** 在东京23区内新放出的房源。
家賃+共益費低于设定价格时，通过 [ntfy](https://ntfy.sh) 推送到手机，同时记录房子从放出到租出用了多久。

> 非官方工具。调用的是 UR / JKK 官网页面背后的接口，网站改版后可能失效。请保持合理的访问频率。

## 功能

- **实时监视**：默认设置下，UR 白天每 5 分钟、JKK 白天每 3 分钟检查一次（JKK 是先到先得，间隔更短）
- **价格筛选**：家賃+共益費低于 `RENT_MAX`（默认 12 万日元）才推送，也可以限制户型
- **推送内容**：区、物件名、房号、户型、面积、楼层、价格、最近车站；点 UR 的通知可直接打开房间页面
- **租出记录**：房子消失后记入 CSV，估算在架时长；用 `rent_stats.py` 按区、户型、价格带统计
- **夜间降频**：夜间每 58 分钟检查一次；切换时推送一条低优先级通知（不响铃）
- **失败退避**：连续失败时自动延长间隔，连续失败 3 轮推送提醒，恢复后再通知

运行后会自动生成以下文件（已在 `.gitignore` 中排除）：

| 文件 | 内容 |
| --- | --- |
| `ur_seen.json` / `jkk_seen.json` | 已经见过的房间（不要删，否则会重新推送） |
| `ur_history.csv` / `jkk_history.csv` | 租出记录，Excel 可直接打开 |

## 环境

- Python 3.8 以上（Linux / macOS）
- 手机安装 ntfy App（[iOS](https://apps.apple.com/app/ntfy/id1625396347) / [Android](https://play.google.com/store/apps/details?id=io.heckel.ntfy)）

## 使用方法

### 1. 设置手机通知

在 ntfy App 里点 **+**，新建 topic 名，例如 `ur-8f3k2q`。

### 2. 下载脚本并设置 topic

```bash
git clone https://github.com/ChouYuduki/ur-jkk-23ku-alert.git ~/house
cd ~/house
echo 'export NTFY_TOPIC=【你刚刚填写的topic名】' >> ~/.bashrc
source ~/.bashrc
```

### 3. 测试通知

```bash
python3 ur_watch.py --test
python3 jkk_watch.py --test
```

手机收到两条测试通知就说明设置成功。

### 4. 开始监视（开两个终端）

```bash
# 终端 1
cd ~/house && python3 ur_watch.py

# 终端 2
cd ~/house && python3 jkk_watch.py
```

- 第一次运行会推送“目前符合条件的空房共 N 间”，之后只推送新放出的房间
- 终端要一直开着，电脑不能休眠

### 5. 查看租出统计

```bash
python3 rent_stats.py          # 所有房源
python3 rent_stats.py --ok     # 只看符合价格条件的
```

## 设置

在每个脚本开头修改：

| 设置 | 默认值 | 说明 |
| --- | --- | --- |
| `RENT_MAX` | `120000` | 家賃+共益費低于这个数才推送，`0` = 不限 |
| `LAYOUTS` | `set()` | 户型限制，例如 `{"1K", "1DK", "1LDK"}`；空 = 全部 |
| `WARDS`（仅 UR） | 23区全部 | 删掉不需要的区 |
| `INTERVAL_SEC` | UR `300` / JKK `180` | 白天检查间隔（秒） |
| `NIGHT_START` / `NIGHT_END` | `0` / `7` | 夜间时段（日本时间，可跨零点，例如 `18` / `9`） |
| `NIGHT_INTERVAL_SEC` | `3480` | 夜间检查间隔（秒，3480 秒 = 58 分钟） |
| `MAX_BACKOFF_SEC` | `3600` | 连续失败时最长等待时间（秒） |
| `GONE_CONFIRM_ROUNDS` | `2` | 连续几轮成功检查都不在，才算租出 |

环境变量：`NTFY_TOPIC`（必填）、`NTFY_SERVER`（自建 ntfy 时使用）、`DISCORD_WEBHOOK`（可选，同时推送到 Discord）。

## 参考

以下开源项目提供了接口和流程上的参考：

- [songqintai/ur-monitor](https://github.com/songqintai/ur-monitor)：UR 按区搜索接口
- [tabikiji/UR_vacancy_notifier](https://github.com/tabikiji/UR_vacancy_notifier)：UR 空房数 + 房间列表的两步取法
- [m-okumura/akiya-monitor](https://github.com/m-okumura/akiya-monitor)：JKKねっと 的 HTTP 检索流程
