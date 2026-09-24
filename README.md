# Kikitan Translator DeepL

An unofficial fork of [Kikitan Translator](https://github.com/YusufOzmen01/kikitan-translator) that adds DeepL API translation while retaining the original Google Translate option.

This project is based on upstream tag [`1.2.5`](https://github.com/YusufOzmen01/kikitan-translator/tree/1.2.5). It is independently maintained and is not an official release from the original Kikitan author or DeepL.

## Main features

- Speech recognition and VRChat chatbox output
- DeepL API Free and DeepL API Pro support
- Google Translate can be selected without an API key
- Translation engine selection from `Settings > Translation`
- DeepL API keys stored in Windows Credential Manager
- Orange UI and separate application identity for easy distinction from upstream Kikitan
- `byDL` or `byGgl` appended only to successful VRChat translation messages
- Light and dark theme support

## Download

Download the latest installer from [GitHub Releases](https://github.com/BBBB5551111/kikitan-deepl/releases).

Important:

- The installer is currently unsigned. Windows SmartScreen may display a warning.
- No DeepL API key is included. Each user must obtain and configure their own key.
- This edition can be installed alongside upstream Kikitan.
- Do not run both editions at the same time. Both use VRChat OSC port `9000` by default and can conflict.
- Google Translate uses Kikitan's original connection method and can be affected by external service changes.

## クイックスタート

1. [Releases](https://github.com/BBBB5551111/kikitan-deepl/releases)から最新版のセットアップファイルをダウンロードしてインストールします。
2. Kikitan Translator DeepLを起動し、歯車アイコンから`Translation`を開きます。
3. DeepLを使う場合は`DeepL API`を選び、契約に合わせて`DeepL API Free`または`DeepL API Pro`を選択します。
4. 自分のDeepL APIキーを入力して`Save`を押し、`Check Connection`で接続を確認します。
5. Google翻訳を使う場合は`Google Translate`を選択します。APIキーは不要です。
6. 設定画面を閉じると選択した翻訳エンジンが適用されます。

DeepL APIキーはWindows資格情報マネージャーへ保存され、アプリの設定ファイルやログには書き込まれません。APIキーをチャット、スクリーンショット、Issueへ掲載しないでください。

## トラブルシューティング

### 声を拾わなくなった（WebView2 Runtime 153）

WindowsのMicrosoft Edge WebView2 Runtimeが`153`に自動更新されると、音声認識が`network`エラーで止まり、Kikitanが声を拾わなくなります。Microsoft側の不具合です（[MicrosoftEdge/WebView2Feedback#5724](https://github.com/MicrosoftEdge/WebView2Feedback/issues/5724)）。Kikitanだけを`152`で動かすと直ります。

**1. 該当するか確認する**

エクスプローラーで`C:\Program Files (x86)\Microsoft\EdgeWebView\Application`を開きます。`153`で始まるフォルダがあれば該当します。WebView2は「インストールされているアプリ」の一覧に表示されないことがあります。

**2. 対処する**

1. [WebView2のダウンロードページ](https://developer.microsoft.com/microsoft-edge/webview2)の「Fixed Version（修正バージョン）」で`152.0.4191.62`と`x64`を選び、「ダウンロード」フォルダに保存します。
2. Kikitanを完全に終了します。
3. スタートボタンを右クリックし、「ターミナル (管理者)」を開きます。
4. 次のブロックを丸ごと貼り付けてEnterを押します。

```powershell
$cab = Get-ChildItem "$env:USERPROFILE\Downloads\Microsoft.WebView2.FixedVersionRuntime.152.*.x64.cab" | Select-Object -First 1
$exe = Get-ChildItem "$env:LOCALAPPDATA","$env:ProgramFiles" -Filter kikitan-translator-deepl.exe -Recurse -ErrorAction SilentlyContinue | Select-Object -First 1 -ExpandProperty FullName
if (-not $cab) { "ダウンロードに 152 の .cab がありません" }
elseif (-not $exe) { "kikitan-translator-deepl.exe が見つかりません" }
else {
  New-Item -ItemType Directory -Force C:\WebView2Fixed | Out-Null
  expand.exe $cab.FullName -F:* C:\WebView2Fixed | Out-Null
  icacls C:\WebView2Fixed /grant "*S-1-15-2-2:(OI)(CI)(RX)" /T | Out-Null
  icacls C:\WebView2Fixed /grant "*S-1-15-2-1:(OI)(CI)(RX)" /T | Out-Null
  $wv = (Get-ChildItem C:\WebView2Fixed -Directory -Filter "Microsoft.WebView2.FixedVersionRuntime.152*" | Select-Object -First 1).FullName
  $bat = "@echo off`r`nset WEBVIEW2_BROWSER_EXECUTABLE_FOLDER=$wv`r`nstart `"`" `"$exe`"`r`n"
  Set-Content -Path "$([Environment]::GetFolderPath('Desktop'))\Kikitan起動.bat" -Value $bat -Encoding Default
  "完了: デスクトップに Kikitan起動.bat を作成しました"
}
```

5. 以後はデスクトップの`Kikitan起動.bat`から起動します。通常のショートカットから起動すると153が使われ、声を拾いません。

**3. Microsoftが修正したあと**

手順1のフォルダ名が153より新しくなったら、通常のショートカットから起動して試してください。声を拾えれば、`Kikitan起動.bat`と`C:\WebView2Fixed`は削除できます。

## Translation engine behavior

Successful translations sent to the VRChat chatbox are marked as follows:

- DeepL API: `byDL`
- Google Translate: `byGgl`

The marker is added only to the VRChat message. It is not added to the on-screen translation, message history, exported data, fallback text, or API request.

If DeepL does not support the selected language combination, the application reports the error without sending a translation request. Translation errors do not stop speech recognition. The optional fallback setting can send the original text to VRChat when translation fails.

## Application identity

Current version: `1.2.5-deepl.9`

The DeepL edition uses:

- Product name: `Kikitan Translator DeepL`
- Application identifier: `com.sagat.kikitan.deepl`
- Executable name: `kikitan-translator-deepl.exe`

Automatic updates from the upstream Kikitan release channel are disabled so that an upstream update cannot silently replace this edition.

## Build from source

### Requirements

- Rust 1.77.2 or newer
- Node.js 22.1.0 or newer
- .NET SDK 9 or newer for the desktop/OpenVR overlay

```powershell
git clone https://github.com/BBBB5551111/kikitan-deepl.git
Set-Location -LiteralPath ".\kikitan-deepl"

npm install
powershell -ExecutionPolicy Bypass -File scripts/build_overlay.ps1

# Development
npm run tauri dev

# Release build
npm run tauri build
```

Code signing is required to distribute a trusted signed installer. The published community build is currently unsigned.

## Upstream project

Kikitan Translator was created by the original project author and contributors:

- [Upstream source repository](https://github.com/YusufOzmen01/kikitan-translator)
- [Original BOOTH page](https://sergiomarquina.booth.pm/items/6073050)
- [Support the original author](https://buymeacoffee.com/sergiomarquina)

See [NOTICE.md](NOTICE.md) for attribution and [CHANGELOG.md](CHANGELOG.md) for changes in this fork.

## License

This fork remains available under the upstream MIT License. See [LICENSE.md](LICENSE.md).

Copyright 2024 SergioMarquina
