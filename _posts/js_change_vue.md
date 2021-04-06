---
title: 原生JS 修改 vue绑定元素值
date: 2021-03-08 11:36:31
categories: web
tags:
toc: true
---
> 如何解决 原生js修改 input标签 vue绑定数据不更新
> <!-- more -->

## 问题
需求是写浏览器插件来实现自动填表

发现一个问题 input直接修改value 后 点击提交系统不识别 

必须人为在输入框输入任意内容后才识别

先通过堆栈报错 发现网站是vue的

然后做了一个简单测试，发现确实如此。

## 有问题的demo
```js
<html>

<head>
    <script src="https://cdn.jsdelivr.net/npm/vue@2.6.12"></script>
</head>

<body>
    <script>
        function change(v) {
            console.log("change....");
        }
    </script>
    <div id="app">
        <input id="in" type="text" v-model="inputValue" onchange="change()" @change="change2" />
        <button @click="click">click</button>
        <button onclick="
            var element = document.getElementById('in');
            element.value = '123';
        ">原生js触发</button>
    </div>

    <script>

        new Vue({
            el: '#app',
            data: {
                inputValue: ''
            },
            methods: {
                click: function () {
                    alert(this.inputValue);
                },
                change2: function () {
                    console.log("vue change....");
                },
            }
        });
    </script>
</body>

</html>
```

## 解决思路
发现直接修改value 不光vue不识别 且onchange方法也不会触发

吃了不熟悉vue的亏，其实vue输入框和数据绑定是通过浏览器input的事件同步的！

最开始还以为是利用onchange  去手动触发onchange 也是不行

所以只需要在修改内容后手动触发input事件即可

## 解决后的代码
```js
<html>

<head>
    <script src="https://cdn.jsdelivr.net/npm/vue@2.6.12"></script>
</head>

<body>
    <script>
        function change(v) {
            console.log("change....");
        }
    </script>
    <div id="app">
        <input id="in" type="text" v-model="inputValue" onchange="change()" @change="change2" />
        <button @click="click">click</button>
        <button onclick="
            var element = document.getElementById('in');
            element.value = '123';
            var evt = document.createEvent('HTMLEvents');
            evt.initEvent('input', false, true);
            element.dispatchEvent(evt);
        ">原生js触发</button>
    </div>

    <script>

        new Vue({
            el: '#app',
            data: {
                inputValue: ''
            },
            methods: {
                click: function () {
                    alert(this.inputValue);
                },
                change2: function () {
                    console.log("vue change....");
                },
            }
        });
    </script>
</body>

</html>
```