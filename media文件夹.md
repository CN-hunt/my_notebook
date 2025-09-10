# media文件夹

media文件夹用于接受用户上传的文件夹，需要一些配置才能使用

1. 首先在`urls.py`里面导入如下

   ```python
   from django.urls import path,re_path
   from django.views.static import serve
   from django.conf import settings
   ```

2. 然后在路径中写入，这是固定写法

   ```python
   re_path(r'media/(?P<path>.*)', serve, {'document_root': settings.MEDIA_ROOT}),
   ```

3. 在`settings.py`中也需要进行一定的配置

   ```python
   import os
   MEDIA_ROOT = os.path.join(BASE_DIR, "media")
   MEDIA_URL = '/media/'
   ```

   BASE_DIR：就是项目的根目录，也就是说**media文件夹**应该被放在根目录（和app01平级），记得手动在该位置创建他

4. 这样就可以在路径中访问他：`[file.png (649×664)](http://localhost:8000/media/file.png)`

   