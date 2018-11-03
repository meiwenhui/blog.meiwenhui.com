---
layout: post
title: "Laravel 目录结构"
description: "How to Download or Use This Theme"
keywords: "laravel"
---


```
laravel-startup
├── CHANGELOG.md
├── app
│   ├── Console
│   │   ├── Commands    // 存放命令Artisan 命令
│   │   └── Kernel.php
│   ├── Events
│   │   └── ArticleCRUDEvent.php
│   ├── Exceptions
│   │   └── Handler.php
│   ├── Helpers.php
│   ├── Http
│   │   ├── Controllers
│   │   ├── Kernel.php
│   │   ├── Middleware
│   │   └── Validators
│   ├── Jobs
│   │   └── SendReminderEmail.php
│   ├── Listeners
│   │   └── ArticleCRUDEventListener.php
│   ├── Models
│   │   ├── ArticleModel.php
│   ├── Providers
│   │   ├── AppServiceProvider.php
│   │   ├── AuthServiceProvider.php
│   │   ├── BroadcastServiceProvider.php
│   │   ├── EventServiceProvider.php
│   │   ├── RouteServiceProvider.php
│   └── User.php
├── artisan
├── bootstrap
│   ├── app.php
│   ├── autoload.php
│   └── cache
│       └── services.php
├── composer.json
├── config
│   ├── app.php
│   ├── auth.php
│   ├── broadcasting.php
│   ├── cache.php
│   ├── database.php
│   ├── filesystems.php
│   ├── mail.php
│   ├── queue.php
│   ├── services.php
│   ├── session.php
│   └── view.php
├── database
│   ├── factories
│   │   └── ModelFactory.php
│   ├── migrations
│   │   ├── 2014_10_12_000000_create_users_table.php
│   └── seeds
│       └── DatabaseSeeder.php
├── package.json
├── phpunit.xml
├── public
│   ├── css
│   │   ├── animate.css
│   │   ├── app.css
│   │   ├── bootstrap.min.css
│   │   └── inspinia.css
│   ├── favicon.ico
│   ├── img
│   │   └── profile_small.jpg
│   ├── index.php
│   ├── js
│   │   ├── app.js
│   │   ├── bootstrap.js
│   │   ├── inspinia.js
│   │   ├── jquery-2.1.1.js
│   │   ├── jquery.validate.v1.13.1.js
│   │   └── plugins
│   ├── robots.txt
│   ├── template
│   │   ├── 404.html
│   │   ├── 500.html
│   │   ....
│   └── web.config
├── readme.md
├── resources
│   ├── assets
│   │   ├── js
│   │   └── sass
│   ├── lang
│   │   └── en
│   └── views
│       ├── auth
│       └── welcome.blade.php
├── routes
│   ├── api.php
│   ├── channels.php
│   ├── console.php
│   └── web.php
├── server.php
├── storage
│   ├── app
│   │   └── public
│   ├── framework
│   │   ├── cache
│   │   ├── sessions
│   │   ├── testing
│   │   └── views
│   └── logs
│       └── laravel.log
├── tests
│   ├── Feature
│   │   ├── ExampleTest.php
│   │   └── PaperTest.php
│   ├── TestCase.php
│   └── Unit
│       ├── EventTest.php
│       └── ExampleTest.php
├── vendor
│   ├── autoload.php
│   ├── composer
│   │   ├── ClassLoader.php
│   │   ├── LICENSE
│   │   ├── autoload_classmap.php
│   │   ├── autoload_files.php
│   │   ├── autoload_namespaces.php
│   │   ├── autoload_psr4.php
│   │   ├── autoload_real.php
│   │   ├── autoload_static.php
│   │   └── installed.json
│   |   .......
└── webpack.mix.js
```

#### Cheers!
