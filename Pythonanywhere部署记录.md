# Pythonanywhere部署记录

首先我们需要将项目文件上传至`github`这样Pythonanywhere就可以直接自从上面拉取代码到服务器上面

***

假设我们存在以下信息(Student_Manager)

- 你的用户名是：`SakuraCN`
- 你的项目目录是：`/home/SakuraCN/Student_mag`
- WSGI 文件是：`/var/www/sakuracn_pythonanywhere_com_wsgi.py`

### 第一步修正：正确创建虚拟环境

1. **打开 Bash 控制台**：

   - 在 PythonAnywhere 的仪表盘中，点击顶部的 **"Consoles"** 标签页。
   - 点击 **"Bash"** 按钮，启动一个新的命令行窗口。

2. **导航到你的项目目录**：

   - 在打开的黑屏命令行中，输入以下命令，进入你的项目文件夹：

     bash

     ```
     cd Student_mag
     ```

   - 输入 `ls` 或 `dir` 命令，你应该能看到 `manage.py` 等文件，确认你就在正确的目录里。

3. **创建虚拟环境**：

   - 在命令行中，输入以下命令来创建一个名为 `venv` 的虚拟环境：

     bash

     ```
     virtualenv venv --python=python3.9
     ```

     *注意：`--python=python3.9` 这个参数必须和你之前创建 Web App 时选择的 Python 版本一致。根据你提供的截图，是 3.9，所以我们就用这个。如果以后创建其他项目，这个版本号一定要改。*

   - 等待命令执行完成。这会创建一个名为 `venv` 的新文件夹。

4. **激活虚拟环境**：

   - 创建完成后，输入以下命令来激活它：

     bash

     ```
     source venv/bin/activate
     ```

   - 激活成功后，你的命令行提示符前面会出现 `(venv)` 字样，像这样：

     text

     ```
     (venv) ~/Student_mag $
     ```

     *这个 `(venv)` 非常重要！它意味着你之后所有的操作（比如安装包）都只在这个隔离的环境中进行，不会影响系统的其他部分。*

### 验证第一步是否成功

现在，我们回到 **"Web"** 标签页的配置界面。

1. 在 **"Virtualenv"** 输入框里，确保路径是：

   text

   ```
   /home/SakuraCN/Student_mag/venv
   ```

   （这个路径应该已经存在了，如果不存在，请手动输入一遍）

2. 点击输入框外的空白处进行保存。

3. 如果路径正确，那个红色的警告 **"Warning: No virtualenv detected at this path"** 就应该消失了，可能会变成一个绿色的对勾提示。

   

***



注意有没有`requirements.txt`这个文件，这个需要在本地端手动创建然后推送到github才对，这样Pythonanywhere拉取代码时才会包含这个文件，如果没有就需要手动创建该文件

### 创建 requirements.txt 文件

在当前的 Bash 终端中（你已经在 `~/Student_mag` 目录下），运行以下命令：

bash

```
# 创建并编辑 requirements.txt 文件
nano requirements.txt
```

在打开的编辑器中，输入以下内容（这些是运行Django项目所需的基本依赖）：

text

```
Django==4.2.7
mysqlclient==2.2.0
```

然后按 `Ctrl+O` 保存文件，按 `Enter` 确认文件名，再按 `Ctrl+X` 退出编辑器。

### 继续部署流程

现在你已经有了 `requirements.txt` 文件，我们可以继续之前的部署步骤了：

1. **创建虚拟环境**（如果你还没有做）：

   bash

   ```
   virtualenv venv --python=python3.9
   ```

2. **激活虚拟环境**：

   bash

   ```
   source venv/bin/activate
   ```

   （命令行前缀应该变成 `(venv)`）

3. **安装依赖**：

   bash

   ```
   pip install -r requirements.txt
   ```

4. **特别安装MySQL客户端**（虽然已经在requirements.txt中，但确保一下）：

   bash

   ```
   pip install mysqlclient
   ```

***

设置WSGI：

### 配置 WSGI 文件（告诉服务器如何运行你的Django项目）

WSGI 文件就像是网站的“总指挥”，它告诉服务器：“嘿，这是一个Django项目，这是它的设置文件在哪，请按这个方式来运行它。”

