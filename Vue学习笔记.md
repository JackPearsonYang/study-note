# Vue学习笔记

## Vue.js基础功能



1. 根节点渲染

   ​	

   ```javascript
   import Vue from 'vue'
   import App from './App'
   
   new Vue({
   	el:'#app',   #挂载点
       render: fuction(h){
       return h(App)
   	}
   })
   ```

   ```html
   <template>  <!--模板文件  -->
       <div>
           
            <!-- 将hello文本中的标签解析为字符串 -->
       <p v-text="hello">
           
           </p>
           
     <!-- 将hello文本中的标签解析为HTML -->
       <p v-html="hello">
           {{ num+1 }}
           {{ status? 'success','fail' }}
            </p>
       <ul>       
           <!--v-for内还可以申明index变量，(item,index)  index表示数组项下表  -->
           
           <!-- 还可以使用属性绑定方法 赋值class    :class="{odd:index %2}"     odd后面的:为选择器，为true/1时赋值给Oddclass-->
           <!--  当数组变成对象时，v-for同样可遍历对象，参数可设置为(value,key) 则value值为对象参数，key为对应的值  -->
   
      	<li v-for="item in list">
           {{ item.name }} - {{ item.price }}
           </li></ul>
       </div>
       <div>
           <componet-a></componet-a>  
           <!--  html内渲染子组件componet  -->
       </div>
       <button  :onclick="addItem">
         addItem   <!--自定义点击出发事件addItem -->
       </button>
       
       </template>
   
   
   <script>
       import componetA from './componets/a'
       #引入子组件componets
       
       #导出
       export default{ 
           methods:{
               addItem () {
                   #addItem方法内可以调用this的属性和方法
                  console.log(this.list)
                   this.list.push({name:'pinapple',price:222})
                   #push将引发页面渲染更新, list的=赋值操作不会更新渲染，而Vue.set全局赋值操作会更新
               }
           }
           componets:{ componetA },  #声明所有子组件
           data () {  
           #省略了返回的函数，完整为 data: fucntion(){retrun ~~~}
               return {
                   hello:'<span>world</span>',
    num:1,
    status:true,
                   #对象数组list
                   list:[
                   {
                   name:'apple',
                   price:34
               },
                   {name:'banana',
                   price:66}
                   ]
                   
               }
           }
           
       }
       
   </script>
   ```


```html
<!-- 子组件Componet  -->
<template>    
	<div>
        {{ hello }}
        <button @click=emitMyEvent>
            
        </button>
    </div>
</template>


<script>
export default{
    data () {
        return {
            hello :'I am component'
        }
    },
    methods:{
        emitMyEvent () {
            this.$emit('my-event',this.hello)
            #可通过emit方法来将子组件的参数(this.hello)传递给父组件（当事件my-event发生时，父节点需要用 @my-event来监听该事件  即v-on:my-event
            #可通过props方法来将需要传输给子节点的参数来进行声明
            #   props:['number']  子节点中属性   {{ number }} 子节点渲染
            #	<com-a number=5></com-a>  父节点中传输参数number
            #亦可使用动态绑定传变量参数
        }
    }

}</script>
```





```html
<input v-model:"myValue">  
<!--  用户输入与myValue双向绑定，myValue变量值变化时input内容变化，input用户输入变化时myValue亦变化  --> 
<!-- 初始值可通过myValue赋值来设置  -->

<input v-model:"myBox" type="checkbox" value="选项1">
<input v-model:"myBox" type="checkbox" value="选项2">
<input v-model:"myBox" type="checkbox" value="选项3">
<!-- 多选框 -->
<!-- myBox需要根据其类型来申明初始值，比如这里为空数组[]

<input v-model:"myBox2" type="ratio" value="选项1">
<input v-model:"myBox2" type="ratio" value="选项2">
<input v-model:"myBox2" type="ratio" value="选项3">
<!-- 单选框 -->
```



```html
<!--  选项列表 ，蛮重要的哦  -->

1.data内需要声明好两项属性，分别是selection和selectOptions，前者初始值为null，后置初始值存放下拉菜单各项的显示值

selection:null,
selectOptions:[
{
	text:'apple'
	value:0
},
{
	text:'banana'
	value:1
}
]

2.模板标签<template></template>内需要有<select>   
</select>标签

<select v-model='selection'>
    <option v-for="item in selectOptions :value=item.value">{{ item.text }}</option>
    <!--  特别注意！变量赋值给标签内需要动态绑定  -->
    <option value='2'>2</option>
</select>

```

```javascript
#监听变量值的改变--watch方法，当myVal改变则执行日志输出函数

watch:{
    myVal: function (val.oldVal){
        console.log(val,oldVal)
    }
}

```





```html
<!--  插槽功能  -->

1.子组件com-a
<slot name="header"></slot>
<slot name="footer"></slot>

2.父组件
<template>
    <com-a>
<p slot="header">
   myheader</p>
    
<p slot="footer">
    myfooter</p>
        
    </com-a>
</template>

<!--  用插槽将公用的头尾部插入子节点
```



## Vue.js 高级功能

### 过渡，动画

```html
<button v-on:click="show =! show">
    Toggle
</button>

<!-- 改变show变量值来切换动画展示 -->

<div>
    <transition name="fade" mode="out-in">
    <p v-show="show">
        i am show
    </p>
        </transition>  <!-- 添加渐隐的动画效果(transition内置组件)  -->
</div>



<!--  Css样式变换  -->
<style>
    .fade-enter-active , .fade-leave-active {
        transition : opacity .5s ease-out;
    }
    
    .fade-enter , .fade-leave-active{
        opacity: 0; <!--设置入点和出点的透明度为0 -->
    }
</style>
 
```





```html
<div :is="currentView"></div>
<!-- 动态挂载组件，可利用method方法来改变currentView变量进而改变显示的子组件  -->
<button :click="togggleCom">Toggle    
</button>

```





## Vue.js项目



### Vue中router的参数传递

​	

```javascript
let router=new VRouter({
    mode:'history',
    routes:[
        {
            path:'apple/:color',
            component:Apple
        }
    ]
    
})
```



​	**routes内数组存放所有的路由Mapping，每个路由Mapping为一个对象，相互之间由逗号分隔，路由Mapping内部由path属性，component属性组成。**

​	**若需要传递URL中的参数，需要通过：color的形式进行传递。****

​	**组件内部获取参数通过this.$route.params来获取URL中传递的参数**





