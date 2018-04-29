> 最近在重构公司项目代码,因为之前这一块不是我写的,所以也抽空看了一下之前同事的代码,可能是因为大家都是初学VUE,现在看来,有些代码还是略显稚嫩的,有一个小组件,虽然很简单,但是还是可以拿出来说一下的,因为它是一个双向绑定组件.

<!-- more -->
## `v-model`是什么

教程上说的很清楚,`v-model`是一个语法糖,它是由 `:value` 和 `@input`构成的(简单来说是这样的,具体可以参看文件结尾的补充)

## 为什么我需要`v-model`

使用`v-model`不仅仅是因为它简单,它的意义一方面是简化了书写,二是简化了逻辑,三是更加语义化,四是让API更加友好,简单易懂.
在VUE2.5之后,[`.sync`][1]修饰符又重新回归了,我们可以通过`.sync`修饰符来处理双向绑定问题,但是我觉得两者的使用场景是不一样的,v-model更多的是接收用户输入,而`.sync`更多的是同步数据.

## 怎么样实现一个可以使用`v-model`的组件

其实这个问题也很简单,只要仔细思考了上面两个问题,应该可以知道如何写,下面我就拿项目中的这个例子来做一个介绍,这个组件的效果是这样的,一个简单的`switch`开关:

![QQ20180429-133843-HD.gif-66.8kB][3]

### 改造之前的代码:

```html
<template>
    <div class="switch-outer"
         :class="{'on':on,'off':!on}"
         @click="handleChange">
        <span class="text"
              v-show="on">{{onText}}</span>
        <span class="text"
              v-show="!on">{{offText}}</span>
        <span class="active-ball"></span>
    </div>
</template>
<script>
export default {
    data() {
        return {
            on: this.isOn
        };
    },
    props: {
        onText: {
            type: String,
            default: '开启'
        },
        offText: {
            type: String,
            default: '关闭'
        },
        isOn: {
            type: Boolean,
            default: true
        }
    },
    methods: {
        handleChange() {
            this.on = !this.on;
            this.$emit('change', this.on);
        }
    },
    watch: {
        isOn() {
            this.on = this.isOn;
        }
    }
};
</script>
<style rel="stylesheet/scss" lang="scss" scoped>
.switch-outer {
    position: absolute;
    cursor: pointer;
    color: #ffffff;

    .active-ball {
        width: 28px;
        height: 28px;
        border-radius: 50%;
        background-color: currentColor;
        position: absolute;
        top: 1px;
    }
}

.on {
    background: #00a4ee;

    .active-ball {
        left: 53px;
        transition: all 0.5s ease;
    }

    .text {
        margin-left: 14px;
        font-size: 14px;
    }
}

.off {
    background: #c0ccda;

    .active-ball {
        left: 2px;
        transition: all 0.5s ease;
    }

    .text {
        margin-left: 44px;
        font-size: 14px;
    }
}
</style>
```

改造之前的代码,并不能使用`v-model`来做双向绑定,但是很显然,这是一个`switch`开关,是一个用户输入组件,所以使用`v-model`是很自然的需求,但是目前这个组件的使用方式是这样的

```html
 <sky-switch class="share-switch"
            @change="handleChange"
            :isOn="group.is_files_share"
            ref="shareSwitch"></sky-switch>
```

```
// 手动监听change事件,修改数据
handleChange(on) {
    this.group.is_files_share = on;
}
```

我们需要自己监听`change`事件,手动的修改值,这显然不符合我们对用户输入组件的认知,我们需要一个像<input>那样的组件,通过 `v-model` 来绑定数据,剩下的交给`vue`来帮助我们处理.

显然这种使用方式并不友好,不能称为一个合格的组件.如果没有文档,我需要看源码才能知道这个组件如何使用,如果源码写的十分复杂,这个组件基本上是不可复用的.

接下来我们就开始魔改这个组件代码,删除不必要的监听和属性,简单直接的利用`v-model`来实现这个功能

