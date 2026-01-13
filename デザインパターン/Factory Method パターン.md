# Factory Method パターン

## 1. パターンの目的

Factory Methodパターンは、**オブジェクトの生成をサブクラスに委ねる**ためのパターンです。

インスタンスの生成方法をスーパークラスで定義し、具体的なクラスのインスタンス化はサブクラスに任せることで、以下を実現します：

- **生成するクラスの決定を遅延**させる
- **newによる直接的なインスタンス生成を隠蔽**する
- **拡張に対して開いて、修正に対して閉じる**（Open-Closed Principle）

Template Methodパターンが「処理の流れ」をテンプレート化するのに対し、Factory Methodパターンは「**インスタンス生成の流れ**」をテンプレート化します。

テンプレートメソッドを使うことで、ファクトリの生成の流れが分かりやすくなる。
テンプレートメソッドパターンとファクトリを組み合わせたデザインパターン
ファクトリの中でテンプレートメソッドが実装されていて、
使用者は、テンプレートメソッドの流れに沿ってインスタンスが生成されるので、生成方法を意識する必要がない。
具体的な生成の方法は、ファクトリのサブクラスで実装する。ファクトリのスーパークラスは抽象メソッドを用意する。
ファクトリで生成する製品が増えた場合は、新たにサブクラスを新設するイメージ

------

## 2. どんな時に使うか

| 状況                                 | 説明                                         |
| ------------------------------------ | -------------------------------------------- |
| **生成するクラスが実行時に決まる**   | 設定や条件に応じて異なるクラスを生成したい   |
| **生成ロジックを一箇所にまとめたい** | newが散在すると変更時の影響範囲が広がる      |
| **フレームワークを作る**             | 具体的なクラスを知らずに生成処理を記述したい |
| **生成処理が複雑**                   | 単純なnewでは済まない初期化処理がある        |

------

## 3. 登場人物（クラス構成）

