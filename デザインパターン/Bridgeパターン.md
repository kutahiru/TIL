# Bridge（ブリッジ）パターン

## 1. パターンの目的

Bridgeパターンは、**「機能のクラス階層」と「実装のクラス階層」を分離する**ためのパターンです。

通常、継承を使うと「機能の追加」と「実装の追加」が同じクラス階層に混在してしまい、クラス数が爆発的に増加します。Bridgeパターンは、この2つの階層を**橋（Bridge）で繋ぐ**ことで、それぞれを独立して拡張できるようにします。

機能の追加は親クラスにはない「新しいメソッド」を追加する継承です。
実装の追加は抽象メソッドを「具体的にどうやるか」で埋める継承です。

```
【問題】継承だけで拡張すると...
                    Shape
                   /      \
           Rectangle      Circle
           /      \       /     \
    RedRect  BlueRect  RedCircle BlueCircle
    
    → 形状×色 の組み合わせ分だけクラスが必要！

【解決】Bridgeパターンで分離すると...
    Shape ---------> Color
      |                |
   Rectangle         Red
   Circle            Blue
   
    → 形状と色を独立して追加できる！
```

## 2. どんな時に使うか

| 場面                         | 具体例                                       |
| ---------------------------- | -------------------------------------------- |
| **プラットフォーム非依存**   | Windows/Mac/Linux で動作するUIコンポーネント |
| **ドライバ/デバイス抽象化**  | 異なるプリンタ、データベースへの対応         |
| **表示形式の分離**           | 同じデータを異なるフォーマットで出力         |
| **機能と実装の独立した拡張** | 新しい機能追加時に既存実装への影響を避けたい |

## 3. 登場人物（クラス構成）

```
┌─────────────────┐          ┌─────────────────┐
│   Abstraction   │─────────▶│   Implementor   │
│    （抽象化）    │  "橋"    │    （実装者）    │
├─────────────────┤          ├─────────────────┤
│ + implementor   │          │ + RawMethod()   │
│ + Operation()   │          └────────┬────────┘
└────────┬────────┘                   │
         │                            │
         │ 継承                        │ 継承
         ▼                            ▼
┌─────────────────┐          ┌─────────────────┐
│RefinedAbstraction│         │ConcreteImplementor│
│ （改善された抽象化）│         │  （具体的実装者）  │
├─────────────────┤          ├─────────────────┤
│ + NewOperation()│          │ + RawMethod()   │
└─────────────────┘          └─────────────────┘
```

| 役割                                       | 責務                                                     |
| ------------------------------------------ | -------------------------------------------------------- |
| **Abstraction（抽象化）**                  | 機能のクラス階層の最上位。Implementorへの参照を保持      |
| **RefinedAbstraction（改善された抽象化）** | Abstractionに機能を追加したクラス                        |
| **Implementor（実装者）**                  | 実装のクラス階層の最上位。基本操作のインターフェース定義 |
| **ConcreteImplementor（具体的実装者）**    | Implementorの具体的な実装                                |

## 4. C#での実装例

### 例：レポート出力システム（機能×出力形式）

csharp

