---
title: 项目一-页面搭建
date: 2025-05-09
tag: 项目练习
category: 前端
---
#### 数据获取成功之后
1.保存数据<br>
<p>
![image](https://github.com/user-attachments/assets/306cec86-f365-455e-9216-22c78adbe167)
 创建一个ref对象存储后端拿到的数据
</p>
2.展示数据<br>
<p>
获取到数据之后用for循环,将vant标签放进去,遍历ref对象数据,展示内容为value.some
<i>注意此处的循环遍历的是对象,(value,key,index)in something</i><br>
</p>
3. 网络请求抽取<br>
<p>
 抽取网络请求到pinia
</p>
4.组件之间监听点击(返回上一级) <br>
<p>
 思路:子组件在标签上监听,获取标签传递的信息,然后emit传递给父组件,父组件放置到相应位置<br>
 
 实操:<br>
 (1)用router.back()和事件总线eventBus.emit("事件名",传递值)<br>
 (2)要传递的对象使用范围大,绑定到store里面<br>
 提示:最好传递对象而非对象的内的数据
</p>
5.死数据变活<br>
<p>
 搭建框架的时候直接写文本,然后在script里面写data数据(此时还是死数据)用插值或者其他方式绑定到框架中,再对数据进行处理
</p>
 
