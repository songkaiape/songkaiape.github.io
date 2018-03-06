这是Flask Mega-Tutorial系列的第三部分，我将告诉你如何使用Web表单。

以下是本系列文章的列表，供大家参考。

- 第1章：你好，世界！
- 第2章：模板
- 第3章：Web窗体 （本文）
- 第4章：数据库
- 第5章：用户登录
- 第6章：配置文件页面和头像
- 第7章：错误处理
- 第8章：如何添加关注
- 第9章：分页
- 第10章：电子邮件支持
- 第11章：改头换面（2018年2月13日发布）
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
- 第22章：后台工作（2018年5月1日发布）
- 第23章：应用程序编程接口（API）（2018年5月8日发布）
*注1：如果您正在寻找本教程的旧版本，它就在这里。*

*注2：如果您想支持我在此博客上的工作，或者只是没有耐心等待每周文章，我将提供本教程的完整版本，打包成电子书或一组视频。有关更多信息，请访问 learn.miguelgrinberg.com。*

在第2章中，我为应用程序的主页创建了一个简单的模板，并使用虚拟对象作为我还没有的对象的替代，例如用户或博客文章。在本章中，我将解决我在这个应用程序中仍然存在的众多漏洞之一，特别是如何通过Web表单接受来自用户的输入。

Web表单是任何Web应用程序中最基本的构建块之一。我将使用表单来允许用户提交博客帖子，也可以登录到应用程序。

在继续阅读本章之前，请确保您已安装了上一章中的微博应用程序，并且您可以无任何错误地运行微博应用程序。

