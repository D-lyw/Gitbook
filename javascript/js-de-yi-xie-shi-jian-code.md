---
description: 记录平时学习、了解到的js实践
---

# js的一些实践code

> 在页面卸载前执行一些js代码

如发送打点信息以记录用户点击了表单的提交按钮。但在大多数情况下， 点击提交按钮会立即开始加载下一个页面， 来不及发送打点信息的js代码

解决方法是拦截该事件以阻止页面卸载。待执行完发现打点信息的js函数后， 再执行提交表单的操作

```javascript
const form = document.getElement('form');
form.addEventListener('submit', function(event) {
    event.preventDefault();    // 取消表单的默认提交行为
    ga('send', 'event', 'some form', 'submit', {    // 发送表单提交操作的打点信息
        hitCallback: function() {
            form.submit();    // 在回调函数在重新提交该表单
        }
    });
});
```

初学者容易忽略的地方是: 超时问题。

如果 ga函数请求失败，则hitCallback回调函数永远不会触发，用户将永远无法提交表单

添加处理超时代码， 如果用户在点击1秒后， hitCallback函数还未触发， 在立即提交表单

```javascript
const form = document.getElement('form');
form.addEventListener('submit', function(event) {
    event.preventDefault();
    const delaySubmit = setTimeout(submitForm, 1000);
    function subminForm() {
        form.submit();
        clearTimeout(delaySubmit);
    }
    ga('send', 'event', 'some form', 'submit', {
        hitCallback: submitForm
    })
})
```

如果较多的使用上述处理模式，则可抽象一个函数处理这种情况

如果所返回的函数在超时时间内得到调用，该函数将清除超时事件并调用输入的函数， 如果返回的函数在规定时间内未被调用， 则立即调用输入的函数

```javascript
const delayExecute = function(callback, time_out) {
    let isExecute = false;
    const fn = function(){
        if (!isExecute) {
            callback();
            isExecute = true;
        }
    }
    setTimeout(fn, time_out);
    return fn;
}
```

