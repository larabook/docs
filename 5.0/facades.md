# فاساد Facades

- [مقدمه](#introduction)
- [تعریف](#explanation)
- [استفاده عملی](#practical-usage)
- [ایجاد فاساد](#creating-facades)
- [ماک کردن فاسادها](#mocking-facades)
- [مرجع فاساد های لاراول](#facade-class-reference)

<a name="introduction"></a>
## مقدمه
فاساد ها  برای دسترسی بصورت `static` به [سرویس های موجود](/docs/{{version}}/container)   درون اپلیکیشن مورد استفاده قرار میگیرند . فاساد های زیادی درون  لاراول وجود دارد که شما بدون دانستن ساختار آن میتوانید از آنها استفاده نمایید . فاساد ، یک روش دسترسی به  سرویس های موجود در اپلیکیشن می باشد و  از طریق آن ها میتوانید متد های موجود در یک سرویس را بصورت استاتیک صدا بزنید .

ممکن است بخواهید برای پکیج های خود فاساد بسازید .  برای این منظورشما را با مفهوم کلاس های فاساد بیشتر آشنا خواهیم کرد .

> **نکته :** قبل از مطالعه بر روی کلاس های فاساد, پیشنهاد میکنیم  با [service container](/docs/{{version}}/container) ها آشنا شوید.

<a name="explanation"></a>
## تعریف
در لاراول ، یک فاساد به منظور دسترسی سریع به یک سرویس درون اپلیکیشن استفاده میشود . منطقی که این عملیات را انجام میدهد درون کلاس `Facade` پایه قرار دارد و هر فاسادی که شما میسازید باید از کلاس `Facade` پایه  ، ارث بری  کند .

کلاس های فاساد ایجاد شده  توسط شما تنها باید یک متد با نام `getFacadeAccessor` در خود داشته باشند ، این متد  نام سرویسی که این فاساد باید به آن دسترسی داشته باشد را تعیین میکند .کلاس `Facade` امکان استفاده از متد جادویی `__callStatic()` را میدهد که به موجب آن میتوانید متد های درون سرویس را با این ترفند به صورت static اجرا نمایید . بنابراین ، 
  فاسادهای لاراول  یک راه آسان برای دسترسی به سرویس ها می باشند .

 برای مثال ، زمانی که فاساد `Cache::get`  را  فراخوانی میکنید لاراول بدنبال آن ،  متد get را از سرویس cache صدا میزند . 

<a name="practical-usage"></a>
## استفاده عملی
در مثال زیر ، یک متد static از فاساد cache صدا زده شده است و شما در نگا ه اول  تصور خواهید کرد متد `get` درون کلاس `cache`  وجود دارد.

	$value = Cache::get('key');

اما , اگر به کلاس `Illuminate\Support\Facades\Cache`  یک نگاه بندازید ، خواهید دید که هیچ متد استاتیکی با نام  `get` وجود ندارد ! :

	class Cache extends Facade {

		/**
		 * Get the registered name of the component.
		 *
		 * @return string
		 */
		protected static function getFacadeAccessor() { return 'cache'; }

	}

کلاس `cache` از کلاس `Facade` ارث بری میکند و یک متد با نام `getFacadeAccessor()` درخود دارد ، بیاد داشته باشید که این متد نام سرویسی که این فاساد میخواهد به آن دسترسی داشته باشد را برمیگرداند.

بنابراین ، فراخوانی `Cache::get` مفهومی همانند کد زیر خواهد داشت :

	$value = $app->make('cache')->get('key');

#### استفاده از یک فاساد
اگر قصد دارید از یک فاساد درون کنترلر های خود استفاده نمایید باید حتما آن را به namespace کنترولر خود اضافه کنید  . تمامی فاساد ها در فضای `global`  قابل دسترس هستند :

	<?php namespace App\Http\Controllers;

	use Cache;

	class PhotosController extends Controller {

		/**
		 * Get all of the application photos.
		 *
		 * @return Response
		 */
		public function index()
		{
			$photos = Cache::get('photos');

			//
		}

	}

<a name="creating-facades"></a>
## ایجاد فاساد
ساخت فاساد برای پروژه یا پکیج بسیار ساده است . تنها 3 چیز را نیاز دارید :

- اضافه کردن سرویس .
- ایحاد یک کلاس فاساد.
- معرفی فاساد به فریموورک .

به مثال زیر توجه نمایید ، ما یک کلاس با نام `PaymentGateway\Payment`  تعریف کرده ایم.

	namespace PaymentGateway;

	class Payment {

		public function process()
		{
			//
		}

	}

باید بتوانیم به کلاس مورد نظر به صورت یک سرویس دسترسی داشته باشیم . پس آن را به سرویس هایتان اضافه کنید :

	App::bind('payment', function()
	{
		return new \PaymentGateway\Payment;
	});

همچنین روش بهتری نیز وجود دارد که ابتدا یک [service provider](/docs/{{version}}/container#service-providers) با نام   `PaymentServiceProvider` اینجاد میکنیم و دستورات بالا را درون متد `register` آن اضافه میکنم و سپس آن را به آرایه سرویس ها در فایل `config/app.php`  اضافه میکنیم .

سپس ، فاساد آن را به روش زیر ایجاد میکنیم :

	use Illuminate\Support\Facades\Facade;

	class Payment extends Facade {

		protected static function getFacadeAccessor() { return 'payment'; }

	}
در پایان ، فاساد را به آرایه `aliases` در فایل  `config/app.php` اضافه میکنیم . حالا میتوانید متد `process` را از سرویس `Payment` به روش زیر فراخوانی کنید :

	Payment::process();

<a name="mocking-facades"></a>
## ماک کردن فاسادها
تست واحد (Unit testing) یکی از مهمترین قسمت های فاساد ها  میباشد که نشان دهنده چگونگی عملکرد آنها می باشد . درواقع تست مهمترین دلیلی است که اهمیت  وجود یک فاساد را مشخص میکند . برای اطلاعات بیشتر به مستندات  [mocking facades](/docs/{{version}}/testing#mocking-facades) رجوع کنید .

<a name="facade-class-reference"></a>
## مرجع فاساد های لاراول
در اینجا لیستی از تمامی فاساد ها به همراه سرویس های مرتبط با آنها نمایش داده شده است . اگر میخواهید بدانید که هر فاساد از کدام سرویس خدمت میگیرد میتوانید از لیست زیر کمک بگیرید . همچنین مشخصه  هر سرویس ([service container binding](/docs/{{version}}/container) )  نیزمشخص شده است .

فاساد |  نام سرویس |  مشخصه سرویس
------------- | ------------- | -------------
App  |  [Illuminate\Foundation\Application](http://laravel.com/api/{{version}}/Illuminate/Foundation/Application.html)  | `app`
Artisan  |  [Illuminate\Console\Application](http://laravel.com/api/{{version}}/Illuminate/Console/Application.html)  |  `artisan`
Auth  |  [Illuminate\Auth\AuthManager](http://laravel.com/api/{{version}}/Illuminate/Auth/AuthManager.html)  |  `auth`
Auth (Instance)  |  [Illuminate\Auth\Guard](http://laravel.com/api/{{version}}/Illuminate/Auth/Guard.html)  |
Blade  |  [Illuminate\View\Compilers\BladeCompiler](http://laravel.com/api/{{version}}/Illuminate/View/Compilers/BladeCompiler.html)  |  `blade.compiler`
Bus  |  [Illuminate\Contracts\Bus\Dispatcher](http://laravel.com/api/{{version}}/Illuminate/Contracts/Bus/Dispatcher.html)  |
Cache  |  [Illuminate\Cache\CacheManager](http://laravel.com/api/{{version}}/Illuminate/Cache/Repository.html)  |  `cache`
Config  |  [Illuminate\Config\Repository](http://laravel.com/api/{{version}}/Illuminate/Config/Repository.html)  |  `config`
Cookie  |  [Illuminate\Cookie\CookieJar](http://laravel.com/api/{{version}}/Illuminate/Cookie/CookieJar.html)  |  `cookie`
Crypt  |  [Illuminate\Encryption\Encrypter](http://laravel.com/api/{{version}}/Illuminate/Encryption/Encrypter.html)  |  `encrypter`
DB  |  [Illuminate\Database\DatabaseManager](http://laravel.com/api/{{version}}/Illuminate/Database/DatabaseManager.html)  |  `db`
DB (Instance)  |  [Illuminate\Database\Connection](http://laravel.com/api/{{version}}/Illuminate/Database/Connection.html)  |
Event  |  [Illuminate\Events\Dispatcher](http://laravel.com/api/{{version}}/Illuminate/Events/Dispatcher.html)  |  `events`
File  |  [Illuminate\Filesystem\Filesystem](http://laravel.com/api/{{version}}/Illuminate/Filesystem/Filesystem.html)  |  `files`
Hash  |  [Illuminate\Contracts\Hashing\Hasher](http://laravel.com/api/{{version}}/Illuminate/Contracts/Hashing/Hasher.html)  |  `hash`
Input  |  [Illuminate\Http\Request](http://laravel.com/api/{{version}}/Illuminate/Http/Request.html)  |  `request`
Lang  |  [Illuminate\Translation\Translator](http://laravel.com/api/{{version}}/Illuminate/Translation/Translator.html)  |  `translator`
Log  |  [Illuminate\Log\Writer](http://laravel.com/api/{{version}}/Illuminate/Log/Writer.html)  |  `log`
Mail  |  [Illuminate\Mail\Mailer](http://laravel.com/api/{{version}}/Illuminate/Mail/Mailer.html)  |  `mailer`
Password  |  [Illuminate\Auth\Passwords\PasswordBroker](http://laravel.com/api/{{version}}/Illuminate/Auth/Passwords/PasswordBroker.html)  |  `auth.password`
Queue  |  [Illuminate\Queue\QueueManager](http://laravel.com/api/{{version}}/Illuminate/Queue/QueueManager.html)  |  `queue`
Queue (Instance) |  [Illuminate\Queue\QueueInterface](http://laravel.com/api/{{version}}/Illuminate/Queue/QueueInterface.html)  |
Queue (Base Class) |  [Illuminate\Queue\Queue](http://laravel.com/api/{{version}}/Illuminate/Queue/Queue.html)  |
Redirect  |  [Illuminate\Routing\Redirector](http://laravel.com/api/{{version}}/Illuminate/Routing/Redirector.html)  |  `redirect`
Redis  |  [Illuminate\Redis\Database](http://laravel.com/api/{{version}}/Illuminate/Redis/Database.html)  |  `redis`
Request  |  [Illuminate\Http\Request](http://laravel.com/api/{{version}}/Illuminate/Http/Request.html)  |  `request`
Response  |  [Illuminate\Contracts\Routing\ResponseFactory](http://laravel.com/api/{{version}}/Illuminate/Contracts/Routing/ResponseFactory.html)  |
Route  |  [Illuminate\Routing\Router](http://laravel.com/api/{{version}}/Illuminate/Routing/Router.html)  |  `router`
Schema  |  [Illuminate\Database\Schema\Blueprint](http://laravel.com/api/{{version}}/Illuminate/Database/Schema/Blueprint.html)  |
Session  |  [Illuminate\Session\SessionManager](http://laravel.com/api/{{version}}/Illuminate/Session/SessionManager.html)  |  `session`
Session (Instance)  |  [Illuminate\Session\Store](http://laravel.com/api/{{version}}/Illuminate/Session/Store.html)  |
Storage  |  [Illuminate\Contracts\Filesystem\Factory](http://laravel.com/api/{{version}}/Illuminate/Contracts/Filesystem/Factory.html)  |  `filesystem`
URL  |  [Illuminate\Routing\UrlGenerator](http://laravel.com/api/{{version}}/Illuminate/Routing/UrlGenerator.html)  |  `url`
Validator  |  [Illuminate\Validation\Factory](http://laravel.com/api/{{version}}/Illuminate/Validation/Factory.html)  |  `validator`
Validator (Instance)  |  [Illuminate\Validation\Validator](http://laravel.com/api/{{version}}/Illuminate/Validation/Validator.html) |
View  |  [Illuminate\View\Factory](http://laravel.com/api/{{version}}/Illuminate/View/Factory.html)  |  `view`
View (Instance)  |  [Illuminate\View\View](http://laravel.com/api/{{version}}/Illuminate/View/View.html)  |
