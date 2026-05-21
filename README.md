# BeepSim

PowerShell / macOS / Linux のビープ音コマンドをブラウザ上で組み立て・試聴するツール。

**[▶ 今すぐ使う](https://budoucha.github.io/beepsim/)**

## 概要

周波数（Hz）と長さ（ms）を並べてシーケンスを作り、ブラウザ上で試聴しながらコマンドをコピーして実際に鳴らす用途。サーバー不要、インストール不要。

## 対応プラットフォーム

| プラットフォーム | コマンド | 必要なもの |
|---|---|---|
| Windows (PowerShell) | `[console]::beep(Hz, ms)` | 標準機能（追加不要） |
| macOS | `play -n synth {秒} square {Hz}` | sox（`brew install sox`） |
| Linux | `beep -f {Hz} -l {ms}` | beep（`apt install beep`） |

## 使い方

1. **[BeepSim](https://budoucha.github.io/beepsim/)** をブラウザで開く
2. 行を追加して周波数・長さを入力
3. `Ctrl+Enter` で試聴
4. タブでプラットフォームを切り替えてコマンドをコピー

### Hz欄を空にするとスリープ

Hz欄を空にした行はスリープ（無音の待機）に変換される。

- PowerShell: `Start-Sleep -Milliseconds ms`
- macOS / Linux: `sleep {秒}`

### Bluetoothイヤホンを使う場合

BT接続の遅延により、最初のビープが聞こえないことがある。  
シーケンスの先頭に **37Hz（人間にほぼ聞こえない最低周波数）を700ms程度** 追加しておくと、イヤホンが起動して以降の音が確実に聞こえる。

```powershell
# 例: 37Hz のウォームアップ → 本番音
[console]::beep(37, 700); [console]::beep(1047, 480); [console]::beep(660, 180)
```

また、ビープとビープの間に `Start-Sleep` を挟むとBT接続が切れて次の音が聞こえなくなる場合がある。その場合はスリープの代わりに 37Hz のビープを挟むと接続を維持できる。

## ショートカット

| 操作 | ショートカット |
|---|---|
| 再生 / 停止 | `Ctrl+Enter` |

## 技術情報

- 単一HTMLファイル（フレームワークなし）
- Web Audio API で試聴（Square波オシレーター）
- localStorage で状態を自動保存
- HTML5 Drag & Drop で行の並び替え
