# HTTP Requests

- [دسترسی به یک نمونه از درخواست](#obtaining-a-request-instance)
- [گرفتن ورودی](#retrieving-input)
- [ورودی قبلی](#old-input)
- [کوکی ها](#cookies)
- [فایلها](#files)
- [دیگر اطلاعات درخواست](#other-request-information)

<a name="obtaining-a-request-instance"></a>
## دسترسی به یک نمونه از درخواست

### با استفاده از فاساد

فاساد `Request` امکان دسترسی به درخواست فعلی که به کانتینر آن وابسته است را ایجاد میکند. برای مثال:

	$name = Request::input('name');

به یاد داشته باشید، در صورتی که در یک فضای نام هستید، باید فاساد `Request` را با استفاده از عبارت `;use Request` در بالای فایل کلاس ایمپورت نمایید.

### با استفاده از تزریق وابستگی - Dependency Injection (DI)

برای به دست آورد نمونه ای از درخواست HTTP با استفاده از DI، باید کلاس را بر روی سازنده کلاس یا متد مورد نظر تایپ_هینت کنید. نمونه درخواست فعلی خودکار توسط [service container](/docs/%7B%7Bversion%7D%7D/container) تزریق خواهد شد:

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

در صورتی که متد کنترلر منتظر ورودی از پارامتر روت باشد، به سادگی میتوانید آرگومانهای روت را پس از دیگر وابستگی ها لیست نمایید:

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

<a name="retrieving-input"></a>
## گرفتن ورودی

#### گرفتن یک مقدار ورودی

با استفاده از چند متد ساده، میتوانید تمامی به ورودیهای کاربران در نمونه `Illuminate\Http\Request` دسترسی داشته باشید. از آنجا که تمامی ورودیها برای تمامی متدهای درخواست به یک شکل مورد دسترس قرار میگیرند، نیازی نیست نگران متد درخواست HTTP استفاده شده باشید.

	$name = Request::input('name');

#### گرفتن یک مقدار پیش فرض درصورتی که مقدار ورودی ارائه نشده باش

	$name = Request::input('name', 'Sally');

#### بررسی ارئه شدن یک مقدار ورودی

	if (Request::has('name'))
	{
		//
	}

#### گرفتن تمامی ورودیهای درخواست

	$input = Request::all();

#### گرفتن تنها برخی ورودیهای درخواست

	$input = Request::only('username', 'password');

	$input = Request::except('credit_card');

وقتی با فرمهایی با ورودیهای "آرایه" ای کار میکنید، میتوانید برای دسترسی به آرایه ها از . (dot notation) استفاده کنید

	$input = Request::input('products.0.name');

<a name="old-input"></a>
## ورودی های قبلی

لاراول امکان نگهداری ورودیهای یک درخواست، در طول درخواست بعدی را فراهم نموده است. برای مثال، ممکن است نیاز داشته باشید، پس از اعتبارسنجی یک فرم آن را دوباره با مقادیر پیشین پر کنید.

#### فلش ورودیهای کاربر درون متغیر نشست (session)

متد `flash` ورودیهای فعلی را درون متغیر [نشست](/docs/%7B%7Bversion%7D%7D/session) قرار میدهد، تا در طول درخواست بعدی کاربر به نرم افزار در دسترس باشد:

	Request::flash();

#### فلش برخی از متغیرها درون متغیر نشست

	Request::flashOnly('username', 'email');

	Request::flashExcept('password');

#### فلش و ردیرکت

از آنجا که اغلب فلش را در زمان ردیرکت به صفحه قبل انجام میدهید، میتوانید به سادگی فلش ورودیها در یک زنجیره عملیات با ردیرکت انجام دهید:

	return redirect('form')->withInput();

	return redirect('form')->withInput(Request::except('password'));

#### بازیابی داده های پیشین

برای بازیابی داده های فلش شده از درخواست پیشین، از متد `old` بر روی نمونه `Request` استفاده کنید:

	$username = Request::old('username');

اگر داده های پیشین را دریک قالب بلید (Blade) نمایش میدهید، بسیار راحتتر است که از تابع کمکی `old` استفاده نمایید:

	{{ old('username') }}

<a name="cookies"></a>
## کوکیها

تمامی کوکیهای ایجاد شده توسط فریمورک لاراول با استفاده از یک کد سنجش هویت رمزنگاری و امضا شده اند، به این معنی که در صورت تغییر آنها توسط کلاینت غیر معتبر خواهند بود.

#### خواندن مقدار کوکی

	$value = Request::cookie('name');

#### پیوست کوکی به یک درخواست جدید

تابع کمکی `cookie` به عنوان یک سازنده جدید برای ایجاد نمونه های جدید از `Symfony\Component\HttpFoundation\Cookie` عمل مینماید. کوکیها را میتوان با استفاده از متد `withCookie` به یک نمونه از `Response` پیوست کرد:

	$response = new Illuminate\Http\Response('Hello World');

	$response->withCookie(cookie('name', 'value', $minutes));

#### ایجاد یک کوکی که تا ابد میماند*

_منظور از "ابد" واقعا پنج سال است_

	$response->withCookie(cookie()->forever('name', 'value'));

#### قراردادن کوکیها در صف

شما میتوانید کوکی را حتی پیش از آنکه پاسخ ایجاد گردد، به "صف" اضاف نمایید تا پیوست شود:

	<?php namespace App\Http\Controllers;

	use Cookie;
	use Illuminate\Routing\Controller;

	class UserController extends Controller
	{
		/**
		 * Update a resource
		 *
		 * @return Response
		 */
		 public function update()
		 {
		 	Cookie::queue('name', 'value');

		 	return response('Hello World');
		 }
	}

<a name="files"></a>
## فایلها

#### گرفتن یک فایل آپلود شده

	$file = Request::file('photo');

#### بررسی وجود فایل در درخواست

	if (Request::hasFile('photo'))
	{
		//
	}

شی بازگردانده شده توسط متد `file` نمونه ای از کلاس `Symfony\Component\HttpFoundation\File\UploadedFile` است، که کلاس `SplFileInfo` از PHP را اکستند میکند و متدهای بسیاری برای کار با فایلها ارائه مینماید.

#### بررسی معتبر بودن فایل آپلود شده

	if (Request::file('photo')->isValid())
	{
		//
	}

#### جابجایی فایل آپلود شده

	Request::file('photo')->move($destinationPath);

	Request::file('photo')->move($destinationPath, $fileName);

### دیگر متدهای فایل

There are a variety of other methods available on `UploadedFile` instances. Check out the [API documentation for the class](http://api.symfony.com/2.5/Symfony/Component/HttpFoundation/File/UploadedFile.html) for more information regarding these methods.
متدهای متنوع بسیاری در نمونه های `UploadedFile` وجود دارند. [مستندات API را برای اطلاعات بیشتر این کلاس](http://api.symfony.com/2.5/Symfony/Component/HttpFoundation/File/UploadedFile.html) بررسی نمایید.

<a name="other-request-information"></a>
## دیگر اطلاعات درخواست

The `Request` class provides many methods for examining the HTTP request for your application and extends the `Symfony\Component\HttpFoundation\Request` class. Here are some of the highlights.
کلاس `Request` متدهای بسیاری برای بررسی درخواستهای HTTP ارسال شده برای نرم افزار کاربردی وب ارائه مینماید و کلاس `Symfony\Component\HttpFoundation\Request` را اکستند میکند.

#### گرفتن URI مروبط به درخواست

	$uri = Request::path();
	
#### بررسی استفاده از AJAX توسط درخواست

	if (Request::ajax())
	{
		//
	}

#### گرفتن ممتد درخواست

	$method = Request::method();

	if (Request::isMethod('post'))
	{
		//
	}

#### بررسی همخوانی مسیر درخواست با الگویی مشخص

	if (Request::is('admin/*'))
	{
		//
	}

#### گرفتن URL درخواست جاری

	$url = Request::url();
