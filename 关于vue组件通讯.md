# 关于vue组件通讯





[TOC]

## 01. 父子组件通讯

###props

使用`props`传递数据作用域是孤立的，它是父组件通过模板传递而来，想接收到父组件传来的数据，需要通过`props选项`来进行接收。

子组件需要显示的声明接收父组件传递来的数据的`数量`，`类型`，`初始值`。

简单的接收可以通过数组的形式来进行接收。

**父组件**
```html
<template>
   <div>
      <demo :msg='msgData' :math = 'mathData' ></demo>
   </div>
</template>
```
```javascript
<script>
import Demo from './Demo.vue'
   export default {
     data () {
       return  {
          msgData:'给子组件传值',
          mathData : 2
       }
     },
     components : {
       Demo
     }
   }
</script>
```
**子组件**
```html
<template>
  <div>
     <p>{{msg}}</p>
     <p>{{math}}</p>
  </div>
</template>
```
```javascript
<script>
export default {
  name: 'demo',
  props: [ 'msg' , 'math'],
}
</script>
```

在子组件中需要通过显示定义好需要从父组件中接收那些数据。

同样的在父组件中在子组件模板中过v-bind来传递子组件中需要显示接收的数据。

语法： ：== v-bind(是封装的语法糖) ：msg = msgData

- msg 第一个参数必须要与子组件的 props 同名
- msgData 则是父组件中需要向子组传递的数据
props 可以显示定义一个或一个以上的数据，对于接收的数据，可以是各种数据类型，同样也可以传递一个函数。
**父组件**
```html
<template>
   <div>
      <demo :fn = 'myFunction'  ></demo>
   </div>
</template>
```
```javascript
<script>
import Demo from './Demo.vue'
   export default {
     components : {
       Demo
     },
     methods: {
       myFunction () {
           console.log('vue')
       }
     }
   }
</script>
```
**子组件**
```html
<template>
  <div>
     <button @click='fn'>按钮</button>
  </div>
</template>
```
```javascript
<script>
export default {
  name: 'demo',
  props: [ 'fn' ],
}
</script>
```
同样，在父组件中也可以向子组件中传递一个`function`，在子组件同样也可以执行父组件传递过来的 myFunction 这个函数。

**字面量语法和动态语法**

对于字面量语法和动态语法，初学者在父组件模板中向子组件中传递数据时加和不加 v-bind 有什么区别，同时会引起什么错语等问题会感觉迷惑。

**在子组件模板上传递数据时加v-bind意味着什么？**

`v-bind:msg = 'msg'` 通过 v-bind 进行传递数据并且传递的数据并不是一个字面量，双引号里的解析的是`一个表达式`，同样也可以是实例上定义的数据和方法(其实就是引用一个变量）"。
`msg='11111'` 没有 v-bind 的模式下只能传递一个字面量，这个字面量只限于 String 类量，字符串类型。
注意：
虽然通过字面量模式下，传任何类型都会被转成字符串类型，但是在子件接收的时候可以通过` typeof `去进行类型检测。

**子组件模板props定义**

camelCased (驼峰式) 和 kebab-case (短横线隔开式) 两者都可以。为了直观性，规范性还是推荐 kebab-case (短横线隔开式)。

**对象传递简写**

props 原子化可以让整体代码逻辑和向外暴露需要传递数据的接口非常清晰，但是同样可以把子组件需要接收的 props 在父组件中以一个对象进行传递。

当传递的数量一旦多到已经让原子化不再结构清晰的时候，通过一个对象传递显得更为简洁明了。

**父组件**
```html
<template>
   <div>
      <demo v-bind= 'msg'  ></demo>
   </div>
</template>
```
```javascript
<script>
import Demo from './Demo.vue'
   export default {
     components : {
       Demo
     },
     data () {
       return {
         msg : {a:1,b:2}
       }
     }
   }
</script>
```

**子组件**
```html
<template>
  <div>
     <button>按钮</button>
  </div>
</template>
```
```javascript
<script>
export default {
  name: 'demo',
  props: ['a','b'],
  created () {
     console.log(this.a)
     console.log(this.b)
  },
}
</script>
```
`<demo v-bind= 'msg' ></demo>` 内部发生了什么？

在子组件模板内部对其`进行了一个封装`，把其展开则跟 props 原子化原理是一个原理

`<demo :a='a' :b='b' ></demo>`

通常情况下建议使用第二种，props 原子化。



##02.`$on` ，`$emit`， `v-on`三者关系

###关于`$emit`与`$on`的关系
 >此文用家庭来描述`$emit`与`$on`的关系
 
- 使用`$on(eventName)`监听事件
- 使用`$emit(eventName)`触发事件

如果把`Vue`看成一个家庭（相当于一个单独的`components`)，女主人一直在家里指派`($emit)`男人做事，而男人则一直监听`($on)`着女士的指派`($emit)里eventName`所触发的事件消息，一旦 `$emit `事件一触发，`$on `则监听到 `$emit `所派发的事件，派发出的命令和执行派执命令所要做的事都是一一对应的。

