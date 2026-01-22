# Strategyパターン

## 1. パターンの目的

アルゴリズム（戦略）を切り替え可能にするパターンです。

アルゴリズムの部分を別のクラスとして分離し、実行時に差し替えられるようにします。これにより、条件分岐（if文やswitch文）を使わずに、振る舞いを柔軟に変更できます。

**解決する問題：**

- アルゴリズムが複数あり、状況に応じて使い分けたい
- アルゴリズムの追加・変更時に既存コードを修正したくない
- 条件分岐が増えてコードが複雑になるのを防ぎたい

## 2. どんな時に使うか

| 場面                   | 具体例                                         |
| ---------------------- | ---------------------------------------------- |
| 複数の計算方法がある   | 税率計算（標準税率/軽減税率/非課税）           |
| 出力形式を切り替えたい | レポート出力（PDF/Excel/CSV）                  |
| 検証ルールが複数ある   | 入力バリデーション（メール/電話番号/郵便番号） |
| ソート方法を変えたい   | 名前順/日付順/価格順                           |
| 支払い方法が複数ある   | 現金/クレジットカード/電子マネー               |

## 3. 登場人物（クラス構成）

```
┌─────────────────────────────────────────────────────┐
│  Context（文脈）                                      │
│  ・Strategyを利用する側                               │
│  ・どのStrategyを使うかを保持                          │
├─────────────────────────────────────────────────────┤
│  - strategy: IStrategy                              │
│  + ExecuteStrategy()                                │
└──────────────────────┬──────────────────────────────┘
                       │ 利用
                       ▼
          ┌────────────────────────┐
          │ <<interface>>          │
          │ IStrategy（戦略）       │
          │ ・アルゴリズムの共通API  │
          ├────────────────────────┤
          │ + Execute()            │
          └────────────────────────┘
                       △
                       │ 実装
        ┌──────────────┼──────────────┐
        │              │              │
┌───────┴───────┐┌─────┴─────┐┌───────┴───────┐
│ConcreteStrategyA││ConcreteStrategyB││ConcreteStrategyC│
│ 具体的な戦略A    ││ 具体的な戦略B    ││ 具体的な戦略C    │
└───────────────┘└───────────┘└───────────────┘
```

**役割の説明：**

| 登場人物                           | 役割                                                 |
| ---------------------------------- | ---------------------------------------------------- |
| **Strategy（戦略）**               | アルゴリズムの共通インターフェースを定義             |
| **ConcreteStrategy（具体的戦略）** | Strategyを実装し、具体的なアルゴリズムを提供         |
| **Context（文脈）**                | Strategyを利用する。どの戦略を使うかを保持・切り替え |

## 4. C#での実装例

### 例：POSシステムの割引計算

csharp

