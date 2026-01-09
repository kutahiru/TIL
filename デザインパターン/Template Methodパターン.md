# Template Method パターン

## 1. パターンの目的

Template Methodパターンは、**アルゴリズムの骨格（テンプレート）をスーパークラスで定義し、具体的な処理内容はサブクラスに委ねる**パターンです。

解決する問題：

- 複数のクラスで処理の流れ（手順）は同じだが、各ステップの具体的な実装が異なる場合
- アルゴリズムの構造を変えずに、一部の処理だけを変更したい場合
- 共通処理の重複を排除し、コードの再利用性を高めたい場合

**「処理の流れは固定、中身は可変」** という考え方がTemplate Methodの本質です。

## 2. どんな時に使うか

| 場面                       | 具体例                                                       |
| -------------------------- | ------------------------------------------------------------ |
| **データ処理パイプライン** | 読み込み→変換→出力の流れは同じだが、フォーマットが異なる     |
| **レポート生成**           | ヘッダー→本文→フッターの構造は同じだが、内容が異なる         |
| **認証処理**               | 入力検証→認証→後処理の流れで、認証方式だけ異なる             |
| **ゲームのターン処理**     | 開始→行動→終了の流れは同じだが、キャラクターごとに行動が異なる |
| **テストフレームワーク**   | setUp→テスト実行→tearDownの流れ                              |

## 3. 登場人物（クラス構成）

```
┌─────────────────────────────────────┐
│      AbstractClass（抽象クラス）       │
├─────────────────────────────────────┤
│ + TemplateMethod()      ←【final】   │
│   処理の流れを定義（骨格）             │
│   Step1() → Step2() → Step3()       │
├─────────────────────────────────────┤
│ # Step1()  ←【abstract】具象に委ねる  │
│ # Step2()  ←【abstract】具象に委ねる  │
│ # Step3()  ←【virtual】フック(任意)   │
└─────────────────────────────────────┘
              △
              │ 継承
    ┌─────────┴─────────┐
    │                   │
┌───┴───┐         ┌─────┴─────┐
│ConcreteA│         │ ConcreteB │
├────────┤         ├───────────┤
│# Step1()│         │ # Step1() │
│# Step2()│         │ # Step2() │
│# Step3()│         │ # Step3() │
└────────┘         └───────────┘
```

| 登場人物           | 役割                                                         |
| ------------------ | ------------------------------------------------------------ |
| **AbstractClass**  | テンプレートメソッドを定義し、アルゴリズムの骨格を決める。抽象メソッドで具体的処理をサブクラスに委ねる |
| **ConcreteClass**  | 抽象メソッドを実装し、具体的な処理内容を提供する             |
| **TemplateMethod** | 処理の流れを定義するメソッド。サブクラスでオーバーライドさせない |
| **Hook Method**    | デフォルト実装を持ち、サブクラスが必要に応じてオーバーライドできるメソッド |

## 4. C#での実装例

### 例1：書籍に近いシンプルな例（文字・文字列の表示）

csharp

```csharp
using System;

namespace TemplateMethodPattern.Basic
{
    /// <summary>
    /// AbstractClass（抽象クラス）
    /// 表示処理のテンプレートを定義
    /// </summary>
    public abstract class AbstractDisplay
    {
        // 抽象メソッド：サブクラスで実装
        protected abstract void Open();
        protected abstract void Print();
        protected abstract void Close();

        /// <summary>
        /// テンプレートメソッド
        /// 処理の流れを定義（サブクラスで変更不可）
        /// </summary>
        public void Display()
        {
            Open();
            for (int i = 0; i < 5; i++)
            {
                Print();
            }
            Close();
        }
    }

    /// <summary>
    /// ConcreteClass：文字を表示する具象クラス
    /// </summary>
    public class CharDisplay : AbstractDisplay
    {
        private readonly char _ch;

        public CharDisplay(char ch)
        {
            _ch = ch;
        }

        protected override void Open()
        {
            Console.Write("<<");
        }

        protected override void Print()
        {
            Console.Write(_ch);
        }

        protected override void Close()
        {
            Console.WriteLine(">>");
        }
    }

    /// <summary>
    /// ConcreteClass：文字列を表示する具象クラス
    /// </summary>
    public class StringDisplay : AbstractDisplay
    {
        private readonly string _text;

        public StringDisplay(string text)
        {
            _text = text;
        }

        protected override void Open()
        {
            PrintLine();
        }

        protected override void Print()
        {
            Console.WriteLine($"|{_text}|");
        }

        protected override void Close()
        {
            PrintLine();
        }

        private void PrintLine()
        {
            Console.WriteLine($"+{new string('-', _text.Length)}+");
        }
    }

    // 使用例
    class Program
    {
        static void Main()
        {
            // 同じDisplayメソッドを呼んでも、結果が異なる
            AbstractDisplay d1 = new CharDisplay('H');
            AbstractDisplay d2 = new StringDisplay("Hello, World");

            d1.Display();
            Console.WriteLine();
            d2.Display();
        }
    }
}
```

