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
    * Extract Sharpest Frame
        * https://github.com/Kotohibi/Extract_sharpest_frame
        * BOOTH版 Windows Binary Edition: https://kotohibi-cg.booth.pm/
* Metashape 360 SfMからCOLMAP形式のCubemap変換ツール
    * Metashape 360 to COLMAP Converter
        * https://github.com/Kotohibi/Metashape_360_to_COLMAP_plane
        * BOOTH版 Windows Binary Edition: https://kotohibi-cg.booth.pm/

* (任意) 3DCGの実寸を推定する為の追加ライセンス
    * Metashape 360 to COLMAP ConverterはAprilTagという二次元マーカーを利用して、3DGSの実寸を推定する機能があります。利用には追加ライセンスが必要です。AprilTagの利用方法は以下の記事で説明しています
    * Add-on Real Sacle 3DGS with AprilTag https://kotohibi-cg.booth.pm/items/8323677
    * 英語版 : https://x.gd/CoWJA
    * 日本語版 : https://x.gd/Isahb

# 手順の説明方法について
以降の手順は処理フローをメインに解説します。各ツールの細かい使い方は以下の手順書を参考にしてください。
*  [[JP]Easy&Fast 3D Gaussian Splatting workflow with 360 Camera]([JP]Easy&Fast%203D%20Gaussian%20Splatting%20workflow%20with%20360%20Camera.md)
# 動画、静止画の準備
本記事では以下の映像素材の例で3DGS制作を解説します

|カメラタイプ|機種|形式|目的|
|---|---|---|---|
|全天球カメラ|Insta 360 X3|動画|シーン全体の再現用|
|平面カメラ|ドローン搭載カメラ|動画|注目領域の再現用|

# 全天球動画から静止画抽出とマスク生成
## カスタムマスクの準備
この全天球カメラはドローンにマウントされている為、上半分にドローン機体が映り込みます。このような場合は上半分をマスクした画像を作成し、カスタムマスクに登録します。
指定したカスタムマスクはYOLOやSAM3マスクに自動で融合されます。
カスタムマスクはオリジナル動画と同じ画サイズの必要があります。

|画像|カスタムマスク|
|---|---|
|![](./images2/output_frame_00100.png)|![](./images2/custom_mask.png)|

## SAM3マスクの準備

# 平面動画から静止画抽出とマスク生成




