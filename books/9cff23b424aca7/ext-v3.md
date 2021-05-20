---
title: "iHerb Affiliates Link Maker MV3設計"
---

# typescript と webpack

まずは成果物へのリンクを記載しておきます。

https://github.com/positrium/iHerb-affiliates-link-maker

## typescript 開発環境を整える

成果物リンクの先にあるpackage.jsonをそのままnpm installすれば良い。

```yaml
{
  "devDependencies": {
    "@types/chrome": "^0.0.137",
    "@types/node": "^15.0.2",
    "copy-webpack-plugin": "^8.1.1",
    "ts-loader": "^9.1.2",
    "typescript": "^4.2.4",
    "webpack": "^5.37.0",
    "webpack-cli": "^4.7.0"
  },
  "dependencies": {
    "yarn": "^1.22.10"
  }
}
```

上記のpackageをインストールすれば開発環境は整います。これに加えて intellij idea ultimate を使っていますが intellij idea ce でもその他エディタを使っても問題ないと思います。

## webpack

手慣れてない人には「css や image を圧縮するやつ？」のように見えるwebpackですが、buildスクリプト相当でした。下記の通り、jsで書かれたbuildスクリプトで、（１）読み込みたいtsファイルentry（２）書き出したいディレクトリ定義（３）トランスパイラの設定（４）その他ファイルのコピー設定の４項目に分かれています。

```js
const path = require("path");
const CopyPlugin = require("copy-webpack-plugin");

module.exports = {
    mode: process.env.NODE_ENV || "development",
    entry: {
        background: path.join(__dirname, "src/background.ts"),
        options: path.join(__dirname, "src/Options.ts"),
        popups: path.join(__dirname, "src/Popup.ts"),
    },
    output: {
        path: path.join(__dirname, "dist"),
        filename: "[name].js",
    },
    module: {
        rules: [
            {
                test: /\.ts$/,
                use: "ts-loader",
                exclude: /(node_modules|dev)/,
            },
        ],
    },
    resolve: {
        extensions: [".ts", ".js"],
    },
    plugins: [
        new CopyPlugin({
            patterns: [
                path.resolve(__dirname, "src/interfaces/ui", "options.html"),
                path.resolve(__dirname, "src/interfaces/ui", "popup.html"),
                {from: "./public", to: "../dist"}
            ]
        })
    ],
    devtool: false
};
```

## ハマりどころ

tsのclassをjavaやその他のようにpackageで括るノリでnamespaceやmoduleで多階層に掘り下げると、トランスパイル後のjsで実ファイルと識別子（domain.affiliate.AffiliateIDのような文字列）の対応が外れてしまい、オブジェクトの参照が出来ない、ということが起こりました。

未検証ですが、moduleよりもnamespaceが誤作動の原因だった可能性があります。後に検証する予定です。
