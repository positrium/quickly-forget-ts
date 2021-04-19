---
title: "はじめに"
---

# 参考

- https://future-architect.github.io/typescript-guide/
- https://www.jetbrains.com/ja-jp/idea/features/editions_comparison_matrix.html

# テーマ

下記 Amazon Affiliates Link Maker を模して、 iHerb Affiliates Link Maker をTypeScriptだけで作って見ようと思います。

- https://chrome.google.com/webstore/detail/amazon-affiliates-link-ma/egkocgaccgbefacjmdnlfmlmccgacjck

ただし、google chrome extension 的な制約(MV3)は考慮しますが、元コードであるAmazon Affiliates Link Makerの設計思想は考慮しません。元コードは Manifest V2で構成されていますが現在のManifestはV3主体なので、ファイル構成や要件外で解決可能な課題の範囲が異なります。

本稿の主目的が「JavaScriptでもclassによる整理を行うこと」であるので、単純にTypeScriptに置き換えるだけとはしたくありません。規模は小さいながらもmodelが存在しているのでDDDで進めていこうと思います。
