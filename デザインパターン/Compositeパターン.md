# Composite（コンポジット）パターン

## 1. パターンの目的

Compositeパターンは、**個々のオブジェクト（葉）とオブジェクトの集合（容器）を同一視して扱う**ためのパターンです。

木構造（ツリー構造）を持つデータを表現する際に、「単一のもの」と「複数のものをまとめたもの」を区別なく統一的に扱えるようにします。これにより、クライアントコードが個別要素と複合要素を意識せずに操作できます。

再帰的な構造で個と集合を同一視したい場合に使用する。
そうすることでクライアント側で余計な型判定などをしなくてよくなる。

------

## 2. どんな時に使うか

- **ファイルシステム**: ファイル（葉）とディレクトリ（容器、ディレクトリも含められる）
- **GUIコンポーネント**: ボタン（葉）とパネル（容器、他のコンポーネントを含む）
- **組織図**: 社員（葉）と部署（容器、社員や他の部署を含む）
- **メニュー構造**: メニュー項目（葉）とサブメニュー（容器）
- **商品カテゴリ**: 商品（葉）とカテゴリ（容器、商品や子カテゴリを含む）

**共通点**: 「入れ物の中に入れ物を入れられる」再帰的な構造

------

## 3. 登場人物（クラス構成）

```
┌─────────────────────────────────────────────────────────┐
│ Client                                                  │
│  - Componentインターフェースを通じて操作                   │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│ Component（抽象クラス/インターフェース）                   │
│  - LeafとCompositeに共通のインターフェースを定義           │
│  - 例: Operation(), Add(), Remove(), GetChild()         │
└─────────────────────────────────────────────────────────┘
                          △
            ┌─────────────┴─────────────┐
            │                           │
┌───────────────────┐       ┌───────────────────────────┐
│ Leaf（葉）         │       │ Composite（複合体）        │
│  - 中身を持たない   │       │  - 子要素を保持            │
│  - 具体的な処理     │       │  - 子に処理を委譲          │
│    を実装          │       │  - Component を複数持つ    │
└───────────────────┘       └───────────────────────────┘
```

| 役割          | 説明                                                      |
| ------------- | --------------------------------------------------------- |
| **Component** | LeafとCompositeに共通のインターフェースを定義             |
| **Leaf**      | 「中身」を表す。他の要素を含まない末端要素                |
| **Composite** | 「容器」を表す。Componentを複数保持し、子要素に処理を委譲 |
| **Client**    | Componentインターフェースを使って操作する                 |

------

## 4. C#での実装例

### 例：ファイルシステムの構造を表現

csharp

~~~csharp
using System;
using System.Collections.Generic;

namespace CompositePattern
{
    // ============================================
    // Component: ファイルとディレクトリの共通インターフェース
    // ============================================
    public abstract class FileSystemEntry
    {
        // エントリの名前
        public string Name { get; }
        
        protected FileSystemEntry(string name)
        {
            Name = name ?? throw new ArgumentNullException(nameof(name));
        }

        // サイズを取得（LeafとCompositeで実装が異なる）
        public abstract long GetSize();

        // 一覧を表示（インデントで階層を表現）
        public abstract void PrintList(string prefix = "");

        // 子要素の追加（デフォルトではエラー）
        public virtual void Add(FileSystemEntry entry)
        {
            throw new InvalidOperationException(
                $"'{Name}' には要素を追加できません");
        }

        // フルパスを含めた表示用文字列
        public override string ToString()
        {
            return $"{Name} ({GetSize()})";
        }
    }

    // ============================================
    // Leaf: ファイル（中身を持たない末端要素）
    // ============================================
    public class File : FileSystemEntry
    {
        private readonly long _size;

        public File(string name, long size) : base(name)
        {
            if (size < 0)
                throw new ArgumentException("サイズは0以上である必要があります", nameof(size));
            _size = size;
        }

        // ファイルのサイズをそのまま返す
        public override long GetSize() => _size;

