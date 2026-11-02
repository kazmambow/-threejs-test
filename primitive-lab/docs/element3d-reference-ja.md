# Element 3D参照仕様とPrimitive Lab実装方針

調査日：2026-11-02（日本時間）

## 利用者の意図

UIの外観だけでなく、提示画像にあるGroup/Particle Replicator/Particle Look/Multi-Object/Deform/Baked Animationの機能を実装する。将来AEのエフェクトへ移植し、共通コマンド・ショートカット・自動操作からCG制作を行える構造とする。不明な製品仕様は利用者へ機能説明を求めず、公式資料を調べる。

## 参照資料と版の扱い

Video Copilot公式の https://www.videocopilot.net/docs/element2/ は https://video-copilot.gitbook.io/element2 へ転送される。公開マニュアルの表題はElement 3D V2.2。現在の製品ページもV2.2を案内。今回、最新パッチのビルド番号を確定する公式リリース履歴は取得できていない。V1の古いPDFにある「アニメーション未対応」等をV2.2仕様として用いない。

- マニュアル：https://video-copilot.gitbook.io/element2
- 製品：https://www.videocopilot.net/products/element2/
- Replicator：https://video-copilot.gitbook.io/element2/animating-3d-objects/particle-replicator
- Particle Look：https://video-copilot.gitbook.io/element2/animating-3d-objects/particle-look
- Baked Animation：https://video-copilot.gitbook.io/element2/animating-3d-objects/baked-animation-options
- Multi-Object：https://video-copilot.gitbook.io/element2/animating-3d-objects/multi-object-mode
- Deform：https://video-copilot.gitbook.io/element2/animating-3d-objects/deform
- Motion Blur：https://video-copilot.gitbook.io/element2/render-settings/motion-blur
- 公式解説：https://www.videocopilot.net/tutorials/playlist/element3d/advanced_3d_noise_and_deform

## Baked Animation：公式確認済み

外部で用意された形状アニメーションを再生する設定。OBJシーケンスへの公式言及あり。リグ・物理シミュレーション・モーションの自動生成を意味しない。

- Loop：先頭から末尾を繰り返す。
- Random Loop：ランダムな開始時刻から繰り返す。
- Freeze at End：一度再生する。末尾で保持する実装はUI名称に基づく解釈。
- Mirror：順再生と逆再生を繰り返す。
- Playback Speed：再生速度。遅い場合は補間用形状データが不足してカクつく可能性を公式が説明。
- Frame Offset：時間を前後へずらす。
- Frame Offset Random：複数インスタンスの時刻をランダムにずらす。
- Random Seed：ランダムな時刻ずれを変える。

Primitive Labには単一OBJ読み込みしかないため、UI追加だけでは動作しない。シーケンスデータ、時間指定、再生・停止、fps、フレーム解決、メモリ管理が必要。同じ時間・Seedなら同じ形状フレームを返す純関数として実装する。これらはPrimitive Labの設計案であり公式の内部仕様ではない。

## Replicator：公式確認済み

配置全体の位置・回転と、個体の見た目を区別する。Countは配置数。ShapeにはPoint、3D Grid、Plane、Sphere、3Dモデル頂点、Layer、Layer Gridなどが記載される。

OrderはRandom、Forward、Backward、Mirror。Repeatは候補の反復回数、Offsetは候補列の前後シフト。Scatterは配置のランダム分散、XYZは軸別分散。Position NoiseはEvolution/Amount/Scaleによる追加変動。Seedは乱数のパターンを変える。

Primitive Labでは利用者指定の均等ランダム配置を優先する。Seed・ノイズの内部計算式、Mirror端点を重複するかなどは文書に完全な数式がなく、独自仕様として明文化してテストする。

## Particle Look：公式確認済み

個体サイズ、サイズ乱数、個体回転、Group色Tint、Force Opacity、独立したSeed。Tintは暗い材質や反射の強い材質では見えにくい場合があると公式が説明。画像で確認したSize XYZ、Orientation、Rotation Random XYZ、Randomize Angle、Rotation Noiseも実装対象。文書の説明以上の数値同一性は保証しない。

## Multi-Object：公式確認済み

モデル内部の事前に分離されたパーツを個別に動かす機能。OBJ等を予め分離・破砕して書き出しておく必要があり、自動でモデルを破砕する機能ではない。

Size、Rotation、Random Rotation、Displace、Displace XYZ、Scatter、Scatter XYZ、Position Noise、Seedが説明される。Displaceはパーツの相対配置に基づく拡散で、Scatterとは別。ある軸に相対的な並びがなければ、その軸のDisplaceでは分散しない。

Primitive Lab：通常はフォルダを一塊にする既存仕様を維持。Multi-Objectを明示的に有効にした場合に限り内部メンバーへ個別操作を許可。OBJの子メッシュとフォルダのメンバーは階層が異なるため、対象単位をデータ上で区別する。

## Deform：確認済みと未確定

公式にTaper/Twist/Bend/Noiseの形状変形を確認。対象は単なるオブジェクト変換ではなくジオメトリ。画像からOrigin、Phase、軸、Direction、Symmetrical、Center、Deform Offset、Resetも必要。

NoiseのMethodは添付画像でNoise/Distortion/3D Distortionを確認。Intensity、Scale、Offset、Exponent、Clamp、XYZ Intensity、Evolution、Seed、Randomizeも画像で確認。

公開Deformページは説明が短く、Noise各方式の厳密な数式、Exponent/Clampの処理順、複数Deformの適用順は今回未確定。名称から推測した式を「Element 3Dと同じ」としない。公式解説で追える挙動と独自実装の差を記録する。

## 自動操作・AE移植の要件（Primitive Labの設計）

安定したパラメーターID、単位・範囲・初期値・型のスキーマ、Group/個体/パーツ/材質のIDを用意する。UIとショートカットは同じコマンドを呼ぶ。例：group.set、object.set、material.assign、timeline.seek、history.undo。複数変更はトランザクションとしてUndoを1件にまとめる。再生時間や乱数を壁時計・描画順へ依存させない。描画が不要な状態取得と、構造化された結果・エラーを用意する。ブラウザの実装だけでAEからの操作やキーフレーム対応が完成したとは扱わない。

## 現在の実装状態

v0.28はGroup1〜8と既存配列の整理まで。上記で追加対象にした全Replicator/Look/Noise/Deform/Baked Animationおよび共通コマンド基盤は未完成。調査済みと実装・動作確認済みを区別する。
