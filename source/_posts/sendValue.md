---
title: Vue学习 -组件间传值整理
date: 2025-5-15
tag: 
-基础整理
-slot
category: 前端
---
#### slot插槽
(1)默认插槽<br>
<p>
  父组件调用子组件标签,在子组件标签中写的东西会被传递到子组件写的slot中
</p>
(2)具名插槽<br> 
<p>
  子组件的slot有name属性是具名,没有name是匿名,父组件中不需要写子组件标签,在需要传值的地方用template包裹,v-slot的value对应子组件name的value
</p>
(3)作用域插槽<br>
<p>
  子组件传值给父组件:slot中传递属性,储存在父组件的v-slot的value中<br>
  解构
</p>
#### props配置项