```
┌─────────────────────────────────────────────────────────────┐
│                      Framework 側                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ┌─────────────┐           ┌─────────────────────┐        │
│   │  Product    │           │      Creator        │        │
│   │  (抽象)     │◄─creates──│      (抽象)         │        │
│   ├─────────────┤           ├─────────────────────┤        │
│   │ + Use()     │           │ + FactoryMethod()   │ ←抽象  │
│   └─────────────┘           │ + Create()          │ ←テンプレート│
│         ▲                   └─────────────────────┘        │
│         │                             ▲                    │
└─────────│─────────────────────────────│────────────────────┘
          │                             │
┌─────────│─────────────────────────────│────────────────────┐
│         │        具体的な実装側        │                    │
├─────────│─────────────────────────────│────────────────────┤
│         │                             │                    │
│   ┌─────────────┐           ┌─────────────────────┐        │
│   │Concrete     │           │  ConcreteCreator    │        │
│   │Product      │◄─creates──│                     │        │
│   ├─────────────┤           ├─────────────────────┤        │
│   │ + Use()     │           │ + FactoryMethod()   │        │
│   └─────────────┘           └─────────────────────┘        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 各登場人物の役割

| 登場人物                            | 役割                                                |
| ----------------------------------- | --------------------------------------------------- |
| **Product（製品）**                 | 生成されるオブジェクトの抽象クラス/インターフェース |
| **ConcreteProduct（具体的製品）**   | Productの具体的な実装                               |
| **Creator（生成者）**               | FactoryMethodを定義し、生成の流れをテンプレート化   |
| **ConcreteCreator（具体的生成者）** | FactoryMethodを実装し、具体的な製品を生成           |

------

## 4. C#での実装例

### 基本例：IDカード生成システム（書籍の例をC#化）

csharp

~~~csharp
using System;
using System.Collections.Generic;

// ============================================
// Framework（フレームワーク側）
// ============================================

namespace Framework
{
    /// <summary>
    /// Product（製品）の抽象クラス
    /// 生成される「もの」の抽象的な表現
    /// </summary>
    public abstract class Product
    {
        /// <summary>
        /// 製品を使用する抽象メソッド
        /// </summary>
        public abstract void Use();
    }

    /// <summary>
    /// Creator（生成者）の抽象クラス
    /// 製品を生成する「工場」の抽象的な表現
    /// </summary>
    public abstract class Factory
    {
        /// <summary>
        /// 製品を生成するテンプレートメソッド
        /// 生成の流れを定義（Create → Register）
        /// </summary>
        public Product Create(string owner)
        {
            // FactoryMethodを呼び出して製品を生成
            Product product = CreateProduct(owner);
            // 生成した製品を登録
            RegisterProduct(product);
            return product;
        }

        /// <summary>
        /// FactoryMethod - サブクラスで具体的な生成処理を実装
        /// </summary>
        protected abstract Product CreateProduct(string owner);

        /// <summary>
        /// 製品を登録する処理（サブクラスで実装）
        /// </summary>
        protected abstract void RegisterProduct(Product product);
    }
}

// ============================================
// 具体的な実装側（IDカード）
// ============================================

namespace IdCard
{
    using Framework;

    /// <summary>
    /// ConcreteProduct - 具体的な製品（IDカード）
    /// </summary>
    public class IdCard : Product
    {
        // C#らしくプロパティで表現
        public string Owner { get; }
        public int CardNumber { get; }

        // コンストラクタはinternalにして、工場経由でのみ生成可能に
        internal IdCard(string owner, int cardNumber)
        {
            Owner = owner;
            CardNumber = cardNumber;
            Console.WriteLine($"[生成] {Owner}のカードを作成します（No.{CardNumber}）");
        }

        public override void Use()
        {
            Console.WriteLine($"[使用] {Owner}のカード（No.{CardNumber}）を使います");
        }

        public override string ToString() => $"[IdCard: {Owner}, No.{CardNumber}]";
    }

    /// <summary>
    /// ConcreteCreator - 具体的な生成者（IDカード工場）
    /// </summary>
    public class IdCardFactory : Factory
    {
        // 発行済みカードを管理するリスト
        private readonly List<IdCard> _registeredCards = new();
        private int _nextCardNumber = 1000;

        protected override Product CreateProduct(string owner)
        {
            // 具体的なIDCardのインスタンスを生成
            return new IdCard(owner, _nextCardNumber++);
        }

        protected override void RegisterProduct(Product product)
        {
            // 生成したカードを登録
            if (product is IdCard card)
            {
                _registeredCards.Add(card);
                Console.WriteLine($"[登録] {card.Owner}のカードを登録しました");
            }
        }

        /// <summary>
        /// 登録済みカードの一覧を取得
        /// </summary>
        public IReadOnlyList<IdCard> GetRegisteredCards() => _registeredCards.AsReadOnly();
    }
}

// ============================================
// クライアントコード
// ============================================

class Program
{
    static void Main()
    {
        // 工場のインスタンスを生成
        var factory = new IdCard.IdCardFactory();

        // 製品（IDカード）を生成
        // クライアントはProductとFactoryだけを知っていればよい
        Framework.Product card1 = factory.Create("田中太郎");
        Framework.Product card2 = factory.Create("鈴木花子");
        Framework.Product card3 = factory.Create("佐藤次郎");

        Console.WriteLine("\n--- カードを使用 ---");
        card1.Use();
        card2.Use();
        card3.Use();

        Console.WriteLine("\n--- 登録済みカード一覧 ---");
        foreach (var card in factory.GetRegisteredCards())
        {
            Console.WriteLine(card);
        }
    }
}
```

**実行結果：**
```
[生成] 田中太郎のカードを作成します（No.1000）
[登録] 田中太郎のカードを登録しました
[生成] 鈴木花子のカードを作成します（No.1001）
[登録] 鈴木花子のカードを登録しました
[生成] 佐藤次郎のカードを作成します（No.1002）
[登録] 佐藤次郎のカードを登録しました

--- カードを使用 ---
[使用] 田中太郎のカード（No.1000）を使います
[使用] 鈴木花子のカード（No.1001）を使います
[使用] 佐藤次郎のカード（No.1002）を使います

--- 登録済みカード一覧 ---
[IdCard: 田中太郎, No.1000]
[IdCard: 鈴木花子, No.1001]
[IdCard: 佐藤次郎, No.1002]
~~~