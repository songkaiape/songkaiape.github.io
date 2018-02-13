
在Flask Mega-Tutorial系列的第二部分中，我将讨论如何使用模板。

以下是本系列文章的列表，供大家参考。

- 第1章：你好，世界！
- 第2章：模板（本文）
- 第3章：Web窗体
- 第4章：数据库
- 第5章：用户登录
- 第6章：配置文件页面和头像
- 第7章：错误处理
- 第8章：追随者
- 第9章：分页
- 第10章：电子邮件支持
- 第11章：焕然一新（2018年2月13日发布）
- 第12章：日期和时间（2018年2月20日可用）
- 第13章：I18n和L10n （2018年2月27日发布）
- 第14章：Ajax （2018年3月6日发布）
- 第15章：更好的应用程序结构（2018年3月13日发布）
- 第16章：全文搜索（2018年3月20日发布）
- 第17章：Linux 上的部署（2018年3月27日发布）
- 第18章：Heroku的部署（2018年4月3日发布）
- 第19章：Docker容器部署（2018年4月10日发布）
- 第20章：一些JavaScript Magic （2018年4月17日发布）
- 第21章：用户通知（2018年4月24日发布）
- 第22章：后台作业（2018年5月1日发布）
- 第23章：应用程序编程接口（API）（2018年5月8日发布）
*注1：如果您正在寻找本教程的旧版本，它就在这里。*

