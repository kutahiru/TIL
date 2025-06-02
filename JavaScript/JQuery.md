# JQuery

```html
<script src="https://ajax.googleapis.com/ajax/libs/jquery/3.3.1/jquery.min.js"></script>
```

## JQuery plugin

### swiper

スライダー機能の追加

```bash
yarn add swiper@11.1.14
```

```javascript
# app/javascript/application.js
// Swiperの読み込み
import Swiper from "swiper/bundle"
import "swiper/css/bundle"

console.log('Swiper:', typeof Swiper) // 確認用

document.addEventListener('DOMContentLoaded', function() {
  const swiperElement = document.querySelector('.swiper')
  
  if (swiperElement) {
    const swiper = new Swiper('.swiper', {
      loop: true,
      speed: 3000,
      autoplay: {
        delay: 1000,
        disableOnInteraction: false,
      },
      pagination: {
        el: ".swiper-pagination",
        clickable: true,
      },
      navigation: {
        nextEl: ".swiper-button-next",
        prevEl: ".swiper-button-prev",
      },
    })
  }
})
```

```html.slim
header.jumbotron.jumbotron-fluid style="position: relative;"
  .swiper style="position: absolute; top: 0; left: 0; width: 100%; height: 100%; z-index: 1;"
    .swiper-wrapper
      - current_site.main_images.each do |image|
        .swiper-slide = image_tag image.variant(resize: '1200x400')
  .container style="position: relative; z-index: 2;"
    h1 = link_to current_site.name, root_path
    p.lead = current_site.subtitle
```



