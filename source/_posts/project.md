---
title: 项目二-(1)用户登录
date: 2025-05-14
tag: 项目练习
category: 前端
---


#### MyLogin页面


  ##### 1.实现功能:<br>
  <p>
  (1)登录表单区域中通过双向绑定,默认账号密码,点击登录时将账号密码信息传给后端,后端验证账号密码(代码在下面)<br>
  (2)重置表单中存在的账号密码信息<br>
  (3)用sessionStorage将数据存储在服务器内,仅在前端使用,不参与服务器通信<br>
  (4)登录后路由跳转到用户主页
</p>

  ##### 2.问题:<br>

<p>
  (1)从哪里验证的账号密码信息<br>
  (2)后端的token<br>
  是服务器生成的一段字符串，代表用户的身份或权限，用于替代传统的用户名和密码进行后续请求的验证。
</p>


##### 3.代码片段
<p>
  <code>
async function login () {
  const { data } = await axios.post('http://localhost:3000/user/login', loginForm.value)
  if (data.code === 2) {
    window.sessionStorage.setItem('userid', data.body.userid)
    window.sessionStorage.setItem('username', data.body.username)
    window.sessionStorage.setItem('roleType', data.body.roleType)
    router.push('/home')
  } else {
    ElMessage.error(data.msg)
  }
}
</code>
</p>

<p>
  (1)const {data} 解构赋值<br>
  通过 const { data } = ... 语法，可以直接从响应对象中提取 data 属性并赋值给同名变量,用于从 Promise 解析后的响应对象中提取特定属性。<br>
  (2)data.code===2<br>
  data.code===2是自己写的,写在后端的user页面中:
  <code>
    
user.post('/login', async (req, res) => {
//从请求体中获取用户提交的数据
  const { body } = req
  let sql = `select * from user_info where userid='${body.userid}' and password='${body.password}'`
  //创建数据库操作实例
  const addDatabase = new DataBase()
  //执行查询
  const info = await addDatabase.getSqlData(sql)
  if (info.length) {
    res.send({
      code: 2,
      body: info[0]
      //返回查询到的第一条数据
    })
  } else {
    res.send({
      code: 4,
      msg: '账号或密码有误！'
    })
  }
})
  </code>
  
</p>


<b>类似片段</b>

<code>
  const login =()=>{
    if(username.value==='admin'&&password.value==='admin'){
        localStorage.setItem('token','Bear...')
        router.push('/home')
    }else{
        localStorage.removeItem('token')
    }
}
</code>
<br>

######  localStorage
<p>
  浏览器提供的客户端储存机制,在浏览器存储键值对数据,长期保留并且不会发送到服务器,一般用于存储无需保密的客户端数据<br>
  localStorage.setItem(key, value);
</p>

