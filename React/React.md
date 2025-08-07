# Reactの環境構築

npm create vite@latest . --template react-ts

## スタイルの適用

CSS Modulesを利用してhome.module.cssファイルのスタイルを読み込む
CSS Modules を使うことで、home.module.css に定義されたクラス名がローカルスコープに限定され、
他のコンポーネントとクラス名が衝突することを防ぐことができる

```tsx
import { Link } from "react-router-dom";
import styles from "../../styles/home.module.css"; //スタイルの読み込み

export default function HomePage() {
  return (
    <div className={styles.container}>
      <h1 className={styles.title}>タスク管理アプリ</h1>
      <p className={styles.description}>
        React×Vite×React Routerで作るタスク管理アプリです。
      </p>
      <Link to="/tasks" className={styles.link}>
        タスク一覧へ
      </Link>
    </div>
  );
}
```

```css
/* src/styles/home.module.css */
.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 2rem;
  text-align: center;
  font-family: Arial, sans-serif;
}

.title {
  font-size: 2.5rem;
  color: #333;
  margin-bottom: 1.5rem;
}

.description {
  font-size: 1.2rem;
  color: #666;
  margin-bottom: 2rem;
}

.link {
  display: inline-block;
  background-color: #0070f3;
  color: white;
  padding: 0.8rem 1.5rem;
  border-radius: 5px;
  text-decoration: none;
  font-size: 1rem;
  transition: background-color 0.3s ease;
}

.link:hover {
  background-color: #0051bb;
}
```



