# Django回顾

知识点的回顾：

- 安装Django

  ```
  pip install django
  ```

- 创建Django项目

  ```
  >>> django-admin startproject mysite
  ```

  注意：Pycharm可以创建。如果用Pycharm创建，记得settings.py中的DIR templates 删除。

- 创建app & 注册

  ```
  >>>python manage.py startapp app01
  >>>python manage.py startapp app02
  >>>python manage.py startapp app03
  ```

  ```
  INSTALLED_APPS = [
      ...
      'app01.apps.App01Config'
  ]
  ```

  注意：否则app下的models.py写类时，无法在数据库中创建表。

- 配置 静态文件**(static)**路径 & 模板**(templates)**的路径（放在app目录下）。

- 配置数据库相关操作（MySQL）

  - 第三方模块（django3版本）

    ```
    pip install mysqlclient
    ```

  - 自己先去MySQL创建一个数据库，ORM只能建表，需要先自己手动建库。

  - 配置数据库连接settings.py

    ```python
    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.mysql',
            'NAME': 'database_name',  # 要链接的数据库名字
            'USER': 'root',
            'PASSWORD': '123456',
            'HOST': '127.0.0.1',  # 那台机器安装了MySQL
            'PORT': 3306,
        }
    }
    ```

  - 在app下的models.py中编写，在这里创建表，而不是写sql语句

    ```python
    from django.db import models
    
    
    class Admin(models.Model):
        """ 管理员 """
        username = models.CharField(verbose_name="用户名", max_length=32)
        password = models.CharField(verbose_name="密码", max_length=64)
    
        # 用于其他表外键链接时直接显示数据内容
        def __str__(self):
            return self.username
    
        
    class Department(models.Model):
        """ 部门表 """
        title = models.CharField(verbose_name='标题', max_length=32)
    
        def __str__(self):
            return self.title
    ```

  - 创建该表需要执行两个命令，每次建新表时都需要该指令，同时删除某张表也只需要将上面表对应的类删掉，然后执行该指令：

    ```python
    >>>python manange.py makemigrations
    >>>python manange.py migrate
    ```

- 在 urls.py 设定路由，在views里面编写对应关系（ URL 和 视图函数的对应关系）。

- 在views.py，视图函数，编写业务逻辑。

- templates目录，编写HTML模板（含有模板语法、继承（`{% extends '模板.html' %}`）、

  页面中加载静态文件时需要使用`{% static 'xx'%}`） 

- ModelForm & Form组件，在我们开发增删改查功能。
  - 生成HTML标签（生成默认值）

  - 请求数据进行校验。

    ```python
    form = AdminForm(request.POST)
    if form.is_valid():
    ```

  - 保存到数据库（ModelForm）

    ```python
    form.save()
    return redirect('http://localhost:8000/admin/')
    ```

  - 获取错误信息。

    ```python
    return render(request, 'admin_add.html', {'form': form})
    ```

  - 汇总，校验、保存、错误返回

    ```python
    form = AdminForm(request.POST)
    if form.is_valid():
        form.save()
        return redirect('http://localhost:8000/admin/')
    return render(request, 'admin_add.html', {'form': form})
    ```

- Cookie和Session，用户登录信息保存起来。

- 中间件，基于中间件实现用户认证 ，基于：`process_request`。

- ORM操作

  ```
  models.User.objects.filter(id="xxx")
  models.User.objects.filter(id="xxx").order_by("-id")
  ```

- 分页组件。



## 1.Ajax请求

- 首先我们创建一个url和views，用于创建一个界面，生成表单接受数据：

  ```python
  path('ajaxmission/', views.ajaxmission),
  ```

  ```python
  class AjaxForm(forms.ModelForm):
      class Meta:
          model = models.ajaxmission
          fields = '__all__'
          widgets = {
              'level': forms.Select(attrs={'class': 'form-control', }),
              'title': forms.TextInput(attrs={'class': 'form-control', }),
              'detail': forms.Textarea(attrs={'class': 'form-control', }),
              'user': forms.Select(attrs={'class': 'form-control'}),
  
          }
  def ajaxmission(request):
      """ajax任务管理"""
      form = AjaxForm()
      return render(request, 'ajax_mission.html', {'form': form})
  ```

- 然后我们在界面上设置一个按钮

  ```html
  <button style="margin-top: 20px" type="button" id="btnAdd" class="btn btn-primary row" >提 交</button>
  ```

  - 这里不是submit提交，因为要采用ajax提交，不用刷新界面，而`type="button"`不会提交表单，所以不需要submit。

  - 通过ajax绑定一个事件，找到`id="btn-Add"`绑定一个点击事件

    

* 在JavaScript中

  ```javascript
  <script type="text/javascript">
      $(function () {
  
          BtnEvent();
      })
      function BtnEvent(){
          $("#btnAdd").click(function () {
              $(".error-msg").empty();
              $.ajax({
                  url: '/ajax_add/',
                  type: "post",
                  data: $("#formAdd").serialize(),
                  dataType: "JSON",
                  success: function (res) {
                      if(res.status){
                          alert('添加成功');
                          location.reload();
                      }else {
                          $.each(res.error,function (name,data){
                              //console.log(name,data)
                              $("#id_"+name).next().text(data[0])
                          })
                      }
                  }
              })
          })
      }
  </script>
  ```

  - 首先我们将错误信息清空

    ```javascript
    $(".error-msg").empty();
    ```

    - 注意错误信息放在这里

      ```html
      <span class="error-msg" style="color: red"></span>
      ```

  - 然后发送一个ajax请求，向`/ajax_add`这个地址以post请求发送，通过`$("#formAdd").serialize(),`将其全部打包发送

    ```html
    <form id="formAdd">
        {% for foo in form %}
            <div class="form-group"></div>
            <label for="">{{ foo.label }}</label>
            {{ foo }}
            <span class="error-msg" style="color: red"></span>
        {% endfor %}
        <div class="col-xs-12">
            <button style="margin-top: 20px" type="button" id="btnAdd" class="btn btn-primary row" >提 交</button>
        </div>
    </form>
    ```

  * 发送之后在后台就可以接受到数据

    ```python
    path('ajax_add/', views.ajax_add),
    ```

    ```python
    @csrf_exempt
    def ajax_add(request):
        print(request.POST)
        form = AjaxForm(data=request.POST)
        if form.is_valid():
            form.save()
            data = {'status': True, }
            return JsonResponse(data)
        # 失败时返回，数据为空之类的
        data = {'status': False, 'error': form.errors}
        return JsonResponse(data)
    ```

    - 首先`@csrf_exempt`用于免除csrf认证，不然会报错，需要导入如下

      ```python
      from django.views.decorators.csrf import csrf_exempt
      ```

    - 然后将数据传给`modelform`做表单的验证，验证成功则保存数据库，并返回`data={'status':True}`给前端

    - 传输成功后就可以拿到**status=True**这个结果，然后JS`res`就可以拿到True或者False判断，False则显示错误信息

      ```javascript
      if(res.status){
          alert('添加成功')
      }else {
          $.each(res.error,function (name,data){
              //console.log(name,data)
              $("#id_"+name).next().text(data[0])
          })
      }
      ```

    - `location.reload();`用于界面的自动刷新



























