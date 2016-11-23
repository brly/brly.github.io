---
layout: post
title: Coursera machine learning
tags: [Coursera, ML, Octave]
---

別にこの日付に終わらせたわけではないのだが、Coursera の Andrew Ng という先生が教えてくれる
Machine learning のネット授業をクリアした。

<img src="/assets/posts/2016-09-21/congra.png">

Unlock certificate と表示されているが、お金を払えば修了証がもらえるらしく linkedin とかに貼ってアピール出来るらしい。

修了証はお金がかかるのだが、それが79ドルであり、若干高いなということと、授業毎の課題毎に毎回何かをするのがめんどくさいのでやらなかった。

そもそもあんまりお金をかけるつもりは無かったし。

授業の開始、終了させた日付は忘れてしまった。開始したとき、既に最初の課題の deadline が 2 週間かそこらに迫っており、毎週毎週課題に追われていた記憶がある。

#### 序盤 (week 1 ~ 4)

とにかく、最急降下法の印象しかない。各課題では fmincg (fminuc) を使い、コスト関数と微分を提供することで
損失が最小となるパラメータを求めていた。線形回帰、ロジスティック回帰の印象。
今見返してみると、Octave のチュートリアルなども行われていたようだ。まぁそりゃそうか。

ちなみに Octave は初めて利用した。慣れてくると使いやすい。

課題中では微分項などのパラメータを計算するために和を取ることがあり、
和の計算はいわゆる for 文を利用したくなるのだが、Octave では for よりもベクトル化して計算するほうが高速に行われるため、
それを意識する必要があった。慣れるまではパズルのような印象だった。別にベクトル化しなくても正しい値は計算されるのだが、
いかんせん処理が遅いので、ベクトル化が必要なシーンは多くあったようなイメージだった。

あくまで個人的な感想だが、この課題中における octave 上の計算のベクトル化が授業を通して一番難しかった。

それくらい、授業内ではなるべく平易な説明となるように Andrew 先生は努めていた。
特に、最初の最初ではパソコン自体の操作が不慣れな人向けへの配慮も見られた。
それでも、段階を追うごとに terminal 上での Octave の操作が必要となるので、
ターミナル操作が無理、という人は脱落してしまうだろうけれども...。それでも、とにかく平易になるように努めている。

#### 中盤 (week 5 ~ 8)

誤差逆伝播法、低バイアス高バリアンス、SVM、教師なし学習(Kmeans と PCA)。
割と知っている分野であったが、個人的にためになったのは week 6 の低バイアス高バリアンスの週だった。
集めるべきデータがあつまり、適用するアルゴリズムの選定、そしてアルゴリズムの中身まで理解していたとしても、
実際学習を行わせてみて、それがうまく行けばよいがうまくいかない場合にはうまく動くようにどうにかしなければならない。
その際の一般的取っ掛かりを week 6 では与えてくれた。

この頃は課題中のメモもいくつかとってあり、抜粋してみると

- SVM の課題、決定境界を良くするような C / σを選べという問題があるが OSX 上でうまくプロットされずパラメータを全探索して対応した
- PCA の課題、共分散行列を作ったら終わってしまった

課題の難易度にムラがあったようだ。

#### 終盤 (week 9 ~ 11)

外れ値検出、協調フィルタリング、確率的勾配降下法、OCR。

外れ値検出で初めて分布が導入された。

強調フィルタリングは推薦システムの構築を例として説明された。あんまりよく知らなかったのでためになった。

week 10 では確率的勾配降下法がメインで、データ量が大きな場合には微分項の和の計算が遅すぎるために導入されたのだ、と説明されている。

OCR では、OCR システムの構築を例とし、機械学習の結果をパイプラインのように繋いでいって最終的に結果を出力するものを作成する。
また、それらシステムの改善のための ceiling analysis についても説明された。これも知らなかったのでためになった。

終盤2週はプログラム課題がなくてかなり軽かった、というかやった実感がないのだが。

#### 感想

やっぱり、課題をやった週の方が記憶に残っている。
また課題をこなしていても、忙しかったので実装の方を急いでおりそこまで理解が出来なかったものもある。
それでも、とりあえず検索して答えを見つけるようなことはせずに出来たので、達成感はあった。

これ、大学生中にやっておけばよかったと思った。

あと、この Andrew 先生の他の授業があれば受けてみたいとも思ったのだが、多分なさそう。