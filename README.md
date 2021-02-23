# JQuery-Express-blog-houtai

​	

时间有限, 只列出了大概，虽然代码中有注释，但还是需要配合后台接口代码一起看，这样才能好好理解前后端交互的思想

想看具体代码的可以去看👉仓库的代码

该博客后台管理系统，功能有四个模块

1. 注册和登录
2. Echart展示的文章和评论数据(只是一个简单的展示，没有接入后台数据)
3. 文章管理(文章类别、文章列表、发布文章)
4. 个人中心(基本资料、修改资料、更换头像、重置密码)

## 1.登录界面

1. 导入layui的css&js文件和当前界面的css&js文件

2. 导入自己封装的baseApi.js文件

   ```js
   // 每次调用 $.get() 或 $.post() 或 $.ajax() 的时候，会先调用 ajaxPrefilter 这个函数
   $.ajaxPrefilter(function(options) {
     options.url = 'http://127.0.0.1:3008' + options.url
     // 统一为有权限的接口，设置 headers 请求头
     if (options.url.indexOf('/my/') !== -1) {
       options.headers = {
         Authorization: localStorage.getItem('token') || ''
       }
     }
   
     // 全局统一挂载 complete 回调函数
     options.complete = function(res) {
       //res.responseJSON 拿到服务器响应回来的数据
       if (res.responseJSON.status === 1 && res.responseJSON.message === '身份认证失败！') {
         // 1. 强制清空 token
         localStorage.removeItem('token')
         // 2. 强制跳转到登录页面
         location.href = '/login.html'
       }
     }
   })
   ```

3. 封装一个验证用户名和密码的函数

   ```js
   // 从 layui 中获取 form 对象
   var form = layui.form
   // 通过 form.verify() 函数自定义校验规则
   form.verify({
     // 自定义了一个叫做 pwd 校验规则
     pwd: [/^[\S]{6,12}$/, '密码必须6到12位，且不能出现空格'],
     // 校验两次密码是否一致的规则
     repwd: function(value) {
       var pwd = $('.reg-box [name=password]').val()
       if (pwd !== value) {
       	return '两次密码不一致！'
   	  }
   	}
   })	
   ```

4. 监听注册按钮和登录按钮，分别发送请求

   ```js
   // 监听注册表单的提交事件
   $('#form_reg').on('submit', function(e) {
     // 1. 阻止默认的提交行为
     e.preventDefault()
     // 2. 发起Ajax的POST请求
     var data = {
       username: $('#form_reg [name=username]').val(),
       password: $('#form_reg [name=password]').val()
     }
     $.post('/api/register', data, function(res) {
       if (res.status !== 0) {
         return layer.msg(res.message)
       }
       layer.msg('注册成功，请登录！')
       // 模拟点击行为
       $('#link_login').click()
     })
   })
   ```

   ```js
   // 监听登录表单的提交事件
   $('#form_login').submit(function(e) {
     // 阻止默认提交行为
     e.preventDefault()
     $.ajax({
       url: '/api/login',
       method: 'POST',
       // 快速获取表单中的数据
       data: $(this).serialize(),
       success: function(res) {
         if (res.status !== 0) {
           return layer.msg('登录失败！')
         }
         layer.msg('登录成功！')
         // 将登录成功得到的 token 字符串，保存到 localStorage 中
         localStorage.setItem('token', res.token)
         // 跳转到后台主页
         location.href = '/index.html'
       }
     })
   })
   ```





## 2.博客首页

使用iframe标签在内容主体区域显示网页内容

```html
<!-- 博客首页默认为图表统计页dashboard.html -- >
<li class="layui-nav-item layui-this">
  <a href="/home/dashboard.html" target="fm"><span class="iconfont icon-home"></span>首页</a>
</li>
```

```html
<div class="layui-body">
  <!-- 内容主体区域 -->
  <iframe name="fm" src="/home/dashboard.html" frameborder="0"></iframe>
</div>
```



