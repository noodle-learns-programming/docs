# HTTP Controllers

- [Introduction](#introduction)
- [Basic Controllers](#basic-controllers)
- [Controller Middleware](#controller-middleware)
- [RESTful Resource Controllers](#restful-resource-controllers)
    - [Partial Resource Routes](#restful-partial-resource-routes)
    - [Naming Resource Routes](#restful-naming-resource-routes)
    - [Nested Resources](#restful-nested-resources)
    - [Supplementing Resource Controllers](#restful-supplementing-resource-controllers)
- [Implicit Controllers](#implicit-controllers)
- [Dependency Injection & Controllers](#dependency-injection-and-controllers)
- [Route Caching](#route-caching)

<a name="introduction"></a>
## Introduction

Instead of defining all of your request handling logic in a single `routes.php` file, you may wish to organize this behavior using Controller classes. Controllers can group related HTTP request handling logic into a class. Controllers are typically stored in the `app/Http/Controllers` directory.

[vi] Thay vì phải định nghĩa tất cả những việc quản lý logic cho những request của bạn trong một file `routes.php` duy nhất, có lẽ bạn sẽ muốn tổ chức hành vi này bằng cách sử dụng Controller classes. Những Controllers có thể gom lại logic xử lý trong mỗi request liên quan với nhau vào một class. Những Controllers thường được lưu trữ trong thư mục `app/Http/Controllers`.

<a name="basic-controllers"></a>
## Basic Controllers

Here is an example of a basic controller class. All Laravel controllers should extend the base controller class included with the default Laravel installation:

[vi] Sau đây là một ví dụ về một controller class đơn giản. Tất cả những Laravel controllers phải kế thừa base controller class đã được bổ sung mặc định trong quá trình cài đặt Laravel.

[note] Dịch chỗ này sẽ làm việc viết và nói tiếng Anh của mình trở nên khó sau này. Vì chuyển ngữ sai ngữ pháp.

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Show the profile for the given user.
         *
         * @param  int  $id
         * @return Response
         */
        public function showProfile($id)
        {
            return view('user.profile', ['user' => User::findOrFail($id)]);
        }
    }

We can route to the controller action like so:

[vi] Chúng ta có thể route đến controller action như vầy:

    Route::get('user/{id}', 'UserController@showProfile');

Now, when a request matches the specified route URI, the `showProfile` method on the `UserController` class will be executed. Of course, the route parameters will also be passed to the method.

[vi] Bây giờ, khi một request mà match với một route URI xác định, `showProfile` method trong `UserController` class sẽ được thực thi. Tất nhiên, những route parameters cũng sẽ được passed đến method.

#### Controllers & Namespaces

It is very important to note that we did not need to specify the full controller namespace when defining the controller route. We only defined the portion of the class name that comes after the `App\Http\Controllers` namespace "root". By default, the `RouteServiceProvider` will load the `routes.php` file within a route group containing the root controller namespace.

[vi] Điều quan trọng phải note lại là chúng ta không cần phải xác định cụ thể full controller namespace khi định nghĩa controller route. Chúng ta chỉ cần định nghĩa portion của cái class name mà nó đi theo sau `App\Http\Controlers` namespace "root". Bởi mặc định, `RouteServiceProvider` sẽ load `routes.php` file bên trong một route group chứa root controller namespace.

If you choose to nest or organize your controllers using PHP namespaces deeper into the `App\Http\Controllers` directory, simply use the specific class name relative to the `App\Http\Controllers` root namespace. So, if your full controller class is `App\Http\Controllers\Photos\AdminController`, you would register a route like so:

[vi] Nếu bạn muốn chọn để nest hoặc organize controllers của mình sử dụng sâu hơn PHP namespaces đến thư mục `App\Http\Controllers`, sử dụng (đơn giản?) class (xác định?) chỉ đến `App\Http\Controllers` root namespace. Vậy, nếu bạn muốn đầy đủ controller class là `App\Http\Controllers\Photo\AdminController`, bạn sẽ muốn đăng kí một route dạng như vầy:

    Route::get('foo', 'Photos\AdminController@method');

#### Naming Controller Routes

Like Closure routes, you may specify names on controller routes:

[vi] Như những Closure routes, bạn có lẽ muốn xác định tên của những controller routes.

    Route::get('foo', ['uses' => 'FooController@method', 'as' => 'name']);

Once you have assigned a name to the controller route, you can easily generate URLs to the action. To generate a URL to a controller action, use the `action` helper method. Again, we only need to specify the part of the controller class name that comes after the base `App\Http\Controllers` namespace:

[vi] Một khi bạn vừa gán một tên đến controller route, bạn sẽ có thể đơn giản(?) generate(?) những URLs đến action. Để tạo (generate) một URL đến một controller action, sử dụng `action` helper method. Một lần nữa, chúng ta chỉ cần xác định cái phần (the part) của controller class name mà nó đi theo (come) `App\Http\Controllers` namespaces:

    $url = action('FooController@method');

You may also use the `route` helper to generate a URL to a named controller route:

[vi] Bạn cũng có thể sử dụng `route` helper để generate một URL để xác định(chỉ ra/đặt tên) (name) controller router:

    $url = route('name');

<a name="controller-middleware"></a>
## Controller Middleware

[Middleware](/docs/{{version}}/middleware) may be assigned to the controller's routes like so:

    Route::get('profile', [
        'middleware' => 'auth',
        'uses' => 'UserController@showProfile'
    ]);

However, it is more convenient to specify middleware within your controller's constructor. Using the `middleware` method from your controller's constructor, you may easily assign middleware to the controller. You may even restrict the middleware to only certain methods on the controller class:

[vi] Tuy nhiên, có điều thuận lợi để xác định middleware bên trọng controller's constructor của bạn. Sử dụng `middleware` method từ controller's constructor của bạn, bạn có thể assign dễ dàng middleware đến controller. Bạn có thể giới hạn (restrict) middleware để chỉ những methods xác định (certain: cụ thể/chắc chắn) ở controller classs:

    class UserController extends Controller
    {
        /**
         * Instantiate a new UserController instance.
         *
         * @return void
         */
        public function __construct()
        {
            $this->middleware('auth');

            $this->middleware('log', ['only' => ['fooAction', 'barAction']]);

            $this->middleware('subscribed', ['except' => ['fooAction', 'barAction']]);
        }
    }
    
[note] Chỗ này không hiểu gì hết trơn.

<a name="restful-resource-controllers"></a>
## RESTful Resource Controllers

Resource controllers make it painless to build RESTful controllers around resources. For example, you may wish to create a controller that handles HTTP requests regarding "photos" stored by your application. Using the `make:controller` Artisan command, we can quickly create such a controller:

    php artisan make:controller PhotoController

The Artisan command will generate a controller file at `app/Http/Controllers/PhotoController.php`. The controller will contain a method for each of the available resource operations.

Next, you may register a resourceful route to the controller:

    Route::resource('photo', 'PhotoController');

This single route declaration creates multiple routes to handle a variety of RESTful actions on the photo resource. Likewise, the generated controller will already have methods stubbed for each of these actions, including notes informing you which URIs and verbs they handle.

#### Actions Handled By Resource Controller

Verb      | Path                  | Action       | Route Name
----------|-----------------------|--------------|---------------------
GET       | `/photo`              | index        | photo.index
GET       | `/photo/create`       | create       | photo.create
POST      | `/photo`              | store        | photo.store
GET       | `/photo/{photo}`      | show         | photo.show
GET       | `/photo/{photo}/edit` | edit         | photo.edit
PUT/PATCH | `/photo/{photo}`      | update       | photo.update
DELETE    | `/photo/{photo}`      | destroy      | photo.destroy

<a name="restful-partial-resource-routes"></a>
#### Partial Resource Routes

When declaring a resource route, you may specify a subset of actions to handle on the route:

    Route::resource('photo', 'PhotoController',
                    ['only' => ['index', 'show']]);

    Route::resource('photo', 'PhotoController',
                    ['except' => ['create', 'store', 'update', 'destroy']]);

<a name="restful-naming-resource-routes"></a>
#### Naming Resource Routes

By default, all resource controller actions have a route name; however, you can override these names by passing a `names` array with your options:

    Route::resource('photo', 'PhotoController',
                    ['names' => ['create' => 'photo.build']]);

<a name="restful-nested-resources"></a>
#### Nested Resources

Sometimes you may need to define routes to a "nested" resource. For example, a photo resource may have multiple "comments" that may be attached to the photo. To "nest" resource controllers, use "dot" notation in your route declaration:

    Route::resource('photos.comments', 'PhotoCommentController');

This route will register a "nested" resource that may be accessed with URLs like the following: `photos/{photos}/comments/{comments}`.

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;

    class PhotoCommentController extends Controller
    {
        /**
         * Show the specified photo comment.
         *
         * @param  int  $photoId
         * @param  int  $commentId
         * @return Response
         */
        public function show($photoId, $commentId)
        {
            //
        }
    }

<a name="restful-supplementing-resource-controllers"></a>
#### Supplementing Resource Controllers

If it becomes necessary to add additional routes to a resource controller beyond the default resource routes, you should define those routes before your call to `Route::resource`; otherwise, the routes defined by the `resource` method may unintentionally take precedence over your supplemental routes:

    Route::get('photos/popular', 'PhotoController@method');

    Route::resource('photos', 'PhotoController');

<a name="implicit-controllers"></a>
## Implicit Controllers

Laravel allows you to easily define a single route to handle every action in a controller class. First, define the route using the `Route::controller` method. The `controller` method accepts two arguments. The first is the base URI the controller handles, while the second is the class name of the controller:

    Route::controller('users', 'UserController');

 Next, just add methods to your controller. The method names should begin with the HTTP verb they respond to followed by the title case version of the URI:

    <?php

    namespace App\Http\Controllers;

    class UserController extends Controller
    {
        /**
         * Responds to requests to GET /users
         */
        public function getIndex()
        {
            //
        }

        /**
         * Responds to requests to GET /users/show/1
         */
        public function getShow($id)
        {
            //
        }

        /**
         * Responds to requests to GET /users/admin-profile
         */
        public function getAdminProfile()
        {
            //
        }

        /**
         * Responds to requests to POST /users/profile
         */
        public function postProfile()
        {
            //
        }
    }

As you can see in the example above, `index` methods will respond to the root URI handled by the controller, which, in this case, is `users`.

#### Assigning Route Names

If you would like to [name](/docs/{{version}}/routing#named-routes) some of the routes on the controller, you may pass an array of names as the third argument to the `controller` method:

    Route::controller('users', 'UserController', [
        'getShow' => 'user.show',
    ]);

<a name="dependency-injection-and-controllers"></a>
## Dependency Injection & Controllers

#### Constructor Injection

The Laravel [service container](/docs/{{version}}/container) is used to resolve all Laravel controllers. As a result, you are able to type-hint any dependencies your controller may need in its constructor. The dependencies will automatically be resolved and injected into the controller instance:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Routing\Controller;
    use App\Repositories\UserRepository;

    class UserController extends Controller
    {
        /**
         * The user repository instance.
         */
        protected $users;

        /**
         * Create a new controller instance.
         *
         * @param  UserRepository  $users
         * @return void
         */
        public function __construct(UserRepository $users)
        {
            $this->users = $users;
        }
    }

Of course, you may also type-hint any [Laravel contract](/docs/{{version}}/contracts). If the container can resolve it, you can type-hint it.

#### Method Injection

In addition to constructor injection, you may also type-hint dependencies on your controller's action methods. For example, let's type-hint the `Illuminate\Http\Request` instance on one of our methods:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use Illuminate\Routing\Controller;

    class UserController extends Controller
    {
        /**
         * Store a new user.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            $name = $request->input('name');

            //
        }
    }

If your controller method is also expecting input from a route parameter, simply list your route arguments after your other dependencies. For example, if your route is defined like so:

    Route::put('user/{id}', 'UserController@update');

You may still type-hint the `Illuminate\Http\Request` and access your route parameter `id` by defining your controller method like the following:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use Illuminate\Routing\Controller;

    class UserController extends Controller
    {
        /**
         * Update the specified user.
         *
         * @param  Request  $request
         * @param  int  $id
         * @return Response
         */
        public function update(Request $request, $id)
        {
            //
        }
    }

<a name="route-caching"></a>
## Route Caching

If your application is exclusively using controller based routes, you may take advantage of Laravel's route cache. Using the route cache will drastically decrease the amount of time it takes to register all of your application's routes. In some cases, your route registration may even be up to 100x faster! To generate a route cache, just execute the `route:cache` Artisan command:

    php artisan route:cache

That's all there is to it! Your cached routes file will now be used instead of your `app/Http/routes.php` file. Remember, if you add any new routes you will need to generate a fresh route cache. Because of this, you may wish to only run the `route:cache` command during your project's deployment.

To remove the cached routes file without generating a new cache, use the `route:clear` command:

    php artisan route:clear