1. **找到 WSGI 文件链接**：

   - 回到 PythonAnywhere 的 **"Web"** 标签页。
   - 找到 **"WSGI configuration file"** 这一项，点击那个链接（通常是 `/var/www/你的用户名_pythonanywhere_com_wsgi.py`）如下所示
   - `/var/www/sakuracn_pythonanywhere_com_wsgi.py?`

2. **编辑 WSGI 文件**：

   - 这会打开一个代码编辑器。**删除里面所有的现有内容**。
   - 将以下代码**完整地复制粘贴**进去。**特别注意：所有 `SakuraCN` 和 `Student` 都要根据你的实际情况修改！**

   python :  <u>Student_mag其实就是进网站创建一个新的app时要求输入的项目民称，并非自己本地的项目名称</u>

   ```python
   import os
   import sys
   
   # ==================== 配置路径 ====================
   # 你的PythonAnywhere用户名
   username = 'SakuraCN'
   # 你的项目目录名
   project_dir = f'/home/{username}/Student_mag'
   # 将项目路径添加到系统路径，这样Python才能找到你的代码
   if project_dir not in sys.path:
       sys.path.append(project_dir)
   
   # ==================== 设置环境变量 ====================
   # ！！！【重要】这里先留空，我们下一步会来配置数据库！！！
   # 告诉Django这是生产环境
   os.environ['DJANGO_SETTINGS_MODULE'] = 'Student.settings'  # 根据你的项目结构，这很可能是 'Student.settings'
   os.environ['DEBUG'] = 'False'
   
   # ==================== 启动Django应用 ====================
   from django.core.wsgi import get_wsgi_application
   application = get_wsgi_application()
   ```

3. **保存文件**：点击编辑器上方的 **Save** 按钮。



***

### 配置数据库链接

- 在PythonAnywhere上，当你初始化MySQL数据库时，它会自动为你生成一个数据库名（通常是 `你的用户名$default` 这样的格式）并要求你设置一个密码，比如：`[SakuraCN$Student_content]`

我们设定数据库信息为：必须四要素

- 数据库名：SakuraCN$Student_content 
- 密码是：20030908xxf@
- Database host address:SakuraCN.mysql.pythonanywhere-services.com 
- Username:SakuraCN

### 在 WSGI 文件中设置环境变量

请按照以下步骤操作：

1. **点击 "WSGI configuration file" 链接**（`/var/www/sakuracn_pythonanywhere_com_wsgi.py`）。

2. **删除文件中的所有现有内容**，然后**完全替换**为以下代码：

   python

   ```
   import os
   import sys
   
   # ==================== 设置环境变量 ====================
   # 数据库配置
   os.environ['DB_NAME'] = 'SakuraCN$Student_content'
   os.environ['DB_USER'] = 'SakuraCN'
   os.environ['DB_PASSWORD'] = '20030908xxf@'
   os.environ['DB_HOST'] = 'SakuraCN.mysql.pythonanywhere-services.com'
   
   # 从你本地的 settings.py 中复制 SECRET_KEY 的值，替换下面的内容
   os.environ['SECRET_KEY'] = '你的Django项目SecretKey' 
   
   # 生产环境设置
   os.environ['DEBUG'] = 'False'
   os.environ['ALLOWED_HOSTS'] = 'sakuracn.pythonanywhere.com'
   
   # ==================== 配置路径 ====================
   # 你的项目路径
   path = '/home/SakuraCN/Student_mag'
   if path not in sys.path:
       sys.path.append(path)
   
   # 告诉Django使用哪个设置文件
   os.environ['DJANGO_SETTINGS_MODULE'] = 'Student.settings'
   
   # ==================== 启动Django应用 ====================
   from django.core.wsgi import get_wsgi_application
   application = get_wsgi_application()
   ```

3. **特别重要**：将 `'你的Django项目SecretKey'` 替换为你本地 `settings.py` 文件中的实际 `SECRET_KEY` 值。这个值对于Django的安全至关重要。

4. **保存文件**。

### 接下来继续执行命令

现在回到你的 **Bash 控制台**（确保在 `~/Student_mag` 目录下且虚拟环境 `(venv)` 已激活），运行以下命令：

bash

```python
# 应用数据库迁移
python manage.py migrate

# 收集静态文件
python manage.py collectstatic
# 当问 "Are you sure you want to do this?" 时，输入 yes
```

***