**表达式：**

`vm.$emit(event,[...args])`

**参数：**
- `{string} event`
- `[...args]`

触发当前实例上的事件。附加参数都会传给监听器回调。

`vm.$on(event,callback)`

监听当前实例上的自定义事件。事件可以由 `vm.$emit `触发。回调函数会接收所有传入事件触发函数的额外参数。

```html
<template>
  <div>
      <p @click='emit'>{{msg}}</p>
  </div>
</template>
```
```javascript
<script>
export default {
  name: 'demo',
  data () {
      return {
         msg : '点击后女人派发事件'
      }
  },
  created () {
      this.$on('wash_Goods',(arg)=> {
          console.log(arg)
      })
  },
  methods : {
      emit () {
         this.$emit('wash_Goods',['fish',true,{name:'vue',verison:'2.4'}])
      }
  }
}
</script>
```
以上案例说了什么呢？在文章开始的时候说了` $emit`的（eventName）是与 `$on(eventName) `是一一对应的，再结合以上两人在组成家庭的之前，女人会给男人列一个手册，告诉男人我会派发` $(emit) `那些事情，男人则会在家庭组成之后`$on(eventName)`应该如何做那些事情。

进一步解释：

参数：

- ` {string} event`（必填）
 1> 第一个参数则是所要派发的事件名，必须是String类型的。
 2> 故事中就是要告诉男人所需要执行的事情。

- `[...args]`（非必填）
第二个参数是一个任何数据类型，如果我们需要传入多个不同的数据类型，则可以写入数组中，像这样[object,Boolean,function,string,...]，只要传一个参数，我们则可以直接写入。
     
>this.$emit('wash_Goods',`'fish'`)

继续延续故事，当女人派发的事情多了，我相信作为男人也会觉得很烦，一旦听到事件的时候肯定会很烦躁，总会抱怨两句。
如果女人在组成家庭之前，告诉男人将要监听那些事情，如果做一件事就抱怨一次，启不是多此一举，所以我们可以通过`Array<string> event`把事件名写成一个数组，在数组里写入你所想监听的那些事件，使用共享原则去执行某些派发事件。

