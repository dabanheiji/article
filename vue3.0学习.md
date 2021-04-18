最近由于客户没有再提过多的需求，所以黑鸡在最近也是闲了下来，虽然现在在公司里使用的是react，但是从心里来说黑鸡还是更喜欢vue，在闲下来的这一段时间内，黑鸡了解了一些php以及laravel框架，但是由于太懒，所以一直没有更新笔记，于是决定在今天摸鱼的时候写点东西。

### vue3.0的数据双向绑定的妙用

在我之前的笔记里说过，vue3.0的代理使用的是ES6中的`proxy`，而不再使用`Object.defineProperty`，两者有一个很大的区别，那就是`Object.defineProperty`只能对对象属性进行绑定，这也就意味着对象的新增属性在vue2.x中我们是监听不到的，这样的话导致的结果就是我们需要给所有的表单项都使用`v-model`来绑定一个属性，且这个属性必须在`data`中定义。比如：

```vue
// vue2.x
<template>
	<form>
        <p>
            <label>
                用户名：
                <input type="text" v-model="form.username"/>
    		</label>
    	</p>
        <p>
            <label>
                密码：
                <input type="text" v-model="form.password"/>
    		</label>
   	 	</p>
        <button @click="submit">提交</button>
    </form>
</template>

<script>
export default {
    data(){
        return {
            form:{
                username:"", //这两项必须在data中出现
                password:""
            }
        }
    },
    methods:{
        submit(){
            console.log(this.form);
            /**
            * 打印结果
            * {username:"xxx",password:"xxx"}
            */
        }
    }
}
</script>
```

这对于一直使用vue的用户来说并没有什么新奇的，因为这都是基本写法，但是一直使用react的黑鸡由于使用react多了，发现像`antd`这种组件的想法是非常棒的，因为我去看了element，iview，antd vue的文档，发现除了antd vue之外其他两个组件库中都必须在data中写出所有表单项绑定的字段，其实黑鸡是不喜欢这种写法的，所以在看了`proxy`后黑鸡就有了一些想法。

首先`proxy`是对整个对象进行代理，这也就意味着整个对象的任何变化都能够被`proxy`监听到基于这一点黑鸡做了一些测试

```vue
//vue3.0
<template>
	<form>
        <p>
            <label>
                用户名：
                <input type="text" v-model="form.username"/>
    		</label>
    	</p>
        <p>
            <label>
                密码：
                <input type="text" v-model="form.password"/>
    		</label>
   	 	</p>
        <button @click="submit">提交</button>
    </form>
</template>

<script>
import { reactive } from 'vue';

export default {
    setup(){
        const form = reactive({});
        const submit = ()=>{
            console.log(form);
            /**
            * 打印结果
            * Proxy:{username:"xxx",password:"xxx"}
            */
        }
        
        return {
            form,
            submit
        }
    }
}
</script>
```

vue3.0中打印出来的是一个代理对象，和vue2.x一致，但是不需要对每一项都进行定义了。

### Vue3.0中的v-model

上面的东西对于很多人来说可能没什么用，但是下面这些就必须得了解一下了，在vue3.0中v-model的改动其实还是比较大的，首先我们先看看vue2.x中的v-model。

v-model是双向数据绑定的指令，在vue2.x版本中v-model是通过oninput时间加上`:value`的props传参来实现的，看下面代码会更容易理解

```vue
<input type="text" v-model="msg"/>

//等价于

<input type="text" :value="msg" @input="msg = $event.target.value"/>
```

由于v-model是用`value`字段进行接收的数据，所以在一个组件上v-model只能出现一次，但是在vue3中就不再使用value字段接收数据了，并且也不再使用input事件了，而是使用了update这个自定义事件，举个栗子

先写一个`MyInput`组件

```vue
// /src/components/MyInput.vue
<template>
	标题<input type="text"/>
	<br>
	页脚<input type="text"/>
</template>
```

然后在`App.vue`中引入

```vue
// /src/App.vue
<template>
	<h1>{{ title }}</h1>
	<MyInput/>
	<h2>{{ footer }}</h2>
</template>

<script>
import { ref, reactive } from 'vue';
import MyInput from './components/MyInput.vue';
export default {
    setup(){
        const title = ref("默认标题");
        const footer = ref("默认页脚");
        
        return {
            title,
            footer
        }
    },
    components:{
        MyInput
    }
}
</script>
```

然后有意思的东西来了，上面说过vue3中v-model不再使用value字段传递数据，现在的一个组件是可以使用多次v-model的，当然写法上也得做一些小小的修改

```vue
// /src/App.vue
<template>
	<h1>{{ title }}</h1>
	<MyInput
     	v-model:title="title"
        v-model:footer="footer"
    />
	<h2>{{ footer }}</h2>
</template>

...
```

`v-model:name`来表示使用`v-model`将name传入到子组件中，在子组件中通过name字段接收，并通过update自定义事件修改name

```vue
// /src/components/MyInput.vue
<template>
	标题<input type="text" @input="$emit('update:title', $event.target.value)"/>
	<br>
	页脚<input type="text" @input="$emit('update:footer',$event.target.value)"/>
</template>

<script>
export default {
    props:{
        title:String,
        footer:String
    }
}
</script>
```

也就是说下面这两种写法是等价的

```vue
<MyInput v-model:title="title"/>

//等价于

<MyInput :title="title" @update:title="title = $event"/>
```

以上就是vue3中对v-model的使用，使用方式基本没有变化，但是原理变化了，还是比较容易理解的。

### v-if与v-for优先级

其实这两个指令是不推荐在一起使用的，这两个指令也是被使用最多的指令，首先按照我们以往的理解，v-for优先级是比v-if要高的，但是这种想法是在2.x版本上的理解，而在3.x中反了过来，v-if的优先级要比v-for高，这个记住即可，面试可能会问