### Invalid HTTP_HOST header: 'sakuracn.pythonanywhere.com'. You may need to add 'sakuracn.pythonanywhere.com' to ALLOWED_HOSTS.

这个错误是Django的安全特性之一，它告诉我们：**“我只接受来自我信任的域名的请求”**。

### 问题解释

- **`HTTP_HOST`**：当你的浏览器访问 `https://sakuracn.pythonanywhere.com` 时，它会发送一个 `Host` 头，其值就是 `sakuracn.pythonanywhere.com`。
- **`ALLOWED_HOSTS`**：这是Django设置中的一个安全列表。Django会检查收到的请求的 `Host` 头是否在这个列表中，如果不在，就会拒绝请求，并抛出这个错误。这是为了防止一种叫做"HTTP Host Header"的攻击。
- 在开发时（使用 `runserver`），`DEBUG=True`，Django对这个检查比较宽松。但在生产环境（`DEBUG=False`），这个检查是强制性的。

### 解决方案

我们需要在你的Django设置中明确告诉它：**“请信任来自 `sakuracn.pythonanywhere.com` 的请求”**。

请按照以下步骤修改你的 `settings.py` 文件：

1. **在你的本地电脑上**，用编辑器打开 `settings.py` 文件。

2. 找到 `ALLOWED_HOSTS` 这个设置。它可能看起来像这样：

   python

   ```
   ALLOWED_HOSTS = []
   ```

   或者

   python

   ```
   ALLOWED_HOSTS = ['localhost', '127.0.0.1']
   ```

3. **修改 `ALLOWED_HOSTS`**，将你的PythonAnywhere域名添加进去：

   python

   ```
   ALLOWED_HOSTS = ['sakuracn.pythonanywhere.com', 'localhost', '127.0.0.1']
   ```

   *如果你的 `ALLOWED_HOSTS` 设置不存在，就直接添加这一行。*

***

### 数据库链接错误

本地端的数据库链接时无法直接拿到线上去用的必须要改为线上版本的数据链接

错误信息：***`(2003, "Can't connect to MySQL server on '127.0.0.1:3306' (111)")`***

问题的根源。当前的数据库配置是硬编码的本地开发环境设置，它试图连接到你本地电脑上的MySQL服务器（`127.0.0.1`），而不是PythonAnywhere提供的MySQL服务器。

应该将数据库`settings.py`里面的配置改为如下所示

```python
import os
from pathlib import Path

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': os.environ.get('DB_NAME', 'studentmanager'),
        'USER': os.environ.get('DB_USER', 'root'),
        'PASSWORD': os.environ.get('DB_PASSWORD', '123456'),
        'HOST': os.environ.get('DB_HOST', 'localhost'),
        'PORT': os.environ.get('DB_PORT', '3306'),
        'OPTIONS': {
            'init_command': "SET sql_mode='STRICT_TRANS_TABLES'",
        }
    }
}
```



***

### 静态文件加载

如果不进行静态文件的配置。那么网站将显示错误，无法加载样式和项目内嵌图片等，

### 问题原因

在Django中，静态文件（CSS、JavaScript、图片）需要被收集到一个统一的目录，并通过Web服务器（如PythonAnywhere的Apache/Nginx）直接提供，而不是通过Django应用本身。您需要完成两个步骤：

1. 配置Django知道在哪里找到静态文件
2. 运行命令收集所有静态文件到一个目录
3. 配置PythonAnywhere的Web应用来提供这些静态文件

### 解决方案

#### 第1步：检查并修改Django设置

打开您的 `Student/settings.py` 文件，检查或添加以下设置：

python

```
# 静态文件的URL前缀，浏览器将通过这个URL访问静态文件
STATIC_URL = '/static/'

# 开发时存放静态文件的目录
STATICFILES_DIRS = [
    os.path.join(BASE_DIR, 'app01/static'),  # 根据您的项目结构，指向app01下的static目录
]

# 执行collectstatic命令后，静态文件将被收集到这个目录
STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')
```

#### 第2步：运行collectstatic命令

在PythonAnywhere的Bash终端中，导航到您的项目目录并运行：

bash

```
python manage.py collectstatic
```

这个命令会将所有应用（包括Django自带admin）的静态文件，以及您在 `STATICFILES_DIRS` 中指定的静态文件，全部复制到 `STATIC_ROOT` 目录（即 `~/Student_mag/staticfiles`）。

