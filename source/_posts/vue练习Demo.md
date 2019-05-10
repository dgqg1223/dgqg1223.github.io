---
title: 5.vue练习Demo
ate: 2019-04-19 11:32:29
tags:
    - Demo
categories:
    - Java
    - Demo
---

# 基础练习
``` html
<!DOCTYPE html>
<html lang="en" xmlns:v-on="http://www.w3.org/1999/xhtml" xmlns:v-bind="http://www.w3.org/1999/xhtml">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style type="text/css">
        .active {
            width: 100px;
            height: 100px;

        }

        .text-danger {
            border: 5px solid #0e7700;
        }

        .text-error {
            background-color: red
        }
    </style>
</head>
<body>
<div id="app">
    <input type="text" v-model="num">
    <button v-on:click="add">点我</button>
    <h2>{{name}}非常帅！！！{{num}}</h2><br/>
    v-text:<span v-text="hello"></span><br/>
    v-html:<span v-html="hello"></span>
    <hr/>
    <h1>"v-model" Demo</h1><br/>
    <input type="checkbox" v-model="language" value="java"/>java<br/>
    <input type="checkbox" v-model="language" value="PHP"/>PHP<br/>
    <input type="checkbox" v-model="language" value="Swift"/>Swift<br/>
    <input type="checkbox" v-model="language" value="python"/>python<br/>
    你选择了：{{language.join(',')}}
    <hr/>
    <h1>"v-on" Demo</h1><br/>
    "v-on:click" 简写为"@click"<br/>
    <!--事件中直接写js片段-->
    <input type="button" value="增加一个" v-on:click="add">
    <!--事件指定一个回调函数，必须是Vue实例中定义的函数-->
    <input type="button" value="减少一个" v-on:click="num--">
    现在的数值是{{num}}

    <hr/>
    <h1>事件修饰符</h1><br/>
    <h2>.stop 阻止事件冒泡到父元素</h2>
    <div style="width:100px;height:100px;background-color:cornsilk " @click="print('div')">
        <button @click.stop="print('button')">点我试试</button>
    </div>
    <br/>
    <h2>.prevent 阻止默认事件发生</h2>
    <a href="www.baidu.com" @click.prevent="print('百度')">百度一下，你就疯了</a>
    <hr/>
    <h1>遍历数组</h1><br/>
    <ul>
        <li v-for="(user in users">
            {{user.name}} - {{user.gender}} - {{user.age}}
        </li>
    </ul>
    <ul>
        <li v-for="(user,index) in users">
            {{index}}.{{user.name}} - {{user.gender}} - {{user.age}}
        </li>
    </ul>
    <hr/>
    <h1>"v-if"和"v-show"</h1><br/>
    <button @click="show = !show">点我呀</button>
    show={{show}} <br/>
    <span v-if="show">v-if={{show}}</span>
    <span v-else>v-else</span>
    <span v-show="show">v-show={{show}}</span>

    <hr/>
    <h1>v-bind</h1>
    通过v-bind修改title属性、class属性、style属性（html标签属性不能使用{{}}的方式）<br/>
    简写:v-bind 简写为:class
    <div v-bind:title="title" v-bind:class="[activeClass,errorClass]"></div>
    <br/>
    <div style="width:100px;height:100px;" v-bind:style="[baseStyles,overridingStyles]"></div>
    <br/>
    <div :class="classObject"></div>
    <br/>
    <div class="text-error" v-bind:class="{active: isActive, 'text-danger': hasError}"></div>
    <br/>

    <hr/>
    <h1>计算属性</h1>
    普通写法：{{new Date(birthday).getFullYear() + '-'+ new Date(birthday).getMonth()+ '-' + new
    Date(birthday).getDay()}}<br/>
    使用计算属性:{{birth}}

    <hr/>
    <h1>watch</h1>
    监控<br/>
    <input type="text" v-model="num"/><br/>

    深度监控<br/>
    <input type="button" value="abc" @click="person.age++"/><br/>
    {{person.age}}<br/>

</div>

<script src="vue/dist/vue.js"></script>
<script>
    var app = new Vue({
            el: "#app",
            data: {
                name: "chen",
                num: 5,
                hello: "大家好！",
                language: [],
                users: [
                    {name: '柳岩', gender: '女', age: 21},
                    {name: '峰哥', gender: '男', age: 18},
                    {name: '范冰冰', gender: '女', age: 24},
                    {name: '刘亦菲', gender: '女', age: 18},
                    {name: '古力娜扎', gender: '女', age: 25}
                ],
                title: "abc",
                show: true,
                activeClass: "active",
                errorClass: ["text-danger", "text-error"],
                baseStyles: {'background-color': 'red'},
                overridingStyles: {border: '1px solid black'},
                classObject: {
                    active: true,
                    'text-danger': true,
                    'text-error': false
                },
                isActive: true,
                hasError: false,

                birthday: 1529032123201,

                num: "",
                person: {
                    name: "jack",
                    age: "21"
                }

            },
            methods: {
                add: function () {
                    this.num++;
                },
                print: function (msg) {
                    console.log(msg);
                },
            },
            created: function () {
                this.num = 100;
            },
            computed: {
                birth() {// 计算属性本质是一个方法，但是必须返回结果
                    const d = new Date(this.birthday);
                    return d.getFullYear() + "-" + d.getMonth() + "-" + d.getDay();
                }
            },
            watch: {
                num(newVal, oldVal) {
                    console.log(newVal, oldVal);
                },
                person: {
                    deep: true,
                    handler(newVal,oldVal) {
                        console.log(newVal.age,oldVal.age);
                    }
                }
            }
        })

    ;
</script>
</body>
</html>
```


# 组件练习
``` html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>


<div id="app">
    <!--使用定义好的全局组件-->
    <counter></counter>
    <counter></counter>

    <!--使用定义好的局部组件-->
    <counter2></counter2>
</div>
<script src="vue/dist/vue.js"></script>
<script type="text/javascript">
    // 定义全局组件，两个参数：1，组件名称。2，组件参数
    Vue.component("counter", {
        template: "<button v-on:click='count++'>全局组件：你点了我 {{ count }} 次，我记住了.</button>",
        data() {
            return {
                count: 0,
            }
        }
    });

    //定义局部组件
    const counter2 = {
        template: '<button v-on:click="count++">局部组件：你点了我 {{ count }} 次，我记住了.</button>',
        data() {
            return {
                count: 0,
            }
        }
    };
    const app = new Vue({
        el: "#app",
        data: {},
        components: {
            counter2,
        }
    });
</script>

</body>
</html>
```