### 改造后的代码

```html
<template>
    <div class="switch-outer"
         :class="{'on':on,'off':!on}"
         @click="handleChange">
        <span class="text"
              v-show="on">{{onText}}</span>
        <span class="text"
              v-show="!on">{{offText}}</span>
        <span class="active-ball"></span>
    </div>
</template>
<script>
export default {
    props: {
        onText: {
            type: String,
            default: '开启'
        },
        offText: {
            type: String,
            default: '关闭'
        },
        value: {  // 将 isOn改成 value ,用来接收 v-model 的传值
            type: Boolean,
            default: true
        }
    },
    methods: {
        handleChange() {
            this.on = !this.on;  // 通过赋值操作来触发属性 on 的 set 方法
            this.$emit('change', this.on); // 发送 change 事件,作为一个事件钩子,方便用户处理自己的逻辑
        }
    },
    computed: {
        on: {  // 去掉了data里面的on,使用计算属性
            get() {  // get时返回 value值
                return this.value; 
            },
            set(val) { // set时发送 input 事件
                this.$emit('input', val);
            }
        }
    }
};
</script>
<style rel="stylesheet/scss" lang="scss" scoped>
.switch-outer {
    position: absolute;
    cursor: pointer;
    color: #ffffff;
    .active-ball {
        width: 28px;
        height: 28px;
        border-radius: 50%;
        background-color: currentColor;
        position: absolute;
        top: 1px;
        transition: all 0.5s ease;
    }
    transition: all 0.5s ease;
}

.on {
    background: #00a4ee;
    .active-ball {
        left: 53px;
        transition: all 0.5s ease;
    }
    .text {
        margin-left: 14px;
        font-size: 14px;
        transition: all 0.5s ease;
    }
}

.off {
    background: #c0ccda;
    .active-ball {
        left: 2px;
    }
    .text {
        margin-left: 44px;
        font-size: 14px;
        transition: all 0.5s ease;
    }
}
</style>
```

这段代码的核心是那个计算属性`on`,巧妙的通过计算属性的`get`和`set`方法,实现监听数据变化,`get`时返回通过`props`传入的`value`,`set`时,向父组件发送`input`事件,这样就可以配合`v-model`来使用了,因为`v-model`的本质就是向组件传一个`value`的`prop`,并且监听`input`方法

`style`部分做了微调,因为这里加入了动画,但是原来的动画只有部分值是动的,会显得很突兀,所以调整为所有变化的属性都有动画,这样整个动画变得很平滑.

改造后的组件,使用方式是这样的:
```
<sky-switch class="share-switch" v-model="group.share_file"></sky-switch>
```
简单明了,一看就是一个用户输入组件,和`<input>`元素一样.

当然,这个组件还不够完善,`API`接口也没有很好的设计,但是用来说明`v-model`这个问题应该是足够了.

## 写在最后

没事多重构自己的点,也可以重构别人的代码,看看别人是怎么实现的,自己又能怎么实现,相互对比,学习,这样才能快速进步.
前两天有事情,清了一天假,项目进度有点落后,今天过来补点进度,剩下点时间,写了这篇文字,虽然是一个很小的点,但是,希望能对对个别人有所启发.

## 一些补充说明:

`v-model`是一个`:value`和`@input`的语法糖,简单的说确实是这样,但其实由什么属性和事件组成,可以通过一个[`model`参数来定制][2],文章只是为了说明用法,并不想研究这些细节,所以都是简单来说;

  [1]: https://cn.vuejs.org/v2/guide/components-custom-events.html#sync-%E4%BF%AE%E9%A5%B0%E7%AC%A6
  [2]: https://cn.vuejs.org/v2/guide/components.html#%E8%87%AA%E5%AE%9A%E4%B9%89%E7%BB%84%E4%BB%B6%E7%9A%84-v-model
  [3]: http://static.zybuluo.com/lesonky/d8u2wpipa4me26cserikstrd/QQ20180429-133843-HD.gif