---
title:项目(2)
date:2025-05-14
---


#### MyLogin页面


  ##### 实现功能:<br>
  <p>
  (1)登录表单区域中通过双向绑定,默认账号密码,并且将账号密码信息传给!!组件<br>
  (2)重置表单中存在的账号密码信息<br>
  (3)用sessionStorage将数据存储在服务器内,仅在前端使用,不参与服务器通信<br>
</p>

  ##### 问题:<br>

<p>
  (1)从哪里验证的账号密码信息<br>
  (2)后端的token<br>
</p>

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