系统会提示您确认，输入 "yes" 继续。

#### 第3步：配置PythonAnywhere的静态文件映射

这是最关键的一步！

1. 登录PythonAnywhere，转到 **Web** 选项卡

2. 向下滚动到 **"Static files"** 部分

3. 添加一条新的静态文件映射：

   - **URL**: `/static/` (这应该与您的 `STATIC_URL` 设置匹配)
   - **Directory**: `/home/SakuraCN/Student_mag/staticfiles` (这应该与您的 `STATIC_ROOT` 路径匹配)

   **注意**：请将 `SakuraCN` 替换为您的实际用户名。

4. 点击 **"Reload"** 按钮重新加载您的Web应用



***

###　github代码克隆：

`git clone https://github.com/CN-hunt/Student_Manager.git Student_mag`

1. **从GitHub克隆代码**：

   - 点击顶部 **"Consoles"** 标签页。

   - 点击 **"Start a new console"** -> **"Bash"**。

   - 在打开的黑色命令行窗口中，运行以下命令来克隆你的代码（请替换成你的GitHub地址）：

     bash

     ```
     git clone https://github.com/你的用户名/你的仓库名.git
     ```

   - 克隆完成后，输入 `ls` 命令，你应该能看到你的项目文件夹出现了。使用 `cd 你的仓库名` 进入项目目录。

2. Pythonanywhere拉取代码：

   ```
   cd ~/Student_mag
   git pull
   ```
   
3. 更新最新代码：

   `git pull origin main`

***

### 总览

### 第一步：在 PythonAnywhere 上创建并设置 MySQL 数据库

1. 登录 PythonAnywhere，进入主仪表板 (Dashboard)。
2. 点击顶部标签页中的 **"Databases"**。
3. 在 "Create a database" 部分，设置一个密码，然后点击 **"Create database"**。
4. 记下 PythonAnywhere 提供给您的信息：
   - **数据库名称** (通常是 `你的用户名$项目名`)
   - **用户名** (通常是你的 PythonAnywhere 用户名)
   - **密码** (你刚刚设置的)
   - **主机地址** (通常是 `你的用户名.mysql.pythonanywhere-services.com`)

### 第二步：准备本地项目

在将代码上传之前，我们需要调整本地设置，为生产环境做好准备。

1. **安装依赖包**：确保你有一个 `requirements.txt` 文件，列出了所有依赖。在本地终端运行：

   bash

   ```
   pip freeze > requirements.txt
   ```

   这会让 PythonAnywhere 知道要安装哪些包。

2. **创建生产环境设置**：为了避免在代码中硬编码生产环境的数据库密码，我们通常会创建一个 `prod.py` 设置文件。在你的 `settings.py` 同级目录下创建 `prod.py`：

   python

   ```
   # prod.py
   from .settings import *  # 导入所有原有设置
   import os
   
   # 重写数据库配置
   DATABASES = {
       'default': {
           'ENGINE': 'django.db.backends.mysql',
           'NAME': os.getenv('PA_DB_NAME'),     # 我们将使用环境变量
           'USER': os.getenv('PA_DB_USER'),
           'PASSWORD': os.getenv('PA_DB_PASSWORD'),
           'HOST': os.getenv('PA_DB_HOST'),
           'OPTIONS': {
               'init_command': "SET sql_mode='STRICT_TRANS_TABLES'",
           }
       }
   }
   
   # 设置静态文件根目录（PythonAnywhere 需要）
   STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')
   
   # 设置允许的hosts（非常重要！）
   ALLOWED_HOSTS = ['你的用户名.pythonanywhere.com']  # 替换成你的
   ```

   *注意：更安全的做法是使用环境变量或 PythonAnywhere 的 Web App 界面里的变量设置功能。*

### 第三步：上传代码到 GitHub

PythonAnywhere 可以直接从 GitHub 拉取代码，这是最简单的方法。

1. 在你的项目根目录，确保已初始化 git 并添加了所有文件：

   bash

   ```
   git init
   git add .
   git commit -m "Initial commit for deployment"
   ```

2. 在 GitHub 上创建一个新的代码库 (Repository)。