```csharp
using System;

namespace StrategyPattern
{
    // ===========================================
    // Strategy: 割引戦略のインターフェース
    // ===========================================
    public interface IDiscountStrategy
    {
        /// <summary>
        /// 割引を適用した金額を計算する
        /// </summary>
        /// <param name="originalPrice">元の金額</param>
        /// <returns>割引後の金額</returns>
        decimal CalculatePrice(decimal originalPrice);

        /// <summary>
        /// 戦略の説明を取得
        /// </summary>
        string Description { get; }
    }

    // ===========================================
    // ConcreteStrategy: 割引なし
    // ===========================================
    public class NoDiscountStrategy : IDiscountStrategy
    {
        public string Description => "割引なし";

        public decimal CalculatePrice(decimal originalPrice)
        {
            return originalPrice;
        }
    }

    // ===========================================
    // ConcreteStrategy: 定率割引（例: 10%OFF）
    // ===========================================
    public class PercentageDiscountStrategy : IDiscountStrategy
    {
        private readonly decimal _percentage;

        public PercentageDiscountStrategy(decimal percentage)
        {
            if (percentage < 0 || percentage > 100)
            {
                throw new ArgumentOutOfRangeException(
                    nameof(percentage), "割引率は0〜100の範囲で指定してください");
            }
            _percentage = percentage;
        }

        public string Description => $"{_percentage}%OFF";

        public decimal CalculatePrice(decimal originalPrice)
        {
            return originalPrice * (100 - _percentage) / 100;
        }
    }

    // ===========================================
    // ConcreteStrategy: 定額割引（例: 500円引き）
    // ===========================================
    public class FixedAmountDiscountStrategy : IDiscountStrategy
    {
        private readonly decimal _discountAmount;

        public FixedAmountDiscountStrategy(decimal discountAmount)
        {
            if (discountAmount < 0)
            {
                throw new ArgumentOutOfRangeException(
                    nameof(discountAmount), "割引額は0以上で指定してください");
            }
            _discountAmount = discountAmount;
        }

        public string Description => $"{_discountAmount:N0}円引き";

        public decimal CalculatePrice(decimal originalPrice)
        {
            // 割引後がマイナスにならないようにする
            return Math.Max(0, originalPrice - _discountAmount);
        }
    }

    // ===========================================
    // ConcreteStrategy: 会員ランク割引
    // ===========================================
    public class MemberDiscountStrategy : IDiscountStrategy
    {
        private readonly MemberRank _rank;

        public MemberDiscountStrategy(MemberRank rank)
        {
            _rank = rank;
        }

        public string Description => $"会員割引（{_rank}）";

        public decimal CalculatePrice(decimal originalPrice)
        {
            // ランクに応じた割引率を適用
            decimal discountRate = _rank switch
            {
                MemberRank.Bronze => 0.03m,   // 3%OFF
                MemberRank.Silver => 0.05m,   // 5%OFF
                MemberRank.Gold => 0.10m,     // 10%OFF
                MemberRank.Platinum => 0.15m, // 15%OFF
                _ => 0m
            };

            return originalPrice * (1 - discountRate);
        }
    }

    public enum MemberRank
    {
        Bronze,
        Silver,
        Gold,
        Platinum
    }

    // ===========================================
    // Context: 買い物かご（Strategyを利用する側）
    // ===========================================
    public class ShoppingCart
    {
        private readonly List<CartItem> _items = new();
        private IDiscountStrategy _discountStrategy;

        public ShoppingCart()
        {
            // デフォルトは割引なし
            _discountStrategy = new NoDiscountStrategy();
        }

        /// <summary>
        /// 割引戦略を設定する（実行時に切り替え可能）
        /// </summary>
        public void SetDiscountStrategy(IDiscountStrategy strategy)
        {
            _discountStrategy = strategy ?? throw new ArgumentNullException(nameof(strategy));
        }

        public void AddItem(string name, decimal price, int quantity)
        {
            _items.Add(new CartItem(name, price, quantity));
        }

        /// <summary>
        /// 小計（割引前）
        /// </summary>
        public decimal Subtotal => _items.Sum(item => item.Price * item.Quantity);

        /// <summary>
        /// 合計金額を計算（割引適用後）
        /// </summary>
        public decimal CalculateTotal()
        {
            // Strategyに処理を委譲
            return _discountStrategy.CalculatePrice(Subtotal);
        }

        public void PrintReceipt()
        {
            Console.WriteLine("========== レシート ==========");
            foreach (var item in _items)
            {
                Console.WriteLine($"{item.Name} × {item.Quantity}: {item.Price * item.Quantity:N0}円");
            }
            Console.WriteLine("------------------------------");
            Console.WriteLine($"小計: {Subtotal:N0}円");
            Console.WriteLine($"適用割引: {_discountStrategy.Description}");
            Console.WriteLine($"合計: {CalculateTotal():N0}円");
            Console.WriteLine("==============================");
        }
    }

    public record CartItem(string Name, decimal Price, int Quantity);

    // ===========================================
    // 実行例
    // ===========================================
    class Program
    {
        static void Main(string[] args)
        {
            var cart = new ShoppingCart();
            cart.AddItem("コーヒー", 350, 2);
            cart.AddItem("サンドイッチ", 480, 1);
            cart.AddItem("ケーキ", 550, 1);

            // 戦略1: 割引なし
            Console.WriteLine("【通常購入】");
            cart.PrintReceipt();

            // 戦略2: 10%OFFクーポン適用
            Console.WriteLine("\n【10%OFFクーポン使用】");
            cart.SetDiscountStrategy(new PercentageDiscountStrategy(10));
            cart.PrintReceipt();

            // 戦略3: 200円引きクーポン適用
            Console.WriteLine("\n【200円引きクーポン使用】");
            cart.SetDiscountStrategy(new FixedAmountDiscountStrategy(200));
            cart.PrintReceipt();

            // 戦略4: ゴールド会員割引
            Console.WriteLine("\n【ゴールド会員割引】");
            cart.SetDiscountStrategy(new MemberDiscountStrategy(MemberRank.Gold));
            cart.PrintReceipt();
        }
    }
}
```

### 実行結果

```
【通常購入】
========== レシート ==========
コーヒー × 2: 700円
サンドイッチ × 1: 480円
ケーキ × 1: 550円
------------------------------
小計: 1,730円
適用割引: 割引なし
合計: 1,730円
==============================

【10%OFFクーポン使用】
========== レシート ==========
コーヒー × 2: 700円
サンドイッチ × 1: 480円
ケーキ × 1: 550円
------------------------------
小計: 1,730円
適用割引: 10%OFF
合計: 1,557円
==============================

【200円引きクーポン使用】
========== レシート ==========
コーヒー × 2: 700円
サンドイッチ × 1: 480円
ケーキ × 1: 550円
------------------------------
小計: 1,730円
適用割引: 200円引き
合計: 1,530円
==============================

【ゴールド会員割引】
========== レシート ==========
コーヒー × 2: 700円
サンドイッチ × 1: 480円
ケーキ × 1: 550円
------------------------------
小計: 1,730円
適用割引: 会員割引（Gold）
合計: 1,557円
==============================
```