```csharp
using System;
using System.Text;
using System.Collections.Generic;

namespace BridgePattern
{
    // =========================================
    // Implementor（実装者）- 出力形式のインターフェース
    // =========================================
    
    /// <summary>
    /// レポートの出力形式を定義するインターフェース（実装のクラス階層）
    /// </summary>
    public interface IReportFormatter
    {
        /// <summary>ヘッダーを出力</summary>
        string FormatHeader(string title);
        
        /// <summary>データ行を出力</summary>
        string FormatRow(string label, string value);
        
        /// <summary>セクション区切りを出力</summary>
        string FormatSeparator();
        
        /// <summary>フッターを出力</summary>
        string FormatFooter();
    }

    // =========================================
    // ConcreteImplementor（具体的実装者）
    // =========================================
    
    /// <summary>
    /// プレーンテキスト形式でのフォーマッター
    /// </summary>
    public class PlainTextFormatter : IReportFormatter
    {
        public string FormatHeader(string title)
        {
            var line = new string('=', 40);
            return $"{line}\n  {title}\n{line}";
        }

        public string FormatRow(string label, string value)
        {
            return $"  {label,-15}: {value}";
        }

        public string FormatSeparator()
        {
            return new string('-', 40);
        }

        public string FormatFooter()
        {
            return new string('=', 40);
        }
    }

    /// <summary>
    /// HTML形式でのフォーマッター
    /// </summary>
    public class HtmlFormatter : IReportFormatter
    {
        public string FormatHeader(string title)
        {
            return $"<html>\n<head><title>{title}</title></head>\n<body>\n<h1>{title}</h1>\n<table border=\"1\">";
        }

        public string FormatRow(string label, string value)
        {
            return $"  <tr><th>{label}</th><td>{value}</td></tr>";
        }

        public string FormatSeparator()
        {
            return "  <tr><td colspan=\"2\"><hr/></td></tr>";
        }

        public string FormatFooter()
        {
            return "</table>\n</body>\n</html>";
        }
    }

    /// <summary>
    /// Markdown形式でのフォーマッター
    /// </summary>
    public class MarkdownFormatter : IReportFormatter
    {
        public string FormatHeader(string title)
        {
            return $"# {title}\n\n| 項目 | 値 |\n|------|-----|";
        }

        public string FormatRow(string label, string value)
        {
            return $"| {label} | {value} |";
        }

        public string FormatSeparator()
        {
            return "";  // Markdownではセクション区切りは空行
        }

        public string FormatFooter()
        {
            return "\n---\n*レポート終了*";
        }
    }

    // =========================================
    // Abstraction（抽象化）- レポートの基本機能
    // =========================================
    
    /// <summary>
    /// レポートの基本クラス（機能のクラス階層）
    /// </summary>
    public abstract class Report
    {
        // 橋（Bridge）- 実装への参照
        protected readonly IReportFormatter _formatter;
        protected readonly StringBuilder _content;
        
        public string Title { get; }

        protected Report(string title, IReportFormatter formatter)
        {
            Title = title ?? throw new ArgumentNullException(nameof(title));
            _formatter = formatter ?? throw new ArgumentNullException(nameof(formatter));
            _content = new StringBuilder();
        }

        /// <summary>レポートを生成する（テンプレートメソッド的）</summary>
        public virtual string Generate()
        {
            _content.Clear();
            _content.AppendLine(_formatter.FormatHeader(Title));
            BuildContent();
            _content.AppendLine(_formatter.FormatFooter());
            return _content.ToString();
        }

        /// <summary>各レポートで内容を構築する</summary>
        protected abstract void BuildContent();

        /// <summary>データ行を追加するヘルパー</summary>
        protected void AddRow(string label, string value)
        {
            _content.AppendLine(_formatter.FormatRow(label, value));
        }

        /// <summary>区切り線を追加するヘルパー</summary>
        protected void AddSeparator()
        {
            var sep = _formatter.FormatSeparator();
            if (!string.IsNullOrEmpty(sep))
            {
                _content.AppendLine(sep);
            }
        }
    }

    // =========================================
    // RefinedAbstraction（改善された抽象化）
    // =========================================
    
    /// <summary>
    /// 商品情報レポート
    /// </summary>
    public class ProductReport : Report
    {
        private readonly string _productName;
        private readonly decimal _price;
        private readonly int _stock;

        public ProductReport(
            string title, 
            IReportFormatter formatter,
            string productName,
            decimal price,
            int stock) 
            : base(title, formatter)
        {
            _productName = productName;
            _price = price;
            _stock = stock;
        }

        protected override void BuildContent()
        {
            AddRow("商品名", _productName);
            AddRow("価格", $"¥{_price:N0}");
            AddRow("在庫数", $"{_stock} 個");
        }
    }

    /// <summary>
    /// 売上サマリーレポート（機能を追加した拡張版）
    /// </summary>
    public class SalesSummaryReport : Report
    {
        private readonly List<(string Name, int Quantity, decimal Amount)> _salesData;
        private readonly DateTime _reportDate;

        public SalesSummaryReport(
            string title, 
            IReportFormatter formatter,
            DateTime reportDate) 
            : base(title, formatter)
        {
            _reportDate = reportDate;
            _salesData = new List<(string, int, decimal)>();
        }

        /// <summary>売上データを追加（機能の追加）</summary>
        public void AddSalesItem(string name, int quantity, decimal amount)
        {
            _salesData.Add((name, quantity, amount));
        }

        protected override void BuildContent()
        {
            AddRow("レポート日", _reportDate.ToString("yyyy/MM/dd"));
            AddSeparator();
            
            decimal total = 0;
            int totalQty = 0;
            
            foreach (var (name, qty, amount) in _salesData)
            {
                AddRow(name, $"{qty}個 / ¥{amount:N0}");
                total += amount;
                totalQty += qty;
            }
            
            AddSeparator();
            AddRow("合計数量", $"{totalQty} 個");
            AddRow("合計金額", $"¥{total:N0}");
        }
    }

    /// <summary>
    /// 詳細売上レポート（さらに機能を追加）
    /// </summary>
    public class DetailedSalesReport : SalesSummaryReport
    {
        private string? _notes;

        public DetailedSalesReport(
            string title, 
            IReportFormatter formatter,
            DateTime reportDate) 
            : base(title, formatter, reportDate)
        {
        }

        /// <summary>備考を設定（さらなる機能追加）</summary>
        public void SetNotes(string notes)
        {
            _notes = notes;
        }

        protected override void BuildContent()
        {
            base.BuildContent();
            
            if (!string.IsNullOrEmpty(_notes))
            {
                AddSeparator();
                AddRow("備考", _notes);
            }
        }
    }

    // =========================================
    // 使用例
    // =========================================
    
    class Program
    {
        static void Main(string[] args)
        {
            // 同じレポートを異なる形式で出力
            Console.WriteLine("【プレーンテキスト形式】");
            var textReport = new ProductReport(
                "商品情報", 
                new PlainTextFormatter(),
                "ワイヤレスマウス",
                2980m,
                50);
            Console.WriteLine(textReport.Generate());
            
            Console.WriteLine("\n【HTML形式】");
            var htmlReport = new ProductReport(
                "商品情報", 
                new HtmlFormatter(),
                "ワイヤレスマウス",
                2980m,
                50);
            Console.WriteLine(htmlReport.Generate());
            
            Console.WriteLine("\n【Markdown形式 - 売上サマリー】");
            var salesReport = new SalesSummaryReport(
                "日次売上レポート",
                new MarkdownFormatter(),
                DateTime.Today);
            salesReport.AddSalesItem("コーヒー", 120, 24000m);
            salesReport.AddSalesItem("サンドイッチ", 45, 18000m);
            salesReport.AddSalesItem("ケーキ", 30, 15000m);
            Console.WriteLine(salesReport.Generate());
        }
    }
}
```

### 実行結果

```
【プレーンテキスト形式】
========================================
  商品情報
========================================
  商品名          : ワイヤレスマウス
  価格            : ¥2,980
  在庫数          : 50 個
========================================

【HTML形式】
<html>
<head><title>商品情報</title></head>
<body>
<h1>商品情報</h1>
<table border="1">
  <tr><th>商品名</th><td>ワイヤレスマウス</td></tr>
  <tr><th>価格</th><td>¥2,980</td></tr>
  <tr><th>在庫数</th><td>50 個</td></tr>
</table>
</body>
</html>

【Markdown形式 - 売上サマリー】
# 日次売上レポート

| 項目 | 値 |
|------|-----|
| レポート日 | 2025/01/21 |
| コーヒー | 120個 / ¥24,000 |
| サンドイッチ | 45個 / ¥18,000 |
| ケーキ | 30個 / ¥15,000 |
| 合計数量 | 195 個 |
| 合計金額 | ¥57,000 |

---
*レポート終了*
```