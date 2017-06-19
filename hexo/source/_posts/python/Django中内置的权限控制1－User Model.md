---
title: Django中内置的权限控制1－User Model
date: 2017-06-15 14:17:27
categories:	django
tags: 
	- django
---

<!-- toc -->


## Django中内置的权限控制1－User Model

在Django的世界中，在权限管理中有内置的Authentication系统。用来管理帐户，组，和许可。还有基于cookie的用户session。这篇blog主要用来探讨这套内置的Authentication系统。

Django内置的权限系统包括以下三个部分：  

    用户（Users）
    许可（Permissions）：用来定义一个用户（user）是否能够做某项任务（task）
    组（Groups）：一种可以批量分配许可到多个用户的通用方式

首先需要在Django中安装这个组件：

    将'django.contrib.auth'和'django.contrib.contenttypes'放到settings.py中的INSTALLED_APPS中。（使用contenttypes的原因是auth中的Permission模型依赖于contenttypes）
    执行manage.py syncdb
装好了就可以开始使用了；我们可以执行manage.py shell来启动脚本，对其中的一些API进行学习和使用。

 

首先最重要的开始就是User模型
User模型对应于一个用户，一个帐户，位于'django.contrib.auth.models'模块中。

User对象有两个多对多的属性分别是：groups和user_permissions：


    >>>from django.contrib.auth.models import User
    >>>es = User.objects.create_user('esperyong','esperyong@gmail.com','123456')
    >>>es.groups
    <django.db.models.fields.related.ManyRelatedManager at 0x10d0642d0>
    >>>es.user_permissions
    <django.db.models.fields.related.ManyRelatedManager at 0x10d014c50>
从上面的代码中的后两行输出我们可以看到，User的这两个多对多属性，都是关于ManyRelatedManager的引用。因此我们可以像对所有对多关系属性一样使用：

直接将一个列表赋值给该属性：

    es.groups = [group_list]
    es.user_permissions = [permission_list]
使用add方法将对象加入：

    es.groups.add(group, group, ...)
    es.user_permissions.add(permission, permission, ...)
使用remove方法将对象删除：

    es.groups.remove(group, group, ...)
    es.user_permissions.remove(permission, permission, ...)
使用clear方法将所有对象删除：

    es.groups.clear()
    es.user_permissions.clear()
　　

User对象有以下几个属性：

    username:字符串类型。必填。30个字符以内。
    first_name:字符串类型。可选。30个字符以内。
    last_name:字符串类型。可选。30个字符以内。
    email:可选。
    password:明文密码的hash或者是某种元数据。该属性不应该直接赋值明文密码，而应该    通过set_password()方法进行赋值，在后面有详细说明TODO。
    is_staff:Boolean类型。用这个来判断是否用户可以登录进入admin site。
    is_active:Boolean类型。用来判断该用户是否是可用激活状态。在删除一个帐户的时候    ，可以选择将这个属性置为False，而不是真正删除。这样如果应用有外键引用到这个用    户，外键就不会被破坏。
    is_superuser:Boolean类型。该属性用来表示该用户拥有所有的许可，而无需明确的赋    予给他。
    last_login:datetime类型。最近一次登陆时间。
    date_joined：datetime类型。创建时间。

除了DjangoModel对象的通用方法之外，User对象有以下特有方法：

    is_anonymous():
    永远返回False.用来将User对象和AnonymousUser(未登录的匿名用户)对象作区分用的识    别方法。通常，最好用is_authenticated()方法。
    is_authenticated():
    永远返回True。该方法不代表该用户有任何的许可，也不代表该用户是active的，而只    是表明该用户提供了正确的username和password。
    get_full_name():
    返回一个字符串，是first_name和last_name中间加一个空格组成。
    set_password(raw_password):
    调用该方法时候传入一个明文密码，该方法会进行hash转换。该方法调用之后并不会保    存User对象。
    check_password(raw_password)：
    如果传入的明文密码是正确的返回True。该方法和set_password是一对，也会考虑hash    转换。
    set_unusable_password()：
    将用户设置为没有密码的状态。调用该方法后，check_password()方法将会永远返回fal    se。但是如果，调用set_password()方法重新设置密码后，该方法将会失效，has_usabl    e_password()也会返回True。
    has_usable_password()：
    在调用set_unusable_password()方法之后，该方法返回False，正常情况下返回True。
    get_group_permissions(obj=None)：
    返回该用户通过组所拥有的许可（字符串列表每一个代表一个许可）。obj如果指定，将    会返回关于该对象的许可，而不是模型。
    get_all_permissions(obj=None):
    返回该用户所拥有的所有的许可，包括通过组的和通过用户赋予的许可。
    has_perm(perm,obj=None)：
    如果用户有传入的perm，则返回True。perm可以是一个格式为：'<app     label>.<permission codename>'的字符串。如果User对象为inactive，该方法永远返回    False。和前面一样，如果传入obj，则判断该用户对于这个对象是否有这个许可。
    has_perms(perm_list,obj=None):
    和has_perm一样，不同的地方是第一个参数是一个perm列表，只有用户拥有传入的每一    个perm，返回值才是True。
    has_module_perms(package_name)：
    传入的是Django app label，按照'<app label>.<permission     codename>'格式。当用户拥有该app     label下面所有的perm时，返回值为True。如果用户为inactive，返回值永远为False。
    email_user(subject,message,from_email=None)：
    发送一封邮件给这个用户，依靠的当然是该用户的email属性。如果from_email不提供的    话，Django会使用settings中的DEFAULT_FROM_EMAIL发送。
    get_profile()：
    返回一个和Site相关的profile对象，用来存储额外的用户信息。这个返回值会在另一片    博文中详细描述。

