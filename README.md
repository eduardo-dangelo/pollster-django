# FIRST DJANGO APP

- [Official Django Documentation](https://www.djangoproject.com/)

## Intallations
- [Install Python](https://www.python.org/downloads/)
- install `pipenv` by running:
    ```
    pip3 install pipenv
    ```
  
## Set up

- Open a terminal window
- Navigate to your `projects` directory
- Create a folder with name `pollster-django`
- Navigate to `pollster-django` directory
```
cd projects
mkdir pollster-django
cd pollster-django
```
- Set up a virtual enviroment
    ```
    pipenv shell
    ```
- Install `Django`
    ```
    pipenv install django
    ```
- Start Project
    ```
    django-admin startproject pollster
    ```
- Navigate to your `pollster` directory
- Run server
    ```
    python3 manage.py runserver
    ```
  > You might be asked to run migrations, if so, stop the server and run
  > ```
  > python3 manage.py migrate
  > ```

## Creating app
```
python3 manage.py startapp polls
```
- Open `pollster/polls/models.py` create the basic models:
    ```
    from django.db import models

    class Question(models.Model):
        title = models.CharField(max_length=200)
        question_text = models.CharField(max_length=200)
        pub_date = models.DateTimeField('date published')

        def __str__(self):
            return self.question_text


    class Choice(models.Model):
        question = models.ForeignKey(Question, on_delete=models.CASCADE)
        choice_text = models.CharField(max_length=200)
        votes = models.IntegerField(default=0)

        def __str__(self):
            return self.choice_text

    ```

- Open `pollster/pollster/settings.py` and add `'polls.apps.PollsConfig'` inside `INSTALLED_APPS`:
    ```
    INSTALLED_APPS = [
        'polls.apps.PollsConfig',
        ...
    ]
    ```
- inside `pollster` directory, run Make migrations command:
    ```
    python3 manage.py makemigrations polls
    ```
- then run migrations:
    ```
    python3 manage.py migrate
    ```
  
> ### Manipulation data inside shell
> ```
> python3 manage.py shell
> ```
> ```
> >>> from polls.models import Question,Choice
> >>> from django.utils import timezone
> >>> q = Question(title="Question 1", question_text="What is you favourite Python framework?", pub_date=timezone.now())
> >>> q.save()
> >>> q = Question.objects.get(pk=1)
> >>> q.choice_set.create(choice_text="Django", votes=0)
> >>> q.choice_set.create(choice_text="Flask", votes=0)
> >>> q.choice_set.create(choice_text="Web2py", votes=0)
> >>> q.choice_set.all()
> >>> quit()
> ```

## Set up Admin area

- Create superuser:
  ```
  python3 manage.py createsuperuser
  ```
- Run server
    ```
    python3 manage.py runserver
    ```
- Go to `localhost:8000/admin` and log-in user superuser credentials
- Open `pollster/polls/admin.py` and register `Question` & `Choice`:
  ```
  from .models import Question, Choice
  
  admin.site.register(Question)
  admin.site.register(Choice)
  ```
  >You should see on `localhost:8000/admin` that 2 items apprear on the admin site
- Modify `admin.py` again to see choices inside question
  ```
  class ChoiceInline(admin.TabularInline):
      model = Choice
      extra = 3
  
  
  class QuestionAdmin(admin.ModelAdmin):
      fieldsets = [
          (None, {'fields': ['title', 'question_text', 'pub_date']}),
      ]
      inlines = [ChoiceInline]
  
  # admin.site.register(Question)
  # admin.site.register(Choice)
  admin.site.register(Question, QuestionAdmin)
  ```
  
## Set up Front Face App

### Create HTML Pages

- Inside `pollster`, create `templates` directory and then create`base.html`:
  ```
  <!DOCTYPE html>
  <html lang="en">
    <head>
      <meta charset="UTF-8" />
      <meta name="viewport" content="width=device-width, initial-scale=1.0" />
      <meta http-equiv="X-UA-Compatible" content="ie=edge" />
      <title>Pollster {% block title %}{% endblock %}</title>
      <link rel="stylesheet" href="style.css" />
      <!-- CSS only -->
      <link
        href="https://cdn.jsdelivr.net/npm/bootstrap@5.2.2/dist/css/bootstrap.min.css"
        rel="stylesheet"
        integrity="sha384-Zenh87qX5JnK2Jl0vWa8Ck2rdkQ2Bzep5IDxbcnCeuOxjzrPF/et3URy9Bv1WTRi"
        crossorigin="anonymous"
      />
    </head>
    <body>
      {% include 'partials/_navbar.html' %}
      <div class="container">
        <div class="row">
          <div class="col-md-6 m-auto">{% block content %}{% endblock %}</div>
        </div>
      </div>
    </body>
  </html>
  ```
- Inside `templates`, create `polls` directory and then create:
  - `index.html`:
    ```
    {% extends 'base.html' %}
    {% block content %}
      <h1 class="text-center mb-3">Poll Question</h1>
      {% if latest_question_list %}
        {% for question in latest_question_list %}
          <div class="card mb-3">
            <div class="card-body">
              <p class="lead">{{ question.title }}</p>
              <p class="lead">{{ question.question_text }}</p>
              <a href="{% url 'polls:detail' question.id %}" class="btn btn-primary btn-sm">
                Vote
              </a>
              <a href="{% url 'polls:results' question.id %}" class="btn btn-secondary btn-sm">
                result
              </a>
            </div>
          </div>
        {% endfor %}
      {% else %}
        <p>No Poll available</p>
      {% endif %}
    {% endblock content %}
    ```
  - `details.html`:
    ```
    {% extends 'base.html' %}
    {% block content %}
    <a class="btn btn-secondary btn-sm mb-3" href="{% url 'polls:index' %}">Back To Polls</a>
    <h1 class="text-center mb-3">{{ question.question_text }}</h1>
  
    {% if error_message %}
    <p class="alert alert-danger">
      <strong>{{ error_message }}</strong>
    </p>
    {% endif %}
  
    <form action="{% url 'polls:vote' question.id %}" method="post">
      {% csrf_token %}
      {% for choice in question.choice_set.all %}
      <div class="form-check">
        <input
            type="radio"
            name="choice"
            class="form-check-input"
            id="choice{{ forloop.counter }}"
            value="{{ choice.id }}"
        />
        <label for="choice{{ forloop.counter }}">{{ choice.choice_text }}</label
        >
      </div>
      {% endfor %}
      <input type="submit" value="Vote" class="btn btn-success btn-lg btn-block mt-4" />
    </form>
    {% endblock %}
    ```
  - `results.html`:
    ```
    {% extends 'base.html' %}
    {% block content %}
    <h1 class="mb-5 text-center">{{ question.question_text }}</h1>
  
    <ul class="list-group mb-5">
      {% for choice in question.choice_set.all %}
      <li class="list-group-item d-flex justify-content-between align-items-center">
        {{ choice.choice_text }}  <span class="badge bg-primary rounded-pill">{{ choice.votes }} vote{{ choice.votes | pluralize }}</span>
      </li>
      {% endfor %}
    </ul>
  
    <a class="btn btn-secondary" href="{% url 'polls:index' %}">Back To Polls</a>
    <a class="btn btn-dark" href="{% url 'polls:detail' question.id %}">Vote again?</a>
    {% endblock %}
    ```
- Inside `templates`, create `partials` directory and then create `_navbar.html`:
  ```
  <nav class="navbar navbar-dark bg-primary mb-4">
    <div class="container">
      <a class="navbar-brand" href="/">Pollster</a>
    </div>
  </nav>
  ```
## Rest of the setup

- Open `polls/views.py` and add the following code:
  ```
  from django.template import loader
  from django.http import HttpResponse, HttpResponseRedirect
  from django.shortcuts import get_object_or_404, render
  from django.urls import reverse
  from .models import Choice, Question
  
  
  def index(request):
      latest_question_list = Question.objects.order_by('-pub_date')[:5]
      context = { 'latest_question_list': latest_question_list }
      return render(request, 'polls/index.html', context)
  
  def detail(request, question_id):
      try:
          question = Question.objects.get(pk=question_id)
      except Question.DoesNotExist:
          raise Http404("Question does not exist")
      return render(request, 'polls/details.html', { 'question': question })
  
  def results(request, question_id):
      question = get_object_or_404(Question, pk=question_id)
      return render(request, 'polls/results.html', { 'question': question })
  
  def vote(request, question_id):
      # print(request.POST['choice'])
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

- inside `pollster/polls`, create a file `urls.py` and add the following code:
  ```
  from django.urls import path
  from . import views
  
  app_name = 'polls'
  urlpatterns = [
      path('', views.index, name='index'),
      path('<int:question_id>/', views.detail, name='detail'),
      path('<int:question_id>/results/', views.results, name='results'),
      path('<int:question_id>/vote/', views.vote, name='vote'),
  ]
  ```
- on `pollster/pollster/urls.py`, lets include polls urls inside `urlpatterns`:
  ```
  urlpatterns = [
      path('polls/', include('polls.urls')),
      ...
  ]
  ```
  > you will need to import `include` from `django.urls`:
  > ```
  > from django.urls import path, include
  > ```
- Open `pollster/pollster/settings.py`, on `TEMPLATES` section, add templates directory:
  ```
  'DIRS': [os.path.join(BASE_DIR, 'templates')],
  ```
  > you may need to import `os` by adding on top of the file: `import os`

### Finally, lets create an app for the landing page:
- Inside `pollster`, run:
  ```
  python3 manage.py startapp pages
  ```
  
- Inside `pollster/pages`, create an `urls.py`:
  ```
  from django.urls import path
  
  from . import views
  
  urlpatterns = [
      path('', views.index, name='index'),
  ]
  ```
  
- Open `views.py` and modify page content:
  ```
  from django.shortcuts import render
  
  def index(request):
    return render(request, 'pages/index.html')
  ```
  
- update `urlpatterns` on `pollster/pollster/urls.py`:
  ```
  urlpatterns = [
      path('', include('pages.urls')),
      path('polls/', include('polls.urls')),
      path('admin/', admin.site.urls),
  ]
  ```