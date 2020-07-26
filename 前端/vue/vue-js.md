el:挂载点

el是用来设置Vue实例挂载（管理）的元素

Vue会管理el选项命中的元素及其内部的后代元素

可以使用其他的选择器，但是建议使用id选择器

可以使用其他的双标签，不能使用HTML和BODY



data数据对象

Vue中用到的数据定义在data中

data中可以写复杂类型的数据

渲染复杂数据类型时，遵守js的语法即可

```html
<body>
    <div id="app">
        {{message}}
        <p>{{school.name}} {{school.mobile}}</p>
        <ul>
            <li>{{campus[0]}}</li>
            <li>{{campus[2]}}</li>
        </ul>
    </div>
    <!-- 开发环境版本，包含了有帮助的命令行警告 -->
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    <script>
        var app = new Vue({
            el:"#app",
            data:{
                message:"hello ben",
                school:{
                    name:"小黑",
                    mobile:"123"
                },
                campus:["深圳","广州","北京"]
            }
        })
    </script>
</body>
```



v-text

作用是设置标签的内容

默认写法会替换全部内容，使用插值表达式{{}}可以替换指定内容

内部支持写表达式



v-html

作用是设置元素的innerHTML

内容中有html结构会被解析为标签

v-text指令无论内容是什么，只会解析为文本

解析文本使用v-text，需要解析html结构使用v-html



v-on

作用是为元素绑定事件

事件名不需要写on

指令可以简写为@

绑定的方法定义在methods属性中

方法内部通过this关键字可以访问定义在data中数据

补充：

事件绑定的方法写成函数调用的形式，可以传入自定义参数

定义方法时需要定义形参来接受传入的实参

事件的后面跟上.修饰符可以对事件进行限制

监听键盘事件时可以添加按键修饰符，.enter可以限制触发的按键为回车

事件修饰符有多种



v-show

作用是根据真假切换元素的显示状态

原理是修改元素的display，实现显示隐藏

指令后面的内容，最终都会解析为布尔值

值为true元素显示，值为false元素隐藏

数据改变之后，对应元素的显示状态会同步更新



v-if

作用是根据表达式的真假状态切换元素的显示状态

本质是通过操纵dom元素来切换显示状态

表达式的值为true，元素存在于dom树中，为false，从dom树中移除

频繁的切换用v-show，反之使用v-if，前者的切换消耗小



v-bind

作用是为元素绑定属性

完整写法是v-bind:属性名

简写的话可以直接省略v-bind，只保留:属性名

需要动态的增删class建议使用对象的方式



v-for

作用是根据数据生成列表结构

数组经常和v-for结合使用

语法是(item,index) in 数据

item和index可以结合其他指令一起使用

数组长度的更新会同步到页面上，是响应式的



v-model

作用是便捷的设置和获取表单元素的值

绑定的数据会和表单元素值相关联

绑定的数据<——>表单元素的值（双向绑定）





axios基本使用

axios必须先导入才可以使用

<script src="https://unpkg.com/axios/dist/axios.min.js"></script>

使用get或post方法即可发送对应的请求

then方法中的回调函数会在请求成功或失败时触发

通过回调函数的形参可以获取响应内容或错误信息



axios_vue

axios回调函数中的this已经改变，无法访问到data中的数据

把this保存起来，回调函数中直接使用保存的this即可

和本地应用的最大区别就是改变了数据来源

```html
<script>
    var app = new Vue({
        el:"#app",
        data:{
            joke:"好笑的笑话"
        },
        method:{
            getJoke:function(){
                var that = this;
                axios.get("https://autumnfish.cn/api/joke")
                    .then(function(response){
                    that.joke = reponse.data;
                },function(err){
                    console.log(err);
                })
            }
        }
    })
</script>
```































