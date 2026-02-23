# Development

## Create an app
- `python manage.py startapp <app_name>`
- Add to app list
  - In `conf/settings.py` add:
    - ```
      INSTALLED_APPS = [
          ...
          "<app_name>",
      ]
      ```
- Add app URL path
  - In `conf/urls.py` add:
    - ```
      urlpatterns = [
          ...
          path("<app_path>", include("<app_name>.urls")),
      ]
      ```

## Creating a view
- Add view URL path
  - In `<app_name>/urls.py` add:
    - ```
      from django.urls import path
      from .views import <view_class>
    
      urlpatterns = [
          path("<app_path>", <view_class>.as_view(), name="<view_name>"),
      ]
      ```
    - `<app_path>` is the relative url to the view, ending in `/`, but not starting with '/'
      - eg. `""`, `"accounts/"`, `"blog/"`, etc.
    - Adding the `name` parameter is optional, but recommended
- Create view
  - In `<app_name>/views.py` add:
    - ```
      from django.views.generic import TemplateView
    
      class HomeView(TemplateView):
          template_name = "templates/<app_name>/home.html

          def get_context_data(self, **kwargs):
              context = super().get_context_data(**kwargs)
              context["name"] = "Sherlock"
              return context
      ```
- Create template
  - In `templates/<app_name>/home.html`
    - ```
      <html>
        <body>
          <h1>My Website</h1>
          <p>Hello, {% name %}!</p>
        </body>
      </html>
      ```

## Create database model
- Create model
  - In `models.py` add:
    - ```
      from django.db import models

      class Article(models.Model):
          author = models.ForeignKey(
              settings.AUTH_USER_MODEL,
              on_delete=models.CASCADE,
          )
          title = models.CharField(max_length=150)
          content = models.TextField()
          date = models.DateTimeField(auto_now_add=True)
    
          def __str__(self):
              return self.title
      ```
- Register model
  - In `<app_name>/admin.py` add:
    - ```
      from .models import Article, Comment
      
      class ArticleAdmin(admin.ModelAdmin):
          list_display = [
              "title",
              "author",
              "date",
              "content",
          ]
              
      admin.site.register(Article, ArticleAdmin)
      ```
- Create migrations
  - `python manage.py makemigrations <app_name>`
- Apply migrations
  - `python manage.py migrate`

## Run the development server
`python manage.py runserver`

Not to be used in a production environment!
