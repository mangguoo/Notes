# mouseLeave不触发

mouseenter/mouseleave（或者mouseover/mouseout）事件通常是成对的，并不会单纯因为鼠标移动过快而丢失。实际情况是，事件已经在队列中，但发生事件的元素本身发生了重大内容变更，这些事件可能会被丢弃。

举例来说，当你mouseenter或mouseover时更新了图标，也就是发生了DOM层面的修改，原本的图标元素被删除，新的图标元素被添加。容易理解，假如你原本的事件是挂在被删除的元素上的，那这些事件随着元素被删除自然就不会被触发。比较复杂的是挂在父级元素上的情况。

```html
<div id="button"></div>

<style>
#button {
    width: 24px;
    height: 24px;
    position: absolute;
    left: 50vw;
    top: 50vh;
}
</style>


<script>
let enterIcon = '<div><svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" aria-hidden="true" class="w-6"><path fill-rule="evenodd" d="M12 2.25c-5.385 0-9.75 4.365-9.75 9.75s4.365 9.75 9.75 9.75 9.75-4.365 9.75-9.75S17.385 2.25 12 2.25zm-1.72 6.97a.75.75 0 10-1.06 1.06L10.94 12l-1.72 1.72a.75.75 0 101.06 1.06L12 13.06l1.72 1.72a.75.75 0 101.06-1.06L13.06 12l1.72-1.72a.75.75 0 10-1.06-1.06L12 10.94l-1.72-1.72z" clip-rule="evenodd"></path></svg></div>';

let leaveIcon = '<div><svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" aria-hidden="true" class="w-6"><path stroke-linecap="round" stroke-linejoin="round" d="M9.75 9.75l4.5 4.5m0-4.5l-4.5 4.5M21 12a9 9 0 11-18 0 9 9 0 0118 0z"></path></svg></div>';

button.innerHTML = leaveIcon

let enter = false;
button.addEventListener("mouseenter", function() {
    console.log("mouseenter");

    if (enter) return;
    enter = true;

    button.innerHTML = enterIcon; // 这里可能会触发第二次 mouseenter 事件

    let a = 1;
    while (a++ < 100000000); // 模拟主线程耗时操作，不够可以再加个 0
});

button.addEventListener("mouseleave", function() {
    console.log("mouseleave");

    enter = false;

    button.innerHTML = leaveIcon;
});
</script>
```

在mouseenter事件回调里更换svg图标，会让父元素再触发一次mouseenter，这次事件event对象的fromElement是那个被从DOM树里删掉的svg，toElement就是button元素

鼠标移动足够快的话，在主线程执行其它代码的时间内，鼠标指针已瞬移到了按钮外部了，第二次的mouseenter事件和mouseleave事件都不会触发，因为就没检测到，而第一次的mouseenter事件因为已经被打断，也不会有对应的mouseleave事件了

mouseenter事件被打断，mouseleave事件不执行的情况有两种：

1. 先考虑mouseout，mouseout是从最内层元素冒泡上来的，最内层元素如果已经被删了，此DOM事件被丢弃也算情理之中

2. mouseleave是从内层到外层元素每层都产生一个单独的事件的，并不是冒泡的，道理上说可以不丢弃，但是mouseleave更像是由mouseout事件产生的后继事件，是有对应性的（虽然Web标准没有明确说明这一点），没有mouseout事件却产生mouseleave事件可能就会有点诡异，从浏览器实现角度来说也会比较麻烦

   