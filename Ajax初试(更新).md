# Ajax初试

**我们使用`JQuery` 的方式来绑定事件实现ajax**

> 假设我们存在`<input id ='btn1' type="button" class="btn btn-primary" value="点击">` 
>
> > ```javascript
> > $(function () {
> >     //界面加载完成后以下代码将会自动执行
> >     Btn1Event();
> > 
> >     function Btn1Event(){
> >         //给id=btn1的元素绑定一个点击事件
> >         $('#btn1').click(function () {
> >             $.ajax({
> >                 url:'目标url',
> >                 type:'请求方式',
> >                 data:{
> >                     data1:"携带数据1",
> >                     data2:"携带数据2"
> >                 },
> >                 //将数据对象变为JSON
> >                 dataType:"JSON"
> >                 //成功后执行
> >                 success:function(res){
> >                     console.log(res);
> >                 }
> >             })
> >         })
> >     }
> > })
> > ```
> >
> > **我们可以这样理解** 
> >
> > - 我们定义了一个函数`Btn1Event()`当界面加载完成执行到这里的时候会执行下面的函数体
> > - 通过`JQuery`的形式，我们将去寻找HTML元素里面存在 <u>id=btn1</u>的元素，给他绑定一个点击事件
> > - 然后执行下面的ajax代码，将数据携带发送过去后，如果成功得到返回值就会返回在`success:function(){}`当中

**ajax返回值**

> 一般来说ajax的返回值都是`json`格式，无论是在前端还是后端都需要进行一定的处理
>
> > 前端接收时需要加上`dataType:"JSON"`将接受得到的数据反序列化成一个前端的JSON对象，否则他得到的仍然是字符串，很难分割其中的数据
> >
> > 后端返回时我们需要
> >
> > ```python
> > def ajax(request):
> >     print(request.POST)
> > 	data_dict = {'status': True, 'data': [11, 22, 33, 44, 55]}
> > 	return HttpResponse(json.dumps(data_dict))
> > ```
> >
> > 以及一个仅用于返回界面的函数
> >
> > ```python
> > def ajaxpag(request):
> >     return render(request, 'ajax.html')
> > ```
> >
> > 或者后端返回时直接使用
> >
> > ```python
> > from django.http import JsonResponse
> > return JsonResponse(data_dict)
> > ```
> >
> > 这样在`console.log(res)`获取数据时就可以以`res.status`以及`res.data`的形式获取数据了
> >
> > - <u>需要注意的是，用于展示界面的函数和接受ajax的返回函数是两个视图函数，一个用于展示界面，一个用于接受数据，及上面写到的`def ajax()`和`def ajaxpage()`别搞错了</u>

**ajax返回动态参数**

> * 我们假设存在以下输入框，希望以ajax的形式传输其中的数据
>
> ```html
> <input type="text" id="user" placeholder="姓名">
> <input type="text" id="gender" placeholder="性别">
> ```
>
> ​		回顾此代码
>
> ```javascript
> $('#btn1').click(function () {
> $.ajax({
>   url:'目标url',
>   type:'请求方式',
>   data:{
>       data1:$("#name").val(),
>       data2:$("#gender").val()"
>   },
>   //成功后执行
>   success:function(res){
>       console.log(res);
>   }
> })
> })
> ```
>
> 我们需要将data1和data2的数据进行动态的改变就需要将其修改为`$("#user").val()`以及`$("#gender").val()`的形式
>
> *<u>另外在ajax通过POST向后端返回数据的时候任然需要csrf验证所以需要以下代码关闭验证</u>*
>
> ```python
> from django.views.decorators.csrf import csrf_exempt
> @csrf_exempt
> ```
>
> * 当界面上的输入框相对较多时也可以结合from表单和`.serialize()`，集合上传，不需要以`.val`的形式发送，例如
>
> ```html
> <form id="form3">
>     <input type="text" name="user" placeholder="姓名"/>
>     <input type="text" name="age" placeholder="年龄"/>
>     <input type="text" name="email" placeholder="邮箱"/>
>     <input type="text" name="more" placeholder="介绍"/>
> </form>
> <input id="btn3" type="button" class="btn btn-primary" value="点击3"/>
> ```
>
> > 此时在script中就可以这样写，注意`data: $("#form3").serialize(),`以及`$("#btn3")`这个是需要结合form表单来使用的，并且HTML元素中必须存在**`name`**属性。 
> >
> > ```javascript
> > function bindBtn3Event() {
> >     $("#btn3").click(function () {
> >         $.ajax({
> >             url: '/task/ajax/',
> >             type: "post",
> >             data: $("#form3").serialize(),
> >             dataType: "JSON",
> >             success: function (res) {
> >                 console.log(res);
> >                 console.log(res.status);
> >                 console.log(res.data);
> >             }
> >         })
> >     })
> > }
> > ```

