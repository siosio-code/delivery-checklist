# 納品チェックリスト（Webアプリ）

納品業務の場面別チェックリスト。Android（Chrome）から片手で使うことを想定した、
静的ファイルのみのPWAです。サーバー・DB・ログインはありません。

## ファイル

| ファイル | 中身 |
| --- | --- |
| `index.html` | アプリ本体。HTML/CSS/JS すべてこの1ファイル |
| `sw.js` | Service Worker（オフライン起動のためのキャッシュ） |
| `manifest.webmanifest` | ホーム画面に追加するための情報 |
| `icons/` | アイコン（192/512/maskable） |

`sw.js` と `manifest.webmanifest` はブラウザの仕様上、HTMLに埋め込めないため別ファイルです。

## ローカルで確認する

`file://` で開くと Service Worker が動かないので、簡易サーバーを立てて開きます。

```bash
python3 -m http.server 8000
```
（このリポジトリを置いたフォルダの中で実行します）

- PC: <http://localhost:8000/>
- スマホ実機: PCと同じWi-Fiにつないで `http://<PCのIPアドレス>:8000/`
  （IPは `ip a` / `ipconfig` で確認。ただし PWA のインストールは `localhost` か HTTPS のみ）

### Chrome の DevTools で見るポイント

- Application → Manifest：エラーが出ていないか
- Application → Service Workers：`activated and is running`
- Network → 「Offline」にしてリロード → 起動すればオフラインOK

### スマホでの動作確認

GitHub Pages に公開してから、Chrome のメニュー →「ホーム画面に追加」。
追加したアイコンから起動すると、アドレスバーなしの全画面になります。

## GitHub Pages で公開する

リポジトリの Settings → Pages → Build and deployment で

- Source: **Deploy from a branch**
- Branch: **main** / フォルダ **/ (root)**

公開URL: `https://siosio-code.github.io/delivery-checklist/`

## 仕様メモ

- **保存先**: 端末の localStorage（キー `deliveryChecklist.v1`）。他の端末とは共有されません
- **日付の区切り**: 午前4時。深夜〜早朝の作業が前日分として続くようにしています
- **自動リセット**: 日付が変わるとチェックは外れます（アプリを開いたままでも30秒ごとに判定）
- **履歴**: チェックした時刻を日付ごとに直近60日分だけ保持。それより古いものは自動削除
- **バックアップ**: 設定画面から JSON を書き出し／読み込み。機種変更・データ削除の前に書き出してください

### データ構造（書き出したJSON）

```jsonc
{
  "version": 1,
  "settings": { "keepAwake": true },
  "scenes": [
    { "id": "…", "name": "出発前", "items": [ { "id": "…", "text": "納品書・伝票を持ったか" } ] }
  ],
  "log": {
    // 業務日 → 場面ID → 項目ID → チェックした時刻(epoch ms)
    "2026-09-01": { "<sceneId>": { "<itemId>": 1756770000000 } }
  }
}
```

## アプリを更新したとき

`index.html` などを変更したら、`sw.js` の `const CACHE = 'checklist-v1'` の数字を上げてください。
上げないと、端末に残った古いキャッシュが表示され続けます。
数字を上げると、次にオンラインで開いたときに「新しいバージョンがあります／更新」と出ます。

## 動作確認済み

Chromium（モバイル表示 384×832）で以下を自動テスト済み：
場面選択・チェック/解除・行全体のタップ判定・時刻記録・進捗表示・リセットの確認ダイアログ・
項目の追加/編集/削除/並べ替え・JSON書き出し/読み込み（不正ファイルの拒否含む）・
リロード後の保持・日付切り替え（午前4時境界／前日分の履歴保持／60日での自動削除）・
Service Worker 制御下でのオフライン起動。
