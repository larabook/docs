# Views

- [کاربرد پایه ای ](#basic-usage)
- [View Composers](#view-composers)

<a name="basic-usage"></a>
## کاربرد ابتدایی

viewها شامل کد HTML نرم افزار وب شما هستند، و روشی ساده برای جداکردن کنترلر و منطق کاری از منطق نمایشی هستند. viewها در دایرکتوری `resources/views`  قراردارند.

یک view ساده به شکل زیر است:

	<!-- View stored in resources/views/greeting.php -->

	<html>
		<body>
			<h1>Hello, <?php echo $name; ?></h1>
		</body>
	</html>

این view را میتوانید به این شکل به مرورگر بفرستید:

	Route::get('/', function()
	{
		return view('greeting', ['name' => 'James']);
	});

همانطور که میبینید، اولین آرگومان فرستاده شده برای تابع کمکی `view` به نام فایل view در `resources/views` دایرکتوری اشاره دارد. آرگومان دوم فرستاده شده به تابع کمکی آرایه ای داده هاست که درview استفاده میشود.

البته، viewها میتوانند در پوشه های تودرتو از دایرکتوری `resources/views` قرار داشته باشند. برای مثال، اگر view در `resources/views/admin/profile.php` ذخیره شده باشد، باید به این شکل استفاده شود:

	return view('admin.profile', $data);

#### ارسال داده به view

	// Using conventional approach
	$view = view('greeting')->with('name', 'Victoria');

	// Using Magic Methods
	$view = view('greeting')->withName('Victoria');

در مثال بالا، متغیر `$name` در دسترس view قرار دارد و حاوی مقدار `Victoria` است.

میتوانید یک آرایه داده را به عنوان پارامتر دوم به تابع کمکی `view` بفرستید:

	$view = view('greetings', $data);

هنگام انتقال اطلاعات به این شکل، `data` باید آرایه ای با مقادیر کلید/مقدار باشد. درون view، میتوانید با استفاده از کلید مربوط به هر مقدار به آن دست پیدا کنید. برای مثال با فرض وجود `$data['key']` میتوان از `{{ $key }}` درview برای دسترسی به داده استفاده کرد.

#### اشتراک داده با تمام viewها

گاهی، ممکن است بخواهید داده ای را در دسترس تمامی viewهای به نمایش درآمده توسط نرم افزارتان قرار دهید. گزینه های زیادی برای این کار دارید: تابع کمکی `view`، [کانترکت](/docs/%7B%7Bversion%7D%7D/contracts) `Illuminate\Contracts\View\Factory`، و یا استفاده از یک وایلدکارد [view کامپوزر](#view-composers).

برای مثال روش استفاده از تابع کمکی `view`:

	view()->share('data', [1, 2, 3]);

همچنین میتوانید از فاساد `View` استفاده کنید:

	View::share('data', [1, 2, 3]);

به طور معمول، فراخوانی متد `share` را در متد `boot` یک service provider قرار میدهیم. اما میتوانید آن را به `AppServiceProvider` بیافزایید یا یک service provider مجزا برای مدیریت به اشتراک گذاری داده با تمام view ها ایجاد کنید.

> **نکته:** هنگامی که تابع کمکی `view` بدون آرگومان فراخوانی شود، یک پیاده سازی از کانترکت `Illuminate\Contracts\View\Factory` ایجاد میکند.

#### بررسی وجود view

اگر میخواهید وجود یک view را بررسی نمایید، میتوانید از متد `exists` استفاده کنید:

	if (view()->exists('emails.customer'))
	{
		//
	}

#### فراخوانی view از مسیر فایل

اگر بخواهید میتوانید یک view با استفاده از آدرس مسیر کامل یک فایل ایجاد نمایید:

	return view()->file($pathToFile, $data);

<a name="view-composers"></a>
## View Composers

view composerها کالبکها یا متدهای کلاس هستند که هنگام رندر view فراخوانی میشوند. اگر داده ای دارید که با هر بار رندرشدن، بخواهید به view بایند شود، view composer آن منطق را در یک مکان مدیریت شده نگاهداری میکند.

#### تعریف یک View Composer

میتوانیم view composerها را درون یک [service provider](/docs/%7B%7Bversion%7D%7D/providers) قرار دهیم. از فاساد `View` برای دستیابی به کانترکت یک نمونه از `Illuminate\Contracts\View\Factory` استفاده میکنیم:

	<?php namespace App\Providers;

	use View;
	use Illuminate\Support\ServiceProvider;

	class ComposerServiceProvider extends ServiceProvider {

		/**
		 * Register bindings in the container.
		 *
		 * @return void
		 */
		public function boot()
		{
			// Using class based composers...
			View::composer('profile', 'App\Http\ViewComposers\ProfileComposer');

			// Using Closure based composers...
			View::composer('dashboard', function($view)
			{

			});
		}

		/**
		 * Register the service provider.
		 *
		 * @return void
		 */
		public function register()
		{
			//
		}

	}

> **نکته:** لاراول پوشه پیش فرضی برای view composerها ندارد. برای دسته بندی آنها هرکجا که بخواهید آزاد هستید. برای مثال، میتوانید یک دایرکتوری `App\Http\ViewComposers` ایجاد کنید.

به یاد داشته باشید، باید service provider را به آرایه `providers` در فایل پیکربندی `config/app.php` بیافزایید.

حال که composer را معرفی کرده ایم، متد `ProfileComposer@compose` با هر بار رندرشدن `profile` فراخوانی میشود. در ادامه کلاس composer را تعریف میکنیم:

	<?php namespace App\Http\ViewComposers;

	use Illuminate\Contracts\View\View;
	use Illuminate\Users\Repository as UserRepository;

	class ProfileComposer {

		/**
		 * The user repository implementation.
		 *
		 * @var UserRepository
		 */
		protected $users;

		/**
		 * Create a new profile composer.
		 *
		 * @param  UserRepository  $users
		 * @return void
		 */
		public function __construct(UserRepository $users)
		{
			// Dependencies automatically resolved by service container...
			$this->users = $users;
		}

		/**
		 * Bind data to the view.
		 *
		 * @param  View  $view
		 * @return void
		 */
		public function compose(View $view)
		{
			$view->with('count', $this->users->count());
		}

	}

درست پیش از رندرشدن view، متد `compose` با نمونه کلاس `Illuminate\Contracts\View\View` فراخوانی میشود. بای بایند داده به view میتوانید از متد `with` استفاده کنید.

> **نکته:** تمامی view composerها از طریق service container تشخیص داده می شوند، بنابراین میتوانید هر وابستگی مورد نیاز را در composer constructor اعلان نوع کنید.

#### Wildcard View Composers

متد `composer` کاراکتر `*` را به عنوان وایلدکارد میپذیرد، و به این واسطه شما میتوانید یک composer را برای تمام viewها استفاده نمایید.

	View::composer('*', function($view)
	{
		//
	});

#### پیوست یک composer به چند view

میتوانید یک view composer را برای چندین view به طور همزمان استفاده نمایید:

	View::composer(['profile', 'dashboard'], 'App\Http\ViewComposers\MyViewComposer');

#### تعریف چند composer

میتوانید از متد `composers` برای ثبت گروهی از composerها به طور همزمان استفاده کنید:

	View::composers([
		'App\Http\ViewComposers\AdminComposer' => ['admin.index', 'admin.profile'],
		'App\Http\ViewComposers\UserComposer' => 'user',
		'App\Http\ViewComposers\ProductComposer' => 'product'
	]);

### view creator - سازنده های view

**سازنده**های view عملکردی شبیه به view composerها دارند؛ هرچند، آنها درست زمان ساخته شدن view اجرا می شوند. برای ثبت یک سازنده view، از متد `creator` استفاده کنید:

	View::creator('profile', 'App\Http\ViewCreators\ProfileCreator');