User对象的Manager，UserManager： 

和其他的模型一样，User模型类的objects属性也是一个Manager对象，但是User的Manager对象是自定义的，增加了一些方法：

    create_user(username,email=None,password=None):该方法创建保存一个is_active=True的User对象并返回。username不能够为空，否则抛出ValueError异常。email和password都是可选的。email的domain部分会被自动转变为小写。password如果没有提供，则User对象的set_unusable_password()方法将会被调用。
    make_random_password(length=10,allowed_chars='abcdefghjkmnpqrstuvwxyzABCDEFGHJKLMNPQRSTUVWXYZ23456789'):该方法返回一个给定长度和允许字符集的密码。其中默认的allowed_chars有一些字符没有，比如i,l等等。
 
## Django admin定制化，User字段扩展[转载]

前言

参考上篇博文，我们利用了OneToOneField的方式使用了django自带的user，http://www.cnblogs.com/caseast/p/5909248.html ， 但这么用有个问题，就是每次增删改查数据，因为有外键的存在都要查询两次（当然可以用select_related方式减少查询次数，参考: Django models对象的select_related方法（减少查询次数） ），另外在admin中需要维护2张表，先创建User，再在UserProfile中进行关联操作。本篇就来介绍一下，如何定制User和admin达到以下目的：1.扩展django自带的User，且不通过OneToOne的方式。2.修改User中的字段，让诸如email这种字段变为必选项（默认为可选）。3.admin表单定制，让不同权限的用户显示不同的页面。
期间踩了很多坑，统一做一次整理，admin可定制的地方很多，但是定制的方法肯定不如自己写的后台那么灵活，需要大体了解django的User和admin的工作模式和源码，怎么取舍还看自己的需求了。

代码实现

扩展User的方法大概有4种，参考这个国外博客：https://simpleisbetterthancomplex.com/tutorial/2016/07/22/how-to-extend-django-user-model.html#proxy ，我用的是里边描述的第四种方法。
models.py

    from django.db import models
    from django.contrib.auth.models import AbstractUser, Group
    # Create your models here.
    
    ''' OneToOne的扩展写法,原来的写法
    class UserProfile(models.Model):
        user = models.OneToOneField(User)
        name = models.CharField(u'姓名', max_length=32, blank=False, null=False)
    
        class Meta:
            verbose_name = u'用户详情'
            verbose_name_plural = u"用户详情"
    '''
    
    class MyUser(AbstractUser): #     继承AbstractUser类，实际上django的User也是继承他，我们要做的就是用自己的类代    替django自己的User
        name = models.CharField(u'中文名', max_length=32, blank=False,     null=False)
    
        class Meta:
            verbose_name = u'用户详情'
            verbose_name_plural = u"用户详情"
这里附上Django User类部分源码

    class User(AbstractUser):
        """
        Users within the Django authentication system are represented by this     model.
        Username, password and email are required. Other fields are optional.
        """
        class Meta(AbstractUser.Meta):
            swappable = 'AUTH_USER_MODEL'
光做以上这些还不够，我们需要告诉django，我们不用你的User了，要用自己的，所以需要在settings.py里重新设定一个变量
settings.py

    AUTH_USER_MODEL = "web_sso.MyUser"  # 我们的app叫web_sso，这个MyUser就是models定义的那个类

看下扩展完的效果，可以看到，我们不用再像之前一样维护“用户”和“用户详情”两张表了，但还是有很多小问题需要解决。


