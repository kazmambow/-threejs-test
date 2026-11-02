# v0.29 Group機能と操作コマンド

## 実装

Group 1〜8は既存のCh 1〜8に対応。通常はフォルダの相対配置を保った一塊を候補として配置する。

- Particle Replicator：既存Grid/円柱表面/単体に加えPoint、Plane、Sphere、3D Object頂点、Layer Alpha、Layer Grid。Particle Countは追加形状の配置数、GridはXYZ個数を用いる。
- Shape Options：均等ランダム、前方、後方、Mirror、Repeat、Offset。Mirrorは端点を重複する往復列。均等ランダムは利用者の指定を優先する。
- Scatter XYZ、Position NoiseのEvolution/Amount/Scale/XYZ倍率。
- Particle Look：個体サイズ、サイズ乱数、Size XYZ、個体回転、軸別回転乱数、Randomize Angle、Rotation Noise、色Tint、Force Opacity、独立Seed。
- Multi-Object：明示的に有効にした場合だけ、フォルダ内部メンバーやOBJ子メッシュの部品を個別にSize/Rotation/Displace/Scatter/Position Noiseで動かす。Displaceは部品中心と集合中心の相対位置に基づく。回転順6種に対応。
- Deform：Taper（Amount/Curve/Symmetrical/Origin）、Twist XYZ/Phase/Origin、Bend（Axis/Angle/Direction/Center/Origin）、Noise（Noise/Distortion/3D Distortion、Intensity/Scale/Evolution/Seed/Randomize、Offset/Exponent/Clamp/XYZ Intensity）、Deform OffsetとReset。
- Baked Animation：OBJ連番の実形状切り替え。Loop/Random Loop/Freeze at End/Mirror、Playback Speed、Frame Offset/Random、Seed。
- 再生/停止、秒指定、前後1フレーム移動、FPS。連番サンプルは24フレームの変形する箱をOBJとして生成し、通常の連番ローダーを通す。

## 操作

ヘッダーの「コマンド」、またはCtrl/⌘+KでJSONコマンドを開く。Ctrl/⌘+Enterで実行。

編集欄以外でSpace：再生/停止、左右矢印：1フレーム移動、Ctrl/⌘+D：選択物を複製、Delete/Backspace：削除、Ctrl/⌘+ZとShift+Ctrl/⌘+Z：Undo/Redo。入力中・IME中・ダイアログ中は制作ショートカットを実行しない。

```json
{"type":"group.set","group":1,"values":{"mode":"sphere","particleCount":20,"surfaceRadius":3,"particleSize":0.5,"positionNoiseAmount":0.3,"particleRotationY":45}}
```

```json
{"type":"group.set","group":1,"values":{"deformEnabled":true,"bendEnabled":true,"bendAxis":"Y","bendAngle":60}}
```

```json
{"type":"timeline.seek","time":1.25}
```

```json
{"type":"batch","commands":[{"type":"object.set","id":1,"values":{"position":{"x":1,"y":0,"z":0}}},{"type":"group.set","group":1,"values":{"forceOpacity":50}}]}
```

`schema.get`でパラメーターとコマンド一覧、`state.get`で現在状態を取得する。スクリプトからは`window.PrimitiveLab.execute(command)`。通常は同期結果`{ok,result}`または`{ok:false,error}`。sequence.importのみFile配列を受け取りPromiseを返す。

コマンド：group.set、object.add/set/select/delete/duplicate、folder.create/move/remove、material.add/set/assign/delete/duplicate、environment.set、lighting.set、camera.set、timeline.seek/play/pause、history.undo/redo、sequence.import/attach、batch、state.get、schema.get。

batchは成功時にUndoを1件にまとめ、失敗時はスナップショットへ戻す。ファイル操作・再生開始・履歴操作を含む非同期トランザクションは対象外。

## 連番の使用

適用先オブジェクトを選択し「OBJ連番」から複数のOBJを選ぶ。ファイル名の数値順で読み込む。読み込み時FPSがシーケンスの基準FPS。複数個体の開始時刻はGroupのFrame Offset Randomで変える。

連番はページ内のみ保持。120フレーム、1回32MB、セッション合計128MBの元ファイルサイズ上限。元ファイルサイズより実際のGPU/CPUメモリ使用量は大きい。素材を消してもUndo用の参照が必要なため、ページ内アセットはページ更新まで保持する。

## 差異と制限

Element 3Dの公開説明に基づく独自実装で、同じ数値で同一画像を生成する互換実装ではない。Noiseの格子補間・変形順序・Bend/Taperの式・Mirror端点などはPrimitive Lab仕様。

DeformはCPUで形状を生成し、同一Group・同一形状は配列内で共有する。多数の変形対象・大量OBJ連番のリアルタイム再生速度はまだ最適化途上。頂点分割が少ない形状は強い変形が角張る。フレーム間の頂点補間、スキニング、C4D読み込みは未対応。既存OBJのMTL自動読込も従来どおり未対応。

Layer系は現在のTexture CHのアルファを32×32でサンプリングする。AEの実レイヤーとは未接続。3D Objectは指定IDの頂点を配置源として使う。

ブラウザの秒・コマンド指定は利用可能だが、AE SDKブリッジ、AEネイティブのキーフレーム、AEライトレイヤー接続は未実装。

## 検証

自動：形状配置数、Repeat/Offset/Mirror、Seedと時間に対するノイズの再現性、再生4モードのフレーム解決、実形状切り替え、Taper/Twist/Bend/Noise、Tint/Opacity、Multi-Object、コマンド失敗・batch復元、既存Group1〜8・フォルダ・材質・IME・Alpha品質切替。

ブラウザ：起動、コマンドからの複合変更、配列・Bend/Taper・Tintの描画、連番24フレームの読み込み・再生・停止・時刻指定、個体の時間差、Ctrl+KとCtrl+Enterでの状態取得、描画エラーなしを確認。全パラメーターの全組み合わせ、Mac日本語IME、重い実制作データの性能は未検証。
