# 입문자를 위한 지식      

## 1-2.1 JQuery
바닐라 자바스크립트의 한계를 깨고 개발자들에게 많은 도움이 됐다.
<br>
하지만 이는 최신 자바스크립트 네이티브 기능들로 인해 거의 필요없게 된 케이스들이 있다.

```tsx
과거 (jQuery)
$(".item").addClass("active");

현재 (Native)
document.querySelectorAll(".item").forEach(el => el.classList.add("active"));
```

## 1-2.2 Backbone
