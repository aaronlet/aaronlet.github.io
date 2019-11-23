---
layout:     post
title:      TypeScripty应用
subtitle:   TypeScripty application
date:       2019-10-14
author:     Aaron
header-img: img/blog/post-bg-typescript.jpg
catalog: true
tags:
    - TypeScript
    - Vue
---

`TypeScript`推出已经很长时间了，在`Angular`项目中开发比较普遍，随着`Vue 3.0`的即将推出，`TypeScript`在`Vue`项目中使用也即将成为很大的趋势，笔者也是最近才开始研究如何在`Vue`项目中使用`TypeScript`进行项目的开发。

> ### 准备

阅读博客前希望读者能够掌握如下技能，文章中也会相对应的讲解一些对于`TypeScript`基础进行讲解。本篇博客基于`Vue cli 3.0`实践，若读者使用的是`Vue cli 2.0`的话，需要对其`webpack`配置进行更改，本文不进行讲解，请读者自行百度，并进行更改。

1. `Vue cli 3.0`环境
2. `Vue`基础应用

读者完全不用担心不懂`TypeScript`基础，本文会一一对其基础进行简单的讲解。

> ### TypeScript基础

创建完项目之后，接下来就可以对项目进行开发，在使用`TypeScript`开发与使用`JavaScript`还是有很大的区别，打开`HelloWorld.vue`文件内容如下：

```html
<template>
  <div></div>
</template>

<script lang="ts">
import { Component, Prop, Vue } from 'vue-property-decorator';

@Component
export default class HelloWorld extends Vue {

}
</script>

<style lang="scss">

</style>
```

读者若经常开发`Vue`项目会发现与之间的`.vue`有了一些变化，主要是在`<script>`标签部分。使用`ES6`开发项目时样文件格式如下：

```html
<template>
  <div></div>
</template>

<script>
export default {

}
</script>

<style lang="scss" scoped></style>
```

对比一下两者之间还是有很大的区别的。原有写法是直接使用`export default`导出一个对象，然而`TypeScript`而是使用`class`如果对`React`的同学应该很熟悉，有点类似于`React`创建的组件的写法了。对于`template`的使用其实与之前的写法是一样的。唯一改变得就是对于`<script>`部分，如果想在项目中使用`TypeScript`需要在`<script>`添加标识：`<script lang="ts">`告知解析器需要使用`TypeScript`进行解析。

在`Vue`中使用`TypeScript`编写项目，需要依赖于`Vue`提供的包`vue-property-decorator`以达到对`TypeScript`的支持。需要使用`@`修饰器其中`Vue`功能才能正常使用。无论在页面还是组件时，一定要使用`@Component`对导出类进行修饰，否则无法正常使用某些功能(如：双向绑定...)。接下来就来实现如何在模板中渲染数据，这里将不再使用`data`而是直接在`class`中写入变量。

关于`vue-property-decorator`的用法会在下面详细介绍。

```html
<template>
  <div>
    <h1>{{message}}</h1>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';

@Component
export default class HelloWorld extends Vue {
    private message:string = "Hello,TypeScript";
}
</script>
```

仔细观察一下`message`的声明，与其之前使用`data`时已经完全不一样了有没有，除了没有使用`data`以外，还有很大的区别：

```
修饰符  变量名：数据类型 = 数据
private message:string = "Hello,TypeScript";
```

> ##### 修饰符

如果没有接触过强类型语言的话对于修饰符可能会感觉到一丝丝的陌生，在`es6`的`class`中已经有了修饰符的概念，在`es6`的`class`中只有一个修饰符`static`修饰符，表示该类的静态方法，但是在`TypeScript`中添加了很多修饰符：

- public：所有定义成`public`的属性和方法都可以在任何地方进行访问(默认值)
- private：所有定义成`private`的属性和方法都只能在类定义内部进行访问
- protected：多有定义成`protected`的属性和方法可以从类定义内部访问，也可以从子类中访问。
- readonly：`readonly`关键字将属性设置为只读的。只读属性必须在声明时或构造函数里被初始化。

