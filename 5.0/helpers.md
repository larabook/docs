# تابع های کمکی - Helpers

- [آرایه ها - Arrays](#arrays)
- [مسیرها - Paths](#paths)
- [نگاشت مسیر - Routing](#routing)
- [Strings](#strings)
- [URLs](#urls)
- [متنوع](#miscellaneous)

<a name="arrays"></a>
## آرایه ها - Arrays

### array_add

تابع `array_add` در صورتی که دوتایی کلید / مقدار در آرایه وجود نداشته باشد، آن را به آرایه می افزاید.

	$array = ['foo' => 'bar'];

	$array = array_add($array, 'key', 'value');

### array_divide

تابع `array_divide` دو آرایه بازمیگرداند، یکی شامل کلیدها و دیگری مقادیر موجود در آرایه اولیه.

	$array = ['foo' => 'bar'];

	list($keys, $values) = array_divide($array);

### array_dot

تابع `array_dot` آرایه های چندبعدی را به آرایه تک بعدی کاهش بعد داده و امکان دسترسی به عمق آرایه با استفاده از "." فراهم میکند.

	$array = ['foo' => ['bar' => 'baz']];

	$array = array_dot($array);

	// ['foo.bar' => 'baz'];

### array_except

تابع `array_except` دوتایی کلید / مقدار را از آرایه حذف میکند.

	$array = array_except($array, ['keys', 'to', 'remove']);

### array_fetch

تابع `array_fetch` آرایه ای تک بعدی شامل مقادیر کلید داده شده باز میگرداند.

	$array = [
		['developer' => ['name' => 'Taylor']],
		['developer' => ['name' => 'Dayle']]
	];

	$array = array_fetch($array, 'developer.name');

	// ['Taylor', 'Dayle'];

### array_first

تابع `array_first` اولین عنصر یک آرایه که شرط ارائه شده را داشته باشد باز میگرداند.

	$array = [100, 200, 300];

	$value = array_first($array, function($key, $value)
	{
		return $value >= 150;
	});

یک مقدار پیش فرض را نیز به عنوان پارامتر سوم میتوان به تابع فرستاد:

	$value = array_first($array, $callback, $default);

### array_last

تابع `array_last` آخرین عنصر آرایه ای که شرط ارائه شده در مورد آن صادق باشد را برمیگرداند.

	$array = [350, 400, 500, 300, 200, 100];

	$value = array_last($array, function($key, $value)
	{
		return $value > 350;
	});

	// 500

یک مقدار پیش فرض را نیز به عنوان پارامتر سوم میتوان به تابع فرستاد:

	$value = array_last($array, $callback, $default);

### array_flatten

تابع `array_flatten` یک آرایه چند بعدی را تبدیل به آرایه ای تک بعدی مینماید.

	$array = ['name' => 'Joe', 'languages' => ['PHP', 'Ruby']];

	$array = array_flatten($array);

	// ['Joe', 'PHP', 'Ruby'];

### array_forget

تابع `array_forget` یک دوتایی را از عمق آرایه ای تو در تو با استفاده از "." حذف مینماید.

	$array = ['names' => ['joe' => ['programmer']]];

	array_forget($array, 'names.joe');

### array_get

تابع `array_get` یک مقدار داده شده را از عمق یک آرایه چند بعدی با استفاده از "." باز میگرداند.

	$array = ['names' => ['joe' => ['programmer']]];

	$value = array_get($array, 'names.joe');

	$value = array_get($array, 'names.john', 'default');

> **نکته:** چیزی شبیه به `array_get` اما برای اشیاء لازم دارید؟ از `object_get` استفاده کنید.

### array_only

متد `array_only` تنها دوتاییهای مشخص شده را از آرایه بازمیگرداند.

	$array = ['name' => 'Joe', 'age' => 27, 'votes' => 1];

	$array = array_only($array, ['name', 'votes']);

### array_pluck

متد `array_pluck` لیستی از دوتایی های کلید / مقدار را از آرایه ایجاد میکند.

	$array = [['name' => 'Taylor'], ['name' => 'Dayle']];

	$array = array_pluck($array, 'name');

	// ['Taylor', 'Dayle'];

### array_pull

متد `array_pull` پس از بازگرداندن یک دوتایی کلید / مقدار از یک آرایه، آن را حذف میکند.

	$array = ['name' => 'Taylor', 'age' => 27];

	$name = array_pull($array, 'name');

### array_set

متد `array_set` مقدار را در عمق یک آرایه با استفاده از "." مقداردهی میکند.

	$array = ['names' => ['programmer' => 'Joe']];

	array_set($array, 'names.editor', 'Taylor');

### array_sort

متد `array_sort` آرایه را با توجه به نتایج کلوژر ارائه شده مرتب سازی میکند.

	$array = [
		['name' => 'Jill'],
		['name' => 'Barry']
	];

	$array = array_values(array_sort($array, function($value)
	{
		return $value['name'];
	}));

### array_where

آرایه را با استفاده از کلوژر ارائه شده فیلتر مینماید.

	$array = [100, '200', 300, '400', 500];

	$array = array_where($array, function($key, $value)
	{
		return is_string($value);
	});

	// Array ( [1] => 200 [3] => 400 )

### head

اولین عنصر در آرایه را بازمیگرداند.

	$first = head($this->returnsArray('foo'));

### last

آخرین عنصر آرایه را بازمیگرداند. در فراخوانی زنجیره ای متدها کاربرد دارد.

	$last = last($this->returnsArray('foo'));

<a name="paths"></a>
## مسیرها - Paths

### app_path

مسیر کامل دایرکتوری `app` را بازمیگرداند.

	$path = app_path();

### base_path

مسیر کامل محل نصب پروژه را باز میگرداند.

### config_path

مسیر کامل به دایرکتوری `config` را بازمیگرداند.

### public_path

مسیرکامل دایرکتوری `public` را بازمیگرداند.

### storage_path

مسیر کامل دایرکتوری `storage` را بازمیگرداند.

<a name="routing"></a>
## Routing

### get

route جدیدی از نوع GET را ثبت میکند.

	get('/', function() { return 'Hello World'; });

### post

route جدیدی از نوع POST را ثبت میکند.

	post('foo/bar', 'FooController@action');

### put

route جدیدی از نوع PUT را ثبت میکند.

	put('foo/bar', 'FooController@action');

### patch

route جدیدی از نوع PATCH را ثبت میکند.

	patch('foo/bar', 'FooController@action');

### delete

route جدیدی از نوع DELETE را ثبت میکند.

	delete('foo/bar', 'FooController@action');
	
### resource

route جدیدی از نوع منبع RESTful را ثبت میکند.

	resource('foo', 'FooController');

<a name="strings"></a>
## Strings

### camel_case

یک رشته متن داده شده را به `camelCase` تبدیل میکند.

	$camel = camel_case('foo_bar');

	// fooBar

### class_basename

نام کلاس داده شده را بدون فضای نام (namespace) ارائه میکند.

	$class = class_basename('Foo\Bar\Baz');

	// Baz

### e

تابع `htmlentities` را بر روی یک string با پشتیبانی از UTF-8 اجرا میکند.

	$entities = e('<html>foo</html>');

### ends_with

بررسی میکند آیا رشته ارائه شده به مقدار مورد نظر ختم می شود.

	$value = ends_with('This is my name', 'name');

### snake_case

رشته ارائه شده را به `snake_case` تبدیل میکند.

	$snake = snake_case('fooBar');

	// foo_bar

### str_limit

تعداد کاراکترها را در یک رشته به اندازه پارامتر ارائه شده محدود میکند.

	str_limit($value, $limit = 100, $end = '...')

Example:

	$value = str_limit('The PHP framework for web artisans.', 7);

	// The PHP...

### starts_with

بررسی میکند آیا رشته ارائه شده با مقدار مورد نظر شروع می شود.

	$value = starts_with('This is my name', 'This');
	

### str_contains

وجود مقدار داده شده در رشته متن را بررسی میکند.

	$value = str_contains('This is my name', 'my');

### str_finish

مقدار ارائه شده را برای یکبار به رشته متن می افزاید. بقیه مقادیر را حذف میکند.

	$string = str_finish('this/string', '/');

	// this/string/

### str_is

همخوانی یک رشته متنی با الگوی ارائه شده را بررسی میکند. برای مشخص کردن وجود "هر" مقدار میتوانید از علامت * استفاده کنید.

	$value = str_is('foo*', 'foobar');

### str_plural

یک رشته را به فرم جمع تبدیل میکند (تنها در زبان انگلیسی).

	$plural = str_plural('car');

### str_random

رشته ای تصادفی ا زکاراکترها با طول داده شده ایجاد میکند.

	$string = str_random(40);

### str_singular

رشته را به شکل مفرد تبدیل میکند (تنها در زبان انگلیسی).

	$singular = str_singular('cars');

### str_slug

از یک رشته متن یک عبارت خوش فرم برای URL میسازد.

	str_slug($title, $separator);

مثال:

	$title = str_slug("Laravel 5 Framework", "-");

	// laravel-5-framework

### studly_case

رشته متنی را به `StudlyCase` تبدیل میکند.

	$value = studly_case('foo_bar');

	// FooBar

### trans

ترجمه یک متن را با توجه به زبان انتخاب شده جایگزین میکند. آلیاس (Alias) برای `Lang::get`.

	$value = trans('validation.required'):

### trans_choice

یک متن را با توجه به تعداد کلمات مشخص شده ترجمه میکند. آلیاس (Alias) برای `Lang::choice`

	$value = trans_choice('foo.bar', $count);

<a name="urls"></a>
## URLs

### action

URL برای مربوط به یک متد کنترلررا ارائه مینماید.

	$url = action('HomeController@getIndex', $params);

### route

URL متناظر با یک route نامگذاری شده را ایجاد میکند.

	$url = route('routeName', $params);

### asset

URL مربوط به منابع استاتیک (عکس، جاوااسکریپت، css) را ایجاد میکند.

	$url = asset('img/photo.jpg');

### secure_asset

ایجاد URL مربوط به منابع استاتیک با استفاده از HTTPS.

	echo secure_asset('foo/bar.zip', $title, $attributes = []);

### secure_url

ایجاد URL کامل به یک مسیر مشخص شده در قالب HTTPS.

	echo secure_url('foo/bar', $parameters = []);

### url

URL کامل برای یک مسیر مشخص شده ایجاد میکند.

	echo url('foo/bar', $parameters = [], $secure = null);

<a name="miscellaneous"></a>
## Miscellaneous

### csrf_token

مقدار توکن CSRF فعلی را باز میگرداند.

	$token = csrf_token();

### dd

متغیر مشخص شده را چاپ میکند و اجرای اسکریپت را متوقف میکند.

	dd($value);

### elixir

مسیر فایل ورژن گذاری شده Elixir را باز میگرداند.

	elixir($file);

### env

مقدار متغیر محلی را باز میگرداند. در صورت نداشتن مقدار مقدار پیش فرض را باز میگرداند.

	env('APP_ENV', 'production')

### event

اجرای یک رویداد

	event('my.event');

### value

اگر مقدار داده شده `Closure` باشد، مقدار بازگردانده شده توسط `Closure` را باز میگرداند. در غیراینصورت مقدار مشخص شده را بازمیگرداند.

	$value = value(function() { return 'bar'; });

### view

نمونه View برای مسیر view مشخص شده را بازمیگرداند.

	return view('auth.login');

### with

شی مشخص شده را بازمیگرداند.

	$value = with(new Foo)->doWork();
