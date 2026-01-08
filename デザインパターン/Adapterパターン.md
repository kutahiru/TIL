# Adapterパターン

# Adapterパターン

## 1. パターンの目的

Adapterパターンは、**既存のクラスのインターフェースを、クライアントが期待する別のインターフェースに変換する**ためのパターンです。

現実世界の例えで言うと、海外旅行で使う「電源アダプター」と同じです。日本のプラグ（既存のインターフェース）を、海外のコンセント（期待されるインターフェース）に合わせて変換します。

## 2. どんな時に使うか

- **既存のクラスを修正せずに、新しいインターフェースで利用したい**とき
- **サードパーティのライブラリ**を自社のインターフェースに合わせたいとき
- **レガシーコードと新しいコード**を統合するとき
- **互換性のないクラス同士**を協調動作させたいとき

## 3. 登場人物（クラス構成）

```
┌─────────────┐         ┌─────────────────┐
│   Client    │────────▶│  Target         │
│             │         │  (目標IF)        │
└─────────────┘         └─────────────────┘
                                 △
                                 │ 実装
                        ┌─────────────────┐       ┌─────────────────┐
                        │    Adapter      │──────▶│    Adaptee      │
                        │   (変換器)       │ 委譲   │  (既存クラス)    │
                        └─────────────────┘       └─────────────────┘
```

| 役割        | 説明                                            |
| ----------- | ----------------------------------------------- |
| **Target**  | クライアントが必要とするインターフェース        |
| **Adaptee** | 既存のクラス（適合させたい対象）                |
| **Adapter** | AdapteeをTargetインターフェースに変換するクラス |
| **Client**  | Targetインターフェースを使う側                  |

## 4. C#での実装例

### 実装方式1: オブジェクトアダプター（委譲を使用）※推奨

csharp

```csharp
using System;

namespace AdapterPattern.ObjectAdapter
{
    // ===== Target: クライアントが期待するインターフェース =====
    public interface IModernLogger
    {
        void LogInfo(string message);
        void LogWarning(string message);
        void LogError(string message, Exception? exception = null);
    }

    // ===== Adaptee: 既存のレガシーログクラス =====
    // このクラスは変更できない（サードパーティ製など）
    public class LegacyLogger
    {
        public void WriteLog(int level, string msg)
        {
            var timestamp = DateTime.Now.ToString("yyyy-MM-dd HH:mm:ss");
            var levelStr = level switch
            {
                0 => "INFO",
                1 => "WARN",
                2 => "ERROR",
                _ => "UNKNOWN"
            };
            Console.WriteLine($"[{timestamp}] [{levelStr}] {msg}");
        }
    }

    // ===== Adapter: レガシーログを新しいインターフェースに適合させる =====
    public class LegacyLoggerAdapter : IModernLogger
    {
        // Adapteeへの参照（委譲）
        private readonly LegacyLogger _legacyLogger;

        public LegacyLoggerAdapter(LegacyLogger legacyLogger)
        {
            _legacyLogger = legacyLogger ?? throw new ArgumentNullException(nameof(legacyLogger));
        }

        public void LogInfo(string message)
        {
            // 新しいインターフェースの呼び出しを、既存クラスの形式に変換
            _legacyLogger.WriteLog(0, message);
        }

        public void LogWarning(string message)
        {
            _legacyLogger.WriteLog(1, message);
        }

        public void LogError(string message, Exception? exception = null)
        {
            var fullMessage = exception != null 
                ? $"{message} | Exception: {exception.Message}" 
                : message;
            _legacyLogger.WriteLog(2, fullMessage);
        }
    }

    // ===== Client: 新しいインターフェースを使用するクラス =====
    public class OrderService
    {
        private readonly IModernLogger _logger;

        public OrderService(IModernLogger logger)
        {
            _logger = logger;
        }

        public void ProcessOrder(string orderId)
        {
            _logger.LogInfo($"注文処理開始: {orderId}");
            
            try
            {
                // 注文処理のロジック...
                _logger.LogInfo($"注文処理完了: {orderId}");
            }
            catch (Exception ex)
            {
                _logger.LogError($"注文処理失敗: {orderId}", ex);
                throw;
            }
        }
    }
}
```

### 実装方式2: クラスアダプター（継承を使用）

csharp

```csharp
namespace AdapterPattern.ClassAdapter
{
    // ===== Target =====
    public interface IModernLogger
    {
        void LogInfo(string message);
        void LogWarning(string message);
        void LogError(string message, Exception? exception = null);
    }

    // ===== Adaptee =====
    public class LegacyLogger
    {
        public void WriteLog(int level, string msg)
        {
            Console.WriteLine($"[Level:{level}] {msg}");
        }
    }

    // ===== Adapter: 継承を使用 =====
    // LegacyLoggerを継承し、IModernLoggerを実装
    public class LegacyLoggerClassAdapter : LegacyLogger, IModernLogger
    {
        public void LogInfo(string message)
        {
            // 継承したメソッドを直接呼び出し
            WriteLog(0, message);
        }

        public void LogWarning(string message)
        {
            WriteLog(1, message);
        }

        public void LogError(string message, Exception? exception = null)
        {
            var fullMessage = exception != null 
                ? $"{message} | Exception: {exception.Message}" 
                : message;
            WriteLog(2, fullMessage);
        }
    }
}
```

### 実行例

csharp

```csharp
class Program
{
    static void Main(string[] args)
    {
        Console.WriteLine("=== オブジェクトアダプター ===");
        
        // 既存のレガシーロガーをラップ
        var legacyLogger = new LegacyLogger();
        IModernLogger modernLogger = new LegacyLoggerAdapter(legacyLogger);
        
        // クライアントは新しいインターフェースだけを知っている
        var orderService = new OrderService(modernLogger);
        orderService.ProcessOrder("ORD-001");
        
        Console.WriteLine("\n=== クラスアダプター ===");
        
        IModernLogger classAdapter = new LegacyLoggerClassAdapter();
        classAdapter.LogInfo("クラスアダプターでのログ出力");
    }
}
```

**出力結果:**

```
=== オブジェクトアダプター ===
[2026-01-07 15:30:45] [INFO] 注文処理開始: ORD-001
[2026-01-07 15:30:45] [INFO] 注文処理完了: ORD-001

=== クラスアダプター ===
[Level:0] クラスアダプターでのログ出力
```