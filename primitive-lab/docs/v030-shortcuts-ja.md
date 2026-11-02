# v0.30 キーボード操作

アプリにフォーカスがある状態で使用。MacのAltはOption。

| キー | 操作 |
|---|---|
| Ctrl/⌘+K | コマンドを開き、入力を全選択 |
| Ctrl/⌘+Enter | コマンド実行 |
| Esc | 入力終了・ダイアログを閉じる |
| Alt+P / Alt+A | Properties / 配列 |
| Alt+O / Alt+M / Alt+F | 物体 / 材質 / フォルダの選択へ |
| ↑↓ | 対象の前後選択（物体はシーン一覧順） |
| Ctrl/⌘+D | 現在の対象を複製 |
| Delete / Backspace | 現在の対象を削除 |
| F2 | 対象の名前変更。Enter確定、Esc取消 |
| Alt+Enter | 選択材質を選択物体または選択フォルダへ適用 |
| Alt+N | フォルダ作成 |
| Alt+1〜8 | 該当Groupを開く |
| Alt+E / L / T | Environment / Lighting / Texture CH |
| Space | 再生・停止 |
| ←→ | 前後1フレーム |
| Ctrl/⌘+Z / Shift+Ctrl/⌘+Z | Undo / Redo |
| F1 | キー一覧 |

材質適用：Alt+Oで物体を選び、Alt+Mで材質を選択しAlt+Enter。フォルダ全体ならAlt+Fで対象フォルダを先に選ぶ。Alt+Oへ移るとフォルダの適用先選択を解除する。

Ctrl/⌘+D、Delete、F2は最後に選択した対象の種類に応じて物体・材質・フォルダへ作用する。フォルダ削除は解除であり内部物体はルートへ移る。フォルダ複製でオブジェクト上限5個を超える場合は拒否する。主な制作ショートカットは入力欄・IME変換・ダイアログ操作中に動作しない。Ctrl/⌘+Kは入力欄でも使えるがIME中は実行しない。

TabとShift+Tabでボタンや入力欄を巡回できる。ショートカットはアプリ内の割当でありOSやブラウザが先に処理するキーを強制的に奪うものではない。

共通コマンドにもui.tab、selection.focus、selection.next、folder.select、material.select、selection.duplicate、selection.delete、material.applySelection、ui.shortcutsを追加。state.getのuiSelection、schema.getのshortcutsで対象種類と割当を取得できる。

自動検証：MacのOption記号を含むキー判定、タブ、物体・材質巡回、文脈別複製、名前変更、入力中Deleteの抑止、IME中の抑止、フォルダ作成、Group選択、ヘルプ。

ブラウザ検証：ポインターのクリック・ドラッグなしで、コマンド実行、既存制作シーン復元、タブ切替、物体と材質の選択・複製・Undo、フォルダ選択、F2による名前編集、F1ヘルプを確認。
