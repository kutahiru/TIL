# Abstract Factory パターン（抽象的な工場）

## 1. パターンの目的

Abstract Factoryパターンは、**関連する一連のオブジェクト群を、その具体的なクラスを意識せずに生成する**ためのパターンです。

「部品」の具体的な実装を隠蔽し、「抽象的な部品」を使って「抽象的な製品」を組み立てます。クライアントは具体的なクラス名を知らなくても、一貫性のあるオブジェクト群を作成できます。

**キーワード**: 「関連するオブジェクト群をまとめて生成」「具体的なクラス名からの解放」

例えば、GUIのボタンを製品と表現している。
ボタンとテキストボックスが両方とも「Windows用」「Mac用」などのファミリー単位で統一されていないと困る。
ファミリーを指定することで「Windows用」のボタンとテキストボックスを生成するのがこのデザインパターン。
実際にボタンを使う側は抽象的な実装になっており、該当のボタンが「Windows用」「Mac用」は意識しない。

## 2. どんな時に使うか

| 場面                     | 具体例                                                      |
| ------------------------ | ----------------------------------------------------------- |
| **UIテーマの切り替え**   | ライトモード/ダークモードで関連するUI部品をまとめて切り替え |
| **OS別の実装**           | Windows/Mac/Linuxで異なるファイル操作やUI                   |
| **データベース切り替え** | SQL Server/PostgreSQL/MySQLで関連するDB操作クラス群         |
| **環境別設定**           | 開発環境/本番環境で異なる接続先やロガー群                   |
| **出力形式の切り替え**   | HTML出力/PDF出力/Excel出力など                              |

## 3. 登場人物（クラス構成）

```
┌─────────────────────────────────────────────────────────────┐
│  Client（利用者）                                            │
│  - AbstractFactory と AbstractProduct のみを使う            │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│  AbstractFactory（抽象的な工場）                             │
│  - 抽象的な製品を作るためのインターフェース                   │
│  - CreateProductA(), CreateProductB() など                  │
└─────────────────────────────────────────────────────────────┘
              △                              △
              │                              │
   ┌──────────┴──────────┐       ┌──────────┴──────────┐
   │  ConcreteFactory1   │       │  ConcreteFactory2   │
   │  (具体的な工場1)     │       │  (具体的な工場2)     │
   └─────────────────────┘       └─────────────────────┘
              │                              │
              ▼                              ▼
   ProductA1, ProductB1          ProductA2, ProductB2
   
┌─────────────────────────────────────────────────────────────┐
│  AbstractProduct（抽象的な製品）                             │
│  - AbstractProductA, AbstractProductB など                  │
│  - 製品の共通インターフェース                                │
└─────────────────────────────────────────────────────────────┘
```

| 役割                | 説明                                             |
| ------------------- | ------------------------------------------------ |
| **AbstractFactory** | 製品を生成するメソッドのインターフェースを定義   |
| **ConcreteFactory** | AbstractFactoryを実装し、具体的な製品を生成      |
| **AbstractProduct** | 生成される製品の共通インターフェース             |
| **ConcreteProduct** | 具体的な製品の実装                               |
| **Client**          | AbstractFactoryとAbstractProductのみを使って処理 |

## 4. C#での実装例

書籍の例（HTMLとテキストでリンク集を生成）に近い形で、**レポート出力システム**を実装します。

csharp

