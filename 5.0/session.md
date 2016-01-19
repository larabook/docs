# نشست - Session

- [پیکربندی](#configuration)
- [کاربرد نشست](#session-usage)
- [Flash Data](#flash-data)
- [نشستهای پایگاه داده](#database-sessions)
- [درایورهای نشست](#session-drivers)

<a name="configuration"></a>
## پیکربندی

از آنجاکه نرم افزارهای مبتنی بر HTTP وضعیت اتصالها را نگاه نمیدارند (stateless)، نشستها به عنوان راه حلی برای نگاهداری اطلاعات کاربران در بین درخواستهاست. لاراول امکانات زیادی برای استفاده در یک API شسته رفته و یکپارچه دارد. پشتیبانی از بک اندهایی مانند [Memcached](http://memcached.org)  و [Redis](http://redis.io) و پایگاههای داده در لاراول انجام می شود.

تنظیمات نشست در فایل `config/session.php` ذخیره می شوند. اطمینان حاصل کنید که گزینه های این فایل را به دقت مطالعه نمایید. به صورت پیش فرض، لاراول برای استفاده از درایور نشست `file` تنظیم شده است. این درایور برای عموم نرم افزارها به خوبی کار میکند.

پیش از استفاده از نشستهای Redis در لاراول باید بسته `predis/predis` (~1.0) را از طریق کامپوزر نصب کنید.

> **نکته:** اگر داده های نشستهای شما باید رمزگذاری شوند، مقدار گزینه `encrypt` را برابر با `true قرار دهید.

> **نکته:** هنگام استفاده از درایور نشست `cookie`، به هیچ وجه نباید middleware با عنوان `EncryptCookie` را از کرنل HTTP حذف نمایید. اگر این middleware را حذف کنید، نرم افزار در مقابل code injection آسیب پذیر خواهد بود.

#### کلیدهای رزروشده

فریمورک لاراول از کلید نشست `flash` به طور پیش فرض استفاده میکند. بنابراین شما نباید موردی را با این اسم به نشست بیافزایید.

<a name="session-usage"></a>
## کاربرد نشست

به چند روش میتوان به نشستها دسترسی داشت، با استفاده از متد `session` از درخواستهای HTTPT، با استفاده از فاساد `Session`، یا تابع کمکی `session`. اگر تابع کمکی `session` بدون آرگومان فراخوانی شود، خروجی آن کل شی نشست خواهد بود. برای مثال:

	session()->regenerate();

#### ذخیره یک عنصر  در نشست

	Session::put('key', 'value');

	session(['key' => 'value']);

#### افزودن یک مقدار به آرایه ای در نشست

	Session::push('user.teams', 'developers');

#### خواندن یک مقدار از نشست

	$value = Session::get('key');

	$value = session('key');

#### خواندن یک مقدار از نشست یا بازگرداندن مقدار پیش فرض

	$value = Session::get('key', 'default');

	$value = Session::get('key', function() { return 'default'; });

#### خواندن مقدار از نشست و حذف آن

	$value = Session::pull('key', 'default');

#### خواندن تمامی مقادیر از نشست

	$data = Session::all();

#### بررسی وجود یک مقدار در نشست

	if (Session::has('users'))
	{
		//
	}

#### حذف یک مقدار از نشست

	Session::forget('key');

#### حذف تمامی مقدارها از نشست

	Session::flush();

#### تولید مجدد ID نشست

	Session::regenerate();

<a name="flash-data"></a>
## فلش داده ها - Flash Data

گاهی میخواهید اطلاعات را تنها برای درخواست بعدی در session ذخیره کنید. اینکار را با استفاده از متد `Session::flash` میتوانید انجام دهید.

	Session::flash('key', 'value');

#### فلش مجدد داده ها فعلی برای session بعدی

	Session::reflash();

#### فلش مجدد تنها بخشی از داده ها

	Session::keep(['username', 'email']);

<a name="database-sessions"></a>
## سشن های پایگاه داده

برای استفاده از درایور نشست `database`، باید جدولی برای نگاهداری عناصر نشست ایجاد شود. در ادامه یک تعریف `Schema` از این جدول ارائه شده است:

	Schema::create('sessions', function($table)
	{
		$table->string('id')->unique();
		$table->text('payload');
		$table->integer('last_activity');
	});

البته میتوانید از فرمان آرتیزان `session::table` برای ایجاد این میگریشن استفاده کنید!

	php artisan session:table

	composer dump-autoload

	php artisan migrate

<a name="session-drivers"></a>
## درایورهای نشست - Session Drivers

درایور نشست مکان ذخیره سازی داده های نشست برای هر درخواست را مشخص میکند. لاراول درایورهای بسیاری با خود دارد:

- `file` - نشستها در `storage/framework/sessions` ذخیره میشوند.
- `cookie` - نشستهای در کوکیهای کدگذاری شده نگهداری میشوند.
- `database` -نشستها در پایگاه داده استفاده شده استفاده شده توسط نرم افزار شما ذخیره میشوند.
- `memcached` / `redis` - نشستها در یکی از این کشهای سریع و امن نگاهداری میشوند.
- `array` - نشستها در آرایه های ساده PHP نگاهداری می شوند و ما بین درخواستهای مختلف حفظ نمی شوند.

> **نکته:** درایورarray معمولا برای اجرای [unit test](/docs/%7B%7Bversion%7D%7D/testing) ها استفاده می شود، بنابراین هیچ داده ای ذخیره دائم نمیشود.
