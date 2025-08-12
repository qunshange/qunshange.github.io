---
title: 组件间传值
date: 2025-05-15
excerpt: slot和props
tag: 
- vue
- slot
- props
category: 前端
---


## 一、slot 插槽

### 1. 默认插槽
- **定义**：父组件在调用子组件标签时，标签内部写的内容会自动填充到子组件的 `<slot>` 中。
- **子组件**：
```vue
<template>
  <div class="child">
    <slot></slot> <!-- 默认插槽 -->
  </div>
</template>
````

* **父组件**：

```vue
<ChildComponent>
  <p>这是插入到子组件默认插槽的内容</p>
</ChildComponent>
```

---

### 2. 具名插槽

* **定义**：给 `<slot>` 添加 `name` 属性，可以指定内容插入的位置。
* **子组件**：

```vue
<template>
  <header>
    <slot name="header"></slot>
  </header>
  <main>
    <slot></slot> <!-- 默认插槽 -->
  </main>
  <footer>
    <slot name="footer"></slot>
  </footer>
</template>
```

* **父组件**：

```vue
<ChildComponent>
  <template v-slot:header>
    <h1>标题部分</h1>
  </template>

  <p>这是默认插槽内容</p>

  <template v-slot:footer>
    <p>底部信息</p>
  </template>
</ChildComponent>
```

> `v-slot:header` 可以简写为 `#header`。

---

### 3. 作用域插槽

* **定义**：子组件向插槽中绑定属性，父组件可以接收并使用这些属性。
* **子组件**：

```vue
<template>
  <div>
    <slot :user="userInfo"></slot>
  </div>
</template>

<script>
export default {
  data() {
    return {
      userInfo: { name: '张三', age: 20 }
    }
  }
}
</script>
```

* **父组件**：

```vue
<ChildComponent v-slot="{ user }">
  <p>用户名：{{ user.name }}</p>
  <p>年龄：{{ user.age }}</p>
</ChildComponent>
```

* 这里 `v-slot="{ user }"` 语法是 **解构插槽参数**。

---

## 二、props 配置项

`props` 用于 **父组件向子组件传递数据**。
定义在子组件中，通过 `props` 接收父组件传递的值。

### 1. 基本用法

* **子组件**：

```vue
<script>
export default {
  props: ['title']
}
</script>

<template>
  <h2>{{ title }}</h2>
</template>
```

* **父组件**：

```vue
<ChildComponent title="这是标题" />
```

---

### 2. 类型校验

* Vue 支持对 props 进行类型校验，并可设置默认值。

```vue
<script>
export default {
  props: {
    title: {
      type: String,     // 类型
      required: true,   // 必传
      default: '默认标题' // 默认值
    },
    count: {
      type: Number,
      default: 0
    }
  }
}
</script>
```

---

### 3. 多类型支持

```vue
props: {
  value: [String, Number]
}
```

表示 `value` 可以是字符串或数字。

---

### 4. 传递对象/数组

```vue
<ChildComponent :user="{ name: '张三', age: 20 }" />
<ChildComponent :list="[1, 2, 3]" />
```

---

## 三、slot 与 props 配合使用

在实际开发中，可以结合 `slot` 与 `props` 实现更灵活的组件：

* 用 `props` 传递静态或父组件控制的数据。
* 用 `slot` 提供可自定义的 UI 渲染区域，让父组件灵活定义展示内容。

示例：

```vue
<ChildComponent :title="pageTitle" v-slot:footer="{ date }">
  <p>这里是主体内容</p>
  <p>当前时间：{{ date }}</p>
</ChildComponent>
```

---

## 总结

* `props`：父传子数据通道，适合结构固定的数据传递。
* `slot`：父子之间灵活的内容分发机制，支持默认、具名和作用域三种方式。
* **组合使用** 可以让组件既可配置（`props`），又可扩展（`slot`）。

---


