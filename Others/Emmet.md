# Emmet

## emmet.includeLanguagesに設定が必要

```
"emmet.includeLanguages": {
  "erb": "html"
}
```

## 使用例

「Emmet」にはチートシートがある。

```
#初期がenなので注意
html
```

```
div.container
↓
<div class="container"></div>

div#test
↓
<div class="test"></div>

div.container#test
↓
<div class="container" id="test"></div>

.container
↓
<div class="container"></div>
```

```
要素 > 要素
ul>li
↓
<ul>
	<li></li>
</ul>

ul>li*3
↓
<ul>
	<li></li>
	<li></li>
	<li></li>
</ul>
```

