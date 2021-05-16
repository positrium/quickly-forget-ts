---
title: "iHerb Affiliates Link Maker MV3設計"
---

(書きかけ)

[MV3](https://developer.chrome.com/docs/extensions/mv3/intro/)が主流になりつつあるのでこれを読みながらvanilla jsで作成中。DDDとは別にMV3での繋げ方を学ばなければならない。

その後、webpackでtsをトランスパイルする流れ。 [参考](https://qiita.com/zaburo/items/26cb6dfb8a631ebbdfbd)

しかし、DDDで構成したtsをトランスパイルするもnamespace周りでトラブル続出。またトランスパイルターゲットはES5とcommonjsのような一般的？なものにしていても、namespace周りのトラブルは解決できないでいる。chromeそのものを使わない方法で問題の切り分け方法を検討する必要がある。