        // ファイル情報を表示
        public override void PrintList(string prefix = "")
        {
            Console.WriteLine($"{prefix}/{this}");
        }
    }

    // ============================================
    // Composite: ディレクトリ（他のエントリを含む容器）
    // ============================================
    public class Directory : FileSystemEntry
    {
        // 子要素のリスト（ファイルもディレクトリも格納可能）
        private readonly List<FileSystemEntry> _children = new();

        public Directory(string name) : base(name) { }

        // ディレクトリのサイズ = 全ての子要素のサイズの合計
        public override long GetSize()
        {
            long totalSize = 0;
            foreach (var child in _children)
            {
                // 再帰的にサイズを計算
                // 子がFileならそのサイズ、Directoryなら更にその子の合計
                totalSize += child.GetSize();
            }
            return totalSize;
        }

        // ディレクトリと中身を再帰的に表示
        public override void PrintList(string prefix = "")
        {
            Console.WriteLine($"{prefix}/{this}");
            foreach (var child in _children)
            {
                // 子要素に対して再帰呼び出し
                child.PrintList($"{prefix}/{Name}");
            }
        }

        // 子要素を追加（ディレクトリのみオーバーライド）
        public override void Add(FileSystemEntry entry)
        {
            if (entry == null)
                throw new ArgumentNullException(nameof(entry));
            _children.Add(entry);
        }

        // メソッドチェーンを可能にする追加メソッド
        public Directory AddEntry(FileSystemEntry entry)
        {
            Add(entry);
            return this;
        }
    }

    // ============================================
    // Client: 使用例
    // ============================================
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("=== Compositeパターン: ファイルシステム ===\n");

            // ルートディレクトリを作成
            var root = new Directory("root");

            // bin ディレクトリ
            var bin = new Directory("bin");
            bin.Add(new File("vi", 10000));
            bin.Add(new File("latex", 20000));

            // tmp ディレクトリ（空）
            var tmp = new Directory("tmp");

            // usr ディレクトリ（ネストした構造）
            var usr = new Directory("usr");
            var yoshuki = new Directory("yoshuki");
            yoshuki.Add(new File("diary.html", 100));
            yoshuki.Add(new File("Composite.java", 200));

            var hanako = new Directory("hanako");
            hanako.Add(new File("memo.tex", 300));

            usr.Add(yoshuki);
            usr.Add(hanako);

            // ルートに追加
            root.Add(bin);
            root.Add(tmp);
            root.Add(usr);

            // 全体を表示（再帰的に処理される）
            root.PrintList();

            Console.WriteLine();
            Console.WriteLine($"root全体のサイズ: {root.GetSize()} bytes");
            Console.WriteLine($"usrのサイズ: {usr.GetSize()} bytes");
            Console.WriteLine($"binのサイズ: {bin.GetSize()} bytes");

            // エラーケース: ファイルに追加しようとする
            Console.WriteLine("\n--- ファイルに追加しようとすると ---");
            try
            {
                var file = new File("test.txt", 100);
                file.Add(new File("other.txt", 50)); // エラー！
            }
            catch (InvalidOperationException ex)
            {
                Console.WriteLine($"エラー: {ex.Message}");
            }
        }
    }
}
```

### 実行結果
```
=== Compositeパターン: ファイルシステム ===

/root (30600)
/root/bin (30000)
/root/bin/vi (10000)
/root/bin/latex (20000)
/root/tmp (0)
/root/usr (600)
/root/usr/yoshuki (300)
/root/usr/yoshuki/diary.html (100)
/root/usr/yoshuki/Composite.java (200)
/root/usr/hanako (300)
/root/usr/hanako/memo.tex (300)

root全体のサイズ: 30600 bytes
usrのサイズ: 600 bytes
binのサイズ: 30000 bytes

--- ファイルに追加しようとすると ---
エラー: 'test.txt' には要素を追加できません
~~~