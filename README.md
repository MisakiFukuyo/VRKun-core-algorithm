# VRKun-core-algorithm

## 簡単な解説

画像を横に並べ、その上に、左右外側方向に若干だけずらした陰影のように見える輪郭を重ねることで、VRに見えます。

## 原理

人間が錯覚しているものと思われます。

## ffmpegによる実装(動画版)

引数

- file_path / 動画ファイルへのパス
- width / 元動画ファイルをスケーリングする時の幅(px単位)
- height / 元動画ファイルをスケーリングする時の高さ(px単位)
- sobel_movement / 輪郭の移動量(px単位)

sobel_movementの量は、最終出力動画の幅が1920px(左右片方ずつで960px)のものに対し、5px程度です

```
ffmpeg -y -i #{file_path} -filter_complex \
"[0] setsar=1/1,scale=w=#{width}:h=#{height},format=rgba,split [main_src][sobel_src];
[sobel_src] sobel=15:1:0 [sobel_green];
[sobel_green]format=gray,gblur=sigma=8:sigmaV=2,format=rgba[sobl_gray_src];
[sobl_gray_src] split [sobl_gray_merge_src][sobl_gray_inv_src];
[sobl_gray_inv_src] negate [sobl_gray_inv];
[sobl_gray_inv][sobl_gray_merge_src] alphamerge [sobl_gray];
[sobl_gray] split [sobel_left][sobel_right];
[main_src] split [main][right];
[main] pad=iw*2 [main_pad];
[main_pad][right] overlay=x=#{width} [main_horiz];
[main_horiz][sobel_left] overlay=x=#{-sobel_movement} [sobel_left_comp];
[sobel_left_comp][sobel_right] overlay=x=#{width+sobel_movement} [sobel_comp];
[sobel_comp] format=yuv420p" converted-media/#{file_name}
```

## より詳細な解説

以下のような手順の処理になります。

```
元となる動画/画像からソーベルフィルタで輪郭抽出
輪郭抽出したものをグレースケール化、ガウシアンぼかしを用いてぼかす
白黒を反転して、透過処理を行い、陰影画像とする
元となる動画/画像を左右に並べる
陰影画像を外側方向に左右にずらし、元となる動画/画像を左右に並べたものの上にオーバーレイする
```
