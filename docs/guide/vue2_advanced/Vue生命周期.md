# Vue 生命周期

## 前言

在日常开发中，Vue 生命周期是使用频率最高的方法之一，而且在面试过程中也是出现比较高频的题目之一，因此我想从使用角度剖析它们。

## 生命周期示例

Vue2 生命周期图示参考[官网](https://v2.cn.vuejs.org/v2/guide/instance.html#%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%E5%9B%BE%E7%A4%BA)

下面举个例子说明嵌套组件初始化，生命周期的执行顺序，部分组件使用了 `element-ui` 库：
```vue
/* App.vue 组件 */
<template>
  <div class="page">
    <p>我是父组件：{{ count }}</p>
    <el-button type="primary" size="small" @click="onChange">点我+1</el-button>
    <el-button type="primary" size="small" @click="onHide">点我隐藏子组件</el-button>
    <el-divider />
    <Child v-if="show" />
  </div>
</template>

<script>
import { defineComponent } from 'vue'
import Child from '@/components/Child.vue'

export default defineComponent({
  name: 'App',
  components: {
    Child,
  },
  data() {
    return {
      count: 0,
      show: true,
    }
  },
  beforeCreate() {
    console.log('App beforeCreate')
  },
  created() {
    console.log('App created')
  },
  beforeMount() {
    console.log('App beforeMount')
  },
  mounted() {
    console.log('App mounted')
  },
  beforeUpdate() {
    console.log('App beforeUpdate')
  },
  updated() {
    console.log('App updated')
  },
  beforeDestroy() {
    console.log('App beforeDestroy')
  },
  destroyed() {
    console.log('App destroyed')
  },
  methods: {
    onChange() {
      this.count++
    },
    onHide() {
      this.show = !this.show
    },
  },
})
</script>

<style lang="scss" scoped></style>
```
::: details 点击查看 Child 组件代码
```vue
/* Child.vue 组件 */
<template>
  <div class="page">
    <p>我是子组件：{{ count }}</p>
    <el-button type="primary" size="small" @click="onChange">点我+1</el-button>
    <el-button type="primary" size="small" @click="onHide">点我隐藏孙组件</el-button>
    <el-divider />
    <Son v-if="show" />
  </div>
</template>

<script>
import { defineComponent } from 'vue'
import Son from './Son.vue'

export default defineComponent({
  name: 'ChildComp',
  components: {
    Son,
  },
  data() {
    return {
      count: 0,
      show: true,
    }
  },
  beforeCreate() {
    console.log('Child beforeCreate')
  },
  created() {
    console.log('Child created')
  },
  beforeMount() {
    console.log('Child beforeMount')
  },
  mounted() {
    console.log('Child mounted')
  },
  beforeUpdate() {
    console.log('Child beforeUpdate')
  },
  updated() {
    console.log('Child updated')
  },
  beforeDestroy() {
    console.log('Child beforeDestroy')
  },
  destroyed() {
    console.log('Child destroyed')
  },
  methods: {
    onChange() {
      this.count++
    },
    onHide() {
      this.show = !this.show
    },
  },
})
</script>

<style lang="scss" scoped></style>
```
:::

::: details 点击查看 Son 组件代码
```vue
/* Son.vue 组件 */
<template>
  <div class="page">
    <p>我是孙组件：{{ count }}</p>
    <el-button type="primary" size="small" @click="onChange">点我+1</el-button>
  </div>
</template>

<script>
import { defineComponent } from 'vue'

export default defineComponent({
  name: 'SonComp',
  data() {
    return {
      count: 0,
    }
  },
  beforeCreate() {
    console.log('Son beforeCreate')
  },
  created() {
    console.log('Son created')
  },
  beforeMount() {
    console.log('Son beforeMount')
  },
  mounted() {
    console.log('Son mounted')
  },
  beforeUpdate() {
    console.log('Son beforeUpdate')
  },
  updated() {
    console.log('Son updated')
  },
  beforeDestroy() {
    console.log('Son beforeDestroy')
  },
  destroyed() {
    console.log('Son destroyed')
  },
  methods: {
    onChange() {
      this.count++
    },
  },
})
</script>

<style lang="scss" scoped></style>
```
:::

浏览器控制台打印结果如下：
```js
App beforeCreate
App created
App beforeMount
Child beforeCreate
Child created
Child beforeMount
Son beforeCreate
Son created
Son beforeMount
Son mounted
Child mounted
App mounted
```
由此得出结论：如果父组件有嵌套子组件，那么等待子组件完成`mounted`后才执行自己的生命周期钩子`mounted`。

通过子组件「点我+1」按钮操作页面响应式数据，浏览器控制台打印结果如下：
```js
Child beforeUpdate
Child updated
```
由此得出结论：当属于该组件的响应式数据更改时，执行该组件的生命周期钩子`beforeUpdate`与`updated`。

通过父组件「点我隐藏子组件」按钮操作页面响应式数据，控制子组件隐藏（销毁）,浏览器控制台打印结果如下：
```js
App beforeUpdate
Child beforeDestroy
Son beforeDestroy
Son destroyed
Child destroyed
App updated
```
由此得出结论：当属于该组件的响应式数据更改时，执行该组件生命周期钩子`beforeUpdate`，然后是执行子组件的销毁生命周期钩子`beforeDestroy`，接着是子孙组件的销毁生命周期钩子`beforeDestroy`与`destroyed`，接着是子组件的销毁生命周期钩子`destroyed`，最后执行该组件的生命周期钩子`updated`。

## 总结

加载渲染过程如下：
1. 父组件=>beforeCreate
2. 父组件=>created
3. 父组件=>beforeMount
4. 子组件=>beforeCreate
5. 子组件=>created
6. 子组件=>beforeMount
7. 子组件=>mounted
8. 父组件=>mounted

更新过程：
1. 父组件=>beforeUpdate
2. 子组件=>beforeUpdate
3. 子组件=>updated
4. 父组件=>updated

销毁过程：
1. 父组件=>beforeDestory
2. 子组件=>beforeDestroy
3. 子组件=>destroyed
4. 父组件=>destroyed

keep-alive 内置组件新增如下两个生命周期：
1. activated=>被 keep-alive 缓存的组件激活时调用
2. deactivated=>被 keep-alive 缓存的组件失活时调用

一般在 created 生命周期执行异步请求数据，原因有如下两点：
1. 能更快获取到服务端数据，减少页面加载时间，用户体验更好
2. SSR 不支持 beforeMount 、mounted 钩子函数，放在 created 中保持一致性