```html
<template>
  <div>
      <p @click='emit'>{{msg}}</p>
      <p @click='emitOther'>{{msg2}}</p>
  </div>
</template>
```
```javascript
<script>
export default {
  name: 'demo',
  data () {
      return {
         msg : '点击后女人派发事件',
         msg2 : '点击后女人派发事件2',
      }
  },
  created () {

      this.$on(['wash_Goods','drive_Car'],(arg)=> {
          console.log('事真多')
      })
      this.$on('wash_Goods',(arg)=> {
          console.log(arg)
      })
      this.$on('drive_Car',(...arg)=> {
          console.log(BMW,Ferrari)
      })
  },
  methods : {
      emit () {
         this.$emit('wash_Goods','fish')
      },
      emitOther () {
         this.$emit('drive_Car',['BMW','Ferrari'])
      }
  }
}
</script>
```
以上案例说明了当女人无论是派发`drive_Car`或者是`wash_Goods`事件，都会打印出`事真多`，再执行一一对应监听的事件。
通常情况下，以上用法是毫无意思的。在平常业务中，这种用法也用不到，通常在写组件的时候，让`$emit在父级作用域中`进行一个触发，通知子组件的进行执行事情。接下来，可以看一个通过在父级组件中，拿到子组件的实例进行派发事件，然而在子组件中事先进行好派发事件监听的准备，接收到一一对应的事件进行一个回调，同样也可以称之为封装组件向父组件暴露的接口。

###`$bus`使用中常见的两个问题
- A页面第一次进行`$emit`通知，B页面第一次没有`$on`接收到，其后的通知才可以接收到。
- A页面只`$emit`通知了一条消息，B页面第一次`$on`接收到一条消息，第二次接收到两条消息，第三次接收到三条消息，每一次都是发现之前的on事件好像没有被撤销掉一样，越累计越多。

###解决方法
-  A页面在`before Destory` 钩子中进行`$emit`传值；

-  B页面在接收到数据后，在`before Destory` 钩子中进行`$off`销毁；