*注2：如果您想支持我在此博客上的工作，或者只是没有耐心等待每周文章，我将提供本教程的完整版本，打包成电子书或一组视频。有关更多信息，请访问[learn.miguelgrinberg.com](https://learn.miguelgrinberg.com/)。*

完成[第1章](TODO)后，您应该有一个完全正常工作，但简单的Web应用程序，它具有以下文件结构：
```
microblog\
  venv\
  app\
    __init__.py
    routes.py
  microblog.py
```
运行你`FLASK_APP=microblog.py`在终端会话中设置的应用程序，然后执行flask run。这将启动带有应用程序的Web服务器，您可以通过在Web浏览器的地址栏中键入`http：// localhost：5000 / URL` 来打开该应用程序。

在本章中，您将继续编写之前的应用程序，重要的是，您将学习如何生成更复杂的网页，这些网页结构复杂且具有许多动态组件。如果您对应用程序或开发工作流程上有什么不明白的，请在继续之前复习[第1章](TODO)内容。

本章的GitHub链接是：[Browse](https://github.com/miguelgrinberg/microblog/tree/v0.1)，[Zip](https://github.com/miguelgrinberg/microblog/archive/v0.1.zip)，[Diff](https://github.com/miguelgrinberg/microblog/compare/v0.0...v0.1)。。

# 什么是模板？
我希望我的microblog的主页有一个欢迎用户的标题。目前，我将忽略应用程序还没有用户这一事实，因为这会在以后编写。因此，我将使用一个模拟用户，我将把它作为一个Python字典来实现，如下所示：
```python
user = {'username': 'Miguel'}
```
创建虚拟对象是一种有用的技术，可让您专注于应用程序的一部分，而无需担心系统中尚不存在的其他部分。我想设计我的应用程序的主页，而且我不希望因为我没有用户系统而影响我的开发，所以我创造一个虚拟用户对象，以便继续向前进行开发。

这个应用程序的视图函数返回一个简单的字符串。我现在想要做的是将返回的字符串扩展为完整的HTML页面，可能是这样的：
```python
#app / routes.py：从视图函数返回完整的HTML页面

from app import app

@app.route('/')
@app.route('/index')
def index():
    user = {'username': 'miguel'}
    return '''
<html>
    <head>
        <title>Home Page - Microblog</title>
    </head>
    <body>
        <h1>Hello, ''' + user['username'] + '''!</h1>
    </body>
</html>'''
```
如果您不熟悉HTML，我建议您阅读维基百科上的[HTML标记](https://en.wikipedia.org/wiki/HTML#Markup)进行简要介绍。

如上所示更新视图函数，并运行程序查看浏览器中网站的外观。

模拟用户

我希望你同意我的观点，上面用于向浏览器提供HTML的解决方案并不好。考虑一下当我有来自用户的博客帖子时，这个视图中的代码将会变得多么复杂，这些帖子会不断发生变化。应用程序也将有更多的视图函数将与其他URL相关联，所以想象如果有一天我决定更改此应用程序的布局，并且必须在每个视图函数中更新HTML。这显然不是我们想要的方案。

如果您可以将应用程序的逻辑与网页的布局区分开来，那么组织起来会更好，您不觉得吗？你甚至可以聘请一个网页设计师来创建一个杀手级网站，同时继续用Python编写应用程序逻辑。

模板有助于实现显示布局和业务逻辑之间的分离。在Flask中，模板被编写为单独的文件，存储在应用程序包内的`模板`文件夹中。因此，确保您位于microblog目录后，创建存储模板的目录：
```
(venv) $ mkdir app/templates
```
在下面你可以看到你的第一个模板，它的功能类似于`index()`上面的视图函数返回的HTML页面。在`app / templates / index.html`中写下这个文件：
```HTML
app / templates / index.html：主页面模板

<html>
    <head>
        <title>{{ title }} - Microblog</title>
    </head>
    <body>
        <h1>Hello, {{ user.username }}!</h1>
    </body>
</html>
```
这是一个非常标准的，非常简单的HTML页面。在这个页面中唯一有趣的事情是有几个动态内容的占位符，在`{{ ... }}`部分中。这些占位符表示页面中可变的部分，并且只在运行时才知道具体值。

现在，页面的显示布局已被分离到HTML模板，可以简化视图功能：
```python
#app / routes.py：使用render \ _template（）函数

from flask import render_template
from app import app

@app.route('/')
@app.route('/index')
def index():
    user = {'username': 'Miguel'}
    return render_template('index.html', title='Home', user=user)
```
这看起来好多了，对吧？试试这个新版本的应用程序来看看模板是如何工作的。在浏览器中加载页面后，您可能需要查看源代码HTML并将其与原始模板进行比较。

将模板转换为完整的HTML页面的操作称为`渲染`。为了呈现模板，我必须导入一个名为Flask框架的函数`render_template()`。此函数接受一个模板文件名和模板参数的变量列表作为参数，并返回相同的模板，但其中的所有占位符都会用实际值替换。

该`render_template()`函数调用与Flask框架附带的[Jinja2](http://jinja.pocoo.org/)模板引擎。`Jinja2`使用`render_template()` 种相应的参数值去替换 `{{ ... }}`。

# 条件声明
您已经看到Jinja2在渲染过程中是如何用实际值替换占位符，但这只是Jinja2在模板文件中支持的许多强大操作之一。例如，模板也支持内部`{{% ... %}}`块中给出的控制语句。*index.html*模板的下一个版本添加了一个条件语句：
```HTML
app / templates / index.html：模板中的条件语句

<html>
    <head>
        {% if title %}
        <title>{{ title }} - Microblog</title>
        {% else %}
        <title>Welcome to Microblog!</title>
        {% endif %}
    </head>
    <body>
        <h1>Hello, {{ user.username }}!</h1>
    </body>
</html>
```
现在这个模板智能了一些。如果视图函数忘记传递`title`占位符变量的值，那么模板将显示一个默认标题，而不是显示空标题。您可以通过删除视图函数调用中的`title`参数来尝试此条件如何工作`render_template()`。

# 循环
登录的用户可能想要查看主页中来自关注用户的最新帖子，因此我现在要做的就是扩展应用程序以支持该功能。

再次，我将依靠虚拟对象技巧来创建一些用户和一些帖子来显示：
```python
app / routes.py：视图函数中的虚拟帖子

from flask import render_template
from app import app

@app.route('/')
@app.route('/index')
def index():
    user = {'username': 'Miguel'}
    posts = [
        {
            'author': {'username': 'John'},
            'body': 'Beautiful day in Portland!'
        },
        {
            'author': {'username': 'Susan'},
            'body': 'The Avengers movie was so cool!'
        }
    ]
    return render_template('index.html', title='Home', user=user, posts=posts)
```
为了展示用户的帖子表示我用一个列表来存储，列表中每个元素是一个包含`author`和`body`键值对的字典。当我开始真正实现用户和博客文章时，我将尽可能保留这些字段名称，以便我这些虚拟对象设计和测试主页模板的所有工作在引入了真正的用户和帖子之后还有效。

在模板方面，我必须解决一个新问题。帖子列表可以具有任意数量的元素，视图函数决定将在页面中呈现多少帖子。该模板无法确定有多少帖子，因此编写好视图函数来发送尽可能多的帖子。

对于这种类型的问题，Jinja2提供了一个for控制结构：
```HTML
app / templates / index.html：模板中的for-loop

<html>
    <head>
        {% if title %}
        <title>{{ title }} - Microblog</title>
        {% else %}
        <title>Welcome to Microblog</title>
        {% endif %}
    </head>
    <body>
        <h1>Hi, {{ user.username }}!</h1>
        {% for post in posts %}
        <div><p>{{ post.author.username }} says: <b>{{ post.body }}</b></p></div>
        {% endfor %}
    </body>
</html>
```
很简单，对吧？尝试一下这个新版本的应用程序，并且记住要在向帖子列表中添加更多内容以查看模板如何适应，并且始终呈现视图函数发送的所有帖子。

模拟帖子

# 模板继承
现在大多数Web应用程序都在页面顶部有一个导航栏，并带有一些常用链接，例如编辑配置文件的链接，登录，注销等。我可以轻松地将导航栏添加到`index.html`模板中更多的HTML，但随着应用程序的增长，我需要在其他页面中使用相同的导航栏。我并不想在许多HTML模板中保留导航栏的多个副本，如果可能的话，不要做重复劳动，这是一个好习惯。

Jinja2具有模板继承功能，专门解决此问题。实质上，您可以将页面布局中所有模板通用的部分移至基本模板，从中派生所有其他模板。

所以我现在要做的就是定义一个名为的基础模板`base.html`，它包含一个简单的导航栏和我之前实现的标题逻辑。您需要在文件`app / templates / base.html`中编写以下模板：
```HTML
app / templates / base.html：带导航栏的基本模板

<html>
    <head>
      {% if title %}
      <title>{{ title }} - Microblog</title>
      {% else %}
      <title>Welcome to Microblog</title>
      {% endif %}
    </head>
    <body>
        <div>Microblog: <a href="/index">Home</a></div>
        <hr>
        {% block content %}{% endblock %}
    </body>
</html>
```
在这个模板中，我使用`block`控制语句来定义派生模板插入的地方。block被赋予一个唯一的名称，派生模板引用它来提供内容。

使用基本模板后，我现在可以通过使index.html继承base.html来简化index.html：
```HTML
app / templates / index.html：从基本模板继承

{% extends "base.html" %}

{% block content %}
    <h1>Hi, {{ user.username }}!</h1>
    {% for post in posts %}
    <div><p>{{ post.author.username }} says: <b>{{ post.body }}</b></p></div>
    {% endfor %}
{% endblock %}
```
由于base.html模板现在将处理常规页面结构，因此我已从index.html中删除了所有这些元素，并仅保留了内容部分。`extends`语句建立了两个模板之间的继承关系，以便Jinja2知道当它被要求渲染index.html时需要将其嵌入到内部base.html。这两个模板具有`block`名称匹配的语句`content`，这就是Jinja2如何知道如何将两个模板合并成一个模板的原因。现在，如果我需要为应用程序创建更多页面，我可以将它们作为来自同一base.html模板的派生模板创建，这就是我可以让应用程序的所有页面共享相同的外观和感觉而不会重复的方式。

模板继承
