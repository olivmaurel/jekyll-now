---
layout: post
title: StreamingHttpResponse() with templates using Jinja
---

[As the official documentation
states](https://docs.djangoproject.com/en/1.10/ref/request-response/#django.http.StreamingHttpResponse),
Django is designed for short-lived requests, and it's preferable to
avoid streaming responses. But there are cases where you prefer to avoid
the normal request-response cycle and stream data in your http response.
Unfortunately, as you may already know, the StreamingHttpResponse()
class is not meant to be used with a template like the normal
HttpResponse() class. So you can stream your data to the browser using a
generator, and the only formatting allowed has to go through the
generator, leading to ugly and unmaintanable blobs of html (cf
myapp/streaminghttpresponse view).
