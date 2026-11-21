# UR / JKK 東京23区 空室ウォッチャー

[English](README.md) | [简体中文](README.zh-CN.md) | **日本語**

**UR賃貸** と **JKK東京（東京都住宅供給公社）の先着順空き家** を監視し、東京23区内の新着物件を検知します。
家賃＋共益費が設定額未満の物件が出たら、[ntfy](https://ntfy.sh) でスマホにプッシュ通知し、物件が掲載されてから成約するまでの時間も記録します。

> 非公式ツールです。UR / JKK の公式サイトが内部で使っているエンドポイントを利用しているため、サイトの仕様変更で動かなくなる可能性があります。アクセス頻度は常識的な範囲に保ってください。

## 機能

- **リアルタイム監視**：デフォルト設定では、日中は UR を5分ごと、JKK を3分ごとにチェック（JKK は先着順のため間隔を短めに設定）
- **家賃フィルター**：家賃＋共益費が `RENT_MAX`（デフォルト12万円）未満のときだけ通知。間取りでの絞り込みも可能
- **通知内容**：区、物件名、部屋番号、間取り、面積、階数、家賃、最寄り駅。UR の通知をタップすると部屋のページが開きます
- **成約記録**：物件が消えたら CSV に記録し、掲載期間を推定。`rent_stats.py` で区・間取り・価格帯ごとに集計
- **夜間モード**：夜間は58分ごとにチェック。切り替え時に優先度の低い通知を送信（音なし）
- **失敗時のバックオフ**：連続で失敗すると間隔を自動で延ばし、3回連続失敗で通知、復旧したら再度通知

実行時に自動生成されるファイル（`.gitignore` で除外済み）：

| ファイル | 内容 |
| --- | --- |
| `ur_seen.json` / `jkk_seen.json` | 確認済みの物件（削除すると全物件が再通知されます） |
| `ur_history.csv` / `jkk_history.csv` | 成約記録。Excel でそのまま開けます |

## 動作環境

- Python 3.8 以上（Linux / macOS）
- スマホに ntfy アプリ（[iOS](https://apps.apple.com/app/ntfy/id1625396347) / [Android](https://play.google.com/store/apps/details?id=io.heckel.ntfy)）

## 使い方

### 1. スマホの通知を設定

ntfy アプリで **+** をタップし、トピック名を作成します（例：`ur-8f3k2q`）。

### 2. スクリプトを取得してトピックを設定

```bash
git clone https://github.com/ChouYuduki/ur-jkk-23ku-alert.git ~/house
cd ~/house
echo 'export NTFY_TOPIC=【作成したトピック名】' >> ~/.bashrc
source ~/.bashrc
```

### 3. 通知のテスト

```bash
python3 ur_watch.py --test
python3 jkk_watch.py --test
```

スマホにテスト通知が2件届けば設定完了です。

### 4. 監視を開始（ターミナルを2つ開く）

```bash
# ターミナル1
cd ~/house && python3 ur_watch.py

# ターミナル2
cd ~/house && python3 jkk_watch.py
```

- 初回は「現在条件に合う空室は N 件」と通知され、以降は新着物件だけが通知されます
- ターミナルは開いたままにし、PC をスリープさせないでください

### 5. 成約統計を見る

```bash
python3 rent_stats.py          # すべての物件
python3 rent_stats.py --ok     # 家賃条件に合う物件だけ
```

## 設定

各スクリプトの冒頭で変更できます：

| 設定 | デフォルト | 説明 |
| --- | --- | --- |
| `RENT_MAX` | `120000` | 家賃＋共益費がこの額未満なら通知。`0` で制限なし |
| `LAYOUTS` | `set()` | 間取りの絞り込み。例：`{"1K", "1DK", "1LDK"}`。空ならすべて |
| `WARDS`（UR のみ） | 23区すべて | 不要な区を削除 |
| `INTERVAL_SEC` | UR `300` / JKK `180` | 日中のチェック間隔（秒） |
| `NIGHT_START` / `NIGHT_END` | `0` / `7` | 夜間の時間帯（日本時間。日付をまたぐ設定も可、例：`18` / `9`） |
| `NIGHT_INTERVAL_SEC` | `3480` | 夜間のチェック間隔（秒。3480秒＝58分） |
| `MAX_BACKOFF_SEC` | `3600` | 連続失敗時の最大待ち時間（秒） |
| `GONE_CONFIRM_ROUNDS` | `2` | 何回連続で見つからなければ成約とみなすか |

環境変数：`NTFY_TOPIC`（必須）、`NTFY_SERVER`（ntfy を自前で運用する場合）、`DISCORD_WEBHOOK`（任意。Discord にも通知）

## 参考

エンドポイントや取得手順について、以下のオープンソースプロジェクトを参考にしました：

- [songqintai/ur-monitor](https://github.com/songqintai/ur-monitor)：UR のエリア検索エンドポイント
- [tabikiji/UR_vacancy_notifier](https://github.com/tabikiji/UR_vacancy_notifier)：UR の空室数 → 部屋一覧の2段階取得
- [m-okumura/akiya-monitor](https://github.com/m-okumura/akiya-monitor)：JKKねっと の HTTP 検索フロー