修饰符不但可以修饰属性，还可以用来修饰方法，若在写入属性和方法时没有使用修饰符对其进行修饰的话默认则会是`public`。

```javascript
class A {
    public message1:string = "Aaron";
    private message2:string = "Angie";
    protected message3:string = "Tom";
    readonly message4:string = "Jerry";
    message5:string = "Sony";
}
class B extends A {
    constructor(){
        //  这里因为受修饰器影响无法读取到`message2`
        //  若写入 message2 在编码工具中则会显示红色波浪线提示错误
        //  Property 'message2' is private and only accessible within class 'A'.
        let {message1,message3,message4,message5} = this;
        console.log(message1,message3,message4,message5);
        //  若去更改 message4 的数据则会抛出下面的错误
        //  Cannot assign to 'message4' because it is a read-only property.
        //  this.message4 = "Sun";
    }
}
```

> ##### 数据类型

在声明变量或属性时需要规定其对应的数据类型是什么，这样一来就可以对其变量以及属性进行更加严格的管理了。

`TypeScript`中提供了一下基本类型：

- string: 字符串
- number：数字
- boolean：布尔值
- array：数组
- enum：枚举，个人理解枚举类型并不陌生，它能够给一系列数值集合提供友好的名称，也就是说枚举表示的是一个命名元素的集合
- any：任意类型
- void:没有任何类型，通常用于函数没有返回值时使用

```javascript
//  定义number型变量
let test:number = 1;
//  定义number型数组
let arr:[]number = [];
let arr1:Array<number> = [];
//  定义对象数据，且对象只能有name字段
let a:{name:string}[] = [];
```

在进行变量或属性声明的时候，一旦使用了数据类型限定的话，如果试图想要对其数据类型进行更改的话，就会提示一个错误，如果类型是多种情况，可以使用`|`进行分割。若不确定使用哪种类型则可以使用`any`。

```javascript
let a:string = "";
// Type '1' is not assignable to type 'string'.
a = 1;

let b:(string|number) = "";
b = 1;
```

注意：**在声明变量时不一定要使用数据限定，如果没有使用数据限定，`TypeScript`则会根据默认值进行数据类型推论作为其变量的数据类型。**

```javascript
let a = 1;
//  Type '"Aaron"' is not assignable to type 'number'.
a = "Aaron";
```

> ##### 函数

对数据已经有了一定了解，之后就是对于函数的说明，在项目开发过程中唯一不可缺少的就是函数，同样的是在`class`中写入的方法，同样也需要使用修饰符，其用法与属性一致。

在函数后面添加了`void`标识符，标识当前函数没有函数返回值，可以根据当前函数的返回值，更改其返回类型。一旦规定了函数的返回值类型，就无法再返回其他的数据的类型，如果强行返回其他类型的话，则会抛出错误。

```html
<template>
  <div>
    <h1>{{message}}</h1>
    <el-button @click="clickMe"></el-button>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';

@Component
export default class HelloWorld extends Vue {
    private message:string = "Hello,TypeScript";
    
    private clickMe():void{
        alert("Aaron!")
        //  Type '"Angie"' is not assignable to type 'void'.
        //  return "";
    }
}
</script>
```

有的时候函数会带有一些参数，该如何去处理这些参数呢？同样当在接收参数(形参)处同样也要规定其数据类型，一旦写入参数在调用该方法时就必须传入该参数，某些参数若不是必填的，则要在定义其类型前使用`?`，这样该参数就不是必填项，可以有可无，`?`与默认值不能共存只能使用一个，一旦使用默认值的话，改参数就是必定存在的了，不会出现不传入的情况了。

```javascript
const fn = (name:string,age?:number) => {
    console.log(`my name is ${name}, age ${age || 18}`)
}
fn("Aaron",3);  //  my name is Aaron, age 3
fn("Angie");    //  my name is Angie, age 18
fn();           //  Expected 1-2 arguments, but got 0.
```