## 3.文章管理

文章管理有三个板块，分别 是文章类别、文章列表、发布文章

```html
<li class="layui-nav-item">
  <a class="" href="javascript:;"><span class="iconfont icon-16"></span>文章管理</a>
  <dl class="layui-nav-child">
    <dd>
      <a href="/article/art_cate.html" target="fm">文章类别</a>
    </dd>
    <dd>
      <a href="/article/art_list.html" target="fm">文章列表</a>
    </dd>
    <dd>
      <a href="/article/art_pub.html" target="fm">发布文章</a>
    </dd>
  </dl>
</li>
```

1. 文章类别

   导入模板引擎

   ```js
   <script src="/assets/lib/template-web.js"></script>
   ```

   获取文章分类

   ```js
   function initArtCateList() {
     $.ajax({
       method: 'GET',
       url: '/my/article/cates',
       success: function(res) {
         var htmlStr = template('tpl-table', res)
         $('tbody').html(htmlStr)
       }
     })
   }
   ```

   表格数据的模版

   ```js
   <script type="text/html" id="tpl-table">
     {{each data}}
     <tr>
       <td>{{$value.name}}</td>
       <td>{{$value.alias}}</td>
       <td>
         <button type="button" class="layui-btn layui-btn-xs btn-edit" data-id="{{$value.Id}}">编辑</button>
         <button type="button" class="layui-btn layui-btn-danger layui-btn-xs btn-delete" data-id="{{$value.Id}}">删除</
   button>
       </td>
     </tr>
     {{/each}}
   </script>
   ```

2. 文章列表

   获取文章列表数据的方法

   ```js
   // 获取文章列表数据的方法
   function initTable() {
     $.ajax({
       method: 'GET',
       url: '/my/article/list',
       data: q,
       success: function(res) {
         if (res.status !== 0) {
           return layer.msg('获取文章列表失败！')
         }
         // 使用模板引擎渲染页面的数据
         var htmlStr = template('tpl-table', res)
         $('tbody').html(htmlStr)
         // 调用渲染分页的方法
         renderPage(res.total)
       }
     })
   }
   ```

3. 发布文章

   ```js
   // 定义一个发布文章的方法
   function publishArticle(fd) {
     $.ajax({
       method: 'POST',
       url: '/my/article/add',
       data: fd,
       // 如果向服务器提交的是 FormData 格式的数据，
       // 必须添加以下两个配置项
       contentType: false,
       processData: false,
       success: function(res) {
         if (res.status !== 0) {
           return layer.msg('发布文章失败！')
         }
         layer.msg('发布文章成功！')
         // 发布文章成功后，跳转到文章列表页面
         location.href = '/article/art_list.html'
       }
     })
   }
   ```

   



## 4.个人中心

1. 个人中心有三个板块，分别 是基本资料、更换头像、重置密码

   ```html
   <li class="layui-nav-item">
     <a href="javascript:;"><span class="iconfont icon-user"></span>个人中心</a>
     <dl class="layui-nav-child">
       <dd>
       	<a href="/user/user_info.html" target="fm">基本资料</a>
       </dd>
       <dd>
       	<a href="/user/user_avatar.html" target="fm">更换头像</a>
       </dd>
       <dd>
       	<a href="/user/user_pwd.html" target="fm">重置密码</a>
       </dd>
     </dl>
   </li>
   ```

