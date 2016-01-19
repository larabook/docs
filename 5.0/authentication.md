# تصدیق هویت

- [مقدمه](#introduction)
- [تصدیق هویت کاربر](#authenticating-users)
- [دریافت مشخصات کاربر تصدیق شده](#retrieving-the-authenticated-user)
- [حافظت از Route ها](#protecting-routes)
- [HTTP Basic Authentication](#http-basic-authentication)
- [ریست و بازگردانی رمز عبور](#password-reminders-and-reset)
- [تصدیق هویت از طریق رسانه های اجتماعی (Social Authentication) ](#social-authentication)

<a name="introduction"></a>
## مقدمه
لاراول عملیات احراز هویت کاربر (authenticating) را به خوبی اجرا میکند . در واقع ، تقریبا همه چیز بصورت حاضر و آماده برای شما تنظیم شده است . فایل تنظیمات در مسیر `config/auth.php` قرار گرفته  ، که تمامی تنظیمات سیستم  Authentication لاراول را به همراه توضیحات هر یک شامل می شود.
لاراول بصورت پیشفرض از مدل `App\User` که در مسیر فولدر `app`  قرار دارد را برای انجام عملیات احراز هویت ،  استفاده میکند .
به یاد داشته باشید : در زمان ساخت جدول مرتبط با این مدل حتما طول فیلد پسوورد را 60 کاراکتر (حداقل) قرار دهید . همچنین مطمئن شود جدول `users` (یا معادل آن) یک فیلد `nullable` از نوع رشته ای با نام `remember_token`  بطول حداقل 100 کاراکتر داشته باشد . این ستون برای ذخیره سازی مقدار "remember me" استفاده میشود. همچنین برای ساخت این ستون میتوانید از دستور `$table->rememberToken();` در Migration های خود استفاده نمایید . البته لاراول 5.0  ، Migration های مربوط به این ستون ها را به صورت پیشفرض برای شما دراختیار گذاشته است !
اگر اپلیکیشن شما از Eloquent استفاد ه نمیکند ، شما میتوانید از درایور `database`  که از `query builder` استفاده می کند برای عملیات تصدیق هویت کاربر استفاده نمایید.

<a name="authenticating-users"></a>
## تصدیق هویت کاربر
لاراول برای تصدیق هویت کاربر دو نمونه کنترلر برای شما به صورت پیشفرض آماده کرده است . کنترلر  `AuthController` وظیفه  ورود کاربر و عضویت کاربر جدید  و  `PasswordController` که وظیفه بازگردانی رمز فراموش شده کاربران  را بر عهده دارد.

هریک از این کنترلر ها از یک trait برای فراهم کردن متد های مورد نیاز خود استفاده میکنند . برای بسیاری از پروژه ها، شما اصلا نیاز نخواهید داشت این کنترلر ها را تغییر دهید . فایل های View که این کنترلر ها آنها را فراخوانی میکنند در فولدر `resources/views/auth`  قرار دارند و میتوانید درصورت لزوم آنها را تغییر دهید .

### ثبت نام کنند کاربر
برای تغییر فیلد های مهمی که برای ثبت نام کاربر لازم است میتوانید فایل `App\Services\Registrar` را تغییر دهید . این کلاس وظیفه Validate کردن و ساختن کاربر جدید را بر عهده دارد ..
متد `validator` از کلاس `Registrar` شامل نقش هایی (Rules) است که برای Validate کردن عضویت کاربرجدید استفاده می شود  درحالایکه متد `create` وظیفه ثبت اطلاعات کاربر جدید در بانک اطلاعاتی را برعهده دارد. شما میتوانید آزادانه هریک از این متد ها را تغییر دهید . `Registrar` از طریق متد هایی که توسط  تریت `AuthenticatesAndRegistersUsers` در کللاس `AuthController` تزریق شده اند ، فراخوانی می شود .

#### تصدیق کاربر به روش دستی (Manual)
اگر قصد ندارید از `AuthController`  پیاده سازی شده استفاده کنید . شما می بایست عملیات تصدیق هویت کاربر را خودتان انجام دهید . بیاید متد   `attempt` را چک کنیم :

	<?php namespace App\Http\Controllers;

	use Auth;
	use Illuminate\Routing\Controller;

	class AuthController extends Controller {

		/**
		 * Handle an authentication attempt.
		 *
		 * @return Response
		 */
		public function authenticate()
		{
			if (Auth::attempt(['email' => $email, 'password' => $password]))
			{
				return redirect()->intended('dashboard');
			}
		}

	}
متد `attempt` یک آرایه ترکیبی از key و value  در آرگومان اول خود دریافت میکند . مقدار `password`  [رمزنگاری (Hash)](/docs/{{version}}/hashing) خواهد شد . مقادیر دیگر در آرایه  برای پیدا کردن کاربر از درون بانک اطلاعاتی استفاده میشود . بنابراین در مثال بالا ، کاربر از طریق مقدار ایمیل جستجو میشود و درصورتی که کاربر پیدا شد ، رمز Hash ذخیره شده در دیتابیس با مقدار Hash شده  از ورودی آرایه مقایسه می شود .اگر هر دو با یکدیگر یکسان باشند ، یک session جدید برای کاربر مورد نظر ایجاد میشود .

متد `attempt` درصورتی که عملیات تصدیق هویت کاربر با موفقیت انجام شود مقدار `true`  و درغیراینصورت مقدار `false` را برمیگرداند .

> **نکته :** در این مثال `email` الزامی نیست و تنها به عنوان مثال استفاد شده است . شما میتوانید از هر ستون از جدول به عنوان username کاربر استفاده نمایید . 

تابع Redirect به روش `intended` کاربر را به همان صفحه ای که قبلا درخواست کرده ولی به دلیل عدم تعیین هویت متوقف شده بوده است ، ریدایرکت می کند. میتوانید یک پارامتر URL به این میتد به عنوان پیشفرض (درصورتی که  مقصد intended موجود نباشد) بدهید.

#### تصدیق هویت کاربر به همراه شرط

همچنین میتوانید شرط های بیشتری را برای تصدیق هویت ایجاد کنید :

	if (Auth::attempt(['email' => $email, 'password' => $password, 'active' => 1]))
	{
		// The user is active, not suspended, and exists.
	}

#### بررسی که آیا کاربر احراز هویت شده است

برای چک کردن اینکه آیا کاربر به وبسایت شما لاگین کرده ,از متد `check` استفاده نمایید :

	if (Auth::check())
	{
		// کاربر احراز هویت شده و لاگین میباشد ...
	}

#### تصدیق هویت کاربر و "به خاطر سپردن" آنها
اگر میخواهد قابلیت بخاطر سپاری ( remember me) را در سایت خودایجاد نمایید ، یک مقدار boolean  به عنوان پارامتر دوم متد `attempt` ارسال نمایید . که باعث میشود کاربر بصورت نامحدود و تازمانی که خوش Logout نکرده ، در سیستم لاگین باقی بماند . درنظر داشته باشید برای این منظور جدول `users`  حتما باید دارای فیلد `remember_token` باشد که برای ذخیره سازی مقدار توکن "remember me" استفاده میشود.

	if (Auth::attempt(['email' => $email, 'password' => $password], $remember))
	{
		// The user is being remembered...
	}
اگر میخوایید بدانید که کاربر شما از کدام طریق  (لاگین کرده یا از حالت بخاطر سپاری رمز) تصدیق هویت شده است ، از متد `viaRemember` استفاده نمایید :

	if (Auth::viaRemember())
	{
		// 
	}

#### تصدیق هویت به وسیله  ID کاربر
برای لاگین کردن یک کاربر به سایت از طریق ID  کاربر ، از متد `loginUsingId` استفاده نمایید :

	Auth::loginUsingId(1);

#### اعتبار سنجی صحت وجود کاربر بدون لاگین کردن آن
متد  `validate`  این امکان را میدهد بدون اینکه کاربر را به سیستم لاگین کند، صحت وجود کاربر را در سیستم بررسی نماید :

	if (Auth::validate($credentials))
	{
		//
	}

#### لاگین کردن کاربر فقط برای اجرای یک درخواست
اگر قصد دارید کاربر را تنها برای اجرای یک درخواست در سیستم لاگین کنید (هیچگونه Cookie و session ذخیره نمی شود) :

	if (Auth::once($credentials))
	{
		//
	}

#### لاگین کاربر به روش دستی (Manually)
اگر نیازمند این میباشید که نمونه ایجاد شده از یک مدل کاربر موجود درسیستم را  لاگین کنید از متد `login` استفاده نمایید :

	Auth::login($user);
که همانند متد  `attempt` عمل میکند .

#### پایان دادن session کاربر (خروج کاربر)

	Auth::logout();
البته ، اگر شما از از Controller پیشفرض لاراول برای عملیات تصدیق هویت کاربر استفاده می کنید ، Action مربوط به ورود و خروج کاربر به صورت پیشفرض درون Controller ایجاد شده است .

#### رویداد های مربوط به تصدیق هویت (Authentication Events)
زمانی که متد `attempt` صدا زده می شود ، یک [رویداد](/docs/{{version}}/events)  `auth.attempt`  نیز درکنار آن اجرا می شود . و اگر عملیات تصدیق هویت کاربر با موفقیت انجام شود و کاربر به سیستم لاگین شود ،رویداد  `auth.login`  اجرا میشود .

<a name="retrieving-the-authenticated-user"></a>
## دریافت مشخصات کاربر تصدیق شده
زمانی که کاربر به سیستم لاگین شد ، برای بدست آوردن کلاس نمونه کاربر لاگین شده چندین راه وجود دارد .
روش اول، شما میتوانید به اطلاعات کاربر لاگین شده از طریق فاساد `Auth` عمل کنید :

	<?php namespace App\Http\Controllers;
	
	use Auth;
	use Illuminate\Routing\Controller;

	class ProfileController extends Controller {

		/**
		 * Update the user's profile.
		 *
		 * @return Response
		 */
		public function updateProfile()
		{
			if (Auth::user())
			{
				// Auth::user() returns an instance of the authenticated user...
			}
		}

	}

روش دوم، استفاده  از  `Illuminate\Http\Request` :

	<?php namespace App\Http\Controllers;

	use Illuminate\Http\Request;
	use Illuminate\Routing\Controller;

	class ProfileController extends Controller {

		/**
		 * Update the user's profile.
		 *
		 * @return Response
		 */
		public function updateProfile(Request $request)
		{
			if ($request->user())
			{
				// $request->user() returns an instance of the authenticated user...
			}
		}

	}

روش سوم , استفاده از type-hint برای  دسترسی به  `Illuminate\Contracts\Auth\Authenticatable`  . این type-hint به constructor کنترلر یا به متد آن یا constructor هر کلاس دیگری  که توسط  [service container](/docs/{{version}}/container) قابل دسترس باشد ، اضافه می شود :

	<?php namespace App\Http\Controllers;

	use Illuminate\Routing\Controller;
	use Illuminate\Contracts\Auth\Authenticatable;

	class ProfileController extends Controller {

		/**
		 * Update the user's profile.
		 *
		 * @return Response
		 */
		public function updateProfile(Authenticatable $user)
		{
			// $user is an instance of the authenticated user...
		}

	}

<a name="protecting-routes"></a>
## حافظت از Route ها
استفاده از [Route middleware](/docs/{{version}}/middleware) این امکان را میدهد که بتوانیم دسترسی کاربر را به برخی از Route ها محدود کنیم ، لاراول به طور پیشفرض برای شما یک Middleware  با نام  `Auth`  فراهم کرده  که در فایل `app\Http\Middleware\Authenticate.php تعریف شده است . تنها کاری که باید بکنید این است که آن را به Route مورد نظرتان نسبت دهید :

	// With A Route Closure...

	Route::get('profile', ['middleware' => 'auth', function()
	{
		// Only authenticated users may enter...
	}]);

	// With A Controller...

	Route::get('profile', ['middleware' => 'auth', 'uses' => 'ProfileController@show']);

<a name="http-basic-authentication"></a>
## تصدیق هویت کاربر بوسیله HTTP
تصدیق هویت بوسیله HTTP یک روش آسان و سریع می باشد که در آن شما  نیازی به ایجاد صفحه لاگین نخواهید داشت. برای شروع میتوانید میدلورر `auth.basic` را به Route خود نسبت دهید :

#### محدود کردن دسترسی به یک Route از طریق تصدیق هویت HTTP  

	Route::get('profile', ['middleware' => 'auth.basic', function()
	{
		// Only authenticated users may enter...
	}]);
به صورت پیشفرض میدلور `basic`   فیلد `email` به عنوان نام کاربری درنظر میگیرد..

#### ایجاد یک فیلتر پایه Stateless HTTP

همچنین بدون استفاده از کوکی در session میتوانید از احرازهویت پایه ای HTTP استفاده کنید. این کار برای احراز هویت در APIها بسیار مفید است. برای اینکار, یک[ Middleware](/docs/{{version}}/middleware) تعریف کنید که متد `onceBasic` را فراخوانی کند :

	public function handle($request, Closure $next)
	{
		return Auth::onceBasic() ?: $next($request);
	}
اگر درحال استفاده از PHP FastCGI هستید ، تصدیق هویت HTTP ممکن است به خوبی برای شما کار نکند ! برای این منظور کد های زیر را به فایل `.htaccess` اضافه کنید :

	RewriteCond %{HTTP:Authorization} ^(.+)$
	RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]

<a name="password-reminders-and-reset"></a>
## ریست و بازگردانی رمز عبور

### Model & Table
فریموورک ها برای ریست کردن رمز عبور از روش های مختص به خود استفاده میکنند و نیاز نیست دیگر برای هر پروژه  آن را مجددا بازسازی کنید . لاراول از یک روش آسان برای  ریست کردن رمز عبور استفاده میکند .

برای شروع ، بررسی کنید که حتما مدل `User` ،  اینترفیس `Illuminate\Contracts\Auth\CanResetPassword`  را implement کرده باشد . البته مدل  پیشفرض `User` که فریموورک  از آن استفاده میکند بطور پیشفرض اینترفیس های لازم را implement میکند و همچنین برای بکارگیری متدهای مورد نیاز از Trait تریت`Illuminate\Auth\Passwords\CanResetPassword` استفاده میکند .

#### تولید Migration های جدول بازگردانی رمز
حال باید یک جدول در بانک اطلاعاتی برای ذخیره توکن های بازآوری رمز ، ایجاد نمایید . البته Migration های مربوط به این جدول بصورت پیشفرض در مسیر   `database/migrations` قرار داده شده اند. بنابراین تنها دستور زیر را در خط  فرمان اجرا نمایید تا عملیات Migration اجرا شوند :

	php artisan migrate

### کنترلر بازآوری رمزعبور
همچنین لاراول کنترلر `Auth\PasswordController` که شامل عملیات منطقی برای ریست کردن رمز است را بصورت پیشفرض مورد استفاده قرار می دهد.  لاراول View های مرتبط را نیز درخود دارد. View ها در مسیر `resources/views/auth` قرار گرفته اند. شما آزادانه میتوانید view ها را مطابق با نوع طراحی خود تغییر دهید .

کاربر شما یک ایمیل حاوی لینک ریست سازی رمز که به اکشن getReset از کنترلر `PasswordController`  لینک شده است . این اکشن فرم ریست سازی رمز عبور که امکان ریست کردن رمز را به کاربر میدهد را به نمایش در می آورد . بعد از اینکه رمز عبور ریست شد، کاربر به صورت اتوماتیک به سایت لاگین شده و به صفحه پیشفرض `/home` ریدایکت میشود که میتوانید مسیر ریدارکت را از طریق خصوصیت  `redirectTo ` درکنترلر مشخص کنید :

	protected $redirectTo = '/dashboard';

> **نکته :** بصورت پیشفرض , توکن ها بعد از گذشت یک ساعت منقضی میشوند . که شما میتوانید مدت زمان آن را از طریق گزینه `reminder.expire` در فایل تنظیمات `config/auth.php` تغییر دهید.

<a name="social-authentication"></a>
## تصدیق هویت از طریق رسانه های اجتماعی (Social Authentication)
علاوه بر روش معمول تصدیق هویت تسوط ورودی کاربر لاراول یک روش آسان نیز برای تصدیق هویت کاربر از طریق رسانه های اجتماعی وجود دارد  . برای این منظور لاراول پکیج [Laravel Socialite](https://github.com/laravel/socialite) بوسیله سرویس OAuth را فراهم کرده است . **این پکیج در حال حاضر Facebook, Twitter, Google, GitHub  و Bitbucket را پشتیبانی میکند .**

برای شروع کار با Socialite , پکیج زیر را در فایل  `composer.json` اضافه کنید :

	"laravel/socialite": "~2.0"

سپس , مسیر `Laravel\Socialite\SocialiteServiceProvider` را به فایل تنظیمات`config/app.php` اضافه نمایید. برای رجیستر [facade](/docs/{{version}}/facades) نیز خط زیر را در آرایه Aliases های خود در همان فایل قرار دهید :

	'Socialize' => 'Laravel\Socialite\Facades\Socialite',

ممکن است بخواهید یک اعتبارنامه جدید برای سرویس OAuth ایاد کنید ، این اعتبار نامه ها در فایل `config/services.php` قرار می گیرند ، و به صورت آرایه ای با اندیس های  `facebook`, `twitter`, `google`, یا `github`, تعریف می شوند . برای مثال :

	'github' => [
		'client_id' => 'your-github-app-id',
		'client_secret' => 'your-github-app-secret',
		'redirect' => 'http://your-callback-url',
	],

حال ، برنامه شما آماده است تا عملیات تصدیق هویت کاربران را به خوبی اجرا کند !  شما به دو Route نیاز خواهید داشت . یکی برای ریدایرکت کردن کاربر به سرویس OAuth ، و دیگری برای دریافت callback از سرویس بعد از تصدیق هویت ... ، نمونه مثال استفاده از فاساد `Socialize` :

	public function redirectToProvider()
	{
		return Socialize::with('github')->redirect();
	}

	public function handleProviderCallback()
	{
		$user = Socialize::with('github')->user();

		// $user->token;
	}

متد redirect وظیفه ریدایرکت کاربر به ارائه دهنده سرویس OAuth مورد نظر را بر عهده دارد . درحالی که متد user تمامی درخواست های رسیده را بررسی کرده و اطلاعات کاربر مورد نظر را از آن دریافت و باز میگرداند .همچنین میتوانید قبل از ریدایرکت کاربر `scope` ها را برای درخواست مشخص نمایید :

	return Socialize::with('github')->scopes(['scope1', 'scope2'])->redirect();
زمانی که یک نمونه شیء از `user` ایجاد شد ، شما میتوانید اطلاعات بیشتری از طریق آن دریافت کنید :

#### بازیابی اطلاعات کاربری

	$user = Socialize::with('github')->user();

	// OAuth Two Providers
	$token = $user->token;

	// OAuth One Providers
	$token = $user->token;
	$tokenSecret = $user->tokenSecret;

	// All Providers
	$user->getId();
	$user->getNickname();
	$user->getName();
	$user->getEmail();
	$user->getAvatar();