3. 将本地代码推送到 GitHub：

   bash

   ```
   git remote add origin https://github.com/你的用户名/你的仓库名.git
   git branch -M main
   git push -u origin main
   ```

### 第四步：在 PythonAnywhere 上配置 Web App

1. 回到 PythonAnywhere 仪表板，点击顶部标签页 **"Web"**。
2. 点击 **"Add a new web app"**。
3. 在弹出窗口中，选择 **"Manual configuration"** (Django 的版本可能变化，手动配置最可靠)。
4. 选择你的 Python 版本（应与你本地开发版本一致）。
5. 配置完成后，进入你的 Web App 配置页面。

### 第五步：在 PythonAnywhere 上部署

在 Web App 配置页面中，进行以下关键操作：

1. **从 GitHub 克隆代码**：

   - 点击 **"Consoles"** 标签页，然后开启一个 **"Bash"** 控制台。

   - 在控制台中运行：

     bash

     ```
     git clone https://github.com/你的用户名/你的仓库名.git
     ```

2. **配置虚拟环境 (Virtualenv)**：

   - 回到 **"Web"** 配置页面。
   - 在 "Virtualenv" 部分，输入你的虚拟环境路径，例如：`/home/你的用户名/你的仓库名/venv`。
   - 点击创建，PythonAnywhere 会自动为你创建并激活虚拟环境。

3. **安装依赖**：

   - 在 **"Bash"** 控制台中，进入你的项目目录：

     bash

     ```
     cd 你的仓库名
     ```

   - 安装依赖：

     bash

     ```
     pip install -r requirements.txt
     pip install mysqlclient # 如果需要，MySQL 驱动
     ```

4. **配置 WSGI 文件**：

   - 在 **"Web"** 配置页面，找到并点击 **"WSGI configuration file"** 链接。

   - 删除文件中的所有内容，替换为以下代码，**注意修改路径和设置文件名为你的实际信息**：

     python

     ```
     import os
     import sys
     
     path = '/home/你的用户名/你的仓库名'  # 你的项目路径
     if path not in sys.path:
         sys.path.append(path)
     
     os.environ['DJANGO_SETTINGS_MODULE'] = '你的项目目录名.prod'  # 指向你的生产设置文件
     
     from django.core.wsgi import get_wsgi_application
     application = get_wsgi_application()
     ```

5. **配置静态文件**：

   - 在 **"Web"** 配置页面，找到 **"Static files"** 部分。
   - 添加一条记录：
     - URL: `/static/`
     - Directory: `/home/你的用户名/你的仓库名/staticfiles` (与 `STATIC_ROOT` 设置一致)

6. **收集静态文件**：

   - 在 **"Bash"** 控制台中运行：

     bash

     ```
     python manage.py collectstatic
     ```

     这会把你所有的静态文件（admin的、你app的）都收集到 `staticfiles` 文件夹中。

7. **应用数据库迁移**：

   - 在运行迁移前，需要设置环境变量。在 **"Web"** 配置页面的 **"Code"** 部分，有一个 **"Environmental variables"** 的输入框。点击并添加你的数据库变量，例如：

     text

     ```
     PA_DB_NAME=你的数据库名
     PA_DB_USER=你的数据库用户名
     PA_DB_PASSWORD=你的数据库密码
     PA_DB_HOST=你的数据库主机地址
     ```

   - 然后在 **"Bash"** 控制台中运行：

     bash

     ```
     python manage.py migrate
     ```

8. **创建一个超级用户**（可选）：

   bash

   ```
   python manage.py createsuperuser
   ```

### 第六步：重启并访问

1. 回到 **"Web"** 配置页面顶部，点击大大的绿色按钮：**Reload 你的用户名.pythonanywhere.com**。
2. 等待几秒钟，然后在浏览器中访问你的网站：`https://你的用户名.pythonanywhere.com`。

### 可能出现的问题及排查

- **500错误**：检查 **"Web"** 页面的 **"Error log"** 链接，那里有详细的错误信息。最常见的原因是数据库连接失败或静态文件路径配置错误。
- **静态文件404**：确保 `STATIC_ROOT` 设置正确，并且运行了 `collectstatic`，并且在 Web App 配置中正确指向了 `staticfiles` 目录。
- **数据库连接错误**：仔细检查数据库名称、用户名、密码和主机地址，确保环境变量已正确设置。







