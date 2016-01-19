# middleware HTTP

- [آشنایی](#introduction)
- [ایجاد middleware](#defining-middleware)
- [ثبت middleware](#registering-middleware)
- [middleware قابل پایان دادن](#terminable-middleware)

<a name="introduction"></a>
## آشنایی

middleware HTTP مکانیزمی راحت برای فیلترنمودن درخواستهای HTTP دریافت شده توسط نرم افزار ایجاد میکند. برای مثال، لاراول یک middleware برای تعیین احراز هویت شدن کاربر نرم افزار دارد. در صورتی که کاربر احراز هویت نشده باشد، middleware کاربر را به صفحه ورود هدایت مینماید. و اگر کاربر احراز هویت شده باشد، middleware به درخواست اجازه پیشروی در نرما افزار را میدهد.

البته، middleware میتواند برای انجام کارهای بسیاری فراتر از احراز هویت نوشته شود. یک middleware CORS ممکن از وظیفه افزودن یک هدر مناسب به تمام پاسخهای خروجی از نرم افزار شما را داشته باشد. یک middleware لاگ (ثبت ردپا) ممکن است تمامی درخواست دریافت شده توسط نرم افزار را ثبت نماید.

middlewareهای بسیاری در فریمورک لاراول وجود دارند، شامل middlewareهایی برای نگهداری (maintenance)، احرازهویت، CSRF، نگهبانی (protection)، و موراد دیگر میباشد. تمامی این middlewareها در دایرکتوری `app/Http/Middleware` قرار دارند.

<a name="defining-middleware"></a>
## ایجاد middleware

برای ایجاد یک middleware، از فرمان آرتیزان `make:middleware` استفاده کنید:

	php artisan make:middleware OldMiddleware

این فرمان یک کلاس `OldMiddleware` جدید در دایرکتوری `app/Http/Middleware` ایجاد میکند. در این middleware، ما تنها در صورتی به مسیر دسترسی داریم که `age` بزرگتر اس 200 باشد. در غیر اینصورت، کاربر به مسیر "home" هدایت میشود.

    <?php namespace App\Http\Middleware;
	
	use Closure;

	class OldMiddleware {

		/**
		 * Run the request filter.
		 *
		 * @param  \Illuminate\Http\Request  $request
		 * @param  \Closure  $next
		 * @return mixed
		 */
		public function handle($request, Closure $next)
		{
			if ($request->input('age') < 200)
			{
				return redirect('home');
			}

			return $next($request);
		}

	}

همانگونه که میبینید، در صورتی که `age` از  `200` کمتر باشد، middleware یک ریدیرکت (تغییردهنده مسیر) HTTP به کلاینت بازمیگرداند؛ در غیر اینصورت، درخواست به مراحل بعدی در نرم افزار فرستاده میشود. برای فرستادن درخواست به عمق نرم افزار (برای اجازه دادن به middleware برای گذر)، کافیست تابع کالبک (فراخوان مجدد) `next$` را با پارامتر `request$` فراخوانی نمایید.

بهتر است middlewareها را به عنوان لایه هایی در نظر بگیرید که درخواستهای HTTP باید از آنها عبور کنند تا به نرم افزار برسند. هر لایه میتواند درخواست را بررسی نماید و حتی آن را کاملا رد نماید.

### اجرای ذستورات قبل و بعد از یک درخواست توسط Middleware

اجرای یک middleware پیش از یا پس از اجرای درخواست به خود middleware وابسته است. این middleware کارهایی را **پیش از** مدیریت درخواست توسط نرم افزار انجام میدهد:

	<?php namespace App\Http\Middleware;
	
	use Closure;

	class BeforeMiddleware implements Middleware {

		public function handle($request, Closure $next)
		{
			// Perform action

			return $next($request);
		}
	}

و این middleware وظیفه خود را **پس از** مدیریت درخواست توسط نرم افزار انجام میدهد:

	<?php namespace App\Http\Middleware;
	
	use Closure;

	class AfterMiddleware implements Middleware {

		public function handle($request, Closure $next)
		{
			$response = $next($request);

			// Perform action

			return $response;
		}
	}

<a name="registering-middleware"></a>
## ثبت middleware

### middleware سراسری

اگر میخواهید middlewareی در طول تمامی درخواستهای HTTP فرستاده شده برای نرم افزار کاربردی شما اجرا شود به سادگی کلاس middleware را در لیست خصوصیت `middleware$` از کلاس `app/Http/Kernel.php` قرار دهید.

### انتصاب middleware به route

اگر میخواهید یک middleware را به یک route مشخص انتصاب دهید، در ابتدا باید به middleware یک کلید کوتاه در فایل `app/Http/Kernel.php` انتصاب دهید. به طور پیش فرض، خصوصیت `routeMiddleware$` این کلاس، دربرگیرنده middlewareهای پیش فرض لاراول میباشد.
پس از تعریف یک middleware در کرنل HTTP، میتوانید از کلید `middleware` در آرایه گزینه های route برای اعمال استفاده کنید:

	Route::get('admin/profile', ['middleware' => 'auth', function()
	{
		//
	}]);

<a name="terminable-middleware"></a>
## middlewareهای با قابلیت پایان دادن

گاهی یک middleware باید کاری را پس از آنکه پاسخ HTTP به مرورگر فرستاده شد، انجام دهد. برای مثال "middleware "session در لاراول _پس از_ آنکه پاسخ به مرورگر فرستاده شد، داده های نشست را ذخیره مینماید. برای انجام این موضوع باید middleware را "terminable" تعریف کنید

    	use Closure;
    use Illuminate\Contracts\Routing\TerminableMiddleware;
	class StartSession implements TerminableMiddleware {

		public function handle($request, Closure $next)
		{
			return $next($request);
		}

		public function terminate($request, $response)
		{
			// Store the session data...
		}

	}

همانطور که میبینید، علاوه بر تعریف متد `handle`، `TerminableMiddleware` یک متد `terminate` هم تعریف میکند. پس از آنکه یک middleware با قابلیت پایان دادن ایجاد نمودید، باید آن را به لیست middlewareهای گلوبال در کرنل HTTP بیافزایید.

