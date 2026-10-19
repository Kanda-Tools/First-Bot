# Bot1 - Discord 通話開始・終了通知 BOT

## 概要
Discord のボイスチャンネルで **通話が始まったとき** と **全員が退出して通話が終わったとき** に、指定したテキストチャンネルへ自動で通知する BOT

## 機能
 
| イベント | 通知内容 |
| --- | --- |
| ボイスチャンネルに最初の1人が参加 | `通話が **チャンネル名** で開始されました` |
| ボイスチャンネルから最後の1人が退出 | `**チャンネル名** の通話が終了` |
| ボイスチャンネル間を移動 | 移動元が無人になれば「終了」、移動先が1人目なら「開始」を通知 |
 
- 2人目以降の参加では通知しません(通話開始時の1回のみ)。
- 通話中のチャンネルは `HashSet<ulong>` で管理し、同じ通話で重複通知しないようにしています。
- 参加者数の取得前に 500ms 待機し、Discord 側のキャッシュ更新を待っています。


## 動作環境
 
- .NET 8 以上(C# 12 のコレクション式 `[]` を使用)
- 使用ライブラリ(NuGet)
  - [Discord.Net](https://www.nuget.org/packages/Discord.Net)
  - [dotenv.net](https://www.nuget.org/packages/dotenv.net)
```bash
dotnet add package Discord.Net
dotnet add package dotenv.net
```

## セットアップ
 
### 1. Discord BOT を作成する
 
1. [Discord Developer Portal](https://discord.com/developers/applications) で **New Application** を作成
2. **Bot** タブで BOT を追加し、**Token** をコピー
3. **OAuth2 → URL Generator** で以下を選択して招待 URL を発行し、サーバーに招待
   - Scopes: `bot`
   - Bot Permissions: `View Channels` / `Send Messages`
> Privileged Gateway Intents(特権インテント)は **不要** です。
> 使用インテントは `AllUnprivileged` + `GuildVoiceStates` のみです。
 
### 2. 通知先チャンネル ID を取得する
 
1. Discord の **設定 → 詳細設定 → 開発者モード** を ON
2. 通知を送りたいテキストチャンネルを右クリック → **チャンネルIDをコピー**
### 3. `.env` ファイルを作成する
 
```env
DISCORD_TOKEN=あなたのBOTトークン
NOTIFY_CHANNEL_ID=通知先テキストチャンネルのID
```
 
| 変数名 | 説明 |
| --- | --- |
| `DISCORD_TOKEN` | BOT のトークン |
| `NOTIFY_CHANNEL_ID` | 通知メッセージを送信するテキストチャンネルの ID |
 
`Program.cs` では `.env` を次のパスから読み込んでいます。環境に合わせて変更してください。
 
```csharp
DotEnv.Load(options: new DotEnvOptions(envFilePaths: new[] { "C:/FirstBot/Bot1/.env" }));
```
 
### 4. 起動する
 
```bash
dotnet run
```
 
起動に成功するとコンソールに `✅ Logged in as ...` と表示されます。
 
## プロジェクト構成
 
```
Bot1/
├─ Program.cs   # エントリポイント(.env 読み込み → Bot 起動)
├─ Bot.cs       # BOT 本体(接続・ボイス状態の監視・通知送信)
└─ .env         # トークンと通知先チャンネルID(Git にコミットしない)
```
 
## 処理の流れ
 
`UserVoiceStateUpdated` イベントを監視し、状態の変化を 3 パターンに分けて処理します。
 
1. **参加**(`before` が null → `after` あり)
   参加後の人数が 1 人で、まだ通話中として記録されていなければ「開始」を通知し `_activeCalls` に追加。
2. **退出**(`before` あり → `after` が null)
   退出後の人数が 0 人で、通話中として記録されていれば「終了」を通知し `_activeCalls` から削除。
3. **移動**(別のボイスチャンネルへ)
   移動元の終了判定と移動先の開始判定をそれぞれ実行。
## 注意事項・既知の課題
 
- **トークンのログ出力**:`RunAsync()` 内の `Console.WriteLine("TOKEN: " + _token);` はトークンが画面やログに残るため、**削除を推奨**します。
- **`.env` を Git に含めない**:`.gitignore` に `.env` を追加してください。トークンが漏れた場合は Developer Portal で **Reset Token** してください。
- **`.env` のパスが固定**:`Program.cs` で絶対パスを指定しているため、別環境では修正が必要です。
- **対象は 1 つの通知チャンネルのみ**:複数サーバーで利用する場合はサーバーごとの通知先設定が必要です。
- **BOT 再起動時の状態**:通話中かどうかはメモリ上でのみ管理しているため、再起動すると状態がリセットされます。
- `Bot.cs` の退出処理にある `updated` 変数は未使用です(削除可能)。
## ライセンス
 
(任意のライセンスを記載してください)
 