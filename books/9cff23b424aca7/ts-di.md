---
title: "ts di lib researches"
---


# ts di lib researches



## はじめに

この記事は、2021年頃の公開情報を元に実験結果をまとめたものです

tsでDDDでchrome extensionを作りたいと思った場合、Repositoryパターンの実現で依存関係をどう解決するかという問題に突き当たりました。Javaの場合はDIツールが揃っているので息をするように`@Inject`や`Interface`で定形作業を行うだけですが、jsに関してはその辺りの知識が薄いことから、改めてDIライブラリについて調べる必要がありました。

今回は、DDD的かつ一番お手軽な方法でRepositoryパターンが実装できる事を念頭に調べています。JSはJavaと違ってクラスパスが存在しないからか、自動bindは不得意のようです。よって、手動bindであっても実験対象になりました。対象外は「記述量が多すぎる」「DIの為に新たなファイルを要求する」「DIのためだけじゃないライブラリ」などです。



## 見つけたライブラリ

- [microsoft/tsyringe](https://github.com/microsoft/tsyringe) 2.2k
- [mgechev/injection-js](https://github.com/mgechev/injection-js) 985
- [typestack/typedi](https://github.com/typestack/typedi) 2.4k
- [inversify/InversifyJS](https://github.com/inversify/InversifyJS) 7.7k
- [tsedio/tsed](https://github.com/tsedio/tsed) 1.7k


## 実験構成

下記の通り、DDDを模した構成でdomain以下は原則アノテーションをつけない。具体的にDIに依存するのは、`infrastructure`と`application`以下のみ。

```
src/main/ts/
├── application/
│   ├── injectionjs/
│   │   ├── injectionjs.html
│   │   ├── DIApplication.ts ... usecase
│   │   └── injectionjs.ts   ... controller
│   ├── tsyringe/
│   │   ├── tsyringe.html
│   │   ├── DIApplication.ts
│   │   └── tsyringe.ts
│   └── typedi/
│       ├── typedi.html
│       ├── DIApplication.ts
│       └── typedi.ts
├── domain/
│   ├── IItemRepository.ts
│   └── Item.ts
└── infrastructure/
    ├── ItemRepositoryInjectionjsImpl.ts
    ├── ItemRepositoryTsyringeImpl.ts
    ├── ItemRepositoryTypediImpl.ts
    └── index.ts
```

## ⭕ microsoft/tsyringe

ms製軽量DIコンテナ

[https://github.com/microsoft/tsyringe#example-with-interfaces](https://github.com/microsoft/tsyringe#example-with-interfaces)



- 手動bind必須。
- interfaceにDIできる
- Constructor injection only



constructor injection のみだが、特に問題はないと思う。見栄えの面では property injectionが一番キレイだけど、DIを使わないとテスト出来ないようなクラスになってしまうのでアンチパターンだと思うし、setter injectionはまあconstructor injectionできればオプショナルだと思う。

### 具象クラス定義

```ts
// ItemRepositoryTsyringeImpl.ts

import {IItemRepository} from "../domain/IItemRepository";
import {Item} from "../domain/Item";

export class ItemRepositoryTsyringeImpl implements IItemRepository {
  find(uuid: string): Item {
    return new Item();
  }

  load(): Array<Item> {
    return new Array<Item>();
  }

  save(item: Item): void {
    console.log("save item !!");
  }
}
```

### 依存関係定義

```ts
// usecase
// DIApplication.ts

import {inject, injectable} from "tsyringe";
import {IItemRepository} from "../../domain/IItemRepository";
import {Item} from "../../domain/Item";

@injectable()
export class DIApplication {

  constructor(@inject('IItemRepository') private repository?: IItemRepository) {
  }

  exec() {
    this.repository?.save(new Item());
  }
}
```

### コンテナ登録　と　usecase単位での依存性解決

```ts
// controller
// tsyringe.ts

import "reflect-metadata";
import {container} from "tsyringe";

import {IItemRepository} from "../../domain/IItemRepository";
import {ItemRepositoryTsyringeImpl} from "../../infrastructure";
import {DIApplication} from "./DIApplication";



container.register("IItemRepository", {
  useClass: ItemRepositoryTsyringeImpl
});

const application = container.resolve(DIApplication);
application.exec();
```



## ⭕ mgechev/injection-js

Angular-jsからスピンアウトしたDIコンテナ

https://github.com/mgechev/injection-js


- 手動bind必須

tsyringe同様に３分割しても使えるので採用しようかなと思わせられる軽量DIコンテナだと思う。けれど、ReflectiveInjectorを使って生成する方法は、tsyringeに比べて可読性が落ちる気がする。

### 具象クラス定義

```ts
// ItemRepositoryInjectionjsImpl.ts

import {IItemRepository} from "../domain/IItemRepository";
import {Item} from "../domain/Item";

export class ItemRepositoryInjectionjsImpl implements IItemRepository {
  find(uuid: string): Item {
    return new Item();
  }

  load(): Array<Item> {
    return new Array<Item>();
  }

  save(item: Item): void {
    console.log("save item !!");
  }
}
```

### 依存関係定義

2022-06-19: なんとなく書き方を間違えているような気がする。

```ts
// usecase
// DIApplication.ts

import {Inject} from "injection-js";

import {IItemRepository} from "../../domain/IItemRepository";
import {ItemRepositoryInjectionjsImpl} from "../../infrastructure";
import {Item} from "../../domain/Item";

export class DIApplication {

  constructor(private repository: IItemRepository) {
    this.repository = repository;
  }

  static get parameters() {
    return [new Inject(ItemRepositoryInjectionjsImpl)];
  }

  exec() {
    this.repository?.save(new Item());
  }
}
```
### usecase単位での依存性解決

2022-06-19: この時点でImplを参照しちゃったら、DIの意味がないんだが・・。書いた頃はやっつけだったかもしれない。

```ts
// controller
// injectionjs.ts

import "reflect-metadata";
import {ReflectiveInjector} from "injection-js";

import {ItemRepositoryInjectionjsImpl} from "../../infrastructure";
import {DIApplication} from "./DIApplication";


const injector = ReflectiveInjector.resolveAndCreate([ItemRepositoryInjectionjsImpl, DIApplication]);
const application = injector.get(DIApplication)
application.exec();
```


## ❌ typestack/typedi

https://github.com/typestack/typedi/tree/develop/docs



- 自動bind
  - @Service({id:string}) が手動bindの糖衣構文になってる
- interfaceにDIできる
- inject対象
    - property
    - constructor
    - setter?

typediは良さそうに見えて、tokenの解決が難しい。他のライブラリ同様に３つに分割するとコンテナへのtoken登録が失敗していて、 `ServiceNotFoundError`が発生してしまう。コメントアウトを外すように書き直し、 controllerにusecaseを直書きしてやって初めてtokenの解決が可能になる。DIApplication.tsもtypedi.tsもapplicationレイヤなので、十分TS的な工夫をして解決する余地はある。しかし、その方法は分からない。

### 具象クラス定義

```ts
// ItemRepositoryTypediImpl.ts

import {IItemRepository} from "../domain/IItemRepository";
import {Item} from "../domain/Item";
import {Service} from "typedi";

@Service({id: 'item.repository'})
export class ItemRepositoryTypediImpl implements IItemRepository {
  find(uuid: string): Item {
    return new Item();
  }

  load(): Array<Item> {
    return new Array<Item>();
  }

  save(item: Item): void {
    console.log("save item !!");
  }
}
```

### 依存関係定義

```ts
// usecase
// DIApplication.ts

import {Inject, Service} from "typedi";

import {IItemRepository} from "../../domain/IItemRepository";
import {Item} from "../../domain/Item";

@Service({id: 'DIApplication'})
export class DIApplication {

  @Inject('item.repository')
  repository?: IItemRepository;

  exec() {
    this.repository?.save(new Item());
  }
}
```
### usecase単位の依存性解決

2022-06-19: 結局解決方法は探していない。

```ts
// controller
// typedi.ts

import {Container} from 'typedi';
import {DIApplication} from "./DIApplication";

// import {Inject, Service} from "typedi";
//
// import {IItemRepository} from "../../domain/IItemRepository";
// import {Item} from "../../domain/Item";
//
// @Service({id: 'DIApplication'})
// export class DIApplication {
//
//   @Inject('item.repository')
//   repository?: IItemRepository;
//
//   exec() {
//     this.repository?.save(new Item());
//   }
// }

let application = Container.get<DIApplication>('DIApplication');
application.exec();
```

## 😫inversify/InversifyJS

[https://github.com/inversify/InversifyJS/blob/master/README.md](https://github.com/inversify/InversifyJS/blob/master/README.md)

star 7.7k（2022年には9k over）の大人気ライブラリのようだけれど、歴史があるだけなんだろうと思う。 サンプルを見る限り `DIの仕組みをコードで全て表現したらこうなりました`的なやたらとDI的な記述が多い。出来ることは、property, constructor, setterな汎用インジェクションが出来るだけ。Javaから流れてきた身としてはアノテーションを用いないゴリゴリのDI記述は、忌避したい習慣です。

2022-06-19: microsoft/tsyringeでコンテナ登録するコードと比較すると、なんだか自由が無さそうに見えた。上の方で書いているライブラリは、interfaceと具象クラスが一対になってないDIApplicationでもコンテナに識別子と一緒に登録しておけば、内包する依存性を解決してくれてインスタンスを返してもらえる。しかし、こちらのライブラリは必ず一対で登録しなければならなさそう。その点が自由が無さそうと思った所以である。

```ts
import { Container } from "inversify";
import { IItemRepository } from "./interfaces";
import { ItemRepositoryInversifyJsImpl } from "./entities";

const myContainer = new Container();
myContainer.bind<IItemRepository>(Symbol.for("IItemRepository")).to(ItemRepositoryInversifyJsImpl);

const itemRepositoryInversifyJsImpl = myContainer.get<IItemRepository>(Symbol.for("IItemRepository"));
```


## 😫tsedio/tsed

https://github.com/tsedio/tsed

node.js用フルスタックフレームワークの中にDIモジュールがあり、JavaのSpringのような印象を受ける。但し、DIしたいだけなら重厚すぎなので実験対象ではないです。


