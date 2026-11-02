# v0.31 配置上限と全体設定

## 配置・Group

- 3D配列のXYZ個数：各1〜100。X=100,Y=1,Z=1なら100配置、100×100×100なら100万配置。
- この版の総実行上限は20,000配置。超過時は変更前に拒否し、要求数と理由を表示する。黙って間引かない。
- Group/チャンネルは1〜5へ変更。共通テクスチャは10chを維持。
- 元モデルの登録数上限は従来どおり5。配置数とは別。
- Scatter（全軸）をScatter XYZの先頭へ移動。全軸値に軸別値を加算した範囲で分散。
- 同一geometry/materialを共有する32個以上の候補をInstancedMeshにまとめる。頂点数そのものは減らないが描画命令を削減する。
- 半透明の品質優先材質は個別描画。速度優先ではInstancingを許可し、個体間の透明面ソートは保証しない。
- レイキャストのinstanceIdから元オブジェクトIDを解決し、材質のドロップ先を維持する。

Element 3Dの実行可能な配置数と同じことを保証する変更ではない。公開Replicator説明にハードウェア共通の最大実行数は明記されていない。ブラウザ・Three.js・GPU・形状・テクスチャ・透明・影によって限界は異なる。20,000は本版の実行上限で、快適なFPSを保証する値ではない。

## World Transform

位置XYZ、アンカーXYZ、回転XYZ、World Scale、除外Group1〜5。変換はT(position+anchor)・R・S・T(-anchor)をチャンネル全体へ適用。除外Groupは自分のGroup変換を維持する。

ブラウザ版World NullをCreateすると軸ヘルパーとコントローラーを作成。Nullの位置・回転・ScaleでWorldを操作できる。Unlinkで現在の値をWorld側へ保持し、以後のNull編集を切り離す。Deleteでヘルパーを削除する。AEのネイティブなヌルレイヤーはまだ作成しない。

## Custom Layers

- Custom Texture Maps：既存Texture CHと同一の10入力。PNG/JPEG/EXRの参照、既読画像への切替・解除。材質とLayer系Replicatorから参照できる。セッション内保存。
- Custom Text and Masks：SVGパスを10入力で管理。SVGの曲線と穴を取り込み、サイズを正規化して押し出しモデルを作成する。文字は事前にアウトライン化したSVGを使う。生のAEテキストレイヤーや任意フォントの文字入力には未接続。

## Render Settings

描画解像度100/75/50/25%、Pixel Ratio 1〜2、Instancing ON/OFF、影ON/OFF、影解像度64〜2048、露出。

Element 3Dの全レンダリングエンジンを実装したという意味ではない。AO、モーションブラー、DOF、反射の追加方式、Unified Renderの複数エフェクト連携は本追加に含まない。

## Output

Full Render、Wireframe、Normals、Depth。背景透過では床・グリッド・ヘルパーを除外。PNG書き出し。Depthはカメラの深度による出力で、線形距離の数値パスやEXR保存は未対応。

## コマンド・キー

world.set、world.null.create/set/unlink/delete、render.set、output.set、output.export、texture.assign、path.import/createを追加。schema.getで全体設定のキー・配置上限を取得する。

```json
{"type":"group.set","group":1,"values":{"mode":"grid","countX":100,"countY":1,"countZ":1,"spacing":0.2,"particleSize":0.1}}
```

```json
{"type":"world.set","values":{"rotationY":30,"scale":1.5,"excluded":[2]}}
```

```json
{"type":"render.set","values":{"resolution":0.5,"instanceBatching":true}}
```

```json
{"type":"output.set","values":{"mode":"full","transparent":true}}
```

```json
{"type":"output.export","filename":"Primitive-Lab.png"}
```

Alt/Option+W：World、+C：Custom Layers、+R：Render Settings、+U：Output。Groupへの移動キーはAlt/Option+1〜5。

## 検証

自動：100インスタンスへの集約、元物体ID保持、上限超過を変更前に拒否、Group5上限、World変換・除外・Null解除、解像度・出力状態、SVG押し出しジオメトリ、Undo復元。既存Look/Deform/連番/材質の回帰テストも実行。

ブラウザ：100個のキューブで1202三角形・床とグリッド込み3 Draw calls、10×10への変更、World回転とGroup2除外、穴のあるSVGの読み込み・押し出し、75%描画、Normals出力を確認。20,000配置のGPU負荷・任意の複雑なSVG・重いモデルの限界FPSは未検証。

## 公式参照

https://video-copilot.gitbook.io/element2/animating-3d-objects/particle-replicator

https://video-copilot.gitbook.io/element2/animating-3d-objects/world-transform

https://video-copilot.gitbook.io/element2/extruding-3d-text-and-mask-shapes/extruding-text
