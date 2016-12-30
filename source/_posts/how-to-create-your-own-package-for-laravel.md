---
title: How to create your own package for laravel
date: 2016-12-16 23:03:21
tags: [Laravel, Package]
---
# 如何创建自己的 Laravel 包
> 为了组(tao)件(bi)化(xue)开(xi)发，在最近做项目的时候，把一部分作为单独的组件来开发。组件化离不开跟包打交道，经过最近的折腾，记录一下自己开发包的过程。

首先，新建一个 Laravel 项目，并在项目根目录下新建 `packages` 文件夹。  
在 `packages` 文件夹中，新建以*你名字*为命名的文件夹，并在此文件夹内建立以*项目名命名*的文件夹。

```
.
├── app
├── artisan
├── bootstrap
├── composer.json
├── composer.lock
├── config
├── database
├── gulpfile.js
├── package.json
├── packages
    └── sliverwing
        └── packagedemo
├── phpunit.xml
├── public
├── readme.md
├── resources
├── routes
├── server.php
├── storage
├── tests
├── vendor
└── yarn.lock

```

如上例，进入 `packages\sliverwing\packagedemo` 运行 `composer init` 命令，进入 `Composer config generator`  
分别键入 `Package name` `Description` `Author` `Minimum Stability` `Package Type` `License` 和交互方式定义所依赖的库 `require` `require-dev` 后便可生成该包的 `composer.json`
接下来，编辑 `composer.json` 增加以下内容:  

```php
    "autoload": {
        "psr-4": {
            "Sliverwing\\PackageDemo\\": "src/"
        }
    },
```

这里定义了，该包发布后 `composer` 如何对项目进行自动加载。
在 Laravel 项目下的 `composer.json` 中，修改 `autoload` 配置，使得在编写包的过程中 Laravel 项目能自动加载包。

```json
    "autoload": {
        "classmap": [
            "database"
        ],
        "psr-4": {
            "App\\": "app/",
            "Sliverwing\\PackageDemo\\": "packages/sliverwing/packagedemo/src/"
        }
    },
```

接下来，在 `packagedemo` 中新建 `src` `config` `migrations` 文件夹。  

```
.
└── packagedemo
    ├── composer.json
    ├── config
    ├── migrations
    └── src
```

`src` 这里是主要存放代码的地方。可以参考 Laravel 中 `app` 的目录来组织 `src` 文件夹中的目录。
`config` 和 `migrations` 比较容易理解，存放包的配置文件和扩展，当然可以根据包内业务的需要加入 `views` 或者其他需要 `pulish` 的文件目录。

我们在 `config` 文件夹下新建 `packagedemo.php`，用来保存包的配置信息。 

回到 `laravel` 项目的根目录，使用 `php artisan make:provider PackageDemoServiceProvider` 创建包的 `ServiceProvider`。
把 `app\Providers\PackageDemoServiceProvider.php` 移动到 `packages\sliverwing\packagedemo\src` 下，并修改 `namespace`。 
> 参考 [https://laravel.com/docs/5.3/packages#service-providers](https://laravel.com/docs/5.3/packages#service-providers)

继续修改，使得在运行 `php artisan vendor:publish` 时，我们的配置文件可以复制到 Laravel 工程的 `config` 文件下。
在 `boot` 方法里，新增代码如下：

```php
    $this->publishes([
        __DIR__.'/../config/packagedemo.php' => config_path('packagedemo.php'),
    ]);
```

接下来，把 `PackageDemoServiceProvider` 加入到 `config\app.php` 中。  

```php
/*
    * Package Service Providers...
    */
Sliverwing\PackageDemo\PackageDemoServiceProvider::class,
        
```

在 Laravel 项目目录下，运行 `composer dump-autload`，然后运行 `php artisan vendor:publish` 可以发现，包下面的配置文件复制到 Laravel 项目中 `config` 文件夹下了。

```bash
[13:30:21] sliverwing:package_demo $ pa vendor:publish
Copied Directory [/vendor/laravel/framework/src/Illuminate/Notifications/resources/views] To [/resources/views/vendor/notifications]
Copied Directory [/vendor/laravel/framework/src/Illuminate/Pagination/resources/views] To [/resources/views/vendor/pagination]
Copied File [/packages/sliverwing/packagedemo/config/packagedemo.php] To [/config/packagedemo.php]
Publishing complete for tag []!
```

接下来，可以在 `src` 文件夹下创建 `Http\Controllers` 文件夹，并根据业务需要，新增包内的 Controller ，也可以对现有的组件进行扩展。
也可以参考其他包项目来进一步完善自己的项目组建。