```csharp
using System;
using System.Collections.Generic;
using System.Text;

namespace AbstractFactoryPattern
{
    // ========================================
    // 抽象的な製品（Abstract Product）
    // ========================================
    
    /// <summary>
    /// 見出し要素を表す抽象クラス
    /// </summary>
    public abstract class ReportTitle
    {
        protected string Content { get; }
        
        protected ReportTitle(string content)
        {
            Content = content;
        }
        
        /// <summary>
        /// 見出しを文字列として出力
        /// </summary>
        public abstract string Render();
    }
    
    /// <summary>
    /// リスト要素を表す抽象クラス
    /// </summary>
    public abstract class ReportList
    {
        protected List<string> Items { get; } = new();
        
        /// <summary>
        /// リストに項目を追加
        /// </summary>
        public void Add(string item)
        {
            Items.Add(item);
        }
        
        /// <summary>
        /// リストを文字列として出力
        /// </summary>
        public abstract string Render();
    }
    
    /// <summary>
    /// ページ全体を表す抽象クラス
    /// </summary>
    public abstract class ReportPage
    {
        protected string Title { get; }
        protected List<object> Elements { get; } = new();
        
        protected ReportPage(string title)
        {
            Title = title;
        }
        
        /// <summary>
        /// 要素（タイトルやリスト）を追加
        /// </summary>
        public void Add(object element)
        {
            Elements.Add(element);
        }
        
        /// <summary>
        /// ページ全体を文字列として出力
        /// </summary>
        public abstract string Render();
    }
    
    // ========================================
    // 抽象的な工場（Abstract Factory）
    // ========================================
    
    /// <summary>
    /// レポート部品を生成する抽象工場
    /// </summary>
    public abstract class ReportFactory
    {
        /// <summary>
        /// 見出しを生成
        /// </summary>
        public abstract ReportTitle CreateTitle(string content);
        
        /// <summary>
        /// リストを生成
        /// </summary>
        public abstract ReportList CreateList();
        
        /// <summary>
        /// ページを生成
        /// </summary>
        public abstract ReportPage CreatePage(string title);
        
        /// <summary>
        /// クラス名から工場インスタンスを生成（Factory Methodとの組み合わせ）
        /// </summary>
        public static ReportFactory GetFactory(string formatType)
        {
            return formatType.ToLower() switch
            {
                "html" => new HtmlReportFactory(),
                "markdown" => new MarkdownReportFactory(),
                "text" => new PlainTextReportFactory(),
                _ => throw new ArgumentException($"未対応の形式: {formatType}")
            };
        }
    }
    
    // ========================================
    // HTML用の具体的な製品（Concrete Product）
    // ========================================
    
    public class HtmlTitle : ReportTitle
    {
        public HtmlTitle(string content) : base(content) { }
        
        public override string Render()
        {
            // HTMLの見出しとして出力
            return $"<h2>{EscapeHtml(Content)}</h2>";
        }
        
        private static string EscapeHtml(string text)
        {
            return text.Replace("&", "&amp;")
                       .Replace("<", "&lt;")
                       .Replace(">", "&gt;");
        }
    }
    
    public class HtmlList : ReportList
    {
        public override string Render()
        {
            var sb = new StringBuilder();
            sb.AppendLine("<ul>");
            foreach (var item in Items)
            {
                sb.AppendLine($"  <li>{EscapeHtml(item)}</li>");
            }
            sb.AppendLine("</ul>");
            return sb.ToString();
        }
        
        private static string EscapeHtml(string text)
        {
            return text.Replace("&", "&amp;")
                       .Replace("<", "&lt;")
                       .Replace(">", "&gt;");
        }
    }
    
    public class HtmlPage : ReportPage
    {
        public HtmlPage(string title) : base(title) { }
        
        public override string Render()
        {
            var sb = new StringBuilder();
            sb.AppendLine("<!DOCTYPE html>");
            sb.AppendLine("<html><head>");
            sb.AppendLine($"  <title>{Title}</title>");
            sb.AppendLine("  <meta charset=\"UTF-8\">");
            sb.AppendLine("</head><body>");
            sb.AppendLine($"<h1>{Title}</h1>");
            
            foreach (var element in Elements)
            {
                if (element is ReportTitle title)
                    sb.AppendLine(title.Render());
                else if (element is ReportList list)
                    sb.AppendLine(list.Render());
            }
            
            sb.AppendLine("</body></html>");
            return sb.ToString();
        }
    }
    
    // ========================================
    // HTML用の具体的な工場（Concrete Factory）
    // ========================================
    
    public class HtmlReportFactory : ReportFactory
    {
        public override ReportTitle CreateTitle(string content) => new HtmlTitle(content);
        public override ReportList CreateList() => new HtmlList();
        public override ReportPage CreatePage(string title) => new HtmlPage(title);
    }
    
    // ========================================
    // Markdown用の具体的な製品
    // ========================================
    
    public class MarkdownTitle : ReportTitle
    {
        public MarkdownTitle(string content) : base(content) { }
        
        public override string Render() => $"## {Content}";
    }
    
    public class MarkdownList : ReportList
    {
        public override string Render()
        {
            var sb = new StringBuilder();
            foreach (var item in Items)
            {
                sb.AppendLine($"- {item}");
            }
            return sb.ToString();
        }
    }
    
    public class MarkdownPage : ReportPage
    {
        public MarkdownPage(string title) : base(title) { }
        
        public override string Render()
        {
            var sb = new StringBuilder();
            sb.AppendLine($"# {Title}");
            sb.AppendLine();
            
            foreach (var element in Elements)
            {
                if (element is ReportTitle title)
                {
                    sb.AppendLine(title.Render());
                    sb.AppendLine();
                }
                else if (element is ReportList list)
                {
                    sb.AppendLine(list.Render());
                }
            }
            
            return sb.ToString();
        }
    }
    
    // ========================================
    // Markdown用の具体的な工場
    // ========================================
    
    public class MarkdownReportFactory : ReportFactory
    {
        public override ReportTitle CreateTitle(string content) => new MarkdownTitle(content);
        public override ReportList CreateList() => new MarkdownList();
        public override ReportPage CreatePage(string title) => new MarkdownPage(title);
    }
    
    // ========================================
    // プレーンテキスト用の具体的な製品
    // ========================================
    
    public class PlainTextTitle : ReportTitle
    {
        public PlainTextTitle(string content) : base(content) { }
        
        public override string Render()
        {
            // 下線付きの見出し
            return $"{Content}\n{new string('-', Content.Length * 2)}";
        }
    }
    
    public class PlainTextList : ReportList
    {
        public override string Render()
        {
            var sb = new StringBuilder();
            for (int i = 0; i < Items.Count; i++)
            {
                sb.AppendLine($"  {i + 1}. {Items[i]}");
            }
            return sb.ToString();
        }
    }
    
    public class PlainTextPage : ReportPage
    {
        public PlainTextPage(string title) : base(title) { }
        
        public override string Render()
        {
            var sb = new StringBuilder();
            var border = new string('=', 50);
            
            sb.AppendLine(border);
            sb.AppendLine($"  {Title}");
            sb.AppendLine(border);
            sb.AppendLine();
            
            foreach (var element in Elements)
            {
                if (element is ReportTitle title)
                {
                    sb.AppendLine(title.Render());
                    sb.AppendLine();
                }
                else if (element is ReportList list)
                {
                    sb.AppendLine(list.Render());
                }
            }
            
            return sb.ToString();
        }
    }
    
    // ========================================
    // プレーンテキスト用の具体的な工場
    // ========================================
    
    public class PlainTextReportFactory : ReportFactory
    {
        public override ReportTitle CreateTitle(string content) => new PlainTextTitle(content);
        public override ReportList CreateList() => new PlainTextList();
        public override ReportPage CreatePage(string title) => new PlainTextPage(title);
    }
    
    // ========================================
    // クライアント（利用者）
    // ========================================
    
    public class Program
    {
        public static void Main(string[] args)
        {
            // 出力形式を指定（実際はコマンドライン引数や設定ファイルから取得）
            string[] formats = { "html", "markdown", "text" };
            
            foreach (var format in formats)
            {
                Console.WriteLine($"=== {format.ToUpper()} 形式 ===\n");
                
                // 工場を取得（具体的なクラス名を知らなくてよい）
                ReportFactory factory = ReportFactory.GetFactory(format);
                
                // レポートを作成（すべて抽象型で操作）
                var report = CreateSalesReport(factory);
                
                Console.WriteLine(report);
                Console.WriteLine();
            }
        }
        
        /// <summary>
        /// 売上レポートを作成するメソッド
        /// 具体的な工場の種類を知らなくても、一貫した方法でレポートを作成できる
        /// </summary>
        private static string CreateSalesReport(ReportFactory factory)
        {
            // ページを作成
            var page = factory.CreatePage("月次売上レポート");
            
            // セクション1: 概要
            var summaryTitle = factory.CreateTitle("売上概要");
            page.Add(summaryTitle);
            
            var summaryList = factory.CreateList();
            summaryList.Add("総売上: ¥1,234,567");
            summaryList.Add("前月比: +15%");
            summaryList.Add("目標達成率: 102%");
            page.Add(summaryList);
            
            // セクション2: 売れ筋商品
            var topItemsTitle = factory.CreateTitle("売れ筋商品TOP3");
            page.Add(topItemsTitle);
            
            var topItemsList = factory.CreateList();
            topItemsList.Add("コーヒー豆（深煎り）: 234個");
            topItemsList.Add("紅茶セット: 189個");
            topItemsList.Add("マグカップ: 156個");
            page.Add(topItemsList);
            
            return page.Render();
        }
    }
}
```

### 実行結果

```
=== HTML 形式 ===

<!DOCTYPE html>
<html><head>
  <title>月次売上レポート</title>
  <meta charset="UTF-8">
</head><body>
<h1>月次売上レポート</h1>
<h2>売上概要</h2>
<ul>
  <li>総売上: ¥1,234,567</li>
  <li>前月比: +15%</li>
  <li>目標達成率: 102%</li>
</ul>
...

=== MARKDOWN 形式 ===

# 月次売上レポート

## 売上概要

- 総売上: ¥1,234,567
- 前月比: +15%
- 目標達成率: 102%
...

=== TEXT 形式 ===

==================================================
  月次売上レポート
==================================================

売上概要
----------------

  1. 総売上: ¥1,234,567
  2. 前月比: +15%
  3. 目標達成率: 102%
...
```