长话短说，直接看一下，我的admin.py改了哪些东西吧.
admin.py

    from django.contrib import admin
    from web_sso import models
    from django.contrib.auth.admin import UserAdmin  # 从django继承过来后进行定制
    from django.utils.translation import ugettext_lazy as _
    from django.contrib.auth.forms import UserCreationForm, UserChangeForm #     admin中涉及到的两个表单
    
    
    class User_exAdmin(admin.ModelAdmin):  # 验证码部分展示
        list_display = ('valid_code', 'valid_time', 'email')
    
    
    # custom user admin
    class MyUserCreationForm(UserCreationForm):  #     增加用户表单重新定义，继承自UserCreationForm
        def __init__(self, *args, **kwargs):
            super(MyUserCreationForm, self).__init__(*args, **kwargs)
            self.fields['email'].required = True   #     为了让此字段在admin中为必选项，自定义一个form
            self.fields['name'].required = True  #     其实这个name字段可以不用设定required，因为在models中的MyUser类中已经设定了bla    nk=False，但email字段在系统自带User的models中已经设定为
            # email = models.EmailField(_('email address'),     blank=True)，除非直接改源码的django（不建议这么做），不然还是自定义一个表单做    一下继承吧。
    
    
    class MyUserChangeForm(UserChangeForm):  #     编辑用户表单重新定义，继承自UserChangeForm
        def __init__(self, *args, **kwargs):
            super(MyUserChangeForm, self).__init__(*args, **kwargs)
            self.fields['email'].required = True
            self.fields['name'].required = True
    
    
    class CustomUserAdmin(UserAdmin):
        def __init__(self, *args, **kwargs):
            super(CustomUserAdmin, self).__init__(*args, **kwargs)
            self.list_display = ('username', 'name', 'email', 'is_active',     'is_staff', 'is_superuser')
            self.search_fields = ('username', 'email', 'name')
            self.form = MyUserChangeForm  #  编辑用户表单，使用自定义的表单
            self.add_form = MyUserCreationForm  # 添加用户表单，使用自定义的表单
            # 以上的属性都可以在django源码的UserAdmin类中找到，我们做以覆盖
    
        def changelist_view(self, request, extra_context=None):  #     这个方法在源码的admin/options.py文件的ModelAdmin这个类中定义，我们要重新定义    它，以达到不同权限的用户，返回的表单内容不同
            if not request.user.is_superuser:  #     非super用户不能设定编辑是否为super用户
                self.fieldsets = ((None, {'fields': ('username', 'password',)}),
                                  (_('Personal info'), {'fields': ('name',     'email')}),  # _ 将('')里的内容国际化,这样可以让admin里的文字自动随着LANGUAGE    _CODE切换中英文
                                  (_('Permissions'), {'fields': ('is_active',     'is_staff', 'groups')}),
                                  (_('Important dates'), {'fields':     ('last_login', 'date_joined')}),
                                  )  #     这里('Permissions')中没有'is_superuser',此字段定义UserChangeForm表单中的具体    显示内容，并可以分类显示
                self.add_fieldsets = ((None, {'classes': ('wide',),
                                              'fields': ('username', 'name',     'password1', 'password2', 'email', 'is_active',
                                                         'is_staff', 'groups'),
                                              }),
                                      )      #此字段定义UserCreationForm表单中的具体显示内容
            else:  # super账户可以做任何事
                self.fieldsets = ((None, {'fields': ('username', 'password',)}),
                                  (_('Personal info'), {'fields': ('name',     'email')}),
                                  (_('Permissions'), {'fields': ('is_active',     'is_staff', 'is_superuser', 'groups')}),
                                  (_('Important dates'), {'fields':     ('last_login', 'date_joined')}),
                                  )
                self.add_fieldsets = ((None, {'classes': ('wide',),
                                              'fields': ('username', 'name',     'password1', 'password2', 'email', 'is_active',
                                                         'is_staff',     'is_superuser', 'groups'),
                                              }),
                                      )
            return super(CustomUserAdmin, self).changelist_view(request,     extra_context)
    
    
    admin.site.register(models.MyUser, CustomUserAdmin)  # 注册一下
    admin.site.register(models.User_ex, User_exAdmin)
    
效果展示

首页面

设定组权限管理

编辑用户页面
管理员：

非管理员：(没有设定超级用户的权限)

新增用户页面


通过以上，基本可以实现一个用户管理后台的需求了。

参考资料

https://simpleisbetterthancomplex.com/tutorial/2016/07/22/how-to-extend-django-user-model.html#proxy
http://www.cnblogs.com/daliangtou/p/5435385.html
https://docs.djangoproject.com/en/1.9/topics/auth/customizing/#django.contrib.auth.models.PermissionsMixin.has_perms
