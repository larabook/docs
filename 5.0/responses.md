#  پاسخ های HTTP

- [مقدمه](#basic-responses)
- [ریدایرکت ها (redirects)](#redirects)
- [انواع دیگر پاسخ ها](#other-responses)
- [Macro ها ](#response-macros)

<a name="basic-responses"></a>
## مقدمه

#### نمونه ساده ارسال یک پاسخ (response) از درون Route

متداول ترین نوع پاسخ در لاراول از درون یک Route، پاسخ String می باشد .

	Route::get('/', function()
	{
		return 'Hello World';
	});

#### ایجاد پاسخ های سفارشی

اما ، در بسیاری از Route ها   و Controller-actions ها ،به شما یک نمونه از کلاس `Illuminate\Http\Response` یا یک  [view](/docs/{{version}}/views) بعنوان پاسخ ارسال میشود که این امکان را میدهد به راحتی `HTTP status code` و `Headers ` ها را دستکاری کنید .
نمونه ایحاد شده  `Response`  از کلاس `Symfony\Component\HttpFoundation\Response` ارث میبرد ، که انواع مختلفی از متدها را برای ساخت یک پاسخ HTTP شامل میشود .

	use Illuminate\Http\Response;

	return (new Response($content, $status))
	              ->header('Content-Type', $value);

برای آسودگی بیشتر ، شما میتوانید از تابع کمکی `response` نیز استفاده کنید :

	return response($content, $status)
	              ->header('Content-Type', $value);

#### ارسال View به عنوان پاسخ

اگر نیازمندید که از قابلیت های  `response` و متد های آن استفاده کنید (مثلا دستکاری Header ها) و همزمان یک View به عنوان پاسخ به خروجی ارسال کنید میتوانید از متد `view`  استفاده نمایید .

	return response()->view('hello')->header('Content-Type', $type);

#### ارسال کوکی در کنار response

	return response($content)->withCookie(cookie('name', 'value'));

#### استفاده ذنجیره ای متد ها درکنار هم

به یاد داشته باشید که بیشتر متدهای `response` قابلیت استفاده ذنجیره ای دارند که کار را برای شما بسیار راحت میکند . 

	return response()->view('hello')->header('Content-Type', $type)
                     ->withCookie(cookie('name', 'value'));

<a name="redirects"></a>
## ریدایرکت ها (redirects)
یکی از مورد استفاده ترین Response ها ریدایرکت ها هستند که یک نمونه از کلاس `Illuminate\Http\RedirectResponse` میباشند . 

#### ارسال پاسخ از نوع ریداریکت (redirect)
روش های مختلفی برای ایجاد یک پاسخ از نوع `RedirectResponse` وجود دارد که ساده ترین آن استفاده از تابع کمکی `redirect` است .  در زمان تست ، ایجاد یک پاسخ ریداریکت ساختگی  متداول نیست ! بنابراین استفاده از تابع کمکی  `redirect` همیشه و در همه حال قابل قبول می باشد . 

	return redirect('user/login');

#### پاسخ ریداریکت به همراه ارسال متغیر
ریداریکت به یک URL جدید و [انتقال متغیر به Session](/docs/{{version}}/session) معمولا به صورت همزمان انجام میشود . بنابراین برای راحتی بیشتر میتوانید یک نمونه از `RedirectResponse` ایجاد کنید و متغیر ها را از طریق یک متد بصورت ذنجیره وار به session انتقال دهید :

	return redirect('user/login')->with('message', 'Login Failed');

#### ریدایرکت به URL پیشین
ممکن است بخواهید کاربر را به مکان قبل (آدرس  URL پیشین) منتقل کنید ، برای مثال ، بعد از اینکه یک فرم اطلاعاتی را در دیتابیس ذخیره کردید ، میتوانید بوسیله متد `back` کاربر را به صفحه ای که فرم در آن ثبت شده انتقال دهید  :

	return redirect()->back();

	return redirect()->back()->withInput();

#### ریداریکت به یک Route از طریق نام آن
زمانی که از تابع کمکی`redirect` بدون پارامتر استفاده میکنید لاراول به شما یک نمونه از کلاس `Illuminate\Routing\Redirector`  برمیگرداند  ، که میتوانید از متد های آن استفاده نمایید . برای مثال یک `RedirectResponse` را برای یک روت از طریق نام آن توسط متد `route` میتوانید ایجاد کنید :

	return redirect()->route('login');

#### ریداریکت به یک Route از طریق نام آن و ارسال پارامتر
اگر روت شما داری پارامتر می باشد ، میتوانید از طریق آرگومان دوم متد `route` آنها را تعریف کنید :

	// For a route with the following URI: profile/{id}

	return redirect()->route('profile', [1]);

درصورتی که پارامتر شما یک مدل Eloquent باشد لاراول بطور پیشفرض خصوصیت فیلد ID را از درون مدل استخراج کرده و به عنوان پارامتر ارسال میکند :

	return redirect()->route('profile', [$user]);

#### ریداریکت به یک Route از طریق نام آن و ارسال مقدار به یک پارامتر نامگذاری شده
	// For a route with the following URI: profile/{user}

	return redirect()->route('profile', ['user' => 1]);

#### ریداریکت به یک کنترلر و اکشن
همانند ریدایرکت به یک Route شما میتوانید به یک [controller actions](/docs/{{version}}/controllers)  خاص نیز ریداریکت کنید :

	return redirect()->action('App\Http\Controllers\HomeController@index');

> **نکته :** درصورتی که مسیر namespace ریشه کنترلرها رو از طریق متد `URL::setRootControllerNamespace` مشخص کنید دیگر نیاز نخواهید داشت `namespace`  کامل را در کلاس URL وارد کنید .

####  ریداریکت به یک  [controller actions](/docs/{{version}}/controllers) و ارسال پارامتر به آن :

	return redirect()->action('App\Http\Controllers\UserController@profile', [1]);

#### ریداریکت به یک  [controller actions](/docs/{{version}}/controllers) و ارسال مقدار به یک پارامتر نامگذاری شده :
	return redirect()->action('App\Http\Controllers\UserController@profile', ['user' => 1]);

<a name="other-responses"></a>
## انواع دیگر پاسخ ها
تابع کمکی `response` برای تولید انواع دیگر پاسخ ها (,JSONP , JSON , دانلود , ...) نیز بکار میرود . زمانیکه تابع کمکی `response` را بدون آرگومان صدا بزنید ، یک Implementation از کلاس `Illuminate\Contracts\Routing\ResponseFactory`  [contract](/docs/{{version}}/contracts) باز گردانده می شود . که این `contract` متد های مفیدی برای تولید انواع پاسخ در خود دارد .

#### ارسال یک پاسخ JSON
متد json بصورت خودکار مقدار هدر `Content-Type`  را `application/json`  قرار میدهد :

	return response()->json(['name' => 'Abigail', 'state' => 'CA']);

#### ارسال یک پاسخ JSONP

	return response()->json(['name' => 'Abigail', 'state' => 'CA'])
	                 ->setCallback($request->input('callback'));

#### ارسال پاسخ به صورت دانلود فایل

	return response()->download($pathToFile);

	return response()->download($pathToFile, $name, $headers);

	return response()->download($pathToFile)->deleteFileAfterSend(true);

> **نکته :** کلاس Symfony HttpFoundation , که وظیفه مدیریت دانلود را بر عهده دارد , نیازمند این می باشد  که نام  فایل دانلودی ، شامل کاراکتر های ASCII باشد .

<a name="response-macros"></a>
## Response Macros
ماکرو ها این امکان را میدهند که متد های `response` را سفارشی کنید و آنها را در قسمت های مختلف پروژه (کنترلر ها و Route ها و ..) استفاده کنید  .
برای مثال , تعریف ماکرو در متد `boot` از یک  [service provider](/docs/{{version}}/providers) :

	<?php namespace App\Providers;

	use Response;
	use Illuminate\Support\ServiceProvider;

	class ResponseMacroServiceProvider extends ServiceProvider {

		/**
		 * Perform post-registration booting of services.
		 *
		 * @return void
		 */
		public function boot()
		{
			Response::macro('caps', function($value)
			{
				return Response::make(strtoupper($value));
			});
		}

	}

متد `macro` یک نام به عنوان اولین آرگومان و یک `Closure`  به عنوان آرگومان دوم دریافت میکند . `Closure` معرفی شده در زمانی که ماکرو مورد نظر صدا زده شود اجرا خواهد شد :

	return response()->caps('foo');
