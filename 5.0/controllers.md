# کنترلرهای HTTP

- [آشنایی](#introduction)
- [کنترلرهای پایه](#basic-controllers)
- [Controller Middleware](#controller-middleware)
- [Implicit Controllers](#implicit-controllers)
- [RESTful Resource Controllers](#restful-resource-controllers)
- [Dependency Injection & Controllers](#dependency-injection-and-controllers)
- [Route Caching](#route-caching)

<a name="introduction"></a>
## آشنایی

به جای تعریف تمام منطقی که درخواستها را پاسخ میدهند دریک فایل `routes.php`، میتوانید این رفتار را با استفاده از کلاسهای کنترلر انجام دهید. کنترلرها میتوانند تمامی منطق مدیریت درخواستهای HTTP رادر یک کلاس مرتب کنند. کنترلرها میتوانند منطق مدیریت درخواستهای HTTP مرتبط را در یک کلاس دسته بندی کنند. کنترلها معمولا در دایرکتوری `app/Http/Controllers` ذخیره میشوند.

<a name="basic-controllers"></a>
## کنترلرهای پایه

درادامه مثالی از یک کلاس کنترلر ابتدایی ارائه شده است:

	<?php namespace App\Http\Controllers;

	use App\Http\Controllers\Controller;

	class UserController extends Controller {

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


برای روت به اکشن متد یک کنترلر به روش زیر عمل میکنیم:

	Route::get('user/{id}', 'UserController@showProfile');

> **نکته:** تمامی کنترلرها باید کلاس کنترلر پایه را اکستند کنند.

#### کنترلرها و فضای نام 

اشاره به این نکته مهم است که نیازی به ارائه کل فضای نام کنترلرنیست، تنها بخشی از نام کلاس که پس از فضای نام "ریشه" `App\Http\Controllers` آورده می شود. به صورت پیش فرض، `RouteServiceProvider` فایل `routes.php` را در گروه route که فضای نام کنترلر ریشه در آن قرار دارد، بارگذاری میکند.

اگر بخواهید کنترلرهایتان را با استفاده از namespace PHP در زیرپوشه های مجزا و دسته بندی شده در دایرکتوری `App\Http\Controllers` قرارگیرد، به سادگی میتوانید از نام آن کلاس نسبت به namespace ریشه `App\Http\Controllers`  استفاده کنید. بنابراین، اگر آدرس کامل کلاس کنترلر شما `App\Http\Controllers\Photos\AdminController` باشد، یک route را باید به شکل زیر ایجاد کنید:

	Route::get('foo', 'Photos\AdminController@method');

#### نامگذاری routeهای کنترلرها

مانند Closure route، میتوانید برای routeهای کنترلرها هم نام تعیین نمایید:

	Route::get('foo', ['uses' => 'FooController@method', 'as' => 'name']);

#### آدرس اکشن متدهای کنترلر

برای ساختن URL به اکشن متد کنترلر، از تابع کمکی `action` استفاده کنید:

	$url = action('App\Http\Controllers\FooController@method');

اگر میخواهید URLی به یک کنترلر بسازید و این کار را با استفاده از مسیر نام نسبی انجام دهید، namespace کنترلر ریشه را به URL generator معرفی کنید:

	URL::setRootControllerNamespace('App\Http\Controllers');

	$url = action('FooController@method');

نام اکشن متد کنترلر فراخوانی شده را با استفاده از متد `currentRouteAction` میتوانید به دست آورید:

	$action = Route::currentRouteAction();

<a name="controller-middleware"></a>
## Controller Middleware

[Middleware](/docs/%7B%7Bversion%7D%7D/middleware) مربوط به کنترلر را به روش زیر میتوان مشخص کرد:

	Route::get('profile', [
		'middleware' => 'auth',
		'uses' => 'UserController@showProfile'
	]);

همچنین میتوانید middleware را درون سازنده کنترلر هم مشخص نمود:

	class UserController extends Controller {

		/**
		 * Instantiate a new UserController instance.
		 */
		public function __construct()
		{
			$this->middleware('auth');

			$this->middleware('log', ['only' => ['fooAction', 'barAction']]);

			$this->middleware('subscribed', ['except' => ['fooAction', 'barAction']]);
		}

	}

<a name="implicit-controllers"></a>
## Implicit Controllers

لاروال امکان تعریف یک route برای مدیریت تمامی اکشن متدها در یک کنترلر را فراهم میکند. ابتدا، route را با استفاده از متد `Route::controller` تعریف کنید:

	Route::controller('users', 'UserController');

اکشن متد `controller` دو آرگومان میپذیرد. آرگومان اول URI پایه به هندل کنترلر است، و دومین آرگومان نام کلاس کنترلر است. پس از آن تنها اکشن متدها را با پیشوند متد HTTP که به آن پاسخ میدهند تعریف کنید:

	class UserController extends Controller {

		public function getIndex()
		{
			//
		}

		public function postProfile()
		{
			//
		}

		public function anyLogin()
		{
			//
		}

	}

متدهای `index` به URI ریشه که توسط کنترلر مدیریت میشوند پاسخ میگویند، که دراینجا `users` است.

اگر اکشن متد کنترلر چندکلمه ای باشد، میتوانید با استفاده از سینتکس "دش -" در URI به آن دسترسی داشته باشید. برای مثال، اکشن متد زیر کنترلردر `UserController` به URI `users/admin-profile` میدهد:

	public function getAdminProfile() {}

#### انتصاب نام route

اگر میخواهید برخی route ها به کنترلرتان را نامگذاری کنید، میتوانید یک پارامتر سوم به متد `controller` بیافزایید:

	Route::controller('users', 'UserController', [
		'anyLogin' => 'user.login',
	]);

<a name="restful-resource-controllers"></a>
## کنترلرهای منبع RESTful

کنترلرهای Resource ساخت کنترلرهای RESTful برای منابع را آسان میکنند. برای مثال، شاید بخواهید کنترلری برای مدیریت درخواستهای HTTP مربوط به "تصویر"های ذخیره شده درون نرم افزار بسازید. با استفاده از فرمان آرتیزان `make:controller`، به سرعت میتوانیم این کنترلر را بسازیم:

	php artisan make:controller PhotoController

در قدم بعد، یک resourceful route را برای کنترلر ثبت میکنیم:

	Route::resource('photo', 'PhotoController');

تعریف این route باعث تعریف چندین route برای مدیریت چندین اکشن متد RESTful بر روی منبع تصویر می شود. به همین ترتیب کنترلر ایجاد شده اکشن متدهایی خواهد داشت که برای هر یک از این عملیاتها در نظر گرفته شده اند. این اکشن متدها به همراه نکته هایی که اطلاعاتی در مورد URI و متد درخواست HTTP مدیریت شده توسط آنها ارائه میدهند، ایجاد میشوند.

#### عملیاتهای مدیریت شده توسط کنترلرهای منبع (resource controller)

Verb      | Path                  | Action       | Route Name
----------|-----------------------|--------------|---------------------
GET       | /photo                | index        | photo.index
GET       | /photo/create         | create       | photo.create
POST      | /photo                | store        | photo.store
GET       | /photo/{photo}        | show         | photo.show
GET       | /photo/{photo}/edit   | edit         | photo.edit
PUT/PATCH | /photo/{photo}        | update       | photo.update
DELETE    | /photo/{photo}        | destroy      | photo.destroy

#### شخصی سازی routهای منبع

همچنین میتوانید route را برای انجام زیرمجموعه ای عملیاتها تعریف نمایید:

	Route::resource('photo', 'PhotoController',
					['only' => ['index', 'show']]);

	Route::resource('photo', 'PhotoController',
					['except' => ['create', 'store', 'update', 'destroy']]);

به صورت پیش فرض، تمامی اکشن متدهای کنترلر دارای یک نام route هستند؛ هرچند، میتوانید با فرستادن یک آرایه `names` به همراه گزینه ها این مقادیر را تغییر دهید:

	Route::resource('photo', 'PhotoController',
					['names' => ['create' => 'photo.build']]);

#### مدیریت کنترلرهای منبع تودرتو - Handling Nested Resource Controllers

برای ایجاد کنترلرهای منبع "تودرتو"، از قالب "." در تعریف route استفاده کنید:

	Route::resource('photos.comments', 'PhotoCommentController');

`photos/{photos}/comments/{comments}`.
این route یک منبع "تودرتو" را که میتواند از طریق URLهایی مانند `photos/{photos}/comments/{comments}` دسترسی شود تعریف میکند.

	class PhotoCommentController extends Controller {

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

#### افزودن routeهای بیشتر به کنترلرهای منبع - Adding Additional Routes to Resource Controllers

در صورت نیاز به افزودن routeهای اضافی به کنترلرهای منبع علاوه بر routeهای منبع پیش فرض، باید آنها را پیش از فراخوانی `Route::resource` فراخوانی نمایید:

	Route::get('photos/popular', 'PhotoController@method');

	Route::resource('photos', 'PhotoController');

<a name="dependency-injection-and-controllers"></a>
## Dependency Injection & Controllers

#### Constructor Injection

[service container](/docs/%7B%7Bversion%7D%7D/container) لاراول برای تشخیص تمامی کنترلرهای لاراول استفاده میشود. بنابراین، میتوانید هر کنترلر مورد نیاز یک کنترلر را اعلان نوع کنید:


	<?php namespace App\Http\Controllers;

	use Illuminate\Routing\Controller;
	use App\Repositories\UserRepository;

	class UserController extends Controller {

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

البته میتوانید هر [کانترکت لاراول](/docs/%7B%7Bversion%7D%7D/contracts) را اعالان نوع نمایید. اگر کانتینتر میتواند توع آن را تشخیص دهد، میتوانید اعالانش کنید.

#### Method Injection

علاوه بر constructor injection، همچنین میتوانید وابستگی ها را بر روی متدهای کنترلرتان هم اعلان نوع کنید. برای مثال، میخواهیم نمونه کلاس `Request` را بر روی یکی از متدهایمان اعلان نوع کنیم:

	<?php namespace App\Http\Controllers;

	use Illuminate\Http\Request;
	use Illuminate\Routing\Controller;

	class UserController extends Controller {

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

اگر اکشن متد کنترلر شما از route هم پارامتر میگیرد، به سادگی آرگومانهای route را پس از وابستگی های دیگر لیست نمایید:

	<?php namespace App\Http\Controllers;

	use Illuminate\Http\Request;
	use Illuminate\Routing\Controller;

	class UserController extends Controller {

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

> **نکته:** Method injection کاملا با [model binding](/docs/%7B%7Bversion%7D%7D/routing#route-model-binding) سازگار است. کانتینر هوشمندانه تشخیص میدهد کدام آرگومانها به مدلی متصل شده اند یا کدام باید inject شود.

<a name="route-caching"></a>
## Route Caching

اگر نرم افزارتان فقط از routeهای کنترلر استفاده میکند، میتوانید از کش route لاراول استفاده کنید. استفاده از کش route به شدت میزان زمان مورد نیاز برای ثبت تمام routeهای نرم افزارتان را کاهش میدهد. در برخی موارد، ثبت route میتواند تا 100 برابر سریعتر شود! برای ایجاد یک کش route، کافیست فرمان آرتیزان `route:cache` را اجرا نمایید:

	php artisan route:cache

این تمام کاریست که باید انجام دهید! از حالا فایل routeهای کش شده به جای فایل `app/Http/routes.php` استفاده می شود. به یاد داشته باشید، اگرroute جدیدی اضاف کنید باید یک کش route جدید بسازید. با توجه به این موضوع بهتر است فرمان `route:cache` را تنها در زمان دیپلوی پروژه اجرا نمایید.

برای حذف فایل routeهای کش شده بدون ایجاد یک فایل جدید کافیست از فرمان `route:clear`استفاده نمایید:

	php artisan route:clear
