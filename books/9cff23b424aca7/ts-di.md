---
title: "ts di lib researches"
---


# ts di lib researches



## はじめに

tsでDDDでchrome extensionを作りたいと思った場合、Repositoryパターンの実現で依存関係をどう解決するかという問題に突き当たりました。Javaの場合はDIツールが揃っているので息をするように`@Inject`や`Interface`で定形作業を行うだけですが、jsに関してはその辺りの知識が薄いことから、改めてDIライブラリについて調べる必要がありました。

今回は、一番お手軽な方法でRepositoryパターンが実装できる事を念頭に調べています。例えば、古代のDIのように手動bindを要求するものは除外しています。せっかく`implements interface`が使える言語なのでそれをクラスパスから拾って依存関係を自動解決してくれなくては困ります。無論、DIを使うシーンでinterfaceとその実装クラスがペアになって存在している、なんてことはあり得ないので手動bindが悪いとは言いませんが、今回の用途ではペアは必須条件なので、手動bindとペア自動解決が出来る事が最低条件になります。



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

typediは良さそうに見えて、tokenの解決が難しい。他のライブラリ同様に３つに分割するとコンテナへのtoken登録が失敗していて、 `ServiceNotFoundError`が発生してしまう。コメントアウトを外すように書き直し、 controllerにusecasewを直書きしてやって初めてtokenの解決が可能になる。DIApplication.tsもtypedi.tsもapplicationレイヤなので、十分TS的な工夫をして解決する余地はある。しかし、その方法は分からない。

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

https://github.com/inversify/InversifyJS/blob/master/README.md

star 7.7k の大人気ライブラリのようだけれど、歴史があるだけなんだろうと思う。 サンプルを見る限り `DIの仕組みをコードで全て表現したらこうなりました`的なやたらとDI的な記述が多い。出来ることは、property, constructor, setterな汎用インジェクションが出来るだけ。Javaから流れてきた身としてはアノテーションを用いないゴリゴリのDI記述は、忌避したい習慣です。


## 😫tsedio/tsed

https://github.com/tsedio/tsed

node.js用フルスタックフレームワークの中にDIモジュールがあり、JavaのSpringのような印象を受ける。但し、DIしたいだけなら重厚すぎなので実験対象ではないです。