**Ajax校验**

> 前端通过ajax返回数据时，后端接收到数据任然需要进行校验，但是不同于使用`modelform`产生的输入框可以直接进行校验，对于ajax的返回数据需要其他方式校验。
>
> > 我们使用仍然使用`modelform`对象，重点在`form = AjaxForm(data=request.POST)`,通过`request.POST`即可将数据得到，然后发给`modelform`对象`AjaxForm`（模型）即可 ,整体效果如下
> >
> > ```python
> > @csrf_exempt
> > def ajax_add(request):
> >     print(request.POST)
> >     form = AjaxForm(data=request.POST)
> >     if form.is_valid():
> >         form.save()
> >         data = {'success': True,}
> >         return JsonResponse(data)
> > ```
> >
> > - 这里我们注意到，和之前有所不同，这里我们不可以重定向`return redirect()`，他不会跳转，所以没有意义。这里同样需要返回前端所需的JSON数据
> >
> >   ***

> 前端返回的数据不符合规定时，
>
> > 需要将错误进行返回，我们采用返回JSON数据`data = {'status': False,'error':form.errors}`的形式。添加如下代码
> >
> > ```python
> > data = {'status': False,'error':form.errors}
> > return JsonResponse(data)
> > ```
> >
> > 组合得到
> >
> > ```python
> > @csrf_exempt
> > def ajax_add(request):
> > print(request.POST)
> > form = AjaxForm(data=request.POST)
> > if form.is_valid():
> > form.save()
> > data = {'status': True,}
> > return JsonResponse(data)
> > # 失败时返回，数据为空之类的
> > data = {'status': False,'error':form.errors}
> > return JsonResponse(data)
> > ```
> >
> > - 这样前端就可以接收到错误了，接下来就是是办法展示到界面上挺复杂的，详见4-12，我们先把代码贴出来
> >
> >   ```javascript
> >   <script type="text/javascript">
> >       $(function () {
> >
> >           BtnEvent();
> >       })
> >       function BtnEvent(){
> >           //每次点击时，先清空错误提示，这样就会仅仅保留仍然存在的错误，避免修改后错误提示仍然存在
> >           $('#error-msg').empty()
> >
> >           $("#btnAdd").click(function () {
> >               $(".error-msg").empty();
> >               $.ajax({
> >                   url: '/ajax_add/',
> >                   type: "post",
> >                   data: $("#formAdd").serialize(),
> >                   dataType: "JSON",
> >                   success: function (res) {
> >                       if(res.status){
> >                           alert('添加成功')
> >                       }else {
> >                           $.each(res.error,function (name,data){
> >                               //这段代码里面的name,data，其实是将错误信息拆成键和值的形式，错误信息的存储格式为							  //{'title':['此项为必填项',]},注意是数组
> >                               //console.log(name,data)
> >                               $("#id_"+name).next().text(data[0])
> >                           })
> >                       }
> >                   }
> >               })
> >           })
> >     }
> >   </script>
> >   ```
> >   
> >     HTML：注意其中的这个`<span class="error-msg" style="color: red"></span>`这个span标签里面看着啥也没写，其实是结合JS代码生成的，错误信息就会展示在这个标签里面。
> >   
> >   ```html
> >   <form id="formAdd">
> >       {% for foo in form %}
> >           <div class="form-group"></div>
> >           <label for="">{{ foo.label }}</label>
> >           {{ foo }}
> >           <span class="error-msg" style="color: red"></span>
> >       {% endfor %}
> >       <div class="col-xs-12">
> >           <button style="margin-top: 20px" type="button" id="btnAdd" class="btn btn-primary row">提 交
> >           </button>
> >       </div>
> >   </form>
> >   ```
> >
> > 

