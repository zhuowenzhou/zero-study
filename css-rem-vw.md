# 移动端动态设置html的font-size

## 背景
由于移动端的终端例如手机有各种各种的分辨率，导致一套设计稿要适应多种屏幕。

## 解决方案
* 不考虑兼容性的话直接vw处理
```css
html {
    font-size: calc(100vw / 7.5)
}
```
这里解释一下代码“100vw”等于屏幕的宽度。相当于把屏幕分成了100份，每份是1vw。至于7.5的原因为是我们想得到1rem = 100px。那么按照设计稿是750px来说 1rem = 100px = 750px/7.5。这样我们就得到了设计稿为750px同时html的fontSize=100px的场景。然后设计稿里面的所有数值 XXXpx = XXX/100 rem，用于方便我们的运算

* js动态计算出font-size
```javascript
(function (doc, win) {
  let docEl = doc.documentElement;
  let resizeEvt = 'orientationchange' in window ? 'orientationchange' : 'resize';
  let recalc = function () {
    let clientWidth = docEl.clientWidth;
    if (!clientWidth) return;
    // 超过750按100px算
    if (clientWidth >= 750) {
      docEl.style.fontSize = '100px';
    } else {
      docEl.style.fontSize = 100 * (clientWidth / 750) + 'px'; 
      /*
      计算出相对于750对于基础值100的换算比例。
      举个栗子，iphone6 clientWidth = 375，则结果是 375/750=0.5,0.5 *100 =50。说明了整体缩放了50%.
      */
    }
  };
  if (!doc.addEventListener) return;
  win.addEventListener(resizeEvt, recalc, false);
  doc.addEventListener('DOMContentLoaded', recalc, false);
})(document, window);

```
