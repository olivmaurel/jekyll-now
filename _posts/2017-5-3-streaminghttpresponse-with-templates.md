---
layout: post
title: Using templates in your StreamingHttpResponse() with Jinja2
---

[As the official documentation
states](https://docs.djangoproject.com/en/1.10/ref/request-response/#django.http.StreamingHttpResponse), Django is designed for short-lived requests, and it's preferable to avoid streaming responses. But there are cases where you prefer to avoid the normal request-response cycle and stream data in your http response.

The problem : no templates with StreamingHttpResponse()
-------------------------------------------------------
In Django, as you may already know, the StreamingHttpResponse() class is not meant to be used with a template like the normal
HttpResponse() class. So you can stream your data to the browser using a generator, and the only formatting allowed has to go through the
generator, leading to ugly and unmaintanable blobs of html.


Solution : use jinja2.generate()
--------------------------------

When I stumbled upon this issue, I looked around on the web for a solution, someone suggested the jinja2 template engine with the generate() method, but with no indication on how to use it in Django. I hope this blog post will help others to implement and make usage of jinja2.generate() in their django projects. 

As a complement, I have created a sample django project illustrating the difference between HttpResponse(), StreamingHttpResponse(), and integrating jinja2.generate() with StreamingHttpResponse(). [Feel free to check it out.](https://github.com/olivmaurel/jinja_httpstream)

The solution in itself is pretty straightforward, requiring only to install jinja2 (covered further down), and adding a few lines of codes in your views.py file:

    import os
    from jinja2 import Environment, FileSystemLoader
    from django.http import StreamingHttpResponse

    def jinja_generate_with_template(template_filename, context):

        template_dir = os.path.dirname(os.path.abspath(__file__)) + '/templates/'
        
        j2_env = Environment(loader=FileSystemLoader(template_dir), trim_blocks=True)
        j2_template = j2_env.get_template(template_filename)
        j2_generator = j2_template.generate(context)
        return j2_generator

What's going on here?

- First, define manually the path for the template directory template_dir. Then create a jinja2.Environment object and point it to your template folder using the loader= attribute, and set the attribute trim_blocks to True to get rid of formatting problems caused by white spaces.
- Create a template object containing your template file using Environment.get_template(template_filename)
- Then you can define your generator using the template.generate() method. 
- The last step will be to return this generator for the StreamingHttpResponse() class to consume.

(Note: I volontarily detailed every step in the above example for the sake of clarity, feel free to modify it and reduce the number of lines in your own implementation)

You can then create a generator in your view function by calling the above function, then return it in a
StreamingHttpGenerator() class, like this:

    def your_view(request):
        your_template = 'jinja2/my_awesome_template.html'
        your_context = {'foo':'bar'}
        stream = jinja_generate_with_template(your_template, your_context)
        return StreamingHttpResponse(stream)

Once this is done, save your views.py file, open the web page in your browser, and see the magic for yourself! 

Example
-------
I have created a sample django project illustrating the difference between HttpResponse(), StreamingHttpResponse(), and integrating jinja2.generate() with StreamingHttpResponse(). [Feel free to check it out.](https://github.com/olivmaurel/jinja_httpstream)

Installing jinja2 in your django project
----------------------------------------

If you want to use jinja2.generate() to stream and render templates in your Django project, you can install jinja2 and with a few settings modifications you should be good to go.

If you already have jinja2 installed in your django project, just skip this part and use directly the example described above in views.py

[The solution described below comes from Jonathan Chu's blog](http://jonathanchu.is/posts/upgrading-jinja2-templates-django-18-with-admin)

1)  Install jinja2:

        pip install jinja2

2)  modify myproject/settings.py (change 'myproject' with the name of your
    app):

        TEMPLATES = [
        {
            'BACKEND': 'django.template.backends.jinja2.Jinja2',
            'DIRS': [
                os.path.join(BASE_DIR, 'templates/jinja2'),
            ],
            'APP_DIRS': True,
            'OPTIONS': {
                'environment': 'myproject.jinja2.environment',
            },
        },
        {
            'BACKEND': 'django.template.backends.django.DjangoTemplates',
            'DIRS': [],
            'APP_DIRS': True,
            'OPTIONS': {
                'context_processors': [
                    'django.contrib.auth.context_processors.auth',
                    'django.template.context_processors.debug',
                    'django.template.context_processors.i18n',
                    'django.template.context_processors.media',
                    'django.template.context_processors.static',
                    'django.template.context_processors.tz',
                    'django.contrib.messages.context_processors.messages',
                ],
            },
        },
        ]

Make sure to keep both jinja2 and django backend, since jinja2 templates may mess with the admin interface

3)  Create a dedicated folder for jinja2 templates under your
    application main folder:

        myproject
        ├── myproject
        │   ├── __init__.py
        │   ├── jinja2.py
        │   ├── settings.py
        │   ├── urls.py
        │   └── wsgi.py
        ├── manage.py
        ├── myapp
        │   └── views.py
        │   └── urls.py
        │   └── templates
        |        └──jinja2
        │           ├── base.html
        │           ├── home.html
        |

4)  Create a jinja2.py file at the same level as your settings.py file, and paste the following code in it:

        def environment(**options):
            env = Environment(**options)
            env.globals.update({
                'static': staticfiles_storage.url,
                'url': reverse,
            })
            return env

5) That's it. Now Django should be using Jinja2 template engine by default, which is by the way a huge improvement from the default template engine. [The official Jinja2 documentation has many examples and use cases](http://jinja.pocoo.org/docs/2.9) (although not this one!)

