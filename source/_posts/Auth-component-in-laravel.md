---
title: Auth-component-in-laravel
date: 2017-03-04 22:48:10
tags: laravel
---
# 简介 Laravel 内置的 Auth 组件  
> Laravel 提供了开箱即用的用户注册、登陆组建，下面通过几个场景简要介绍 Auth 组件和我们的业务结合。 

## 注册

### 将用户名 (username) 替换为其他字段.

在 `App\Http\Controllers\Auth\RegisterController` 中修改方法 `create` 如下

```php

protected function create(array $data)
{
    return User::create([
        'name' => $data['name'],
        'phone' => $data['phone'],
        'password' => bcrypt($data['password']),
    ]);
}

```

本例中讲默认的 `email` 替换为 `phone`, 此外还需要修改 controller 中的 validator 方法:

```php

protected function validator(array $data)
{
    return Validator::make($data, [
        'name' => 'required|max:255',
        'phone' => 'required|min:255|unique:users',
        'password' => 'required|min:6|confirmed',
    ]);
}

```

### 注册后分配用户组

修改 'App\Http\Controllers\Auth\RegisterController' 中的 `create` 方法：

```php

protected function create(array $data)
{
    $user-> User::create([
        'name' => $data['name'],
        'phone' => $data['phone'],
        'password' => bcrypt($data['password']),
    ]);

    $user->attachRole($roleId);
}

```


## 登陆

### 将用户名 (username) 替换替换成其他字段  
#### 替换为手机号 (phone)  

在 `App\Http\Controllers\Auth\LoginController` 中添加如下方法:  
```php

public function username()
{
    return 'phone'
}

```

#### 动态判断用户名字段

修改: `App\Http\Controllers\Auth\LoginController` :  
```php 

//...

protected $redirectTo = '/home';
protected $username = 'username'; // 新增该属性

//...

protected function credentials(Request $request)
{
    if ( strlen ($request->input('username')) > 11 )
    {
        $this->username = 'id_number';
        return ['id_number' => $request->input('username'), 'password' => $request->input('password')];
    }
    else
    {
        $this->username = 'phone';
        return ['phone' => $request->input('username'), 'password' => $request->input('password')];
    }
}

public function username()
{
    return $this->username;
}

```
> 本例中实现了对用户 id_number 用户手机号的动态判断。

## 根据用户身份重定向到特定 URL

依然在 `App\Http\Controllers\Auth\LoginController` 添加如下方法:

```php

protected function authenticated(Request $request, $user)
{
    if ($user->can('admin.login'))
    {
        $this->redirectTo = '/admin';
    }
}

```

> * 文章提到的用户名 (username) 是沿用 Auth 组件中的描述: `Illuminate\Foundation\Auth\AuthenticatesUsers@credentials`

```php
protected function credentials(Request $request)
{
        return $request->only($this->username(), 'password');
}
```

> * 请在修改业务逻辑之余不要忘记修改数据库结构 (migration).

