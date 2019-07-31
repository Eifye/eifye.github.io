---
layout: post
title: StyleGANでアニメ顔画像合成
---

## やりたいこと

StyleGAN[公式実装](https://github.com/NVlabs/stylegan)で学習された、[アニメ顔生成モデル](https://www.gwern.net/Faces)をChainerでimportしていいように遊ぶ。
以下を試したい。

- 画像の遷移(補間)

    実験結果
    ![interpolate.gif]({{ site.baseurl }}/images/2019-7-31-interpolate.gif)

- 画像のスタイル合成

    実験結果
    ![mixing.gif]({{ site.baseurl }}/images/2019-7-31-style-mixing.png)

## 必要な作業

- TensorFlow(TF)のモデルをChainerにimport

  - TensorFlowはあまり使ったことがないので、個人的に使いやすいChainerで使えるようにする。

- 画像から潜在変数を算出するロジックの実装
  - ランダムに生成した潜在変数を元に画像を生成する仕組みのため、既にある画像で遊ぶには画像に対応する潜在変数を逆算する仕組みが必要

### モデルのimportについての覚書

[onnx](https://github.com/onnx/onnx)を利用して、「TFモデル変換」->「Chainerで読み込み」のAPIを呼ぶだけの想定だったが全くうまくいかなかった。

- モデル配布元で配布しているデータを、onnxで変換可能な形式にすることができず断念
   -> 仕方なくネットワークの重みの数値を直接取り出して使うことにした
  - ChainerのStyleGANは[PFN公式実装](https://github.com/pfnet-research/chainer-stylegan)を使ったが、実装やパラメータの細かい違いがあり多少手直しが必要だった。

### 潜在変数の算出についての覚書

モデル配布元では、[潜在変数算出の文献](https://arxiv.org/abs/1904.03189)についても触れているのでそのまま採用した。
文献に記載されている内容から調整した個所は以下の通り。

- 算出する潜在変数を下図の3.から2.に変更
-> 文献で採用している3.では、StyleMixingを実施した際に画像が崩れてしまい品質が良くなかったため。
![net.png]({{ site.baseurl }}/images/2019-7-31-net.png)
- Perseptual Lossで使用する特徴ベクトルの調整
  - 文献では実顔画像を対象にVGGの中間層4層分の特徴ベクトル誤差を係数1で合算してる
    -> 同じ条件では画像が崩れやすかったため、VGGの4，5層目のみを使用した
