# Service Container

- [مقدمه](#introduction)
- [نمونه ساده](#basic-usage)
- [تزریق Interfaces ها به دورن Implementation ها](#binding-interfaces-to-implementations)
- [تزریق شرطی (Contextual Binding)
](#contextual-binding)
- [تگ کردن (Tagging)](#tagging)
- [استفاده عملی](#practical-applications)
- [رویداد ها (Container Events)](#container-events)

<a name="introduction"></a>
## مقدمه
service container  یکی از بخشهای مهم و اساسی لاراول می باشد  بوسیله آن می توانید وابستگی های کلاس ها  (dependencies) را مدیریت کنید . تزریق وابستگی ها (Dependency injection) یکی از روش های پرطرفدار در دنیای برنامه نویسی می باشد و مفهوم آن این است : شما میتوانید به وسیله آن ،  کلاس ها و پکیج های مورد نیاز خود (dependencies) را از طریق متد سازنده  کلاس  (constructor) یا در برخی مواقع بوسیله متد "setter" لود نمایید .

به یک مثال ساده توجه کنید :

	<?php namespace App\Handlers\Commands;

	use App\User;
	use App\Commands\PurchasePodcastCommand;
	use Illuminate\Contracts\Mail\Mailer;

	class PurchasePodcastHandler {

		/**
		 * The mailer implementation.
		 */
		protected $mailer;

		/**
		 * Create a new instance.
		 *
		 * @param  Mailer  $mailer
		 * @return void
		 */
		public function __construct(Mailer $mailer)
		{
			$this->mailer = $mailer;
		}

		/**
		 * Purchase a podcast.
		 *
		 * @param  PurchasePodcastCommand  $command
		 * @return void
		 */
		public function handle(PurchasePodcastCommand $command)
		{
			//
		}

	}
در این مثال ،    `PurchasePodcast` یک ایمیل زمانی که یک پادکست خریداری میشود ارسال میکند . بنابراین ما به یک سرویس برای ارسال ایمیل نیاز خواهیم تاشت که آن را در constructor  کلاس  تزریق میکنیم.

مطالعه و فهم  عمیق  service container  در لاراول لازمه ساخت پروژه های بزرگ  و  همچنین مشارکت درتوسعه هسته لاراول است .

<a name="basic-usage"></a>
## نمونه ساده

### درون ریزی Binding
تقریبا  تمامی عملیات معارفه سرویس ها در [service providers](/docs/{{version}}/providers) ها صورت میگیرد . بنابراین تمامی این مثال ها به ما نشان می دهد که چگونه از Service Container ها در بستر پروژه مان استفاده کنیم . اما اگر قصد دارید از این سرویس ها در جاهای دیگری مثل کلاس Factory استفاده کنید ، میتوانید کانتراکت `Illuminate\Contracts\Container\Container` را تایپ هینت (Type-Hint) کنید . در نتیجه یک نمونه از سرویس مورد نظر برای شما ایجاد میگردد. همچنین روش دیگری نیزوجود دارد که توسط فاساد `app` میتوانید به Container سرویس ها دسترسی داشته باشید .

#### ثبت سرویس در درون منبع سرویس ها
میتوانید بوسیله `$this->app` از درون Service Provider به منبع سرویس ها (service container)  دسترسی داشته باشید .
چندین روش مختلف برای ثبت کردن یک سرویس (dependence) در منبع سرویس ها (service container)  وجود دارد . روش اول تزریق Interface ها به دورن Implementation ها  است و روش  دوم ایجاد توابع Closure برای سرویس مورد نظر می باشد که در ابتدا این روش را بررسی خواهیم نمود  .
هر Closure با یک نام (معمولا نام کلاس سرویس مربوطه) درون منبع سرویس ها تعریف می شود و در زمانی که درخواست فراخوانی برای سرویس مورد نظر صادر میشود، Closure مربوطه اجرا و مقادیر آن برمیگردد  که این مقادیر عموما یک نمونه از کلاسی که سرویس مورد نظر را پیاده سازی میکند می باشد :

	$this->app->bind('FooBar', function($app)
	{
		return new FooBar($app['SomethingElse']);
	});

#### ثبت سرویس به صورت Singleton
ممکن است بخواهید سرویسی ایجاد کنید که تنها یک بار درون منبع سرویس ها (service container) لود شود و برای دفعات بعد همان نمونه ایجاد شده از سرویس مورد نظر برای شما باز گردد :

	$this->app->singleton('FooBar', function($app)
	{
		return new FooBar($app['SomethingElse']);
	});

#### ثبت یک نمونه ایجاد شده از کلاس به عنوان سرویس
 همچنین میتوانید یک نمونه ایجاد شده از یک کلاس را به وسیله متد  `instance`  در لیست سرویس های موجود ثبت کنید ، با اینکار هرزمان درخواست فراخوانی سرویس مورد نظر صادر شود ، نمونه کلاس ایجاد شده باز میگردد :

	$fooBar = new FooBar(new SomethingElse);

	$this->app->instance('FooBar', $fooBar);

### فراخوانی یک سرویس
روشهای مختلفی برای دسترسی به سرویس های موجود درون منبع وجود دارد ، عنوان اولین روشی میتوانید از متد  `make` استفاده نمایید :

	$fooBar = $this->app->make('FooBar');

دومین روش , دسترسی به صورت آرایه ای می باشد :

	$fooBar = $this->app['FooBar'];
و اما روش آخر اما مرسوم ترین روش ! ، شما به راحتی می توانید وابستگی ها (dependencies) را از طریق متد `constructor` کلاسی که در منبع سرویس ها (service container) ثبت شده است مانند  event listener ، controllers ، queue jobs و ... ،معرف نمایید . بنابراین سرویس مورد نظر بطور اتوماتیک در بدنه کلاس تزریق می شود :

	<?php namespace App\Http\Controllers;

	use Illuminate\Routing\Controller;
	use App\Users\Repository as UserRepository;

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

		/**
		 * Show the user with the given ID.
		 *
		 * @param  int  $id
		 * @return Response
		 */
		public function show($id)
		{
			//
		}

	}

<a name="binding-interfaces-to-implementations"></a>
## تزریق Interfaces ها به دورن Implementation ها

### تزریق وابستگی ها
یک ویژگی خوب Service Container توانایی تزریق Interface ها به درون Implementation ها می باشد . برای مثال ، ممکن است برنامه ما برای ارسال و دریافت لحظه ای event ها به سرویس [Pusher](https://pusher.com) وابستگی داشته باشد ،  اگر ما از Pusher SDK در پروژه مان استفاده کرده باشیم ، لاراول یک نمونه از کلاس Pusher  به درون کلاس تزریق میکند :

	<?php namespace App\Handlers\Commands;

	use App\Commands\CreateOrder;
	use Pusher\Client as PusherClient;

	class CreateOrderHandler {

		/**
		 * The Pusher SDK client instance.
		 */
		protected $pusher;

		/**
		 * Create a new order handler instance.
		 *
		 * @param  PusherClient  $pusher
		 * @return void
		 */
		public function __construct(PusherClient $pusher)
		{
			$this->pusher = $pusher;
		}

		/**
		 * Execute the given command.
		 *
		 * @param  CreateOrder  $command
		 * @return void
		 */
		public function execute(CreateOrder $command)
		{
			//
		}

	}
در مثال بالا ، تزریق وابستگی های کلاس یک عمل خوب می باشد ولی باعث شده  برای ارسال و دریافت اطلاعات به کامپوننت Pusher وابسته باشیم  و اگر بخواهیم بجای  سرویس Pusher از سرویس دیگری استفاده کنیم تنها کافی است کلاس `CreateOrderHandler` را تغییر دهیم .

### تغییر برنامه به Interface
به منظور حذف وابستگی میان  `CreateOrderHandler` و سرویس Pusher ( به این معنی که دیگر جایگزینی سرویس جدید کدهای این بخش را تحت تاثیر قرار ندهد) . یک interface با نام `EventPusher` و یک implementation با نام `PusherEventPusher`  ایجاد میکنم :

	<?php namespace App\Contracts;

	interface EventPusher {

		/**
		 * Push a new event to all clients.
		 *
		 * @param  string  $event
		 * @param  array  $data
		 * @return void
		 */
		public function push($event, array $data);

	}

بعد از نوشتن کدهای  `PusherEventPusher` ، میتوانیم آنرا در منبع سرویس ها(service container) ثبت کنیم :

	$this->app->bind('App\Contracts\EventPusher', 'App\Services\PusherEventPusher');

این دستور به Container میگوید هرزمان ما نیاز به اینترفیس `EventPusher` داشتیم آنگاه یک implementation از کلاس `PusherEventPusher` را تزریق کند . حالا میتوانیم اینترفیس `EventPusher` را از طریق متد سازنده کلاس تزریق (type-hint) نماییم :

		/**
		 * Create a new order handler instance.
		 *
		 * @param  EventPusher  $pusher
		 * @return void
		 */
		public function __construct(EventPusher $pusher)
		{
			$this->pusher = $pusher;
		}

<a name="contextual-binding"></a>
## تزریق شرطی (Contextual Binding)
کاهی اوقات شما دو کلاس دارید که همزمان از یک سرویس استفاده میکنند اما میخواهید لاراول برای یکی از آنها یک implementation متفاوت تزریق کند . برای مثال زمانی که سیستم سفارشی دریافت میکند بجای ارسال event ها بوسیله سرویس Pusher میخواهید event ها از طریق سرویس PubNub ارسال شوند . لاراول یک روش آسان برای انجام این کار دارد :

	$this->app->when('App\Handlers\Commands\CreateOrderHandler')
	          ->needs('App\Contracts\EventPusher')
	          ->give('App\Services\PubNubEventPusher');

<a name="tagging"></a>
## تگ کردن (Tagging)
ممکن است بخواهیم مجموعه ای از سرویس ها ، برای مثال درحال تولید گزارشی هستید که آرایه ای از سرویس های مختلف  را درخود استفاده میکند . بعد از ثبت در منبع سرویس ها (service container) میتوانید برای آنها از طریق متد `tag` یک تگ تعریف کنید :

	$this->app->bind('SpeedReport', function()
	{
		//
	});

	$this->app->bind('MemoryReport', function()
	{
		//
	});

	$this->app->tag(['SpeedReport', 'MemoryReport'], 'reports');
زمانیکه  سرویسها تگ شدند ، میتوانید آنها را بصورت یکجا از طریق متد `tagging` دریافت نمایید :

	$this->app->bind('ReportAggregator', function($app)
	{
		return new ReportAggregator($app->tagged('reports'));
	});

<a name="practical-applications"></a>
## استفاده عملی
لاراول امکان دسترسی به منبع سرویس ها به چندین روش به منظور انعطاف پذیری و تست پذیری بیشتر می دهد . یک مثال عمده در زمان فراخوانی کنترلرها (Controllers) می باشد. همه کنترلر ها در منبع سرویس ها ثبت و فراخوانی می شوند به این معنی که شما میتوانید تمامی وابستگی های (Dependencies) یک کنترلر از طریق متد سازنده (Constructor) آن تزریق (type-hint) نمایید .

	<?php namespace App\Http\Controllers;

	use Illuminate\Routing\Controller;
	use App\Repositories\OrderRepository;

	class OrdersController extends Controller {

		/**
		 * The order repository instance.
		 */
		protected $orders;

		/**
		 * Create a controller instance.
		 *
		 * @param  OrderRepository  $orders
		 * @return void
		 */
		public function __construct(OrderRepository $orders)
		{
			$this->orders = $orders;
		}

		/**
		 * Show all of the orders.
		 *
		 * @return Response
		 */
		public function index()
		{
			$orders = $this->orders->all();

			return view('orders', ['orders' => $orders]);
		}

	}


#### مثال دیگری از استفاده container
البته همانطور که اشاره کردیم کنترلر ها تنها قسمت هایی نیستند که از طریق منبع سرویس ها (service container) قابل دسترسند! شما میتوانید وابستگی های مورد نیاز دیگر مانند Filters,events,queue jobs و...  را به درون متد سازنده کنترلر (constructor)  یا  Closure  یک Route ، تزریق (type-hint) نمایید .

<a name="container-events"></a>
## رویداد ها (Container Events)

#### ایجاد یک Listener
هر زمان که درخواست فراخوانی یک سرویس از container داده میشود ، یک رویداد (event) به دنبال آن اجرا میگردد و شما می توانید از طریق متد `resolving` این event ها را معرفی نمایید :

	$this->app->resolving(function($object, $app)
	{
		// Called when container resolves object of any type...
	});

	$this->app->resolving(function(FooBar $fooBar, $app)
	{
		// Called when container resolves objects of type "FooBar"...
	});
شیء که فراخوانی می شود به عنوان آرگمان تابع event ارسال می شود.