原理： [点击这里](https://www.jianshu.com/p/fde85549e3b0)

###关于`v-on`与`$emit`的关系

**父组件可以在使用子组件的引入模板直接用 v-on 来监听子组件触发的事件。**

v-on接着用故事直观的说法就是，在家里装了一个电话，父母随时听着电话，同样也有一本小册子，在组成家庭之前，也要去监听那些事。

`不能用 $on 监听子组件释放的事件，而必须在模板里直接用 v-on 绑定。`

**区别：**`$emit和$on只能作用在一一对应的同一个组件实例，而v-on只能作用在父组件引入子组件后的模板上。`

就像下面这样： `<children v-on:eventName="callback"></children>`

```html
<div id="counter-event-example">
  <p>{{ total }}</p>
  <button-counter v-on:increment="incrementTotal"></button-counter>
  <button-counter v-on:increment="incrementTotal"></button-counter>
</div>
```
```javascript
Vue.component('button-counter', {
  template: '<button v-on:click="incrementCounter">{{ counter }}</button>',
  data: function () {
    return {
      counter: 0
    }
  },
  methods: {
    incrementCounter: function () {
      this.counter += 1
      this.$emit('increment')
    }
  },
})
new Vue({
  el: '#counter-event-example',
  data: {
    total: 0
  },
  methods: {
    incrementTotal: function () {
      this.total += 1
    }
  }
})
```
这样的好处在哪里？虽然 `Vue `是进行数据单向流的，但是子组件不能直接改变父组件的数据，(也不是完全不能，但不推荐用)，标准通用明了的用法，则是通过父组件在子组件模板上进行一个` v-on` 的绑定监听事件，同时再写入监听后所要执行的回调。
在`counter-event-example`父组件里，声明了两个`button-count`的实列，通过` data` 用闭包的形式，让两者的数据都是单独享用的，而且`v-on `所监听的` eventName` 都是当前自己实列中的` $emit` 触发的事件，但是回调都是公用的一个` incrementTotal `函数，因为个实例所触发后都是执行一种操作！
如果你只是想进行简单的进行父子组件基础单个数据进行双向通信的话，在模板上通过 `v-on `和所在监听的模板实例上进行 `$emit `触发事件的话，未免有点多余。通常来说通过 `v-on` 来进行监听子组件的触发事件的话，我们会进行一些多步操作。

**子组件**
```html
<template>
  <div>
      <p @click='emit'>{{msg}}</p>
  </div>
</template>
```
```javascript
<script>
export default {
  name: 'demo',
  data () {
      return {
         msg : '点击后改变数据',
      }
  },
  methods : {
      emit () {
         this.$emit('fromDemo')
      },
  }
}
</script>
```
**父组件**
```html
<template>
  <div class="hello">
     <p>hello {{msg}}</p>
     <demo v-on:fromDemo='Fdemo'></demo>
  </div>
</template>
```
```javascript
<script>
import Demo from './Demo.vue'
export default {
  name: 'hello',
  data () {
    return {
       msg: '数据将在一秒后改变'
    }
    
  },
  methods: {
    waitTime() {
      return new Promise(resolve=>{
        setTimeout(()=> {
            this.msg = '数据一秒后改变了'
            resolve(1)
        },1000)
      })
    },
    async Fdemo () {
        let a = await this.waitTime();
        console.log(a)
    }
  },
  components : {
     Demo
  }
}
</script>
```
从上面 demo 可以看出当子组件触发了 `fromDemo `事件，同时父组件也进行着监听。
当父组件接收到子组件的事件触发的时候，执行了`async `的异步事件，通过一秒钟的等秒改变` msg`，再打印出回调后通过 `promise `返回的值。
接下来想通的此例子告诉大家，这种方法通常是通过监听子组件的事件，让父组件去执行一些多步操作，如果我们只是简单的示意父组件改变传递过来的值用此方法就显的多余了。

我们进行一些改动：

children
```html
<template>
  <div>
      <p @click='emit'>{{msg}}</p>
  </div>
</template>
```
```javascript
<script>
export default {
  name: 'demo',
  props: [ 'msg' ],
  methods : {
      emit () {
         this.$emit('fromDemo','数据改变了')
      },
  }
}
</script>
```

parent
```html
<template>
  <div class="hello">
     <demo v-on:fromDemo='Fdemo' :msg='msg'></demo>
  </div>
</template>
```
```javascript
<script>
import Demo from './Demo.vue'
export default {
  name: 'hello',
  data () {
    return {
       msg: '数据没有改变'
    }
  },
  methods: {
    Fdemo (arg) {
      this.msg = arg 
    }
  },
  components : {
     Demo
  }
}
</script>
```
上面` demo `中子组件从父组件接收一个` msg `数据，但是想点击按钮的时候，改变父组件的` msg`，进行父组件的数据改动，同时再次改变子组件的 `msg`。

###通过`$emit `，`v-on `，`$on` 三者结合使用

简单地讲就是`b->a->c`，意思就是`a`去通知`b`，`b`对a进行监听，当`b`监听到以后，进行向`c`触发，`c`的内部在进行监听操作。

**同级子组件 First**
```html
<template>
  <div>
    <p @click="$emit('fromFirst','来自A组件')">first组件</p>
  </div>
</template>
```
```javascript
<script>
  export default {
    name: 'first'
  }
</script>
```
先定义一个同级子组件，当点击的时候向外触发一个`eventName`为`fromFirst`的事件,传递一个来自A组件的参数,开始了`a->b`的通知,
接下来就是`b`的监听，监听后`b`进行一个通知。


**父组件**
```html
<template>
   <div>
     <p>父组件</p>
     <first v-on:fromFirst="hanlderFromA"></first>
     <second ref="second"></second>
  </div>
<template>
```
```javascript
<script>
     import First from './first.vue'
  import Second from './second.vue'
  export default {
    name: 'login',
    components: {
      First,
      Second
    },
    methods: {
      hanlderFromA (Bmsg) {
        let second = this.$refs.second
        second.$emit('fromLogin', Bmsg)
      }
    }
  }
<script>
```
**在此处强调一点：**
`v-on`与`$emit`是进行`父子组件事件通信`，作用在`父子组件`两个层面上，在`First`组件模版上进行一个`v-on监听`，一旦监听到触发`fromFirst` 事件，则进行`hanlderFromA`函数。

**同级子组件Second**

```html
<template>
  <div>
    <p>{{Bmsg}}</p>
    <p>second组件</p>
  </div>
</template>
```
```javascript
<script>
  export default {
    name: 'second',
    created () {
      this.$on('fromLogin', (Bmsg) => {
        this.Bmsg = Bmsg
        console.log('通信成功')
      })
    },
    data () {
      return {
        Bmsg: ''
      }
    }
  }
</script>
```
OK!

##03.attrs与listeners

简单来说就是一个深层次数据传递，或者进行某种交互时需要向上通知或父层组件数据改变时，不进行跨路由之间页面的共享。

###先看$attr与inheritAttrs之间的关系

默认情况下父作用域的不被认作` props 的特性绑定 (attribute bindings)` 将会“回退”且作为普通的 HTML 特性应用在子组件的根元素上。当撰写包裹一个目标元素或另一个组件的组件时，这可能不会总是符合预期行为。通过设置 `inheritAttrs`到 false，这些默认行为将会被去掉。而通过 (同样是 2.4 新增的) 实例属性` $attrs `可以让这些特性生效，且可以通过 v-bind 显性的绑定到非根元素上。

**子组件**
```html
<template>
   <div>
      {{first}}
   </div>
</template>
```
```javascript
<script>
export default {
   name: 'demo',
   props: ['first']
}
</script>
```
**父组件**
```html
<template>
  <div class="hello">
     <demo :first="firstMsg" :second="secondMessage"></demo>
  </div>
</template>
```
```javascript
<script>
  import Demo from './Demo.vue'
  export default {
    name: 'hello',
    components: {
      Demo
    },
    data () {
      return {
        firstMsg: 'first props',
        secondMessage: 'second props'
      }
    },
  }
</script>
```
父组件在子组件中进行传递 firstMsg 和secondMsg 两个数据，在子组件中，应该有相对应的 props 定义的接收点，如果在 props 中定义了，你会发现无论是 firstMsg 和 secondMsg 都成了子组件的接收来的数据了，可以用来进行数据展示和行为操作。
虽然在父组件中在子组件模版上通过 props 定义了两个数据，但是子组件中的 props 只接收了一个，只接收了 firstMsg，并没有接收 secondMsg，没有进行接收的此时就会成为子组件根元素的属性节点。

interitAttrs = false 发生了什么 ？
```html
<template>
   <div>
      {{first}}
   </div>
</template>
```
```javascript
<script>
export default {
   name: 'demo',
   props: ['first'],
   inheritAttrs: false,
}
</script>
```
对子组件进行一个改动，我们加上 `inheritAttrs: false`，从字面上的翻译的意思，取消继承的属性，然而 `props `里仍然没有接收 `seconed`，发现就算 `props` 里没有接收` seconed`，在子组件的根元素上并没有绑定任何属性。

>$attrs

包含了父作用域中不被认为 (且不预期为) props 的特性绑定 (class 和 style 除外)。当一个组件没有声明任何 props 时，这里会包含所有父作用域的绑定 (class 和 style 除外)，并且可以通过 v-bind="$attrs" 传入内部组件——在创建更高层次的组件时非常有用。
在前面的例子中，子组件`props中并没有接受seconed`，设置选项` inheritAttrs： false`，同样也不会作为根元素的属性节点，整个没有接收的数据都被` $attr` 实例属性给接收，里面包含着所有父组件传入而子组件并没有在 Props里显示接收的数据。
为了验证事实，可以在子组件中加上
```javascript
created () {
       console.log(this.$attrs)
    }
```
打印出来则是一个对象` {second: "secondMsg", third: "thirdMsg"}`

想要通过 `$attr` 接收，但必须要保证设置选项 `inheritAttrs: false`，不然会默认变成根元素的属性节点。
开头说了，最有用的情况则是在深层次组件运用的时候，创建第三层孙子组件，作为第二层父组件的子组件，在子组件引入的孙子组件，在模版上把整个` $attr `当数作数据传递下去，中间则并不用通过任何方法去手动转换数据。

**子组件**
```html
<template>
   <div>
      <next-demo v-bind="$attrs"></next-demo>
   </div>
</template>
```
**孙子组件**
```html
<template>
  <div>
      {{second}}{{third}}
  </div>
</template>
```
```javascript
<script>
  export default {
     props : [ 'second' , 'third']
  }
</script>
```
孙子组件在 props 接收子组件中通过` $attr `包裹传来的数据，同样是通过父组件传来的数据，只是在子组件用了`$attrs`进行了统一接收，再往下传递，最后通过孙子组件进行接收。
以此类推孙子组件仍然不想接收，再传入下级组件，我们仍然需要对孙子组件实力选项进行设置选项` inheritAttrs: false`，否则仍然会成为孙子组件根元素的属性节点。
从而利用 `$attrs`来接收` props`为接收的数据再次向下传递是一件很方便的事件，深层次接收数据我们理解了，那从深层次向浅层次请求改变数据如何实现。意思就是让顶层数据和最底层数据进行一个双向绑定。

###$listeners

listeners 可以认为是监听者。

`$listeners` 和` $attrs `两者表面层都是一个意思，`$attrs` 是向下传递数据，`$listeners `是向下传递方法，通过手动去调用 `$listeners `对象里的方法，原理就是 `$emit `监听事件，`$listeners` 也可以看成一个包裹监听事件的一个对象。

**父组件**

```html
<template>
  <div class="hello">
     {{firstMsg}}
     <demo v-on:changeData="changeData" v-on:another = 'another'></demo>
  </div>
</template>
```
```javascript
<script>
  import Demo from './Demo.vue'
  export default {
    name: 'hello',
    components: {
      Demo
    },
    data () {
      return {
        firstMsg: '父组件',
      }
    },
    methods: {
      changeData (params) {
         this.firstMsg = params
      },
      another () {
        alert(2)
      }
    }
  }
</script>
```
在父组件中引入子组件，在子组件模板上面进行 changeData 和 another 两个事件监听，其它这两个监听事件并不打算被触发，而是直接被调用，再简单的理解则是向下传递两个函数。

```html
<template>
   <div>
      <p @click="$emit('another')">子组件</p>  
      <next-demo  v-on='$listeners'></next-demo>
   </div>
   
</template>
```
```javascript
<script>
import NextDemo from './nextDemo.vue'
export default {
   name: 'demo',
   components: {
       NextDemo
   },
   created () {
     console.log(this.$listeners)
   },
}
</script>
```
在子组件中，引入孙子组件 nextDemo。在子组件中像 `$attrs `一样，可以用 `$listeners` 去整体接收监听的事件，`{changeData: ƒ, another: ƒ}`以一个对象去接收，此时在父组件中子组件模板上监听的两个事件不但可以被子组件实例属性 `$listeners `去整体接收，并且同时可以在子组件进行触发。
同样在孙子 nextDemo 组件中，继续向下传递，通过` v-on` 把整个 `$listeners` 所接收的事件传递到孙子组件中，只是通过 `$listeners `把其所有在父组件拿到监听事件一并通过 `$listeners` 一起传递到孙子组件里。

**孙子组件**
```html
<template>
  <div class="hello">
      <p @click='$listeners.changeData("change")'>孙子组件</p>
  </div>
</template>
```
```javascript
<script>
export default {
  name: 'demo',
  created () {
      console.log(this.$listeners)
  },
}
</script>
```
依然能拿到从子组中传递过来的`$listeners`所有的监听事件，此时并不是通过`$emit`去触发，而是像调用函数一样，`$emit`只是针对于父子组件的双向通信，`$listeners`包了一个对象，分别是 changeData 和 another，通过`$listeners.changeData('change')`等于直接触发了事件，执行监听后的回调函数，就是通过函数的传递，调用了父组件的函数。
通过 `$attrs `和 `$listeners` 可以很愉快地解决深层次组件的通信问题，更加合理的组织你的代码。

###本文参考于以下作者
作者：混元霹雳手
链接：https://juejin.im/post/5bd97e7c6fb9a022852a71cf
来源：掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。