**出力結果：**

```
<<HHHHH>>

+------------+
|Hello, World|
|Hello, World|
|Hello, World|
|Hello, World|
|Hello, World|
+------------+
```

### 例2：実務レベルの例（データエクスポート処理）

csharp

```csharp
using System;
using System.Collections.Generic;
using System.Text;
using System.Text.Json;
using System.IO;

namespace TemplateMethodPattern.Practical
{
    /// <summary>
    /// エクスポート対象のデータモデル
    /// </summary>
    public record SalesRecord(
        int Id,
        DateTime Date,
        string ProductName,
        int Quantity,
        decimal UnitPrice
    )
    {
        public decimal TotalPrice => Quantity * UnitPrice;
    }

    /// <summary>
    /// AbstractClass：データエクスポートのテンプレート
    /// </summary>
    public abstract class DataExporter
    {
        // テンプレートで使用するデータ
        protected IEnumerable<SalesRecord> Records { get; private set; } = [];
        protected string OutputPath { get; private set; } = string.Empty;

        // 抽象メソッド：サブクラスで必ず実装
        protected abstract string GetFileExtension();
        protected abstract string FormatHeader();
        protected abstract string FormatRecord(SalesRecord record);
        protected abstract string FormatFooter(IEnumerable<SalesRecord> records);

        // フックメソッド：必要に応じてオーバーライド
        protected virtual void OnExportStarting() { }
        protected virtual void OnExportCompleted(string filePath) { }
        protected virtual bool ValidateRecord(SalesRecord record) => true;

        /// <summary>
        /// テンプレートメソッド：エクスポート処理の流れを定義
        /// </summary>
        public void Export(IEnumerable<SalesRecord> records, string basePath)
        {
            Records = records ?? throw new ArgumentNullException(nameof(records));
            OutputPath = $"{basePath}.{GetFileExtension()}";

            // 1. 開始処理（フック）
            OnExportStarting();

            var content = new StringBuilder();

            // 2. ヘッダー出力
            content.AppendLine(FormatHeader());

            // 3. 各レコードの出力
            foreach (var record in Records)
            {
                // バリデーション（フック）
                if (!ValidateRecord(record))
                {
                    Console.WriteLine($"警告: レコードID {record.Id} をスキップしました");
                    continue;
                }
                content.AppendLine(FormatRecord(record));
            }

            // 4. フッター出力
            content.AppendLine(FormatFooter(Records));

            // 5. ファイル書き込み
            File.WriteAllText(OutputPath, content.ToString(), Encoding.UTF8);

            // 6. 完了処理（フック）
            OnExportCompleted(OutputPath);
        }
    }

    /// <summary>
    /// ConcreteClass：CSV形式でエクスポート
    /// </summary>
    public class CsvExporter : DataExporter
    {
        protected override string GetFileExtension() => "csv";

        protected override string FormatHeader()
        {
            return "ID,日付,商品名,数量,単価,合計";
        }

        protected override string FormatRecord(SalesRecord record)
        {
            return $"{record.Id},{record.Date:yyyy-MM-dd}," +
                   $"\"{record.ProductName}\",{record.Quantity}," +
                   $"{record.UnitPrice},{record.TotalPrice}";
        }

        protected override string FormatFooter(IEnumerable<SalesRecord> records)
        {
            var total = records.Sum(r => r.TotalPrice);
            return $",,,,合計,{total}";
        }

        protected override void OnExportCompleted(string filePath)
        {
            Console.WriteLine($"CSVエクスポート完了: {filePath}");
        }
    }

    /// <summary>
    /// ConcreteClass：HTML形式でエクスポート
    /// </summary>
    public class HtmlExporter : DataExporter
    {
        protected override string GetFileExtension() => "html";

        protected override string FormatHeader()
        {
            return """
                <!DOCTYPE html>
                <html><head>
                <meta charset="UTF-8">
                <title>売上レポート</title>
                <style>
                    table { border-collapse: collapse; width: 100%; }
                    th, td { border: 1px solid #ddd; padding: 8px; text-align: left; }
                    th { background-color: #4CAF50; color: white; }
                    tr:nth-child(even) { background-color: #f2f2f2; }
                    .total { font-weight: bold; background-color: #fff3cd; }
                </style>
                </head><body>
                <h1>売上レポート</h1>
                <table>
                <tr><th>ID</th><th>日付</th><th>商品名</th><th>数量</th><th>単価</th><th>合計</th></tr>
                """;
        }

        protected override string FormatRecord(SalesRecord record)
        {
            return $"<tr><td>{record.Id}</td><td>{record.Date:yyyy-MM-dd}</td>" +
                   $"<td>{record.ProductName}</td><td>{record.Quantity}</td>" +
                   $"<td>¥{record.UnitPrice:N0}</td><td>¥{record.TotalPrice:N0}</td></tr>";
        }

        protected override string FormatFooter(IEnumerable<SalesRecord> records)
        {
            var total = records.Sum(r => r.TotalPrice);
            return $"""
                <tr class="total"><td colspan="5">合計</td><td>¥{total:N0}</td></tr>
                </table>
                <p>出力日時: {DateTime.Now:yyyy-MM-dd HH:mm:ss}</p>
                </body></html>
                """;
        }

        protected override void OnExportStarting()
        {
            Console.WriteLine("HTMLレポートを生成中...");
        }

        protected override void OnExportCompleted(string filePath)
        {
            Console.WriteLine($"HTMLエクスポート完了: {filePath}");
        }
    }

    /// <summary>
    /// ConcreteClass：JSON形式でエクスポート
    /// </summary>
    public class JsonExporter : DataExporter
    {
        private readonly List<object> _recordList = [];

        protected override string GetFileExtension() => "json";

        protected override string FormatHeader()
        {
            _recordList.Clear();
            return string.Empty; // JSONはヘッダー不要
        }

        protected override string FormatRecord(SalesRecord record)
        {
            _recordList.Add(new
            {
                record.Id,
                Date = record.Date.ToString("yyyy-MM-dd"),
                record.ProductName,
                record.Quantity,
                record.UnitPrice,
                record.TotalPrice
            });
            return string.Empty; // 後でまとめてシリアライズ
        }

        protected override string FormatFooter(IEnumerable<SalesRecord> records)
        {
            var result = new
            {
                ExportDate = DateTime.Now.ToString("yyyy-MM-dd HH:mm:ss"),
                TotalRecords = _recordList.Count,
                GrandTotal = records.Sum(r => r.TotalPrice),
                Records = _recordList
            };

            var options = new JsonSerializerOptions
            {
                WriteIndented = true,
                Encoder = System.Text.Encodings.Web.JavaScriptEncoder.UnsafeRelaxedJsonEscaping
            };

            return JsonSerializer.Serialize(result, options);
        }

        // バリデーションをオーバーライド：数量0以下は除外
        protected override bool ValidateRecord(SalesRecord record)
        {
            return record.Quantity > 0;
        }

        protected override void OnExportCompleted(string filePath)
        {
            Console.WriteLine($"JSONエクスポート完了: {filePath}");
        }
    }

    // 使用例
    class Program
    {
        static void Main()
        {
            // テストデータ
            var salesData = new List<SalesRecord>
            {
                new(1, DateTime.Now.AddDays(-2), "コーヒー豆 500g", 3, 1200),
                new(2, DateTime.Now.AddDays(-1), "紅茶パック 20袋", 5, 480),
                new(3, DateTime.Now, "マグカップ", 2, 850),
                new(4, DateTime.Now, "キャンセル品", 0, 100), // JSON出力時にスキップされる
            };

            // 同じデータを異なる形式でエクスポート
            // テンプレートメソッドのおかげで、呼び出し側は形式を意識しない
            DataExporter[] exporters =
            [
                new CsvExporter(),
                new HtmlExporter(),
                new JsonExporter()
            ];

            foreach (var exporter in exporters)
            {
                exporter.Export(salesData, "sales_report");
            }
        }
    }
}
```