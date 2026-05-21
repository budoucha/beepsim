# BeepSim

`~/devel/beepsim/index.html` に全機能が入った単一HTMLファイルのアプリ。
サーバー不要、ブラウザで直接開いて使う（file://）。

## 概要

PowerShell の `[console]::beep(Hz, ms)` コマンドを視覚的に組み立てるツール。
ブラウザ上（Web Audio API）で試聴しながら、コマンドをコピーして実際に鳴らす用途。

## 技術スタック

- 素のHTML/CSS/JS（フレームワークなし）
- Web Audio API（SquareWaveオシレーター）
- localStorage（状態の永続化）
- HTML5 Drag & Drop API（行の並び替え）

## アーキテクチャ

状態は `rows` 配列で管理。各rowは `{ id, hz, ms }` を持つ。
`renderRows()` が唯一のDOM再構築関数で、変更のたびに全行を再描画する。
`updatePreview()` がPreviewのHTML生成とlocalSave両方をトリガーする。

## PowerShellコマンドの仕様

- 周波数: 37〜32767 Hz（範囲外はArgumentOutOfRangeException）
- 長さ: 1ms以上（ただしWindowsタイマー精度の都合で実質15ms未満は意味がない）
- 周波数欄が空の行 → `Start-Sleep -Milliseconds ms` に変換される

## Bluetoothイヤホン対策の知見

BT接続の遅延（数百ms）により短いビープが聞こえない問題がある。
対策として、本番音の前に**37Hz（最低周波数）で長めのダミービープ**を入れて
イヤホンを起こしてから鳴らす。

```powershell
[console]::beep(37, 700); [console]::beep(1047, 480); [console]::beep(660, 180); [console]::beep(1047, 180)
```

- `37Hz` = 人間にはほぼ聞こえない疑似無音として使用
- ビープとビープの間の `Start-Sleep` をそのまま使うとBTが切れて次の音が聞こえなくなる
  → 代わりに `37Hz` の短いビープを挟むことで接続を維持できる

## ショートカット

| 操作 | ショートカット |
|---|---|
| 再生 / 停止 | `Ctrl+Enter`（入力欄内からも動作） |
| 追加・リセットは未決定 | 候補: Alt系 or Ctrl系 |

ブラウザショートカットとの衝突に注意:
- `Ctrl+Shift+Delete` → Chromeの閲覧データ削除（使用禁止）
- `Ctrl+R` → リロード
- `Alt+D` → アドレスバー
- `Alt+Left/Right` → 戻る/進む
- `Ctrl+Space` → IME切り替え（Windows日本語環境）

安全な候補: `Alt+P`, `Alt+A`, `Alt+N`, `Alt+Delete`, `Ctrl+Shift+Enter`, `Ctrl+Shift+X`

## デザイン

- テーマ: ダーク、オシロスコープ/シンセ風
- アクセントカラー: `#f0a500`（アンバー）
- サブカラー: `#00c8d4`（シアン、sleep行に使用）
- フォント: `Share Tech Mono`（モノスペース）、`Zen Kaku Gothic New`（日本語）
- スキャンラインオーバーレイあり（`body::after`）

## 既知の制約・注意点

- `type="number"` inputのスピナー（▲▼）は非表示にしていない。必要なら CSS で `-webkit-appearance: none` を使う
- ドラッグ中に input をクリックしてもドラッグが始まらないよう `mousedown` の `stopPropagation` を設定済み
- `renderRows()` は全行を再描画するため、フォーカスが外れる。入力中に他の行が更新されることはないので実用上は問題ない
