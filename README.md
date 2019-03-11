# Django-siteuser

Integrate user registration, login, upload avatar, third-party login and more. The goal of this project is to not write the user system when creating a new site.


## attention

* js in the project depends on jQuery, so make sure your site introduces jQuery
* Only third-party account login is implemented, and the function of binding multiple social accounts to a local account is not implemented.
* siteuser does not use the User system that comes with Django.


## Features

* register log in
* Login user to change password
* Reset password when you forget your password
* Upload avatar (with crop preview function)
* Third party account login
* notification


## how to use

#### Installation

```bash
Pip install django-siteuser
```

At the same time, the dependencies of this project will be installed:

* [socialoauth](https://github.com/yueyoum/social-oauth) - Third party login
* [django-celery](https://github.com/celery/django-celery) - Send mail asynchronously



#### Introducing the necessary js files in your template

```html
<script type="text/javascript" src="{{ STATIC_URL }}js/jquery.js"></script>
<script type="text/javascript" src="{{ STATIC_URL }}js/siteuser.js"></script>
```


#### Setting the settings.py file

* First set up [django-celery] (https://github.com/celery/django-celery)

* Add `siteuser` to `INSTALLED_APPS`
    ```python
    INSTALLED_APPS = (
        # ...
        'djcelery',
        'siteuser.users',
        'siteuser.upload_avatar',
        'siteuser.notify',
    )
    ```

    `siteuser.users` is ** must add ** app, the other two are used to upload avatars, one is a simple notification system, ** can not be added **


* Add `siteuser.SITEUSER_TEMPLATE` to `TEMPLATE_DIRS`

    **Note**: This is not a string, so you need to `import siteuser` in the `settings.py` file.


* Add `'siteuser.context_processors.social_sites'` to `TEMPLATE_CONTEXT_PROCESSORS`
* Add `'siteuser.middleware.User'` to `MIDDLEWARE_CLASSES`
* Add `url(r'', include('siteuser.urls'))` to the project's `urls.py`
* `AVATAR_DIR` - tells siteuser where to place the uploaded avatar
* `USING_SOCIAL_LOGIN` Whether to enable third-party account login. **If not set, the default is False**
* `SOCIALOAUTH_SITES` - Set only if `USING_SOCIAL_LOGIN` is True. The configuration required for third-party login. [See the socialauth documentation] (https://github.com/yueyoum/social-oauth/blob/master/doc.md#-settingspy)
* `SITEUSER_EXTEND_MODEL`
    If you don't set this item, the example can run as well, but in the actual project, the user field will definitely be set according to the project itself.
    For the default fields, please see [SiteUser](/siteuser/users/models.py#L108).
    
    Support two ways to extend the SiteUser field
    * Defined directly in `settings.py`
    
        ```python
        # project settings.py
        From django.db import models
        Class SITEUSER_EXTEND_MODEL(models.Model):
            # some fields...
            Class Meta:
                Abstract = True
        ```

    * Write the definition of this model in a different file and specify it in settings.py.

    
    The second type used by `example`, you can view the `example` project.

* `SITEUSER_ACCOUNT_MIXIN`
    Siteuser provides login, registered template, but just login, registration form template,
    And siteuser doesn't know how to embed this form template into your site, and doesn't know what additional context to pass in when rendering the template.
    So you need to define this setting yourself.
    
    As with `SITEUSER_EXTEND_MODEL`, there are also two method definitions.
    * Defined directly in `settings.py`
    
        ```python
        Class SITEUSER_ACCOUNT_MIXIN(object):
            Login_template = 'login.html' # Your project's login page template
            Register_template = 'register.html' # Registration page template for your project
            Reset_passwd_template = 'reset_password.html' # Forgot password reset password template
            Change_passwd_template = 'change_password.html' # Login User Change Password Template
            Reset_passwd_email_title = u'reset password' # reset password to send email header
            Reset_passwd_link_expired_in = 24 # Reset password link expires after several hours
            
            Def get_login_context(self, request):
                Return {}
                
            Def get_register_context(self, request):
                Return {}
        ```
        
        These two methods are just like their names. The request is the request that django passes to the view. You can return the context that needs to be passed to the template.
        Check the default [SiteUserMixIn] (/siteuser/users/views.py#L73) here.
        
    * The second method is to define this Mixin in a file and then specify it in settings.py
    
    The second type used by `example`, you can view the `example` project.


* `SITEUSER_EMAIL`

    Siteuser wrote his own email method, so he didn't use Django's own EMAI settings.
    So you need to set `SITEUSER_EMAIL`, see [local_settings.py.example](/example/example/local_settings.py.example)

    Which needs to be explained is the `display_from` setting.
    If you don't set up SMTP SERVER yourself, you don't have to buy a service to send mail.
    Instead, I applied for an email address directly at the email provider gmail, 163, sohu, etc.
    Then your mailbox is definitely in the form of name@gmail.com.
    At this time, if you want to display `abc@def.com` in someone else's inbox.
    Then set `display_from` to this value. If not set, the default is `from`


#### Template

You need to complete the login.html, register.html, account_settings.html templates yourself. (The name can be taken by yourself, as long as it is in the code
Just wait for it.) You only need to do one thing, that is, the corresponding siteuser template in your template's `include`.

For example, login.html is in the location you defined `{% include 'siteuser/login.html' %}`,

`{% include 'siteuser/upload_avatar.html' %} in account_settings.html

For details, please refer to the `example` project.

After the above settings, `python manage.py validate` is detected, syncdb, runserver can test registration, login, avatar, third-party login