2. 基本资料

   基本资料展示了登录用户的数据，也可以在这里对用户基本信息进行修改

   ```js
   // 初始化用户的基本信息的ajax请求
   function initUserInfo() {
     $.ajax({
     method: 'GET',
     url: '/my/userinfo',
     success: function(res) {
       if (res.status !== 0) {
       	return layer.msg('获取用户信息失败！')
       }
       // console.log(res)
       // 调用 form.val() 快速为表单赋值
       form.val('formUserInfo', res.data)	
      }
     })
   }
   ```

   ```js
   // 修改用户信息的ajax请求
   $('.layui-form').on('submit', function(e) {
     // 阻止表单的默认提交行为
     e.preventDefault()
     // 发起 ajax 数据请求
     $.ajax({
       method: 'POST',
       url: '/my/userinfo',
       data: $(this).serialize(),
       success: function(res) {
         if (res.status !== 0) {
           return layer.msg('更新用户信息失败！')
         }
         layer.msg('更新用户信息成功！')
         // 调用父页面中的方法，重新渲染用户的头像和用户的信息
         window.parent.getUserInfo()
       }
     })
   })
   ```

   

   2. 更换头像

   3. 导入 `cropper.css` 样式和 `cropper.js`

   4. 实现基本裁剪效果

   ```html
   <!-- html -->
   <img id="image" src="/assets/images/sample.jpg" />
   ```

   ```js
   // 1 获取裁剪区域的 DOM 元素
   var $image = $('#image')
   // 2 配置选项
   const options = {
     // 纵横比
     aspectRatio: 1,
     // 指定预览区域
     preview: '.img-preview'
   }
   
   // 3 创建裁剪区域
   $image.cropper(options)
   ```

   3. 更换裁剪的图片

   ```js
   // 为文件选择框绑定 change 事件
   $('#file').on('change', function(e) {
     // console.log(e);
     // 获取用户选择的文件
     var filelist = e.target.files
     if (filelist.length === 0) {
     return layer.msg('请选择照片！')
     }
   
     // 1. 拿到用户选择的文件
     var file = e.target.files[0]
     // 2. 将文件，转化为路径
     var imgURL = URL.createObjectURL(file)
     // 3. 重新初始化裁剪区域
     $image
     .cropper('destroy') // 销毁旧的裁剪区域
     .attr('src', imgURL) // 重新设置图片路径
     .cropper(options) // 重新初始化裁剪区域
   })
   ```

   4. 为确定按钮绑定点击事件

   ```js
   // 为确定按钮，绑定点击事件
   $('#btnUpload').on('click', function() {
     // 1. 要拿到用户裁剪之后的头像
     var dataURL = $image
     .cropper('getCroppedCanvas', {
       // 创建一个 Canvas 画布
       width: 100,
       height: 100
     })
     .toDataURL('image/png') // 将 Canvas 画布上的内容，转化为 base64 格式的字符串
     // 2. 调用接口，把头像上传到服务器
     $.ajax({
       method: 'POST',
       url: '/my/update/avatar',
       data: {
         avatar: dataURL
       },
       success: function(res) {
         if (res.status !== 0) {
           return layer.msg('更换头像失败！')
         }
         layer.msg('更换头像成功！')
         window.parent.getUserInfo()
       }
     })
   })
   ```

   3. 更换密码

   ```js
   var form = layui.form
   
   form.verify({
     pwd: [/^[\S]{6,12}$/, '密码必须6到12位，且不能出现空格'],
     samePwd: function(value) {
       if (value === $('[name=oldPwd]').val()) {
         return '新旧密码不能相同！'
       }
     },
     rePwd: function(value) {
       if (value !== $('[name=newPwd]').val()) {
         return '两次密码不一致！'
       }
     }
   })
   
   $('.layui-form').on('submit', function(e) {
     e.preventDefault()
     $.ajax({
       method: 'POST',
       url: '/my/updatepwd',
       data: $(this).serialize(),
       success: function(res) {
         if (res.status !== 0) {
           return layui.layer.msg('更新密码失败！')
         }
         layui.layer.msg('更新密码成功！')
         // 重置表单
         $('.layui-form')[0].reset()
       }
     })
   })
   ```

   

到这里就结束了，很抱歉没有详细列出来各个文件之间的联系，但是我在代码中给出了注释，希望能帮助到你们^_^

