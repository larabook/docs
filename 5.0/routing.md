# HTTP Routing

- [آشنایی](#basic-routing)
- [جلوگیری از حمله CSRF](#csrf-protection)
- [متدهای ساختگی ارسال فرم](#method-spoofing)
- [پارامترها در Routing](#route-parameters)
- [نام گذاری Route ها](#named-routes)
- [گروه بندی Route ها](#route-groups)
- [Bind کردن مدل](#route-model-binding)
- [ارسال خطای 404](#throwing-404-errors)

<a name="basic-routing"></a>
## آشنایی

ما معمولا Route های پروژه را در درون فایل  `app/Http/routes.php` ذخیره میکنیم که بوسیله   `App\Providers\RouteServiceProvider`   سرویسدهی میشوند ، ساده ترین روش معرفی  Route ها  استفاده از URL و یک تابع (`Closure`) می باشد :

#### تعریف ساده یک Route برای درخواست GET

	Route::get('/', function()
	{
		return 'Hello World';
	});

#### درخواست های Post,Put,Delete

	Route::post('foo/bar', function()
	{
		return 'Hello World';
	});

	Route::put('foo/bar', function()
	{
		//
	});

	Route::delete('foo/bar', function()
	{
		//
	});

#### معرفی Route برای انواع مشخصی از درخواستها
	Route::match(['get', 'post'], '/', function()
	{
		return 'Hello World';
	});

#### معرفی Route برای انواع درخواست ها 

	Route::any('foo', function()
	{
		return 'Hello World';
	});

گاهی اوقات ممکن است بخواهید آدرس URL یک Route را در سایت نمایش دهید برای این منظور از تابع کمکی  `url` استفاده میکنیم  :

	$url = url('foo');

<a name="csrf-protection"></a>
##  جلوگیری از حمله CSRF

لاراول بطور خودکار  وبسایت شما را در مقابل حملات  [CSRF](http://en.wikipedia.org/wiki/Cross-site_request_forgery) محافظت میکند .  CSRF درخواست های مخربی هستند که به موجب آن عملیات غیرمجازی با هویت کاربر سایت یا مدیر سیستم اجرا میشوند.

لاراول بصورت خودکار برای هر درخواست یک توکن CSRF  ایجاد میکند . این توکت درحقیقت تایید میکند که کاربری که درخواست را ارسال کرده ، همان کاربری است که تعیین هویت شده است.  لاراول شما را مجبور میسازد تا این توکن را همراه با درخواست هایی Put ، patch ، delete ارسال کنید چراکه اجرای اینگونه درخواستها بصورت غیرمحافظت شده ممکن است عملیات مخربی بر روی اطلاعات شما  به بار آورد .

#### ساخت یک توکن CSRF در فرم درخواست

	<input type="hidden" name="_token" value="<?php echo csrf_token(); ?>">

و البته روش ساده  [blade](/docs/{{version}}/templates) :

	<input type="hidden" name="_token" value="{{ csrf_token() }}">

شما مجبور نیستید عملیات تصدیق توکن CSRF برای درخواست های POST  ، PUT ، ِDELETE را بصورت دستی انجام دهید !  `VerifyCsrfToken` [HTTP middleware](/docs/{{version}}/middleware)  بطور خودکار برای  تمامی درخواست های از این نوع ، توکن ارسال شده آنها را با توکن ذخیره شده در SESSION مقایسه و تایید میکند . 
 
#### X-CSRF-TOKEN
Middleware علاوه بر بررسی توکن ارسالی توسط متد POST به بررسی هدر `X-CSRF-TOKEN` می پردازد . برای مثال شما میتوانید توکن خود را درون یک متا تگ html ذخیره نمایید و آن را به هدر تمامی درخواست های AJAX از طریق jQuery اضافه نمایید . 

	<meta name="csrf-token" content="{{ csrf_token() }}" />

	$.ajaxSetup({
			headers: {
				'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
			}
		});

حال توکن CSRF به همراه  تمامی درخواست های AJAX  به صورت اتوماتیک ارسال می شود. 

	$.ajax({
	   url: "/foo/bar",
	})

#### X-XSRF-TOKEN
لاراول همچنین توکن CSRF را درون یک کوکی `XSRF-TOKEN`  ذخیره می نماید . شما میتوانید از مقدار این کوکی برای مقداردهی Header  درخواست  `X-XSRF-TOKEN`  استفاده نمایید . برخی از فریموورک های جاوااسکریپت مانند Angular این عملیات را به صورت خودکار انجام میدهند .

> توجه : تفاوت میان  `X-CSRF-TOKEN` و `X-XSRF-TOKEN` این می باشد که X-CSRF  مقداری از نوع رشته ای را درخود ذخیره میکنید ولی X-XSRF مقدار  رمزنگاری شده در کوکی ذخیره میکند زیرا کوکی ها در لاراول همیشه رمزنگاری میشوند . اگر از تابع `csrf_token()` استفاده کنید لاراول مقدار توکن  X-CSRF را به شما برمی گرداند .

<a name="method-spoofing"></a>
## متدهای ساختگی ارسال فرم (Method Spoofing)
فرم های HTML متد های PUT  و PATH و DELETE را پشتیبانی نمیکند . به همین جهت برای Route کردن درخواست های از اینگونه ،  شما بایستی در فرم HTML یک ورودی با نام `_method` از نوع Hidden ارسال نمایید  ، تا روتر شما پس از شناسایی نوع متد درخواست شده ، آن را به route مربوطه انتقال دهد . 

مقداری که درون فیلد `_method`  قرار میگیرد به عنوان نوع متد درخواست شناسایی میشود . برای مثال :

	<form action="/foo/bar" method="POST">
		<input type="hidden" name="_method" value="PUT">
		<input type="hidden" name="_token" value="<?php echo csrf_token(); ?>">
	</form>

<a name="route-parameters"></a>
## پارامترها در Routing
شما میتوانید قسمت های از یک URL ارسالی را به عنوان پارامتر دریافت نمایید .

#### نمونه ساده یک Route  به همراه پارامتر

	Route::get('user/{id}', function($id)
	{
		return 'User '.$id;
	});

> **توجه :** شما نمیتوانید از کاراکتر `-`  در تعریف پارامتر ها استفاده کنید ، از یک کاراکتر  (`_`) به جای آن استفاده نمایید .

#### پارامتر اختیاری

	Route::get('user/{name?}', function($name = null)
	{
		return $name;
	});

#### پارامتر اختیاری با مقدار پیشفرض

	Route::get('user/{name?}', function($name = 'John')
	{
		return $name;
	});

#### اعتبار سنجی پارامتر ها بوسیله Regular Expression 

	Route::get('user/{name}', function($name)
	{
		//
	})
	->where('name', '[A-Za-z]+');

	Route::get('user/{id}', function($id)
	{
		//
	})
	->where('id', '[0-9]+');

#### اعتبار سنجی بوسیله آرایه

	Route::get('user/{id}/{name}', function($id, $name)
	{
		//
	})
	->where(['id' => '[0-9]+', 'name' => '[a-z]+']);

#### معرفی الگوهای اعتبارسنجی به صورت سراسری (Global Patterns)
اگر قصد دارید یک پارامتر را در تمامی Route هایی که از آن استفاده شده اعتبارسنجی کنید از متد   `pattern` در سرویس `RouteServiceProvider` استفاده نمایید . شما میتوانید Pattern های خود را در متد boot از این سرویس تعریف نمایید .

	$router->pattern('id', '[0-9]+');

pattern معرفی شده شما درتمامی Route هایی که از آن  پارامتر استفاده میکنند اعمال خواهد شد .

	Route::get('user/{id}', function($id)
	{
		// Only called if {id} is numeric.
	});

#### دریافت پارامتر از Route

ممکن است بخواهید به پارامترهای  ارسالی از یک Route دسترسی داشته باشید برای این منظور از متد `input` استفاده کنید  :

	if ($route->input('id') == 1)
	{
		//
	}

برای دسترسی به پارامتر های Route جاری ، یک Type Hint از نوع `Illuminate\Http\Request` یا نام مستعار `Request` معرفی کنید . Instance ساخته شده از این کلاس به Route جاری اشاره دارد و شما میتوانید پارامتر های ارسالی به آن را از طریق آن دریافت کنید :

	use Illuminate\Http\Request;

	Route::get('user/{id}', function(Request $request, $id)
	{
		if ($request->route('id'))
		{
			//
		}
	});

<a name="named-routes"></a>
## نام گذاری Route ها
نام گذاری route ها به شما این امکان را میدهد که URL های آن ها رو تولید کنید یا از آنها برای Redirect کردن در برنامه استفاده کنید . برای نامگذاری Route  از کلمه کلیدی `as`  در آرایه استفاده کنید .

	Route::get('user/profile', ['as' => 'profile', function()
	{
		//
	}]);

همچنین برای ارجاع Route به یک Controller و Action از این طریق عمل کنید :

	Route::get('user/profile', [
		'as' => 'profile', 'uses' => 'UserController@showProfile'
	]);

استفاده از نام Route برای تولید URL یا Redirect کردن :

	$url = route('profile');

	$redirect = redirect()->route('profile');

متد `currentRouteName` نام Route ی که درخواست HTTP جاری را یدک میکشد را برمیگرداند :

	$name = Route::currentRouteName();

<a name="route-groups"></a>
## گروه بندی Route ها
گاهی اوقات Route های شما خصوصیات های مشابهی از قبیل middleware ،URL segments ، namespaces   با یکدیگر دارند .  بجای اینکه تعریف هریک از این خصوصیات را برای هرکدام تکرار کنید میتوانید آنها را درون یک گروه تعریف کنید و سپس تمامی خصوصیات تعریف شده بر روی کلیه Route های عضو گروه اعمال میشود . 

خصوصیت ها  به صورت آرایه در پارامتر اول متد  `Route::group` تعریف می شوند .

<a name="route-group-middleware"></a>
### Middleware

Middleware بر روی تمامی Route هایی که درون گروه قرار دارند به صورت زیر اعمال می شود . Middleware ها به ترتیبی که درون آرایه تعریف شده اند بر روی Route های عضو گروه اجرا خواهد شد .

	Route::group(['middleware' => ['foo', 'bar']], function()
	{
		Route::get('/', function()
		{
			// Has Foo And Bar Middleware
		});

		Route::get('user/profile', function()
		{
			// Has Foo And Bar Middleware
		});

	});

<a name="route-group-namespace"></a>
### Namespaces
برای تعریف `namespace` به صورت زیر عمل می کنیم . namespace مشخص می کند که Route مورد نظر به کدام Controller ارجاع شود .

	Route::group(['namespace' => 'Admin'], function()
	{
		// Controllers Within The "App\Http\Controllers\Admin" Namespace

		Route::group(['namespace' => 'User'], function()
		{
			// Controllers Within The "App\Http\Controllers\Admin\User" Namespace
		});
	});

> **توجه :** به صورت پیشفرض  `RouteServiceProvider` فایل  `routes.php`  را درون یک namespace  لود میکند که این امکان را میدهد Controller ها را بدون معرفی namespace  آن (مثلا `App\Http\Controllers`  ) درون route های عضو یک گروه تعریف کنید .
 

<a name="sub-domain-routing"></a>
### زیر دامنه در route
همچنین روتر لاراول  استفاده از sub-domain  ها را نیز پشتیبانی میکند و دسترسی به پارامتر های wildcard را ممکن میسازد .

#### معرفی یک زیر دامنه برای گروه از Route ها

	Route::group(['domain' => '{account}.myapp.com'], function()
	{

		Route::get('user/{id}', function($account, $id)
		{
			//
		});

	});
	
<a name="route-prefixing"></a>
### Route Prefixing
اعمال پسوند URL برای Route های عضو یک گروه : 

	Route::group(['prefix' => 'admin'], function()
	{
		Route::get('users', function()
		{
			// Matches The "/admin/users" URL
		});
	});
همچنین میتوانید پسوندها را همانند پارامترها معرفی کنید و از آنها استفاده نمایید :

	Route::group(['prefix' => 'accounts/{account_id}'], function()
	{
		Route::get('detail', function($account_id)
		{
			//
		});
	});

میتوانید مقادیر پارامتر های استفاده شده درRoute های عضو یک گروه را اعتبار سنجی و محدود کنید  :

	Route::group([
		'prefix' => 'accounts/{account_id}',
		'where' => ['account_id' => '[0-9]+'],
	], function() {

		// روت ها را در اینجا تعریف کنید .
	});

<a name="route-model-binding"></a>
## Bind کردن مدل
مدل Binding یک روش آسان برای ایجاد instance  از مدل معرفی شده با توجه با نام پارامتر ارسالی از Route می باشد برای مثال میخواهید هرزمان که پارامتری حاوی ID کاربر ارسال شد سیستم بطور خودکار یک instance از مدل کاربری که با این ID یکسان می باشد برای شما ایجاد کند .
  برای معرفی آن ، دستورات زیر را  در متد `boot` از کلاس `RouteServiceProvider` قرار میدهیم .
  
#### Bind کردن یک پارامتر در یک مدل (Model Binding)

	public function boot(Router $router)
	{
		parent::boot($router);

		$router->model('user', 'App\User');
	}

سپس , یک Route به همراه پارامتر `{user}` تعریف نمایید :

	Route::get('profile/{user}', function(App\User $user)
	{
		//
	});
حال اگر درخواست `profile/1` ارسال شود  یک  instance از مدل `User` که ID آن برابر 1 باشد ایجاد میشود . 

> **توجه :** اگر رکورد مدل درخواستی در بانک اطلاعاتی موجود نباشد یک خطای 404 به بیرون thrown خواهد شد .

و اگر قصد دارید یک behavior از نوع "not found"  بصورت دستی ایجاد کنید :

	Route::model('user', 'User', function()
	{
		throw new NotFoundHttpException;
	});

روش زیر درصورتی که بخواهید محدودیت های بیشتر ایجاد کنید یا عمل Bind کردن مدل را به نحوی دیگر (برای مثال ایجاد شرط برای فیلد نام) تغییر دهید . 

	Route::bind('user', function($value)
	{
		return User::where('name', $value)->first();
	});

<a name="throwing-404-errors"></a>
## ارسال خطای 404
بطورکلی 2 روش برای ارسال خطای 404 وجود دارد : 

1- استفاده از تابع کمکی  `abort`:


	abort(404);

تابع کمکی  `abort` یک Exception  از نوع  `Symfony\Component\HttpKernel\Exception\HttpException` با وضعیت مشخص شده ایجاد میکند .

2- ارسال Exception بوسیله  instance ساخته شده از کلاس `Symfony\Component\HttpKernel\Exception\NotFoundHttpException`.

جهت دریافت اطلاعات بیشتر درمورد exception  ها و خطاها به بخش  [errors](/docs/{{version}}/errors#http-exceptions) رجوع فرمایید .