开发过程中难免会遇到一些特殊的函数，函数内部无法确定其参数个数，但是传入的类型都是统一类型的，在`JavaScript`中提供了`arguments`属性，对于`TypeScript`有其他处理方式：

```javascript
const fn = (...foo:number):number => {
    let res:number = 0;
    //  如果不使用foo，可以替换成arguments也是一样的
    for(let i = 0;i<foo.length;i++){
        res += foo[0];
    }
    return res;
}
fn(1,2,3,4,5,6,8,7,9)   //  45
```

笔者也尝试使用过`arguments`，但是如果使用`arguments`时会抛出一个错误`Cannot find name 'arguments'.`

函数的重载，在没有接触过强语言的话可能是很陌生的，什么是重载？[重载](https://note.youdao.com/)就是函数或者方法有相同的名称，但是参数列表不相同的情形，这样的同名不同参数的函数或者方法之间，互相称之为重载函数或者方法。（节选自百度百科）

```javascript
function abs(name:string):string;
function abs(name:{name:string}):string;
function abs(name:string|{name:string}):string{
  if(typeof name === "object"){
    return name.name;
  }
  return name;
}
abs("Aaron");
abs({name:"Angie"});
```

上述中前两个都属于抽象函数，最后一个函数为对于抽象函数的实现，实现签名必须兼容所有的重载签名，总是在参数列表的最后，接受一个`any`类型或联合类型的参数作为他的参数。如果没有按照实现所传入参数则会抛出错误。

> ##### 泛型

泛型是程序设计语言的一种特性。允许程序员在强类型程序设计语言中编写代码时定义一些可变部分，那些部分在使用前必须作出指明。各种程序设计语言和其编译器、运行环境对泛型的支持均不一样。将类型参数化以达到代码复用提高软件开发工作效率的一种数据类型。泛型类是引用类型，是堆对象，主要是引入了类型参数这个概念。

笔者在最开始接触泛型的时候也是一脸懵逼，因为一直都在听后端的同学说，泛型什么什么的...一直都只是知道有泛型这个东西，但是泛型应该怎么用，想问却又不敢...因为是不知道从何开始问起。

我对于泛型的理解就是去规定一种数据类型，无论是函数接收参数，还是返回结果，或者一种类型的数组，对象，等等等都可以使用泛型进行约束，泛型就是在定义方法不确定需要返回什么样类型的数值，但是当调用函数值时需要去限定返回该类型。

使用泛型时分为两种情况(只会提及`TypeScript`的泛型)，一种是接口泛型，一种是类泛型其实两种方法是类似的。

```javascript
//  类泛型
class User {
  name:string  = "";
  age:number = 0;
}
//  接口泛型
interface User {
  name:string,
  age:number
}

//  获取数据
function getData(){
  // ... axios操作等
  return [{}];
};
//  T 泛型的形参
//  T 可以自定义
function getUsers<T> ():T[]{
  return getData();
}
let users:User[] = getUsers<User>();
```

上面代码中使用`class`和接口实现了两种泛型，两种都是可用的，一般不推荐使用去使用泛型，因为在使用类去做泛型的时候需要对其中的属性进行初始化，否则会抛出错误。

> ##### 接口

上面提到了接口，对于接口也是前端没有涉及的一部分，接口泛指实体把自己提供给外界的一种抽象化物（可以为另一实体），用以由内部操作分离出外部沟通方法，使其能被内部修改而不影响外界其他实体与其交互的方式。

对于接口来说，去定义一些抽象的方法或属性，在使用时对其进行实现，使用`implements`关键字对其接口进行实现，可以同时实现多个接口以`,`分割。

接口可以继承类，但是类不可以继承接口，只能实现接口，如果接口继承类的话，不会继承类的实现只会继承类的签名规范。

```javascript
class Friend {
  public friends:object[] = [];
  getFriends(){
    return this.friends;
  }
}
interface U extends Friend {
  name:string,
  age:number,
  seyHi(name:string):string
}
interface S {
  job:string
}
class User implements U,S {
  public friends:object[] = [];
  constructor(
    public name:string,
    public age:number,
    public job:string){}
  seyHi(name:string):string{
    return `Hi ${name} ~,my name is ${this.name}`;
  }
  getFriends(){
    return this.friends;
  }
}
new User("Aaron",3,"Code");
```

> ##### 类

有关于`TypeScript`的类，与`es6`中的类是一样的，这里着重说一下抽象类。抽象类是对其属性方法进行规定，在继承时对其进行实现。

```javascript
abstract class Animal{
  public name:string;
  constructor(name:string){
      this.name=name;
  }
  //  抽象方法 ，不包含具体实现，要求子类中必须实现此方法
  abstract eat():any;
  //  非抽象方法，无需要求子类实现、重写
  run(){
    console.log('非抽象方法，不要子类实现、重写');
  }
}
class  Dog extends Animal{
  //  子类中必须实现父类抽象方法，否则ts编译报错
  eat(){
    return this.name+"吃肉";
  }
}
```

在类中需要对其内部的属性进行初始化赋值，可以写入默认值，同样也可以根据传入的值进行赋值，也就是在进行实例化的时候传入参数对其属性进行初始化。以下三种方法都可以为类中的属性进行初始化。

```javascript
//  第一种方法
class A {
  public name:string = "Aaron";
}
//  第二种方法
class A {
  public name:string;
  constructor(name:string){
    this.name = name;
  }
}
//  第三种方法
class A {
  constructor(public name:string){}
}
```

> ### Vue中使用TypeScript

上面已经对`TypeScript`的基础做了一些简单的讲解，在开发中是没有问题的了，接下来就开始在`Vue`项目中开始实战。

对于创建项目就不做过多赘述了，只需要在创建项目时选中`TypeScript`即可，其他配置项读者可以根据项目需求自行选择。

当`Vue`中使用`TypeScript`编写项目以后，很多东西发生了变化，上面已经提到了事件与数据，对一些常用的方法进行简单的说明。

> ##### 组件传值

父组件传入子组件的值通过`vue-property-decorator`中`Prop`进行接收，传入的方式与`Vue`中的使用是相同的。

```JavaScript
import { Component, Prop, Vue } from 'vue-property-decorator';
@Component
export default class HelloWorld extends Vue {
  @Prop() private msg!: string;
}
```

> ##### 组件挂载

组件挂载则是在`Component`中进行挂载，除了位置发生了变化，其他并没有任何不同。

```JavaScript
import {Component, Vue } from 'vue-property-decorator';
import Search from '@/components/business/Search.vue';
@Component({
  components: {
    Search
  }
})
export default class CreateItem extends Vue {}
```

通过上述代码已经可以正常使用组件了`<Search/>`。

> ##### 事件消息

事件消息使用`Emit`进行返回事件

注意：

1. 当使用`Emit`时执行事件函数所返回的值，则作为在函数中传给父组件的值。
2. Emit中可以接收参数，第一个参数可以自定义事件名称，参数什么样，在父组件绑定的事件名称必须一致，否则事件不会执行，如果不传入事件名称，则会默认使用子组件中触发事件的函数名称，在父组件调用时则使用`烤串形式`进行拼接。

```JavaScript
import {Component, Vue, Ref, Emit} from 'vue-property-decorator';
@Component
export default class CreateItem extends Vue {
  @Emit("clickMe")  //  执行事件 @clickMe
  private async onClickMe():string{
    return "onClick";
  }
  @Emit             //  执行事件 @on-click-me
  private async onClickMe():string{
    return "onClick";
  }
}
``` 

> ##### Ref

```html
<template>
  <div>
    <button ref="btn"><button/>
  </div>
</template>

<script lang="ts">
import {Component, Vue, Ref} from 'vue-property-decorator';
@Component
export default class CreateItem extends Vue {
  //  定义类型可以根据当前为什么元素写入对应的名称的element
  //  如果不确定元素可以直接写 HTMLElement
  @Ref("btn") readonly formEle!:HTMLButtonElement;
};
</script>
```

> ##### mixins

对于混入的话有两种混入方法，可以依赖于`vue-class-component`模块中的混入，也可以在`Component`中进行混入。

第一种方法：

```javascript
//  混入文件
import { Vue, Component} from 'vue-property-decorator';

@Component
export default class myMixins extends Vue {
  values: string = 'Hello'

  created() {
    console.log("混入第二种方法!!!")
  }

}

//  混入
import HomeMixins1 from "@/mixins/Home1";
@Component({
  mixins:[HomeMixins1]
})
export default class Home extends Vue {
    
}
```

第二种方法：

```JavaScript
//  混入文件
import Vue from 'vue';
import Component from 'vue-class-component';

@Component
export default class HomeMixin extends Vue {
  valueq: string = "Hello"

  created() {
    console.log("混入第一种方法!!")
  }
}
//  混入
import  Component,{mixins}  from 'vue-class-component';
import HomeMixins from "@/mixins/Home";
export default class Home extends mixins(HomeMixins) {}
```

两种混入方法都是可行的，同样都可以混入多种方法，官网推荐使用第二种方法进行混入。

> ##### 过滤器

过滤器分文两种，一种全局过滤器，一种局部过滤器，对于全局过滤器几乎没有发生变化。

**全局过滤器**

全局过滤器写在`main.ts`中

```javascript
Vue.filter('capitalize', function (value:string) {
  if (!value) return ''
  return value.charAt(0).toUpperCase() + value.slice(1)
})
```

**局部过滤器**

局部过滤器写在各个组件`@Component`中。

```JavaScript
import {Component, Vue} from 'vue-property-decorator';

@Component({
  filters:{
    thumb(url:string){
      return url += "/abcd";
    }
  }
})
export default class Home extends Vue {}
```

> ### 项目结构

简单说一下笔者在使用`Vue`开发项目式使用的项目结构，可能不是最好的，但是对于项目的可维护性确实有了很大的提高。在项目开始时需要创建项目结构，`Vue`项目中为了更好的结构化，需要将项目进行分层处理，以达到高度维护的目的。

目录结构：

```
├─api           //  数据请求
├─assets        //  资源
├─components    //  组件
│  ├─basis          //  基础组件
│  └─business       //  业务组件 
├─domain        //  业务
├─interface     //  接口
├─middleware    //  中间件
├─mixins        //  混入
├─style         //  样式
├─store         //  状态管理
├─router        //  路由
└─views         //  视图
```

上面对其项目进行了项目结构进行了划分，若对工程化不太了解的同学可能不太能理解每一层的目的到底是为了什么？他们相互之间又应该如何搭配使用？简单对每一层进行简单的描述。

1. api：其中封装的是请求后端接口数据的请求函数
2. assets：静态资源
3. components：组件，在文件中分为了两个文件夹，业务组件和基础组件
4. domain：项目中的业务逻辑
5. interface：用户TypeScript限制接口数据格式
6. middleware：Vue项目中使用的中间件
7. mixins：需要混入的内容
8. style：样式
9. store：全局状态管理
10. router：路由

这里需要说明一点，在创建项目时如果读者选择了`vue-router`和`vuex`，创建项目后会自动生成`router.ts`和`store.ts`笔者这里并没有使用原有的文件，而是单独对其进行处理。考虑到若项目业务较多可以使用使用文件或文件夹再对其路由和状态管理根据其业务对其进行文件划分，单独维护，这样更加的有利于项目的可维护性。

> ### 总结

文章篇幅过长，用了过长的篇幅讲解了`TypeScript`简单应用，对于`Vue`中的使用也只是简单的讲解了一下。与其之前是很类似的。

文章些许潦草，但是很感谢大家能够坚持读完本篇文章。若文章中出现错误请大家在评论区提出，我会尽快做出改正。