本章的GitHub链接是：[Browse](https://github.com/miguelgrinberg/microblog/tree/v0.3)，[Zip](https://github.com/miguelgrinberg/microblog/archive/v0.3.zip)，[Diff](https://github.com/miguelgrinberg/microblog/compare/v0.2...v0.3)。

# Flask-WTF简介
为了处理这个应用程序中的Web表单，我将使用Flask-WTF扩展，这是一个封装了WTForms包扩展，它很好地与Flask集成在一起。这是我向你展示的第一个Flask扩展，但它不会是最后一个。扩展程序是Flask生态系统中非常重要的一部分，因为它们为Flask故意没有斟酌的问题提供了解决方案。

Flask扩展是与之一起安装的常规Python包pip。您可以继续在您的虚拟环境中安装Flask-WTF：

(venv) $ pip install flask-wtf
# 配置
到目前为止，该应用程序非常简单，因此我不必担心其配置。但对于除最简单应用程序之外的任何应用程序，您都会发现Flask（以及可能还有您使用的Flask扩展）为如何做事提供了一定的自由度，你需要手动去配置一些变量。

应用程序有几种格式可以指定配置选项。最基本的解决方案是将变量定义为键入app.config，它使用字典格式来处理变量。例如：
```python
app = Flask(__name__)
app.config['SECRET_KEY'] = 'you-will-never-guess'
# ... add more variables here as needed
```
虽然上面的语法足以为Flask创建配置选项，但我喜欢更精细的划分配置，所以不是将我的配置放在创建我的应用程序的相同位置，而是使用稍微更精细的结构，以便让我将我的配置保存在单独的文件中。

我非常喜欢这种格式，因为它具有很强的扩展性，它使用一个类来存储配置变量。为了保持组织良好，我将在单独的Python模块中创建配置类。您可以在下面看到此应用程序的新配置类，它存储在顶层目录的config.py模块中。
```python
#config.py：密钥配置

import os

class Config(object):
    SECRET_KEY = os.environ.get('SECRET_KEY') or 'you-will-never-guess'
```
很简单，对吧？配置设置在类中定义为类变量Config。由于应用程序需要更多配置，可以将它们添加到此类中，稍后，如果我发现需要多个配置集，则可以创建它的子类。但现在还不要担心这一点。

`SECRET_KEY`作为唯一配置项添加的一个配置变量是大多数Flask应用程序中的重要组成部分。Flask及其一些扩展使用密钥的值作为加密密钥，用于生成签名或令牌。Flask-WTF扩展使用它来保护网页表单免受名为Cross-Site Request Forgery或CSRF（发音为“seasurf”）的恶意攻击。顾名思义，密钥应该是保密的，因为由它产生的令牌和签名的强度取决多少人知道。

密钥的值被设置为由两个项组成的表达式，由`or`连接。第一个函数查找当前环境变量中是否包含`SECRET_KEY`。第二项只是一个硬编码的字符串。这是一种你会看到我经常使用配置变量的模式。这个想法是首选来自环境变量的值，但如果环境没有定义变量，那么将使用硬编码字符串。在开发此应用程序时，安全性要求较低，因此您可以忽略此设置并使用硬编码字符串。但是，当这个应用程序部署在生产服务器上时，我将在环境中设置一个独特且难以猜测的值，以便服务器拥有一个别人不知道的安全密钥。

现在我有一个配置文件，我需要告诉Flask读取它并应用它。这可以在使用app.config.from_object()方法加载配置类：
```python
#app / __init__.py：Flask配置

from flask import Flask
from config import Config

app = Flask(__name__)
app.config.from_object(Config)

from app import routes
```
我导入这个Config类的方式一开始可能会让人感到困惑，但如果你看看Flask从flask包（小写字母“f”）导入的类（大写字母“F ”），你会注意到我正在做同样的事情与配置。小写“config”是Python模块config.py的名称，显然带有大写“C”的是实际的类。

正如我上面提到的那样，配置项可以通过字典语法来访问app.config。在这里你可以看到与Python解释器的快速会话，我在这里检查密钥的值：
```python
>>> from microblog import app
>>> app.config['SECRET_KEY']
'you-will-never-guess'
```
# 用户登录表单
Flask-WTF扩展使用Python类来表示Web表单。表单类只是将表单的字段定义为类变量。

考虑到问题再次分离，我将使用新的app / forms.py模块来存储我的Web表单类。首先，我们定义一个用户登录表单，要求用户输入用户名和密码。该表单还将包含一个“记住我”复选框和一个提交按钮：
```python
#app / forms.py：登录表单

from flask_wtf import FlaskForm
from wtforms import StringField, PasswordField, BooleanField, SubmitField
from wtforms.validators import DataRequired

class LoginForm(FlaskForm):
    username = StringField('Username', validators=[DataRequired()])
    password = PasswordField('Password', validators=[DataRequired()])
    remember_me = BooleanField('Remember Me')
    submit = SubmitField('Sign In')
```
大多数Flask扩展`flask_<name>`为其顶级导入符号使用命名约定。在这种情况下，Flask-WTF具有其所有的符号`flask_wtf`。所以在app / forms.py的顶部导入了基类`FlasekForm`。

由于Flask-WTF扩展不提供自定义版本，因此表示我用于此表单的字段类型的四个类直接从WTForms包导入。对于每个字段，都会在`LoginForm`类中创建一个对象作为类变量。每个字段都有一个描述或标签作为第一个参数。

可选参数`validators`将验证行为附加到字段。该`DataRequired`验证程序只是简单地检查该字段不会提交空值。还有更多的验证器可用，其中一些将以其他形式使用。

# 表单模板

下一步是将表单添加到HTML模板，以便它可以在网页上呈现。好消息是，在`LoginForm`该类中定义的字段知道如何将自己呈现为HTML，所以这个任务非常简单。您可以在下面看到登录模板，我将在文件`app / templates / login.html`中存储该模板：
```html
app / templates / login.html：登录表单模板

{% extends "base.html" %}

{% block content %}
    <h1>Sign In</h1>
    <form action="" method="post">
        {{ form.hidden_tag() }}
        <p>
            {{ form.username.label }}<br>
            {{ form.username(size=32) }}
        </p>
        <p>
            {{ form.password.label }}<br>
            {{ form.password(size=32) }}
        </p>
        <p>{{ form.remember_me() }} {{ form.remember_me.label }}</p>
        <p>{{ form.submit() }}</p>
    </form>
{% endblock %}
```
对于这个模板，我将通过模板继承语句`extend`再次使用base.html模板，如第2章所示。事实上所有的模板都将集成这个`base.html`，以确保在应用程序的所有页面中顶部导航栏的布局一致。

这个模板需要一个实例化的`LoginForm`类表单对象作为参数，你可以看到它被引用为form。这个参数将通过登录视图功能发送，这部分我们后面会介绍。

`HTML <form>`元素用作Web表单的容器。`action`属性用于告诉浏览器在提交用户在表单中输入的信息时之后应该使用的URL。将操作设置为空字符串时，表单将提交给当前位于地址栏中的URL，该地址栏是在页面上呈现表单的URL。该`method`属性指定将表单提交给服务器时应使用的HTTP请求方法。默认情况下是发送一个`GET`请求，但几乎在所有情况下，使用`POST`请求可以提供更好的用户体验，因为此类型的请求可以在请求主体中提交表单数据，而GET请求则会将表单字段添加到URL，混乱浏览器地址栏。

该`form.hidden_tag()`模板参数生成一个隐藏字段，其中包含一个用来防止CSRF攻击形式的令牌。让这个表单免受攻击你只需要做的事情是包含这个语句并且在在Flask配置`SECRET_KEY`。`Flask-WTF`会为你做其余的事情。

如果您以前编写过HTML Web表单，那么您可能会发现这个模板中没有HTML字段是很奇怪的。这是因为表单对象中的字段知道如何将自己呈现为HTML。我需要做的只是在我想要的地方添加标签`{{ form.<field_name>.label }}`，以及用`{{ form.<field_name>() }}`添加字段。对于需要额外HTML属性的字段，这些字段可以作为参数传递。此模板中的用户名和密码字段将`size`作为参数添加到`<input>HTML`元素作为属性。这也是你可以附加CSS类或ID来表示字段的方式。

# 表单视图
在浏览器中查看此表单之前的最后一步是在应用程序中编写一个新的视图函数，该函数可呈现上一节中的模板。

因此，让我们编写一个映射到创建表单的`/login` URL 的新视图函数，并将其传递给模板进行呈现。这个视图函数也可以在前面的app / routes.py模块中进行：
```python
#app / routes.py：登录视图函数

from flask import render_template
from app import app
from app.forms import LoginForm

# ...

@app.route('/login')
def login():
    form = LoginForm()
    return render_template('login.html', title='Sign In', form=form)
```
我在这里做的是`LoginForm`从`forms.py`中导入类，从中实例化一个对象，并将其发送到模板。`form=form`语法可能看起来很奇怪，但简单地使form在上面的行中创建（和示出右侧）对象的名称的模板form（在左侧示出）。这是获取表单域所需的全部内容。

为了方便访问登录表单，基本模板可以在导航栏中包含指向它的链接：
```html
app / templates / base.html：导航栏中的登录链接

<div>
    Microblog:
    <a href="/index">Home</a>
    <a href="/login">Login</a>
</div>
```
此时，您可以运行该应用程序并在Web浏览器中查看表单。在应用程序运行时，输入`http://localhost:5000/`浏览器的地址栏，然后单击顶部导航栏中的“登录”链接以查看新的登录表单。很酷，对吧？


# 接收表格数据
如果您尝试按提交按钮，浏览器将显示“Method Not Allowed”错误。这是因为上一节中的登录视图功能迄今只完成了一半的工作。它可以在网页上显示表单，但它没有逻辑来处理用户提交的数据。这是Flask-WTF使工作变得非常简单的另一个方面。以下是视图函数的更新版本，它接受和验证用户提交的数据：
```python
#app / routes.py：接收登录凭证

from flask import render_template, flash, redirect

@app.route('/login', methods=['GET', 'POST'])
def login():
    form = LoginForm()
    if form.validate_on_submit():
        flash('Login requested for user {}, remember_me={}'.format(
            form.username.data, form.remember_me.data))
        return redirect('/index')
    return render_template('login.html', title='Sign In', form=form)
```
这个版本中的第一个新东西是`methods`路由装饰器中的参数。这告诉Flask该视图函数接受GET和POST请求，覆盖默认值，即只接受GET请求。HTTP协议规定GET请求是那些将信息返回给客户端（在这种情况下是网络浏览器）的请求。到目前为止，应用程序中的所有请求均属于此类型。POST请求通常在浏览器向服务器提交表单数据时使用（实际上GET请求也可用于此目的，但这不是推荐的做法）。由于浏览器试图发送一个错误，因此浏览器向您显示的“Method Not Allowed”错误出现POST请求和应用程序未配置为接受它。通过提供methods参数，您告诉Flask哪些请求方法应该被接受。

该`form.validate_on_submit()`方法完成所有表单处理工作。当浏览器发送GET请求接收带有表单的网页时，这个方法将返回False，所以在这种情况下，函数会跳过if语句并直接跳到在函数的最后一行中生成模板。

当浏览器`POST`由于用户按下提交按钮而发送请求时，form.validate_on_submit()将收集所有数据，运行附加到字段的所有验证器，并且如果一切都正常，它将返回True，表明数据是有效的并且可以由应用程序处理。但是，如果至少有一个字段未通过验证，那么函数将返回False，并且这将导致表单返回给用户，就像GET请求情况一样。稍后我会在验证失败时添加一条错误消息。

当`form.validate_on_submit()`返回`True`，登录视图功能调用了两个从Flask种导入新的函数。`flash()`函数是向用户显示消息的有用方法。许多应用程序都使用这种技术让用户知道某些操作是否成功。在这种情况下，我将使用这种机制作为临时解决方案，因为我没有所有必要的基础设施来登录真正的用户。我现在可以做的最好的是显示一条消息，确认应用程序已收到凭据。

登录视图函数中使用的第二个新函数是`redirect()`。此功能指示客户端Web浏览器自动导航到作为参数给出的页面。该视图函数使用它将用户重定向到应用程序的索引页面。

当您调用该`flash()`函数时，Flask会存储​​该消息，但闪烁的消息不会奇迹般地出现在网页中。应用程序的模板需要合适的网页布局来呈现这些闪烁的消息。我将把这些消息添加到基本模板，以便所有模板都继承此功能。这是更新的基本模板：
```html
app / templates / base.html：在基本模板中闪烁的消息

<html>
    <head>
        {% if title %}
        <title>{{ title }} - microblog</title>
        {% else %}
        <title>microblog</title>
        {% endif %}
    </head>
    <body>
        <div>
            Microblog:
            <a href="/index">Home</a>
            <a href="/login">Login</a>
        </div>
        <hr>
        {% with messages = get_flashed_messages() %}
        {% if messages %}
        <ul>
            {% for message in messages %}
            <li>{{ message }}</li>
            {% endfor %}
        </ul>
        {% endif %}
        {% endwith %}
        {% block content %}{% endblock %}
    </body>
</html>
```
在这里，我使用一个with构造函数将get_flashed_messages()的返回值赋值给messages变量，全部在模板的上下文中。`get_flashed_messages()`功能来自Flask，并返回`flash()`以前注册的所有消息的列表。接下来的条件检查是否messages有一些内容，并且在这种情况下，\<ul\>每个消息将呈现一个元素作为\<li\>列表项。这种渲染风格看起来不太好，我们会在后续章节改进。

这些闪存消息的一个有趣属性是，一旦通过`get_flashed_messages`函数请求了一次消息，它们将从消息列表中删除，因此它们在flash()调用该函数后仅出现一次。

这是再次尝试应用程序并测试表单如何工作的好时机。请确保您尝试提交空白的用户名或密码字段的表单，以查看`DataRequired`验证程序如何暂停提交过程。

# 改善字段验证
附加到表单域的验证器可防止无效数据被接收到应用程序中。应用程序处理无效表单输入的方式是通过重新显示表单来让用户进行必要的更正。

如果您尝试提交无效数据，我相信您注意到虽然验证机制运作良好，但没有向用户指示表单出现问题，用户只能看见表单没有变化。下一个任务是通过在验证失败的每个字段旁边添加有意义的错误消息来改善用户体验。

事实上，表单验证器已经生成了这些描述性错误消息，所以缺少的是模板中的一些其他逻辑来呈现它们。

以下是登录模板，在用户名和密码字段中添加了字段验证消息：
```html
app / templates / login.html：登录表单模板中的验证错误

{% extends "base.html" %}

{% block content %}
    <h1>Sign In</h1>
    <form action="" method="post">
        {{ form.hidden_tag() }}
        <p>
            {{ form.username.label }}<br>
            {{ form.username(size=32) }}<br>
            {% for error in form.username.errors %}
            <span style="color: red;">[{{ error }}]</span>
            {% endfor %}
        </p>
        <p>
            {{ form.password.label }}<br>
            {{ form.password(size=32) }}<br>
            {% for error in form.password.errors %}
            <span style="color: red;">[{{ error }}]</span>
            {% endfor %}
        </p>
        <p>{{ form.remember_me() }} {{ form.remember_me.label }}</p>
        <p>{{ form.submit() }}</p>
    </form>
{% endblock %}
```
我所做的唯一更改是在用户名和密码字段之后添加for循环，用于呈现红色验证器添加的错误消息。作为一般规则，任何附有验证程序的字段都将添加错误消息form.<field_name>.errors。这将成为一个列表，因为字段可以附加多个验证程序，并且多个验证程序可能会提供错误消息以向用户显示。

如果您尝试使用空的用户名或密码提交表单，您现在会看到一条红色的错误消息。

表单验证

# 生成链接
登录表单现在已经相当完整，但在结束本章之前，我想讨论在模板和重定向中包含链接的正确方法。到目前为止，您已经看到了一些定义链接的实例。例如，这是基本模板中的当前导航栏：
```html
    <div>
        Microblog:
        <a href="/index">Home</a>
        <a href="/login">Login</a>
    </div>
```
登录视图函数还定义了传递给该`redirect()`函数的链接：
```python
@app.route('/login', methods=['GET', 'POST'])
def login():
    form = LoginForm()
    if form.validate_on_submit():
        # ...
        return redirect('/index')
    # ...
```
直接在模板和源文件中编写链接的一个问题是，如果有一天您决定重新组织链接，那么您将不得不在整个应用程序中搜索并替换这些链接。

为了更好地控制这些链接，Flask提供了一个名为的函数`url_for()`，它使用URL的内部映射来生成URL以查看函数。例如，url_for('login')返回/login并url_for('index')返回'/index。参数url_for()是端点名称，它是视图函数的名称。

您可能会问，为什么使用函数名称而不是URL更好？事实是，网址更可能改变比查看功能名称，这完全是内部的。第二个原因是，稍后您会了解到，某些网址中包含动态组件，因此手动生成这些网址需要连接多个元素，这很枯燥且容易出错。该url_for()还能够产生这些复杂的URL。

所以从现在开始，每当我需要生成一个应用程序URL时，我都会使用url_for()。基本模板中的导航栏会变成：
```html
app / templates / base.html：使用链接的url_for（）函数

    <div>
        Microblog:
        <a href="{{ url_for('index') }}">Home</a>
        <a href="{{ url_for('login') }}">Login</a>
    </div>
```
这里是更新后的login()视图功能：
```python
app / routes.py：使用链接的url \ _for（）函数

from flask import render_template, flash, redirect, url_for

# ...

@app.route('/login', methods=['GET', 'POST'])
def login():
    form = LoginForm()
    if form.validate_on_submit():
        # ...
        return redirect(url_for('index'))
    # ...
```
