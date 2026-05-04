# LINE 家計簿 Bot

LINE Messaging API と Google Apps Script（GAS）を使った家計簿管理ボットです。  
LIFF フォームから支出を入力し、Google スプレッドシートに自動記録します。

---

## プロジェクト構成

```
gas/
├── expenses/       # シンプル版（項目・金額）
└── s_expenses/     # 詳細版（項目・内容・金額）
```

どちらも独立した GAS プロジェクトとして動作します。

---

## 機能

### 支出入力（LIFF フォーム）

LINE アプリ内のフォームから支出を入力します。

| バージョン | 入力項目 |
|---|---|
| expenses | 項目（カテゴリ）、金額 |
| s_expenses | 項目（カテゴリ）、内容（メモ）、金額 |

**カテゴリ一覧**  
食費 / 外食費 / 日用品費 / 交通費 / 医療費 / 交際費 / 雑費 / 娯楽費

### トークコマンド

LINE のトーク画面にテキストを送信することで情報を取得できます。

| コマンド | 内容 |
|---|---|
| `月別合計` | 今年度の月ごとの支出合計を返信 |
| `支払い合計` | 支払者ごとの合計金額を返信 |

### スプレッドシート記録

支出を送信すると Google スプレッドシートに自動で記録されます。

| 列 | expenses | s_expenses |
|---|---|---|
| A | 日付 | 日付 |
| B | 月 | 月 |
| C | 費目 | 費目 |
| D | 金額 | 内容 |
| E | 支払者 | 金額 |
| F | — | 支払者 |

年ごとにシートが作成されます。新年度のシートが存在しない場合は前年のシートをコピーして自動作成します。

### 友だち追加・削除

- **友だち追加時**：ユーザー情報（ID・表示名・日時）を `user` シートに登録
- **ブロック時**：`user` シートから該当ユーザーを削除

---

## セキュリティ

- **機密情報の管理**：チャネルトークン・スプレッドシートID・LIFF ID はすべて GAS Script Properties で管理し、ソースコードには含まれません
- **LINE 外アクセス拒否**：`liff.isInClient()` でブラウザからの直接アクセスをブロック
- **ログイン確認**：`liff.isLoggedIn()` で未認証ユーザーを LINE ログインへ誘導
- **入力バリデーション**：クライアント・サーバー両側でカテゴリのホワイトリスト検証・金額の範囲チェックを実施
- **XSS 対策**：送信前に特殊文字（`<>&"'\`` ）を除去
- **二重送信防止**：送信中はボタンを無効化してフラグ管理

---

## セットアップ

### 1. GAS プロジェクトの作成

`expenses` と `s_expenses` をそれぞれ別の GAS プロジェクトとして作成し、各ファイルをコピーします。

### 2. Script Properties の設定

各 GAS プロジェクトの「プロジェクトの設定」→「スクリプトプロパティ」に以下を登録します。

| プロパティ | 値 |
|---|---|
| `CHANNEL_TOKEN` | LINE チャネルアクセストークン |
| `SPREADSHEET_ID` | Google スプレッドシートの ID |
| `LIFF_ID` | LINE Developers で発行した LIFF ID |

### 3. ウェブアプリとしてデプロイ

GAS エディタから「デプロイ」→「新しいデプロイ」を選択し、以下の設定でデプロイします。

- 種類：ウェブアプリ
- 次のユーザーとして実行：自分
- アクセスできるユーザー：全員

### 4. LINE Developers の設定

1. [LINE Developers Console](https://developers.line.biz/) でチャネルを作成
2. Messaging API の Webhook URL にデプロイした GAS の URL を設定
3. LIFF アプリを追加し、エンドポイント URL に GAS の URL を設定
4. 発行された LIFF ID を Script Properties に登録

---

## 使用技術

- [Google Apps Script](https://workspace.google.com/products/apps-script/)
- [LINE Messaging API](https://developers.line.biz/ja/services/messaging-api/)
- [LIFF（LINE Front-end Framework）](https://developers.line.biz/ja/docs/liff/)
- [Google スプレッドシート](https://www.google.com/intl/ja/sheets/about/)
- [Bootstrap 4](https://getbootstrap.com/docs/4.4/)
