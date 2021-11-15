

# Django Practice

###### created by Weber Huang at 2021-11-15

This repo is created to practicing the Python web framework, django referring to the official [documentation](https://docs.djangoproject.com/en/3.2/)

### Quick Start

#### Tutorial 01: Build up a poll view

Link: [Tutorial 01: Build up a poll view](https://docs.djangoproject.com/en/3.2/intro/tutorial01/)

Install Django

> Ignore this step if you use Pycharm Professional

```bash
# install package
$ python -m pip install Django
# check the version
$ python -m django --version

# create the project
$ cd <your dir>
$ django-admin startproject mysite

# create a polls practice
$ python manage.py startapp polls

# Write your first view at `polls/views.py`
# edit `polls/urls.py`
# edit `mysite/urls.py`

# verification
$ python manage.py runserver
```

file structure

```bash
mysite/
    manage.py
    mysite/
        __init__.py
        settings.py
        urls.py
        asgi.py
        wsgi.py
		polls/
		    __init__.py
		    admin.py
		    apps.py
		    migrations/
		        __init__.py
		    models.py
		    tests.py
		    views.py
```

The development server

```bash
# the defaut address with port is 127.0.0.1:8000
$ python manage.py runserver

# changing port
$ python manage.py runserver 8080

# changing the deployment address to 0.0.0.0
$ python manage.py runserver 0:8000
```

#### Tutorial 02: Database, Model, API 

Link: [Tutorial 02: Database, Model, API](https://docs.djangoproject.com/en/3.2/intro/tutorial02/)

預設使用sqlite，修改 `mysite/settings.py DATABASE` 設定來配置網站資料庫:

(記得先創建資料庫)

例如 :

```python
# default.ini
[DatabaseInfo]
ENGINE = django.db.backends.mysql
NAME = 
USER = 
PASSWORD = 
HOST = 
PORT = 
```

```python
config = ConfigParser()
config.read('default.ini')

DATABASES = {
    'default': {
        'ENGINE': config['DatabaseInfo']['ENGINE'],
        'NAME': config['DatabaseInfo']['NAME'],
        'USER': config['DatabaseInfo']['USER'],
        'PASSWORD': config['DatabaseInfo']['PASSWORD'],
        'HOST': config['DatabaseInfo']['HOST'],
        'PORT': config['DatabaseInfo']['PORT'],
    }
}
```

時間跟語言順便設定:

```bash
LANGUAGE_CODE = 'en-us' # or 'zh-hant'
TIME_ZONE = 'Asia/Taipei'
USE_I18N = True
USE_L10N = True
USE_TZ = False
```

設定完之後:

```bash
# 基於設定檔創建資料表
$ python manage.py migrate
```

創建模型:

```python
# polls/models.py
from django.db import models

class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')

class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
```

啟動模型:

```python
# mysite/settings.py
INSTALLED_APPS = [
    'polls.apps.PollsConfig', # 加入自定義models
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```

合併功能:

```bash
$ python manage.py makemigrations polls

Migrations for 'polls':
  polls/migrations/0001_initial.py
    - Create model Question
    - Create model Choice
```

> By running **`makemigrations`**, you’re telling Django that you’ve made some changes to your models (in this case, you’ve made new ones) and that you’d like the changes to be stored as a *migration*.

> Migrations are how Django stores changes to your models (and thus your database schema) - they’re files on disk. You can read the migration for your new model if you like; it’s the file **`polls/migrations/0001_initial.py`**

寫入資料庫:

```bash
$ python manage.py sqlmigrate polls 0001
```

自動產出寫入內容:

- 以下為支援 PostgreSQL 語法，不過 Django 會根據資料庫格式設定自動轉換為相對應的資料庫語法，無須手動調整。

```sql
BEGIN;
--
-- Create model Question
--
CREATE TABLE "polls_question" (
    "id" serial NOT NULL PRIMARY KEY,
    "question_text" varchar(200) NOT NULL,
    "pub_date" timestamp with time zone NOT NULL
);
--
-- Create model Choice
--
CREATE TABLE "polls_choice" (
    "id" serial NOT NULL PRIMARY KEY,
    "choice_text" varchar(200) NOT NULL,
    "votes" integer NOT NULL,
    "question_id" integer NOT NULL
);
ALTER TABLE "polls_choice"
  ADD CONSTRAINT "polls_choice_question_id_c5b4b260_fk_polls_question_id"
    FOREIGN KEY ("question_id")
    REFERENCES "polls_question" ("id")
    DEFERRABLE INITIALLY DEFERRED;
CREATE INDEX "polls_choice_question_id_c5b4b260" ON "polls_choice" ("question_id");

COMMIT;
```

執行創建對應資料表:

```bash
$ python manage.py migrate

Operations to perform:
  Apply all migrations: admin, auth, contenttypes, polls, sessions
Running migrations:
  Rendering model states... DONE
  Applying polls.0001_initial... OK
```

> Migrations are very powerful and let you change your models over time, as you develop your project, without the need to delete your database or tables and make new ones - it specializes in upgrading your database live, without losing data.

remember the three-step guide to making model changes:

- Change your models (in `models.py`)
- Run `python manage.py makemigrations` to create migrations for those changes
- Run `python manage.py migrate` to apply those changes to the database

Playing with the API:

```bash
# 進入互動式操作介面
$ python manage.py shell

>>> from polls.models import Choice, Question  # Import the model classes we just wrote.

# No questions are in the system yet.
>>> Question.objects.all()
<QuerySet []>

# Create a new Question.
# Support for time zones is enabled in the default settings file, so
# Django expects a datetime with tzinfo for pub_date. Use timezone.now()
# instead of datetime.datetime.now() and it will do the right thing.
>>> from django.utils import timezone
>>> q = Question(question_text="What's new?", pub_date=timezone.now())

# Save the object into the database. You have to call save() explicitly.
>>> q.save()

# Now it has an ID.
>>> q.id
1

# Access model field values via Python attributes.
>>> q.question_text
"What's new?"
>>> q.pub_date
datetime.datetime(2012, 2, 26, 13, 0, 0, 775217, tzinfo=<UTC>)

# Change values by changing the attributes, then calling save().
>>> q.question_text = "What's up?"
>>> q.save()

# objects.all() displays all the questions in the database.
>>> Question.objects.all()
<QuerySet [<Question: Question object (1)>]>
```

修改 model 加上 `__str__`

> It’s important to add `__str__()` methods to your models, not only for your own convenience when dealing with the interactive prompt, but also because objects’ representations are used throughout Django’s automatically-generated admin.

```python
from django.db import models

class Question(models.Model):
    # ...
    def __str__(self):
        return self.question_text

class Choice(models.Model):
    # ...
    def __str__(self):
        return self.choice_text
```

```python
import datetime

from django.db import models
from django.utils import timezone

class Question(models.Model):
    # ...
    def was_published_recently(self):
        return self.pub_date >= timezone.now() - datetime.timedelta(days=1)
```

```python
>>> from polls.models import Choice, Question

# Make sure our __str__() addition worked.
>>> Question.objects.all()
<QuerySet [<Question: What's up?>]>

# Django provides a rich database lookup API that's entirely driven by
# keyword arguments.
>>> Question.objects.filter(id=1)
<QuerySet [<Question: What's up?>]>
>>> Question.objects.filter(question_text__startswith='What')
<QuerySet [<Question: What's up?>]>

# Get the question that was published this year.
>>> from django.utils import timezone
>>> current_year = timezone.now().year
>>> Question.objects.get(pub_date__year=current_year)
<Question: What's up?>

# Request an ID that doesn't exist, this will raise an exception.
>>> Question.objects.get(id=2)
Traceback (most recent call last):
    ...
DoesNotExist: Question matching query does not exist.

# Lookup by a primary key is the most common case, so Django provides a
# shortcut for primary-key exact lookups.
# The following is identical to Question.objects.get(id=1).
>>> Question.objects.get(pk=1)
<Question: What's up?>

# Make sure our custom method worked.
>>> q = Question.objects.get(pk=1)
>>> q.was_published_recently()
True

# Give the Question a couple of Choices. The create call constructs a new
# Choice object, does the INSERT statement, adds the choice to the set
# of available choices and returns the new Choice object. Django creates
# a set to hold the "other side" of a ForeignKey relation
# (e.g. a question's choice) which can be accessed via the API.
>>> q = Question.objects.get(pk=1)

# Display any choices from the related object set -- none so far.
>>> q.choice_set.all()
<QuerySet []>

# Create three choices.
>>> q.choice_set.create(choice_text='Not much', votes=0)
<Choice: Not much>
>>> q.choice_set.create(choice_text='The sky', votes=0)
<Choice: The sky>
>>> c = q.choice_set.create(choice_text='Just hacking again', votes=0)

# Choice objects have API access to their related Question objects.
>>> c.question
<Question: What's up?>

# And vice versa: Question objects get access to Choice objects.
>>> q.choice_set.all()
<QuerySet [<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]>
>>> q.choice_set.count()
3

# The API automatically follows relationships as far as you need.
# Use double underscores to separate relationships.
# This works as many levels deep as you want; there's no limit.
# Find all Choices for any question whose pub_date is in this year
# (reusing the 'current_year' variable we created above).
>>> Choice.objects.filter(question__pub_date__year=current_year)
<QuerySet [<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]>

# Let's delete one of the choices. Use delete() for that.
>>> c = q.choice_set.filter(choice_text__startswith='Just hacking')
>>> c.delete()
```

Create Admin account:

```python
$ python manage.py createsuperuser
Username:
Email address:
Password:

# start the service
$ python manage.py runserver
```

Make the poll app modifiable in the admin

```python
# polls/admin.py
from django.contrib import admin

from .models import Question

admin.site.register(Question)
```

#### Tutorial 03: Views

Link: [Tutorial 03: Views](https://docs.djangoproject.com/en/3.2/intro/tutorial03/)

在 polls 中新增更多 views:

```python
# polls/views.py
def detail(request, question_id):
    return HttpResponse("You're looking at question %s." % question_id)

def results(request, question_id):
    response = "You're looking at the results of question %s."
    return HttpResponse(response % question_id)

def vote(request, question_id):
    return HttpResponse("You're voting on question %s." % question_id)
```

連接 views 與 urls:

```python
from django.urls import path

from . import views

urlpatterns = [
    # ex: /polls/
    path('', views.index, name='index'),
    # ex: /polls/5/
    path('<int:question_id>/', views.detail, name='detail'),
    # ex: /polls/5/results/
    path('<int:question_id>/results/', views.results, name='results'),
    # ex: /polls/5/vote/
    path('<int:question_id>/vote/', views.vote, name='vote'),
]
```

新增 view 實際功能:

```python
from django.http import HttpResponse

from .models import Question

def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    output = ', '.join([q.question_text for q in latest_question_list])
    return HttpResponse(output)

# Leave the rest of the views (detail, results, vote) unchanged
```

創建 index 功能相對應模板:

```python
# pools/templates/polls/index.html

{% if latest_question_list %}
    <ul>
    {% for question in latest_question_list %}
        <li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
    {% endfor %}
    </ul>
{% else %}
    <p>No polls are available.</p>
{% endif %}
```

更新 view index 功能，加上模板資訊:

```python
from django.http import HttpResponse
from django.template import loader

from .models import Question

def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    template = loader.get_template('polls/index.html')
    context = {
        'latest_question_list': latest_question_list,
    }
    return HttpResponse(template.render(context, request))
```

加入錯誤代碼訊息至 detail:

```python
# polls/views.py
from django.http import Http404
from django.shortcuts import render

from .models import Question
# ...
def detail(request, question_id):
    try:
        question = Question.objects.get(pk=question_id)
    except Question.DoesNotExist:
        raise Http404("Question does not exist")
    return render(request, 'polls/detail.html', {'question': question})
```

```python
# polls/templates/polls/detail.html
{{ question }}
```

修改錯誤代碼，引用 shortcut

```python
from django.shortcuts import get_object_or_404, render

from .models import Question
# ...
def detail(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, 'polls/detail.html', {'question': question})
```

```python
<h1>{{ question.question_text }}</h1>
<ul>
{% for choice in question.choice_set.all %}
    <li>{{ choice.choice_text }}</li>
{% endfor %}
</ul>
```

移除硬編碼問題:

> Remember, when we wrote the link to a question in the `polls/index.html` template, the link was partially hardcoded like this:

```python
# change
<li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
# to
<li><a href="{% url 'detail' question.id %}">{{ question.question_text }}</a></li>
# 可以這麼改原因在於我們在 url模組中有定義路徑
...
# the 'name' value as called by the {% url %} template tag
path('<int:question_id>/', views.detail, name='detail'),
... 
```

在pools/urls 模組中加入命名空間，避免與其他app混淆

```python
# polls/urls.py
from django.urls import path

from . import views

# add namesapce
app_name = 'polls'

urlpatterns = [
    path('', views.index, name='index'),
    path('<int:question_id>/', views.detail, name='detail'),
    path('<int:question_id>/results/', views.results, name='results'),
    path('<int:question_id>/vote/', views.vote, name='vote'),
]

# pools/templates/polls/index.html
# change 
<li><a href="{% url 'detail' question.id %}">{{ question.question_text }}</a></li>
# to 
<li><a href="{% url 'polls:detail' question.id %}">{{ question.question_text }}</a></li>
```

#### Tutorial 04: Form

Link: [Tutorial 04: Form](https://docs.djangoproject.com/en/3.2/intro/tutorial04/)

創建簡易表單:

```python
# polls/templates/polls/detail.html
<form action="{% url 'polls:vote' question.id %}" method="post">
{% csrf_token %}
<fieldset>
    <legend><h1>{{ question.question_text }}</h1></legend>
    {% if error_message %}<p><strong>{{ error_message }}</strong></p>{% endif %}
    {% for choice in question.choice_set.all %}
        <input type="radio" name="choice" id="choice{{ forloop.counter }}" value="{{ choice.id }}">
        <label for="choice{{ forloop.counter }}">{{ choice.choice_text }}</label><br>
    {% endfor %}
</fieldset>
<input type="submit" value="Vote">
</form>
```

> `forloop.counter` indicates how many times the `for` tag has gone through its loop. Since we’re creating a POST form (which can have the effect of modifying data), we need to worry about Cross Site Request Forgeries. Thankfully, you don’t have to worry too hard, because Django comes with a helpful system for protecting against it. In short, all POST forms that are targeted at internal URLs should use the `{% csrf_token %}` template tag.

修改 vote view:

```python
from django.http import HttpResponse, HttpResponseRedirect
from django.shortcuts import get_object_or_404, render
from django.urls import reverse

from .models import Choice, Question
# ...
def vote(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    try:
        selected_choice = question.choice_set.get(pk=request.POST['choice'])
    except (KeyError, Choice.DoesNotExist):
        # Redisplay the question voting form.
        return render(request, 'polls/detail.html', {
            'question': question,
            'error_message': "You didn't select a choice.",
        })
    else:
        selected_choice.votes += 1
        selected_choice.save()
        # Always return an HttpResponseRedirect after successfully dealing
        # with POST data. This prevents data from being posted twice if a
        # user hits the Back button.
        return HttpResponseRedirect(reverse('polls:results', args=(question.id,)))
```

> We are using the `reverse()` constructor in this example. This function helps avoid having to hardcode a URL in the view function. It is given the name of the view that we want to pass control to and the variable portion of the URL pattern that points to that view.

修改 results view and html:

```python
from django.shortcuts import get_object_or_404, render

def results(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, 'polls/results.html', {'question': question})
```

```python
<h1>{{ question.question_text }}</h1>

<ul>
{% for choice in question.choice_set.all %}
    <li>{{ choice.choice_text }} -- {{ choice.votes }} vote{{ choice.votes|pluralize }}</li>
{% endfor %}
</ul>

<a href="{% url 'polls:detail' question.id %}">Vote again?</a>
```

> The code for our `vote()` view does have a small problem. It first gets the `selected_choice` object from the database, then computes the new value of `votes`, and then saves it back to the database. If two users of your website try to vote at *exactly the same time*, this might go wrong: The same value, let’s say 42, will be retrieved for `votes`. Then, for both users the new value of 43 is computed and saved, but 44 would be the expected value. This is called a *race condition*. If you are interested, you can read [Avoiding race conditions using F()](https://docs.djangoproject.com/en/3.2/ref/models/expressions/#avoiding-race-conditions-using-f) to learn how you can solve this issue.

使用共通 view 程式碼減量:

1. Convert the `URLconf`.
2. Delete some of the old, unneeded views.
3. Introduce new views based on Django’s generic views.

```python
#polls/urls.py
from django.urls import path

from . import views

app_name = 'polls'
urlpatterns = [
    path('', views.IndexView.as_view(), name='index'),
    path('<int:pk>/', views.DetailView.as_view(), name='detail'),
    path('<int:pk>/results/', views.ResultsView.as_view(), name='results'),
    path('<int:question_id>/vote/', views.vote, name='vote'),
]

#polls/views.py
from django.http import HttpResponseRedirect
from django.shortcuts import get_object_or_404, render
from django.urls import reverse
from django.views import generic

from .models import Choice, Question

class IndexView(generic.ListView):
    template_name = 'polls/index.html'
    context_object_name = 'latest_question_list'

    def get_queryset(self):
        """Return the last five published questions."""
        return Question.objects.order_by('-pub_date')[:5]

class DetailView(generic.DetailView):
    model = Question
    template_name = 'polls/detail.html'

class ResultsView(generic.DetailView):
    model = Question
    template_name = 'polls/results.html'

def vote(request, question_id):
    ... # same as above, no changes needed.
```

> We’re using two generic views here: `ListView` and `DetailView`. Respectively, those two views abstract the concepts of “display a list of objects” and “display a detail page for a particular type of object.”

#### Tutorial 05: Automated tests

Link: [Tutorial 05: Automated tests](https://docs.djangoproject.com/en/3.2/intro/tutorial05/)

See the above tutorial for studying auto test in-depth

#### Tutorial 06: Static file

Link: [Tutorial 06: Static file](https://docs.djangoproject.com/en/3.2/intro/tutorial06/)

建立靜態資料夾: `polls/static/polls/`

建立靜態檔案:

```css
/* polls/static/polls/style.css */
li a {
    color: green;
}
```

> Just like templates, we *might* be able to get away with putting our static files directly in `polls/static` (rather than creating another `polls` subdirectory), but it would actually be a bad idea. Django will choose the first static file it finds whose name matches, and if you had a static file with the same name in a *different* application, Django would be unable to distinguish between them. We need to be able to point Django at the right one, and the best way to ensure this is by *namespacing* them. That is, by putting those static files inside *another* directory named for the application itself.

更新 index.html:

```html
<!-- 放在最前面 -->
{% load static %}

<link rel="stylesheet" type="text/css" href="{% static 'polls/style.css' %}">
```

> The `{% static %}` template tag is not available for use in static files which aren’t generated by Django, like your stylesheet. You should always use **relative paths** to link your static files between each other, because then you can change `STATIC_URL` (used by the `static` template tag to generate its URLs) without having to modify a bunch of paths in your static files as well.

加入背景圖片: `polls/static/polls/images/background.gif`

```css
/* polls/static/polls/style.css */
/* ... */
body {
    background: white url("images/background.gif") no-repeat;
}
```

```bash
$ python manage.py runserver
```

#### Tutorial 07: customized admin site

Link: [Tutorial 07: customizing admin site](https://docs.djangoproject.com/en/3.2/intro/tutorial07/)

客製化 admin 表格 (question):

```python
# polls/admin.py
from django.contrib import admin
from .models import Question

class QuestionAdmin(admin.ModelAdmin):
    fields = ['pub_date', 'question_text']

admin.site.register(Question, QuestionAdmin)

# ----
from django.contrib import admin
from .models import Question

class QuestionAdmin(admin.ModelAdmin):
    fieldsets = [
        (None,               {'fields': ['question_text']}),
        ('Date information', {'fields': ['pub_date']}),
    ]

admin.site.register(Question, QuestionAdmin)
```

客製化 admin 表格 (choice):

```python
# ---- adding choice 
from django.contrib import admin

from .models import Choice, Question
# ...
admin.site.register(Choice)

# ---- optimize choice with question
from django.contrib import admin
from .models import Choice, Question

class ChoiceInline(admin.StackedInline):
    model = Choice
    extra = 3

class QuestionAdmin(admin.ModelAdmin):
    fieldsets = [
        (None,               {'fields': ['question_text']}),
        ('Date information', {'fields': ['pub_date'], 'classes': ['collapse']}),
    ]
    inlines = [ChoiceInline]

admin.site.register(Question, QuestionAdmin)

# ---- change the display way from stacked to tabular
class ChoiceInline(admin.TabularInline):
    #...
```

客製化 change list:

```python
# polls/admin.py
class QuestionAdmin(admin.ModelAdmin):
    # ...
    list_display = ('question_text', 'pub_date')

# ----
class QuestionAdmin(admin.ModelAdmin):
    # ...
    list_display = ('question_text', 'pub_date', 'was_published_recently')

# ----
# Oops, it seems `was_published_recently` display isn't in a good way
# change it by edit polls/models.py
from django.contrib import admin

class Question(models.Model):
    # ...
    @admin.display(
        boolean=True,
        ordering='pub_date',
        description='Published recently?',
    )
    def was_published_recently(self):
        now = timezone.now()
        return now - datetime.timedelta(days=1) <= self.pub_date <= now
```

加上 filter 與 search bar:

```python
# polls/admin.py
from django.contrib import admin
from .models import Choice, Question

class QuestionAdmin(admin.ModelAdmin):
# ...
list_filter = ['pub_date']
search_fields = ['question_text']
```

客製化專案級模板:

- 在根目錄的 templates 創建 admin 資料夾

- 複製套件中模板 `admin/base_site.html` 原代碼 (`venv/lib/site-packages/django/contrib/admin/templates`) 至剛創建的路徑

  > `$ python -c "import django; print(django.__path__)"` 找出原代碼位置

- 複寫 `base_site.html` 檔案內容

  ```html
  <!-- 
  Replace all `{{ site_header|default:_('Django administration') }}` 
  with project name
  -->
  
  {% block branding %}
  <h1 id="site-name"><a href="{% url 'admin:index' %}">Polls Administration</a></h1>
  {% endblock %}
  ```

> Note that any of Django’s default admin templates can be overridden. To override a template, do the same thing you did with `base_site.html` – copy it from the default directory into your custom directory, and make changes.