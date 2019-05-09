# Google thanos手套彩蛋效果

本文基于[Github的一个项目](https://github.com/jeiDev/animation-thanos-of-google)，不过这个仓库应该也就是Google页面的整理。github上很多类似的实现，不过这个是完全静态，也是最简洁的，本文来分析一下原理，文末造了一个demo。

本文需要一定的web基础，首先看项目文件夹，就是静态页面老三样html，css，js，这个项目只需要html2canvas库，不需要jquery和任何其他库，可以轻松引入自己的项目。具体的动画逻辑在app.js里

---

## 代码解析

先看index.html
```html
<div class="header-main" id="headerMain">
    <p>Cerca de <span id="cantSearch"></span> resultados (<span id="seconds"></span> segundos)</p>
</div>
```
这个是彩蛋效果中，搜索结果数的标签，最好用span，方便操作innerHTML，这个数字变动的效果，关联于app.js里的 animateValue。

```html
 <div class="gauntlet" id="gauntlet">
     <img src="./images/thanos_idle.png">
 </div>
 ```
这个就是灭霸手套，要注意CSS的定义，决定了手套点击后的动画能覆盖住原图。
```js
function animateValue(selector, start, end, duration, type = false) {
     let range = end - start
     let current = start
     let increment = !type ? -100000 : 100000
     let stepTime = Math.abs(Math.floor(duration / range))

     let timer = setInterval(function () {
          if ((start - end) - 2000000 > current) increment = !type ? -100000 : 100000
          current += increment
          selector.innerHTML = `${comas(""+current)}`

          if (current <= end && !type) {
               clearInterval(timer)
                finish = true
          }else if(current >= end && type){
               clearInterval(timer)
               finish = true
          }
     }, stepTime);

     return timer;
}
```
很简单，type是决定数字自增还是自减，然后用setInterval不断减小数字。 comas就是给数字打上逗号分隔符的。

接下来看init函数，略去了一些代码
```js
function init() {
    ···
    ···

    let click = true
    let c = RandomNumber(20000000, 95000000)   // 随机生成搜索结果数，仅demo用
    let s = Math.random().toFixed(2)           // 搜索用时
    
    ···

    gauntlet.onclick = () => {
        if (!finish) return
        finish = false  //如果在动画中，直接返回
        if (!click) {
            click = true   // 复原效果
            let canvas = document.createElement("canvas")
            let audio = new Audio("/mp3/thanos_reverse_sound.mp3")
            canvas.style.position = "absolute"  //很关键，决定位置
            canvas.style.cursor = "pointer"
            gauntlet.appendChild(canvas)
            audio.play()
            animationImage(canvas, "./images/thanos_time.png", res => {
                // animationImage函数用来做手套动画
                let animationTransition = document.querySelectorAll("animation-transition")

                for (let i = 0; i < animationTransition.length; i++) animationTransition[i].classList.add("reset")
                Object.keys(save).forEach((key, i) => { 
                    save[key].parent.removeChild(save[key].dom)
                    
                    ···
            
                    animateValue(cantSearch, numAnt, c, 10, true) //复原数字
                    headerMain.style.color = "#006621"

                    setTimeout(() => {
                         headerMain.style.color = "rgba(119, 119, 119, 0.63)"
                    }, 600)

                    if ((i + 1) == Object.keys(save).length) {
                        let items = document.querySelectorAll(".none")

                        for (let i = 0; i < items.length; i++) items[i].classList.add("animation-in")
                    }//逐个复原消失的搜索结果
                })
            })
            return
        }

        for (let i = 0; i < 6; i++) {
            let num = RandomNumber(0, items.length - 1)
            if (node.indexOf(num) == -1) node.push(num)
            else i -= 1
        } // 随机选取6个搜索结果，用于消失

        click = false // 消失动画开始
        let canvas = document.createElement("canvas")
        let audio = new Audio("./mp3/thanos_snap_sound.mp3")
        
        ···
        
        animationImage(canvas, "./images/thanos_snap.png", res => {
            gauntlet.removeChild(canvas)  
            //动画完成后清除手套snap的canvas
            for (let i = 0; i < node.length; i++) createCanvas(i)
            //创建结果消失动画
        })

    }
}
```

## 手套动画原理

Google的方法是利用了48帧合成的一张[Sprite图](https://zh.wikipedia.org/wiki/%E7%B2%BE%E7%81%B5%E5%9B%BE)进行动画的.

![thanos_snap](./images/thanos.png)

当然这个项目的实现是不是谷歌的实现我也不知道，不过大同小异：
```js
    interval = setInterval(() => {
        if (count < 48) {
            let left = count * 80
            canvas.style.background = "white"
            ctx.clearRect(0, 0, 80, 80)
            ctx.drawImage(img, left, 0, 80, 80, 0, 0, 80, 80)
            count += 1
        } else {
            clearInterval(interval)
            cb()
        }
    }, 50)
```
css里已经把手套的图片设置好可叠加，于是只要在手套上创建一个canvas，然后设置一个interval从左到右截取`sprite`图的一个矩形部分，在canvas内绘制即可。

## 消失动画原理

Google的实现很巧妙，它将需要沙化消失内容的html通过`html2canvas`渲染成canvas，然后将其转换为图片后的每一个像素点随机地分配到32块canvas中，最后对每块画布进行随机地移动和旋转即达到了沙化消失的效果。

在`init()`里，手套snap后对每个结果框进行了`createCanvas`，每个`createCanvas`先是将html转换为canvas，然后调用了一次`NewFrame`即，分割为32个小canvas进行沙化。

```js
function NewFrame(canvas, count = 32) {
    let {
        width,
        height
    } = canvas;
    let ctx = canvas.getContext("2d");
    let originalData = ctx.getImageData(0, 0, width, height);
    let imageDatas = [...Array(count)].map((_, i) => ctx.createImageData(width, height))

    for (let x = 0; x < width; ++x)
        for (let y = 0; y < height; ++y) {
            for (let i = 0; i < repetition; ++i) {
                let dataIndex = Math.floor(count * (Math.random() + 2 * x / width) / 3);
                let pixelIndex = (y * width + x) * 4;
                for (let offset = 0; offset < 4; ++offset) 
                imageDatas[dataIndex].data[pixelIndex + offset] = originalData.data[pixelIndex + offset];
            }
        }
    return imageDatas.map(data => {
        let clone = canvas.cloneNode(true);
        clone.getContext("2d").putImageData(data, 0, 0);
        return clone;
    });
}
```

每个小canvas沙化：
```js
    frames.forEach((frame, i) => {
        let randomRadian = 2 * Math.PI * (Math.random() - 0.5)
        frame.style.transform = `rotate(${15 * (Math.random() - 0.5)}deg) translate(${60 * Math.cos(randomRadian)}px, ${30 * Math.sin(randomRadian)}px) rotate(${15 * (Math.random() - 0.5)}deg)`;
        frame.style.opacity = 0;
        //frame就是每个小canvas，给它设置好随机旋转的transform，最终渐变为透明

        //后略
```

## Trump snap demo

可以查看我的仓库：https://github.com/RoyalAliceAcademyOfSciences/thanos-trump

## Reference

https://juejin.im/post/5cc652adf265da03540316e3
https://github.com/jeiDev/animation-thanos-of-google