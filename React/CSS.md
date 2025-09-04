# CSS

cssの反映方法は複数の選択肢がある。

## CSS Modulesの使用

```bash
npm install sass
```

```jsx
// src/CssModules.jsx
import classes from "./CssModules.module.scss";

export const CssModules = () => {
  return (
    <div className={classes.container}>
      <p className={classes.title}>CSS Modulesです</p>
      <button className={classes.button}>ボタン</button>
    </div>
  );
};
```

```scss
// src/CssModules.module.scss
.container {
  border: solid 1px #aaa;
  border-radius: 20px;
  padding: 8px;
  margin: 8px;
  display: flex;
  justify-content: space-around;
  align-items: center;
}

.title {
  margin: 0;
  color: #aaa;
}

.button {
  background-color: #ddd;
  border: none;
  padding: 8px;
  border-radius: 5px;
  &:hover {
    background-color: #aaa;
    color: #fff;
    cursor: pointer;
  }
}
```

## Styled JSXの使用

プレーンなReactプロジェクトでは使わない。
Next.jsで作成したプロジェクトでとりあえずCSS-in-JSを使う場合に最適

```bash
npm install styled-jsx
```

```jsx
// src/StyledJsx.jsx
export const StyledJsx = () => {
  return (
    <>
      <div className="container">
        <p className="title">Styled JSXです</p>
        <button className="button">ボタン</button>
      </div>
  
      <style jsx>{`
      .container {
        border: solid 1px #aaa;
        border-radius: 20px;
        padding: 8px;
        margin: 8px;
        display: flex;
        justify-content: space-around;
        align-items: center;
      }
      
      .title {
        margin: 0;
        color: #aaa;
      }
      
      .button {
        background-color: #ddd;
        border: none;
        padding: 8px;
        border-radius: 5px;
        &:hover {
          background-color: #aaa;
          color: #fff;
          cursor: pointer;
        }
      }
      `}
      </style>
    </>
  );
}
```

## Styled Componentsの使用

scss記法がそのまま使えるため、既存のCSSを使用したシステムから移行がスムーズ

```bash
npm install styled-components
```

```jsx
import styled from "styled-components";

export const StyledComponents = () => {
  return (
  <SContainer>
    <STitle>Styled Componentsです</STitle>
    <SButton>ボタン</SButton>
  </SContainer>
  );
};

const SContainer = styled.div`
  border: solid 1px #aaa;
  border-radius: 20px;
  padding: 8px;
  margin: 8px;
  display: flex;
  justify-content: space-around;
  align-items: center;
`

const STitle = styled.p`
  margin: 0;
  color: #aaa;
`;

const SButton = styled.button`
  background-color: #ddd;
  border: none;
  padding: 8px;
  border-radius: 5px;
  &:hover {
    background-color: #aaa;
    color: #fff;
    cursor: pointer;
`;
```

## Emotionの使用

上の3つのいずれの方法でも選択可能になる
自由が高いから保守性が低下するかも

```bash
npm install @emotion/react @emotion/styled
```

