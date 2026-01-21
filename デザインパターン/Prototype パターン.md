# Prototype パターン

## 1. パターンの目的

Prototypeパターンは、**既存のインスタンスをコピー（複製）して新しいインスタンスを作る**パターンです。

解決する問題：

- `new`でインスタンスを作るのが複雑・コストが高い場合
- クラス名を指定せずにインスタンスを生成したい場合
- 同じ状態を持つオブジェクトを効率的に作りたい場合

「原型（プロトタイプ）」となるインスタンスを元に、コピーして新しいインスタンスを生み出します。

ここに記載がある例はパターンの説明であって、必要性はない。
インスタンス生成に時間がかかる場合などで、コピーすれば時間がかからないって話

## 2. どんな時に使うか

| 場面         | 具体例                                       |
| ------------ | -------------------------------------------- |
| 複雑な初期化 | 多くの設定が必要なオブジェクトを複製         |
| 動的な種類   | 実行時に決まる種類のオブジェクトを生成       |
| コスト削減   | DB読み込みや計算結果を持つオブジェクトを複製 |
| 履歴・Undo   | 状態のスナップショットを保存                 |
| テンプレート | 雛形を元に少しずつ変えたものを作成           |

## 3. 登場人物（クラス構成）

```
┌─────────────────────────────────┐
│      <<interface>>              │
│        Prototype                │
├─────────────────────────────────┤
│ + Clone(): Prototype            │  ← 自分自身を複製するメソッド
└─────────────────────────────────┘
              △
              │
    ┌─────────┴─────────┐
    │                   │
┌───────────┐     ┌───────────┐
│ConcreteA  │     │ConcreteB  │
├───────────┤     ├───────────┤
│+ Clone()  │     │+ Clone()  │  ← 具体的な複製処理
└───────────┘     └───────────┘

┌─────────────────────────────────┐
│          Manager                │
├─────────────────────────────────┤
│ - prototypes: Dictionary        │  ← 名前とプロトタイプの対応
├─────────────────────────────────┤
│ + Register(name, proto)         │  ← プロトタイプを登録
│ + Create(name): Prototype       │  ← 名前から複製を生成
└─────────────────────────────────┘
```

**役割の説明**

- **Prototype**: 複製のためのインターフェースを定義
- **ConcretePrototype**: 実際の複製処理を実装
- **Client/Manager**: プロトタイプを管理し、複製を利用する

## 4. C#での実装例

### 基本実装（ICloneableを使用）

csharp

```csharp
using System;

/// <summary>
/// プロトタイプの基底クラス
/// C#標準のICloneableを実装
/// </summary>
public abstract class Prototype : ICloneable
{
    public string Name { get; set; } = "";
    
    // 抽象メソッド - 具象クラスで実装
    public abstract object Clone();
    
    // 使用時に表示するメソッド
    public abstract void Use(string message);
}

/// <summary>
/// 具体的なプロトタイプ：下線付きメッセージ
/// </summary>
public class UnderlinePen : Prototype
{
    private char _underlineChar;

    public UnderlinePen(char underlineChar)
    {
        _underlineChar = underlineChar;
    }

    public override object Clone()
    {
        // MemberwiseClone()でシャローコピーを作成
        return MemberwiseClone();
    }

    public override void Use(string message)
    {
        Console.WriteLine(message);
        Console.WriteLine(new string(_underlineChar, message.Length));
    }
}

/// <summary>
/// 具体的なプロトタイプ：枠付きメッセージ
/// </summary>
public class MessageBox : Prototype
{
    private char _boxChar;

    public MessageBox(char boxChar)
    {
        _boxChar = boxChar;
    }

    public override object Clone()
    {
        return MemberwiseClone();
    }

    public override void Use(string message)
    {
        int length = message.Length + 4;
        string border = new string(_boxChar, length);
        
        Console.WriteLine(border);
        Console.WriteLine($"{_boxChar} {message} {_boxChar}");
        Console.WriteLine(border);
    }
}

/// <summary>
/// プロトタイプを管理するマネージャー
/// </summary>
public class PrototypeManager
{
    // 名前とプロトタイプの対応を保持
    private Dictionary<string, Prototype> _prototypes = new();

    /// <summary>
    /// プロトタイプを登録
    /// </summary>
    public void Register(string name, Prototype prototype)
    {
        _prototypes[name] = prototype;
    }

    /// <summary>
    /// 登録されたプロトタイプを複製して返す
    /// </summary>
    public Prototype Create(string name)
    {
        if (!_prototypes.TryGetValue(name, out var prototype))
        {
            throw new KeyNotFoundException($"プロトタイプ '{name}' が見つかりません");
        }
        return (Prototype)prototype.Clone();
    }
}

// 使用例
public class BasicDemo
{
    public static void Run()
    {
        var manager = new PrototypeManager();

        // プロトタイプを登録
        manager.Register("strong", new MessageBox('*'));
        manager.Register("warning", new MessageBox('#'));
        manager.Register("underline", new UnderlinePen('-'));
        manager.Register("wave", new UnderlinePen('~'));

        // 登録名から複製を生成して使用
        var p1 = manager.Create("strong");
        p1.Use("Hello, World!");

        Console.WriteLine();

        var p2 = manager.Create("warning");
        p2.Use("警告メッセージ");

        Console.WriteLine();

        var p3 = manager.Create("underline");
        p3.Use("見出しテキスト");

        Console.WriteLine();

        var p4 = manager.Create("wave");
        p4.Use("波線の装飾");
    }
}
```

**出力結果**

```
****************
* Hello, World! *
****************

##############
# 警告メッセージ #
##############

見出しテキスト
--------

波線の装飾
~~~~~~
```