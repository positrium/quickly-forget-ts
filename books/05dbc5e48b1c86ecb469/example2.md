---
title: "class"
---

# まずはclassの構文

## java

```java
package dev.zenn.foo;

import dev.zenn.hoge.MovableHands;

/**
 * contract marker interface
 */
public interface dev.zenn.hoge.MovableHands{
  }

/**
 * contract interface
 */
public interface Communicatable extends MovableHands {
  public String shakeHands();
}

/**
 * abstract class
 */
public abstract class Ahh {
  public abstract String sayAhhh();
}

/**
 * class
 */
public class Sample extends Ahh implements Communicatable {
  /**
   * immutable inner value
   */
  private final String value;

  public Sample(String value) {
    this.value = value;
  }

  /**
   * class method
   */
  public static String sayLove() {
    return "Love ! <3";
  }

  /**
   * instance method
   */
  public String saySomething() {
    return this.prependCap();
  }

  /**
   * private instance method
   */
  private String prependCap() {
    return "wow wow " + this.value;
  }

  /**
   * concrete method
   */
  @Override
  public String sayAhhh() {
    return "Ahhh!!!!";
  }

  /**
   * interface contract method
   */
  @Override
  public String shakeHands() {
    return "OK, hug next!";
  }
}

//  Sample.sayLove(); // returns 'Love ! <3'
//  Sample s1 = new Sample("hoo!");
//  s1.saySomething(); // returns 'wow wow hoo!'
//  s1.sayAhhh(); // returns 'Ahhh!!!!'
//  s1.shakeHands(); // returns 'OK, hug next!'
```

## typescript

基本的に名前空間に対するJavaのpublicと対になっているのが、export。外から参照した場合に「見ることができる」という読み替えのように感じる。

使い方の面で、namespaceをdotで区切ると、vanilla javascriptが入れ子になって読みづらい。ts>jsのコードを読むタイミングは限られるが、デバッグ時に消耗したくないなら読みづらさを増やす行動は慎むべきだろうか。

他の差異は、親クラスの引数なしコンストラクタであっても暗黙的に呼び出されることは無いようなので、手動で書いてやらなければならないところ。intellijを使っていればアラートしてくれるので気を張る必要はない。

```ts
namespace dev.zenn.hoge {
  /**
   * contract marker interface
   */
  export interface MovableHands {
  }
}
namespace dev.zenn.foo {

  import MovableHands = dev.zenn.hoge.MovableHands;

  /**
   * contract interface
   */
  export interface Communicatable extends MovableHands {
    shakeHands(): string;
  }

  /**
   * abstract class
   */
  export abstract class Ahh {
    abstract sayAhhh(): string;
  }

  /**
   * class
   */
  export class Sample extends Ahh implements Communicatable {
    /**
     * immutable inner value
     */
    private readonly value: string;

    constructor(value) {
      super(); // Can't be implicit
      this.value = value;
    }

    /**
     * class method
     */
    public static sayLove(): string {
      return "Love ! <3";
    }

    /**
     * instance method
     */
    public saySomething(): string {
      return this.prependCap();
    }

    /**
     * private instance method
     */
    private prependCap(): string {
      return "wow wow " + this.value;
    }

    /**
     * concrete method
     */
    public sayAhhh(): string {
      return "Ahhh!!!!";
    }

    /**
     * interface contract method
     */
    public shakeHands(): string {
      return "OK, hug next!";
    }
  }
}

import Sample = dev.zenn.foo.Sample;

Sample.sayLove();
const s1 = new Sample("hoo!");
s1.saySomething();
s1.sayAhhh();
s1.shakeHands();
```

## vanilla js

vanilla jsでは、s1.prependCap()にアクセスできそうだが、intellijでは候補に出てこない。ts>jsを実行した時に一緒に生成されたソースマップがアクセス可否を制御しているのかな？

```javascript
var dev;
(function (dev) {
    var zenn;
    (function (zenn) {
        var foo;
        (function (foo) {
            /**
             * abstract class
             */
            class Ahh {
            }
            foo.Ahh = Ahh;
            /**
             * class
             */
            class Sample extends Ahh {
                constructor(value) {
                    super(); // Can't be implicit
                    this.value = value;
                }
                /**
                 * class method
                 */
                static sayLove() {
                    return "Love ! <3";
                }
                /**
                 * instance method
                 */
                saySomething() {
                    return this.prependCap();
                }
                /**
                 * private instance method
                 */
                prependCap() {
                    return "wow wow " + this.value;
                }
                /**
                 * concrete method
                 */
                sayAhhh() {
                    return "Ahhh!!!!";
                }
                /**
                 * interface contract method
                 */
                shakeHands() {
                    return "OK, hug next!";
                }
            }
            foo.Sample = Sample;
        })(foo = zenn.foo || (zenn.foo = {}));
    })(zenn = dev.zenn || (dev.zenn = {}));
})(dev || (dev = {}));

var Sample = dev.zenn.foo.Sample;
Sample.sayLove();
const s1 = new Sample("hoo!");
s1.saySomething();
s1.sayAhhh();
s1.shakeHands();
//# sourceMappingURL=TsSample.js.map
```
