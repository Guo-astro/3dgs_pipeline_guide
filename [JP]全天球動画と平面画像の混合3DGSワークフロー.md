# はじめに
本資料の読者は以下のユーザーを対象にしています
* 以下の手順書を理解している
  * [[JP]Easy&Fast 3D Gaussian Splatting workflow with 360 Camera]([JP]Easy&Fast%203D%20Gaussian%20Splatting%20workflow%20with%20360%20Camera.md)
* 全天球カメラを用いた3DGSの制作フローを一通り理解している
* さらにステップアップして高品位な3DGSを制作したい
  

# 概要
OSMO360やATATA360等の全天球カメラを用いたい3DGS制作は手軽な反面品質面には限界がありました。一方ミラーレスカメラ等の平面画像を用いたワークフローは品質が高いものの撮影の手間が大きくシーン全体の再現が難しい課題がありました。
本ワークフローは全天球カメラと平面カメラを用いた混合ワークフローを提案するものです。
シーン全体を全天球カメラで撮影、注目領域は平面カメラで精緻に撮影し3DGSの結果を向上させることを主眼に置きます。

# 参考

# 必要な物

* 全天球カメラ
  * DJI OSMO 360
  * DJI AVATA 360
  * Insta 360
  * etc...
* 平面カメラ
  * ミラーレスカメラ
  * アクションカム
  * etc... 

* ハイエンドPCとNVIDIA GPU
    * 3DGSの学習には高性能なGPUが必要になります。特にVRAMは多い方が良いです。最低でも12GB以上のVRAMを搭載したGPUを推奨します。
* Metashape Standard (Professional版は未サポート)
    * 全天球画像を直接SfM可能で、非常に高速&ロバストです
    * https://www.agisoft.com/features/standard-edition/

* 3D Gaussian Splatting software
    * Postshot: https://www.jawset.com/
    * LichtFeld Studio(LFS): https://github.com/MrNeRF/LichtFeld-Studio
    * Brush: https://github.com/ArthurBrussee/brush
* 動画から静止画切り出しツール
    * Extract Sharpest Frame (別名:360 Extractor)
        * https://github.com/Kotohibi/Extract_sharpest_frame
        * BOOTH版 Windows Binary Edition: https://kotohibi-cg.booth.pm/
* Metashape 360 SfMからCOLMAP形式のCubemap変換ツール
    * Metashape 360 to COLMAP Converter (別名:360 MCConverter)
        * https://github.com/Kotohibi/Metashape_360_to_COLMAP_plane
        * BOOTH版 Windows Binary Edition: https://kotohibi-cg.booth.pm/

* (任意) 3DCGの実寸を推定する為の追加ライセンス
    * Metashape 360 to COLMAP ConverterはAprilTagという二次元マーカーを利用して、3DGSの実寸を推定する機能があります。利用には追加ライセンスが必要です。AprilTagの利用方法は以下の記事で説明しています
    * Add-on Real Sacle 3DGS with AprilTag https://kotohibi-cg.booth.pm/items/8323677
    * 英語版 : https://x.gd/CoWJA
    * 日本語版 : https://x.gd/Isahb

# 手順の説明方法について
以降の手順は処理フローをメインに解説します。ツールの細かい使い方、オプション詳細は以下の手順書を参考にしてください。
*  [[JP]Easy&Fast 3D Gaussian Splatting workflow with 360 Camera]([JP]Easy&Fast%203D%20Gaussian%20Splatting%20workflow%20with%20360%20Camera.md)
# 動画、静止画の準備
本記事では以下の映像素材の例で3DGS制作を解説します<br>
**事前に全天球カメラと平面カメラ間の露出や色調は現像ソフト等で合わせておくことを強く推奨します。
露出、色調が大きく異なる場合3DGSの結果は悪化します。**

|カメラタイプ|機種|形式|目的|
|---|---|---|---|
|全天球カメラ|Insta 360 X3|動画|シーン全体の再現用|
|平面カメラ|ドローン搭載カメラ|動画|注目領域の再現用|

# 全天球動画から静止画抽出とマスク生成
## カスタムマスクの準備（任意）
本事例は全天球カメラはドローン下部にマウントされている為、上半分にドローン機体が映り込みます。このような場合は上半分をマスクした画像を作成し、カスタムマスクに登録します。
指定したカスタムマスクはYOLOやSAM3マスクに自動で融合されます。
カスタムマスクはオリジナル動画と同じ画サイズの必要があります。
DJI AVATA360等の全天球カメラドローンを使用する場合はこの手順は不要になります。

|画像|カスタムマスク|
|---|---|
|![](./images2/output_frame_00100.png)|![](./images2/custom_mask.png)|

## 動画を読み込む
360 Extractorで全天球動画を読み込みます。本ツールは複数動画をバッチ処理可能です。
* "複数動画の出力を１つにフォルダにまとめる"オプションをオンにすると、動画ファイル毎に連番の接頭辞が付与され静止画抽出され一つのフォルダに格納されます。マスクも同様に一つのフォルダに格納されます。
  * ２つの動画ファイル処理時の静止画ファイル名例
    * 動画1|マスク : 001_[出力ファイル名パターン], ...
    * 動画2|マスク : 002_[出力ファイル名パターン], ...
* 静止画抽出設定は共通設定です。動画ファイル毎に異なる条件で抽出したい場合は、抽出処理を複数回に分けてください。
* その場合、Output folderを同一にして、Output patternを変えることで、抽出ファイルの上書き保存を回避でき、また同一フォルダにまとめることが可能です。
* "Start", "End"で処理対象のタイムコードを限定でき、テストランに有効です。
<img src="./images2/extractor_1.png" width="80%">

## SAM3マスク設定
本ツールは、二通りのSAM3マスク設定をすることが可能です。
"SAM3 Mask 1", "SAM3 Mask 2"タブがその機能に相当します。
カメラアラインメント用と3DGS学習用のマスクを分けることで高品質な3DGSを生成することができます。
|用途|説明|マスクプロンプト例|マスク例|
|---|---|---|---|
|カメラアラインメント|可能な限り動体をマスク|sky, cloud, tree, vehicle, drone, people|![](./images2/sam3_1.png)|
|3DGS学習用|最低限の動体をマスク|drone, people|![](./images2/sam3_2.png)|

# 平面動画から静止画抽出とマスク生成
次に平面動画から静止画抽出とマスク生成を行います。
本記事では平面動画はドローン搭載カメラで１ファイルとします。
静止画とマスクは全天球処理結果のフォルダにまとめたい為、同じ出力フォルダを指定します。
また上書き保存を避ける為、[出力ファイル名パターン]を以下のように被らないように変更します。<br>

|出力ファイル名パターン|
|---|
|`003_output_frame_%05d.png`|

<img src="./images2/extractor_planar_1.png" width="80%">

## Tips
Extract Sharpest Frameは動画だけなく、静止画からマスクを作成することも可能です。
その場合は以下の[静止画マスクモード]をオンにして、静止画フォルダを選択してください。
<img src="./images2/mask_only_mode.png" width="80%">
#####
以下執筆中



