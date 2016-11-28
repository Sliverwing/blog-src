---
title: Better Way To Handle Modal Status In Laravel
date: 2016-11-27 23:27:48
tags: Laravel
---
# 更优雅的方式，来处理 Laravel 中模型的状态
> 在开发过程中，我们时常用特定的数字来描述当前记录的状态，当状态输出到模板时，除了`@if ... @elseif ... @endif`，是否有更好的办法呢？

## Step 1  
通过 `composer new StatusDemo` 获取一份新的 Laravel 应用。
## Step 2
使用 `php artisan make:model Student -m` 生成示例数据库模型和迁移。
修改 `create_students_table` :

    public function up()
    {
        Schema::create('students', function (Blueprint $table) {
            $table->increments('id');
            $table->string('name');
            $table->integer('age');
            $table->integer('status')->default(0);
            $table->timestamps();
        });
    }
## Step 3
修改 `App\Student.php` : 

    public function getStatus(){
        switch($this->status){
            case 0:
                return "正常";
                break;
            case 1:
                return "已被锁定";
                break;
        }
    }

## Step 4
使用 `php artisan make:seeder StudentSeeder` 生成 `StudentSeeder`。
修改 `StudentSeeder`：

    for($i=0; $i<10; $i++){
        DB::table('students')->insert([
            'name' => str_random(10),
            'age' => rand(20, 30),
            'status' => rand(0, 1),
        ]);
    } 

## Step 5
接下来，我们用扩展模板的方法完成状态的显示。  
首先使用 `php artisan make:provider BladeExtendingServiceProvider` 生成扩展模板的 provider 。
并在 `config\app.php` 增加 `App\Providers\BladeExtendingServiceProvider::class,`。
修改 `BladeExtendingServiceProvider`:

    public function boot()
    {
        Blade::directive('getstatus', function ($item) {
            return '<?php echo '. $item .'->getStatus(); ?>';
        });
    }

## Step 5
为了方便我使用 `php artisan make:auth` 认证组件中的 Controller 和 View。
在 `http\Controller\HomeController` 中修改：

    public function index()
    {
        $items = Student::all();
        return view('home', ['items' => $items]);
    }

在 `views/home.blade.php` 中添加：

    <table class="table">
        <thead>
            <tr> 
            <th>Name</th> 
            <th>Age</th> 
            <th>Status</th> 
            </tr>
        </thead>
        <tbody>
            @foreach($items as $item)
                <tr>
                    <th>{{ $item->name }}</th>
                    <th>{{ $item->age }}</th>
                    <th> @getstatus($item) </th>
                </tr>
            @endforeach
        </tbody>
    </table>

效果如图所示：
![ScreenShot](better-way-to-handle-modal-status-in-laravel-screen-shot-0.png)

## 几点补充
* 本质上来讲，扩展 Blade 到生成模板缓存的过程，是替换字符串的过程。
* 可以在扩展的方法中使用 `echo e()` 的方法，转义 html。
* 如果需要更新扩展中的逻辑，需要进行 `view:clear` 重新缓存模板。

> 请署名并附原链转载，谢谢。