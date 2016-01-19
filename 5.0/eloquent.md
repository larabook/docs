# Eloquent ORM

- [آشنایی](#introduction)
- [آغاز به کار](#basic-usage)
- [Mass Assignment](#mass-assignment)
- [درج، به روزرسانی، حذف - Insert, Update, Delete](#insert-update-delete)
- [حذف موقت - Soft Deleting](#soft-deleting)
- [تایم استمپها - Timestamps](#timestamps)
- [حوزه پرس و جوها - Query Scopes](#query-scopes)
- [حوزه های گلوبال - Global Scopes](#global-scopes)
- [رابطه ها - Relationships](#relationships)
- [پرس و جوی رابطه ها - Querying Relations](#querying-relations)
- [بارگذاری خوشبینانه - Eager Loading](#eager-loading)
- [درج مدلهای وابسته - Inserting Related Models](#inserting-related-models)
- [تغییر تایم استمپ پرنت - Touching Parent Timestamps](#touching-parent-timestamps)
- [کار با جدولهای پیوت - Working With Pivot Tables](#working-with-pivot-tables)
- [کالکشنها - Collections](#collections)
- [Accessors & Mutators](#accessors-and-mutators)
- [Date Mutators](#date-mutators)
- [Attribute Casting](#attribute-casting)
- [رویدادهای مدل - Model Events](#model-events)
- [Model Observers](#model-observers)
- [سازنده URL به مدل - Model URL Generation](#model-url-generation)
- [تبدیل به آرایه - Converting To Arrays / JSON](#converting-to-arrays-or-json)

<a name="introduction"></a>
## آشنایی

ابزار ORM ایلوکوئنت (Eloquent) که در کنار لاراول ارائه شده، پیاده سازی ساده و زیبایی از ActiveRecord برای کار با پایگاه داده دارد. در این پیاده سازی هر جدول از پایگاه داده دارای یک مدل است. از این مدل برای تعامل با جدول در پایگاه داده استفاده میشود.

پیش ازآغاز، باید کانکشن به پایگاه داده را در فایل `config/database.php` ایجاد نمایید.

<a name="basic-usage"></a>
## آغاز به کار

برای شروع، یک مدل ایلوکوئنت (Eloquent) بسازید. مدلها معمولا در دایرکتوری `app` قراردارند، اما میتوانید آنها را کجا که امکان بارگذاری خودکار (auto-load) وجود دارد قرار دهید. این موضوع را از طریق فایل `composer.json` میتوانید بررسی کنید. تمامی مدلهای ایلوکوئنت از `Illuminate\Database\Eloquent\Model` ارث بری میکنند.

#### تعریف یک مدل ایلوکوئنت

	class User extends Model {}

همچنین مدلهای Eloquent را میتوانید با استفاده از فرمان `make:model`  ایجاد نمایید:

	php artisan make:model User

توجه داشته باشید ما به Eloquent در مورد اینکه از کدام جدول برای مدل User استفاده کند چیزی نگفتیم. در این مواقع حالت جمع انگلیسی نام کلاس در برای اسم جدول استفاده میشود، اما میتوانید نام جدول را هم استفاده کنید. بنابراین در این مورد، Eloquent فرض را بر این میگذارید که مدل `User` رکوردها را در جدول `users` ذخیره میکند. در صورت متفاوت بودن نام مدل با نام جدول، آن را میتوان با استفاده از خصوصیت `table` بر روی مدل تعریف کرد.

	class User extends Model {

		protected $table = 'my_users';

	}

> **نکته:** پیش فرض دیگر Eloquent وجود یک ستون کلید اصلی (primary key) با نام `id` در هر جدول است. برای شکستن این قانون و استفاده از اسم متفاوت میتوانید از خصوصیت `primaryKey` استفاده کنید. همینطور برای تعریف نام اتصال پایگاه داده مربوط به مدل میتوانید خصوصیت `connection` را با مقدار مورد نظر خود مقداردهی کنید.

پس از تعریف مدل، آماده بازیابی و ایجاد رکورد در جدول خود هستید. توجه داشته باشید که باید در همه جدولها دو ستون `updated_at` و `created_at` را بیافزایید. اگر نمیخواهید Eloquent به طور خودکار این ستونها را مدیریت نماید، خصوصیت `$timestamps` را بر روی مدل خود با `false` مقداردهی کنید.

#### بازیابی تمامی رکوردها

	$users = User::all();

#### بازیابی یک رکورد با استفاده از کلید اصلی

	$user = User::find(1);

	var_dump($user->name);

> **نکته:** تمامی متدهایی که در [query builder](/docs/queries) استفاده می شوند در استفاده از مدلهای Eloquent هم در دسترس هستند.

#### بازیابی یک مدل با استفاده از کلید اصلی یا ایجاد exception

گاهی ممکن است بخواهید در صورت پیدا نشدن مدل مورد نظر exception ایجاد کنید. برای این کار، از متد `firstOrFail` استفاده کنید:

	$model = User::findOrFail(1);

	$model = User::where('votes', '>', 100)->firstOrFail();

این کار به شما کمک میکند exception را مدیریت نمایید و در صورت نیاز لاگ کنید و صفحه خطا نمایش دهید. برای مدیریت exception مربوط به `ModelNotFoundException` باید مقداری کد به فایل `app/Exceptions/Handler.php` بیافزایید.

	use Illuminate\Database\Eloquent\ModelNotFoundException;

	class Handler extends ExceptionHandler {

		public function render($request, Exception $e)
		{
			if ($e instanceof ModelNotFoundException)
			{
				// Custom logic for model not found...
			}

			return parent::render($request, $e);
		}

	}

#### پرس و جوی پایگاه داده با استفاده از مدلهای Eloquent

	$users = User::where('votes', '>', 100)->take(10)->get();

	foreach ($users as $user)
	{
		var_dump($user->name);
	}

#### توابع تجمعی Eloquent - Eloquent Aggregates

برای انجام عملیات تجمعی میتوانید از توابع تجمعی query builder استفاده کنید.

	$count = User::where('votes', '>', 100)->count();

اگر نمیتوانید پرس و جوی مورد نظرتان را با استفاده از اینترفیس fluent بسازید، به راحتی میتوانید از `whereRaw` استفاده کنید:

	$users = User::whereRaw('age > ? and votes = 100', [25])->get();

#### مدیریت حجم بالای نتایج با دستور Chunk

اگر نیاز به پردازش حجم بالای (چندهزار) رکورد Eloquent دارید، استفاده از دستور `chunk` امکان انجام این کار بدون اشغال تمامی حافظه میدهد:

	User::chunk(200, function($users)
	{
		foreach ($users as $user)
		{
			//
		}
	});

اولین آرگومان فرستاده شده به متد تعداد رکوردهایی است که میخواهید در هر بسته پردازشی دریافت کنید. کلوژری (Closure) که به عنوان آرگومان دوم فرستاده میشود در هر واکشی بسته پردازشی از پایگاه داده فراخوانی میشود.


#### مشخص کردن اتصال هر پرس و جو

امکان مشخص کردن کانکشن مربوط به پایگاه داده، برای اجرای یک پرس وجو وجود دارد. برای این کار به سادگی از متد `on` استفاده کنید:

	$user = User::on('connection-name')->find(1);

اگر از [کانکشنهای read/write](/docs/%7B%7Bversion%7D%7D/database#read-write-connections) استفاده میکنید، برای اجبار پرس و جو به استفاده از کانکشن "نوشتن - write" از متد زیر استفاده کنید:

	$user = User::onWriteConnection()->find(1);

<a name="mass-assignment"></a>
## انتصاب کلی - Mass Assignment

در زمان ایجاد یک مدل جدید، آرایه ای از خصوصیتها را به متد سازنده آن میفرستید. این خصوصیتها با استفاده از ویژگی انتصاب کلی به مدل انتصاب داده میشوند. این کار بسیار ساده است؛ هر چند اگر بدون کنترل ورودیهای کاربر به مدل منتقل شود میتواند یک مشکل امنیتی **جدی** ایجاد کند. اگر ورودیهای کاربر بدون کنترل به مدل منتقل شود، کاربر میتواند **تمامی** یا **هر کدام** از خصوصیتهای مدل را تغییر دهد. به این دلیل، تمامی مدلهای Eloquent در حالت پیش فرض در مقابل انتصاب کلی محافظت شده اند.

برای شروع، مقادیر خصوصیتهای `fillable` یا `guarded` را در مدل مقدار دهی نمایید.

#### تعریف خصوصیات با امکان مقداردهی (fillable) در مدل

خصوصیت `fillable` مشخص میکند کدام خصوصیتها را میتوان در انتصاب کلی مقداردهی کرد. این کار را میتوان در سطح کلاس یا نمونه کلاس انجام داد.

	class User extends Model {

		protected $fillable = ['first_name', 'last_name', 'email'];

	}

در این مثال، تنها سه خصوصیت لیست شده، در انتصاب کلی مقداردهی میشوند.

#### تعریف خصوصیتهای محافظت شده (guarded) در مدل

`guarded` برعکس `fillable` است، و به عنوان یک "لیست سیاه" عمل میکند:

	class User extends Model {

		protected $guarded = ['id', 'password'];

	}

> **نکته:** وقتی از `guarded` استفاده میکنید، هنوز نباید مقدار `Input::get()` یا آرایه ای از ورودیهایی که تنها توسط کاربر کنترل شده اند را به متدهای `save` یا `update` بفرستید، چراکه هر ستونی که محافظت نشده باشد ممکن است به روزرسانی شود.

#### بلاک کردن تمامی خصوصیتها از انتصاب کلی

درمثال بالا، خصوصیتهای `id` و `password` به شکل کلی مقداردهی **نمیشوند**. تمامی خصوصیات دیگر میتوانند به صورت کلی انتصاب مقدار شوند. شما میتوانید **تمامی** خصوصیتها را با استفاده از خصوصیت `guarded` از انتصاب کلی محافظت نمایید:

	protected $guarded = ['*'];

<a name="insert-update-delete"></a>
## درج، به روزرسانی، حذف - Insert, Update, Delete

برای ایجاد یک رکورد جدید در پایگاه داده از یک مدل، کافیست یک نمونه از آن مدل بسازید و متد `save` را فراخوانی کنید:

#### ذخیره یک مدل جدید

	$user = new User;

	$user->name = 'John';

	$user->save();

> **نکته:** معمولا مدلهای Eloquent کلیدهای auto-increment دارند. هر چند اگر بخواهید کلیدهای خود را معرفی کنید، خصوصیت `incrementing` را در مدل موردنظر با `false` مقداردهی کنید.

همچنین میتوانید در یک خط از متد `create` برای ذخیره مدل استفاده نمایید. مدل ذخیره شده، به عنوان خروجی متد قابل دسترس است. پیش از انجام این کار باید خصوصیت `fillable` یا `guarded` را برای مدل مقداردهی کنید، چرا که تمامی مدلهای Eloquet در مقابل انتصاب کلی محافظت شده هستند.

پس از ذخیره یا ایجاد یک مدل جدید که از IDهای auto-increment استفاده میکنند، میتوانید با استفاده از خصوصیت `id` شی به شناسه ID آن دست یابید:

	$insertedId = $user->id;

#### مقداردهی خصوصیت guarded برای یک مدل

	class User extends Model {

		protected $guarded = ['id', 'account_id'];

	}

#### استفاده از متد create مدل

	// یک کاربر جدید در پایگاه داده ایجاد میکند...
	$user = User::create(['name' => 'John']);

	// اگر کاربری با این خصوصیات وجود داشته باشد، آن را باز میگرداند و در غیر اینصورت آن را ایجاد میکند...
	$user = User::firstOrCreate(['name' => 'John']);

	// در صورت وجود کاربری با خصوصیات ارائه شده آن را باز میگرداند و در غیر اینصورت یک مدل نمونه چدید میسازد...
	$user = User::firstOrNew(['name' => 'John']);

#### به روزرسانی یک مدل واکشی شده

برای به روزرسانی یک مدل، میتوانید آن را واکشی کرده، خصوصیات آن را تغییر دهید و از متد `save` استفاده کنید:

	$user = User::find(1);

	$user->email = 'john@foo.com';

	$user->save();

#### ذخیره یک مدل و ارتباطها

گاهی میخواهید به همراه یک مدل مدلهایی که با آنها در ارتباط هستند را هم ذخیره کنید، برای این کار از متد `push` استفاده کنید:

	$user->push();

همچنین میتوانید به روزسانی را در قالب پرس و جویی که بر روی مجموعه ای از مدلها انجام میدهید اجرا نمایید:

	$affectedRows = User::where('votes', '>', 100)->update(['status' => 2]);

> **نکته:** هنگام به روز رسانی مجموعه ای از مدلها با استفاده از پرس و جو ساز Eloquent هیچ یک از رویدادهای مدل فراخوانی نمیشوند.

#### حذف یک مدل

برای حذف یک مدل، کافیست متد `delete` را بر روی نمونه مدل اجرا کنید:

	$user = User::find(1);

	$user->delete();

#### حذف یک مدل با استفاده از کلید

	User::destroy(1);

	User::destroy([1, 2, 3]);

	User::destroy(1, 2, 3);

همچنین میتوانید دستور حذف را بر روی مجموعه ای مدلها هم اجرا کنید:

	$affectedRows = User::where('votes', '>', 100)->delete();

#### به روزرسانی timestamp مربوط به مدلها

اگر تنها میخواهید timestamp های مربوط به یک مدل را به روزرسانی کنید، میتوانید از متد `touch` استفاده کنید:

	$user->touch();

<a name="soft-deleting"></a>
## حذف موقت - Soft Deleting

حذف موقت در حقیقت باعث حذف آن از پایگاه داده نمیشود. این کار باعث مقداردهی تایم استمپ `deleted_at` بر روی رکورد میشود. برای ایجاد امکان حذف موقت برای یک مدل، `SoftDeletes` را بر روی مدل اعمال نمایید:

	use Illuminate\Database\Eloquent\SoftDeletes;

	class User extends Model {

		use SoftDeletes;

		protected $dates = ['deleted_at'];

	}

برای اضاف کردن ستون `deleted_at` به جدولتان، از متد `softDeletes` در migration استفاده کنید:

	$table->softDeletes();

حال هنگامی که متد `delete` بر روی مدل فراخوانی شود، ستون `deleted_at` به مقدار timestamp فعلی مقداردهی میشود. هنگام پرس و جوی یک مدل که از حذف موقت استفاده میکند، مدلهای "حذف شده" در نتیجه پرس و جو در نظر گرفته نمی شوند.

#### اجبار برای در نظر گرفتن مدلهای حذف موقت شده در نتایج

برای اینکه مدلهایی که حذف موقت شده اند را هم در مجموعه نتایج داشته باشید، از متد `withTrashed` بر روی پرس و جو استفاده کنید:

	$users = User::withTrashed()->where('account_id', 1)->get();

متد `withTrashed` را بر روی یک رابطه تعریف شده میتوان استفاده نمود:

	$user->posts()->withTrashed()->get();

اگر میخواهید **تنها** مدلهایی که حذف موقت شده اند را در نتایج خود داشته باشید، میتوانید از متد `onlyTrashed` استفاده نمایید:

	$users = User::onlyTrashed()->where('account_id', 1)->get();

برای بازگرداندن یک مدل موقتا حذف شده به حالت فعال، از متد `restore` استفاده کنید:

	$user->restore();

همچنین میتوانید از متد `restore` بر روی یک پرس و جو استفاده کنید:

	User::withTrashed()->where('account_id', 1)->restore();

مانند `withTrashed`، متد `restore` نیز میتواند بر روی یک رابطه استفاده شود:

	$user->posts()->restore();

اگر میخواهید واقعا یک مدل را از پایگاه داده حذف کنید، میتوانید از متد `forceDelete` استفاده کنید:

	$user->forceDelete();

متد `forceDelete` بر روی رابطه ها هم کار میکند:

	$user->posts()->forceDelete();

برای تعیین حذف موقت شدن یک مدل میتوانید از متد `trashed` استفاده کنید:

	if ($user->trashed())
	{
		//
	}

<a name="timestamps"></a>
## تایم استمپ - Timestamps

به صورت پیش فرض، Eloquent ستونهای `created_at` و `updated_at` را بر روی جدول پایگاه داده مدیریت میکند. برای این منظور به سادگی  ستونها را به جدول خود بیافزایید و لاراول بقیه کارها را مدیریت میکند. اگر نمیخواهید Eloquent این مقادیر را نگاه دارد، خصوصیت زیر را به مدلتان بیافزایید:

#### غیرفعال کردن تایم استمپ خودکار

	class User extends Model {

		protected $table = 'users';

		public $timestamps = false;

	}

#### ارائه یک timesatmp اختصاصی

اگر میخواهید قالب timestamp را تغییر دهید، میتوانید متد `getDateFormat` را در مدلتان بازنویسی کنید:

	class User extends Model {

		protected function getDateFormat()
		{
			return 'U';
		}

	}

<a name="query-scopes"></a>
## حوزه های پرس و جو

#### تعریف حوزه یک پرس و جو - Define a Query Scope

حوزه ها (scope) امکان استفاده مجدد منطق پرس و جو را در مدلها فراهم میکنند. برای تعریف scope کافیست عبارت `scope` پیش از متد موردنظر در مدل اضاف شود.

	class User extends Model {

		public function scopePopular($query)
		{
			return $query->where('votes', '>', 100);
		}

		public function scopeWomen($query)
		{
			return $query->whereGender('W');
		}

	}

#### استفاده از scope پرس و جو

	$users = User::popular()->women()->orderBy('created_at')->get();

#### scopeهای پویا

گاهی نیاز به تعریف حوزه هایی با امکان دریافت پارامتر داریم. تنها پارامترهایتان را به تابع scope بیافزایید:

	class User extends Model {

		public function scopeOfType($query, $type)
		{
			return $query->whereType($type);
		}

	}

آنگاه پارامتر را هنگام فراخوانی به scope بفرستید:

	$users = User::ofType('member')->get();

<a name="global-scopes"></a>
## scopeهای گلوبال - Global Scopes

گاهی نیاز به تعریف scopeهای برای پوشش تمام پرس و جوهایی است که بر روی یک مدل اجرا میشوند. ویژگی "حذف موقت" Eloquent به این شکل کار میکند. scopeهای گلوبال با ترکیبی از PHP trait و یک پیاده سازی از `Illuminate\Database\Eloquent\ScopeInterface` تعریف شده اند.

ابتدا اجازه دهید trait را تعریف کنیم. برای این مثال، از `SoftDeletes` که در لاراول قرار دارد استفاده میکنیم:

	trait SoftDeletes {

		/**
		 * Boot the soft deleting trait for a model.
		 *
		 * @return void
		 */
		public static function bootSoftDeletes()
		{
			static::addGlobalScope(new SoftDeletingScope);
		}

	}

اگر یک مدل Eloquent از یک trait با متدی که از قوانین نامگذاری `bootNameOfTrait` استفاده میکند، آن متد trait هنگام بوت مدل Eloquent فراخوانی میشود و امکان ثبت یک scope گلوبال یا انجام هر کاری دیگری را فراهم میکند. یک scope باید اینترفیس `ScopeInterface`، با دو متد `remove` و `apply` را پیاده سازی نماید.

متد `apply` یک شی کوئری ساز `Illuminate\Database\Eloquent\Builder` و `Model`ی که باید بر روی آن به کارگرفته شود را دریافت میکند و مسئول افزودن هر تعداد عبارت `where` است که scope اضاف میکند. متد `remove` هم یک شی `Builder` و `Model` دریافت میکند، و مسئول عکس کردن عملیات انجام شده به وسیله `apply` است، به بیانی دیگر، `remove` باید اولین عبارت `where` (یا هر عبارت دیگر) افزوده شده را حذف نماید. بنابراین برای `SoftDeletingScope`، متدهای ما چیزی شبیه موارد زیر است:

	/**
	 * Apply the scope to a given Eloquent query builder.
	 *
	 * @param  \Illuminate\Database\Eloquent\Builder  $builder
	 * @param  \Illuminate\Database\Eloquent\Model  $model
	 * @return void
	 */
	public function apply(Builder $builder, Model $model)
	{
		$builder->whereNull($model->getQualifiedDeletedAtColumn());

		$this->extend($builder);
	}

	/**
	 * Remove the scope from the given Eloquent query builder.
	 *
	 * @param  \Illuminate\Database\Eloquent\Builder  $builder
	 * @param  \Illuminate\Database\Eloquent\Model  $model
	 * @return void
	 */
	public function remove(Builder $builder, Model $model)
	{
		$column = $model->getQualifiedDeletedAtColumn();

		$query = $builder->getQuery();

		foreach ((array) $query->wheres as $key => $where)
		{
			// If the where clause is a soft delete date constraint, we will remove it from
			// the query and reset the keys on the wheres. This allows this developer to
			// include deleted model in a relationship result set that is lazy loaded.
			if ($this->isSoftDeleteConstraint($where, $column))
			{
				unset($query->wheres[$key]);

				$query->wheres = array_values($query->wheres);
			}
		}
	}

<a name="relationships"></a>
## ارتباطها - Relationships

مسلما جدولهای پایگاه داده شما با هم در ارتباط هستند. برای مثال، یک پست میتواند تعدادی کامنت داشته باشد، یا یک سفارش با کاربری که آن را ایجاد کرده در ارتباط باشد. Eloquent کار با و مدیریت این ارتباطها را ساده میکند. لاراول انواع رابطه ها را پشتیبانی میکند:

- [یک به یک - One To One](#one-to-one)
- [یک به چند - One To Many](#one-to-many)
- [چند به چند - Many To Many](#many-to-many)
- [ارتباط با چند با واسطه - Has Many Through](#has-many-through)
- [ارتباطهای پلی مرفیک - Polymorphic Relations](#polymorphic-relations)
- [ارتباطهای پلی مرفیک چند به چند - Many To Many Polymorphic Relations](#many-to-many-polymorphic-relations)

<a name="one-to-one"></a>
### یک به یک - One To One

#### تعریف یک رابطه یک به یک

یک رابطه یک به یک بسیار ابتدایی است. برای مثال، یک مدل `User` میتواند یک `Phone` داشته باشد. میتوان این رابطه را در Eloquent تعریف کرد.

	class User extends Model {

		public function phone()
		{
			return $this->hasOne('App\Phone');
		}

	}
	
اولین آرگومان فرستاده شده به متد `hasOne` نام مدل مربوطه است. هنگامی که رابطه تعریف شد، میتوانیم با استفاده از [خصوصیات پویا](%28#dynamic-properties%29) در Eloquent به آن دسترسی داشته باشیم.

	$phone = User::find(1)->phone;

عبارت SQL اجرا شده توسط این عبارت در ادامه ارائه شده است:

	select * from users where id = 1

	select * from phones where user_id = 1

به این نکته توجه داشته باشید که Eloquent نام کلید خارجی رابطه را براساس نام مدل مربوطه در نظر میگیرد. در این حالت، مدل `Phone` باید دارای یک کلید خارجی با نام `user_id` باشد. اگر میخواهید از نام دیگری استفاده کنید، و این قاعده را بشکنید، میتوانید آن را به عنوان آرگومان دوم به متد `hasOne` بفرستید. علاوه بر آن، میتوانید آرگومان سومی را برای تعیین کلید محلی که در رابطه استفاده میشود به متد بفرستید:

	return $this->hasOne('App\Phone', 'foreign_key');

	return $this->hasOne('App\Phone', 'foreign_key', 'local_key');

#### تعریف طرف دوم یک رابطه

برای تعریف دوم یک رابطه بر روی مدل `Phone`، میتوانید ازمتد `belongsTo` استفاده کنید:

	class Phone extends Model {

		public function user()
		{
			return $this->belongsTo('App\User');
		}

	}

در مثال بالا، Eloquent در جدول `phones` به دنبال ستون `user_id` میگردد. اگر میخواهید یک ستون کلید خارجی دیگر تعریف کنید، میتوانید آن را به عنوان آرگومان دوم به متد `belongsTo` بفرستید:

	class Phone extends Model {

		public function user()
		{
			return $this->belongsTo('App\User', 'local_key');
		}

	}

همچنین برای مشخص کردن ستون رابطه نام آن را به عنوان آرگومان سوم به متد بفرستید:

	class Phone extends Model {

		public function user()
		{
			return $this->belongsTo('App\User', 'local_key', 'parent_key');
		}

	}

<a name="one-to-many"></a>
### یک به چند - One To Many

مثال رابطه یک به چند پست بلاگی است که چندین کامنت دارد. میتوانیم این رابطه را به شکل زیر تعریف کنیم: 

	class Post extends Model {

		public function comments()
		{
			return $this->hasMany('App\Comment');
		}

	}

حال میتوان از طریق [خصوصیتهای پویا](#dynamic-properties%29) به کامنتهای پست دسترسی داشت:

	$comments = Post::find(1)->comments;

اگر میخواهید شرطهایی برای محدود کردن کامنتهای واکشی شده قرار دهید، میتوانید از متد `comments` استفاده کنید و در ادامه شرط تعریف نمایید (chaining):

	$comments = Post::find(1)->comments()->where('title', '=', 'foo')->first();

مجددا اشاره میکنیم که میتوانید با فرستادن نام مورد نظر به عنوان آرگومان دوم به متد `hasMany` قرارداد پیش فرض Eloquent را تغییر دهید. و همانند رابطه `hasOne`، ستون محلی را هم میتوان مشخص نمود:

	return $this->hasMany('App\Comment', 'foreign_key');

	return $this->hasMany('App\Comment', 'foreign_key', 'local_key');

#### تعریف طرف دوم یک رابطه

برای تعریف طرف دوم رابطه در مدل `comment`، از متد `belongsTo` استفاده میکنیم:

	class Comment extends Model {

		public function post()
		{
			return $this->belongsTo('App\Post');
		}

	}

<a name="many-to-many"></a>
### چند به چند

ارتباطهای چند به چند پیچیده ترین نوع رابطه هستند. یک مثال از این نوع ارتباط یک کاربر با چندین نقش است، که درآن هر نقش میتواند به افراد مختلفی داده شود. برای مثال کاربران زیادی میتوانند نقش "ادمین" را داشته باشند. برای این رابطه به سه جدول درپایگاه داده احتیاج است: `roles` ،`users`، و `role_user`. جدول `role_user` از کنار هم قرارگرفتن نام جداول با ترتیب الفبایی نام آنها گرفته شده است، و باید دو ستون `user_id` و `role_id` داشته باشد.

ارتباط چند به چند را از طریق متد `belongsToMany`میتوان تعریف نمود:

	class User extends Model {

		public function roles()
		{
			return $this->belongsToMany('App\Role');
		}

	}

حال میتوانیم نقشها را با استفاده از مدل `User` به دست آوریم:

	$roles = User::find(1)->roles;

اگر میخواهید برای جدول واسطه نامی غیررایج را استفاده کنید، میتوانید آن را به عنوان دومین پارامتر به متد `belongToMany` بفرستید:

	return $this->belongsToMany('App\Role', 'user_roles');

همچنین میتوانید قانون کلیدهای رابطه را هم بازنویسی کنید:

	return $this->belongsToMany('App\Role', 'user_roles', 'user_id', 'foo_id');

همچنین میتوانید طرف دیگر را بطه را هم بر روی مدل `Role` تعریف کنید:

	class Role extends Model {

		public function users()
		{
			return $this->belongsToMany('App\User');
		}

	}

<a name="has-many-through"></a>
### دارای ارتباط چندتایی با واسطه

ارتباط چند با واسطه یک مسیرکوتاه برای دستیابی به مدلهایی که در عمق یک رابطه هستند  با استفاده از یک رابطه میانی است. برای مثال، یک مدل `Country` میتواند با واسطه مدل `User` چندین `Post` داشته باشد. جدول این رابطه به شکل زیر است:

	countries
		id - integer
		name - string

	users
		id - integer
		country_id - integer
		name - string

	posts
		id - integer
		user_id - integer
		title - string

هر چند جدول `posts` ستونی برای `country_id` ندارد، ارتباط `hasManyThrough` امکان دسترسی به پستهایی که یک کاربر برای یک مدل `Country` نوشته است از طریق `$country->posts` به وجود میآید.

	class Country extends Model {

		public function posts()
		{
			return $this->hasManyThrough('App\Post', 'App\User');
		}

	}
	
اگر می خواهید کلید رابطه را دستی خودتان مشخص نمایید، میتوانید آنها را به عنوان آرگومانهای سوم و چهارم به متد بفرستید:

	class Country extends Model {

		public function posts()
		{
			return $this->hasManyThrough('App\Post', 'App\User', 'country_id', 'user_id');
		}

	}

<a name="polymorphic-relations"></a>
### ارتباطهای پلی مرفیک - Polymorphic Relations

ارتباطهای پلی مرفیک امکان ارتباط یک مدل با چندین مدل دیگر، بر روی یک رابطه، را میدهند. برای مثال، ممکن است یک مدل `Photo` داشته باشید که به هر دو مدل `Staff` و `Order` تعلق داشته باشند. این رابطه را به شکل زیر تعریف میکنیم:

	class Photo extends Model {

		public function imageable()
		{
			return $this->morphTo();
		}

	}

	class Staff extends Model {

		public function photos()
		{
			return $this->morphMany('App\Photo', 'imageable');
		}

	}

	class Order extends Model {

		public function photos()
		{
			return $this->morphMany('App\Photo', 'imageable');
		}

	}

#### واکشی یک رابطه پلی مرفیک

حال میتوانیم عکسهای هر یک از مدلهای کارکنان یا درخواستها را واکشی نماییم: 

    $staff = Staff::find(1);

	foreach ($staff->photos as $photo)
	{
		//
	}

#### واکشی صاحب یک را یطه پلی مرفیک

قدرت واقعی "پلی مرفیک" زمانی خود را نشان میدهد که میخواهدی به `staff` یا `order` از طریق مدل `Photo` دسترسی داشته باشید:

	$photo = Photo::find(1);

	$imageable = $photo->imageable;

رابطه `imageable` بر روی مدل `Photo`، نمونه های کلاسهای مدل `Staff` یا `Order` رو با توجه به اینکه تصویر مربوط به کدام مدل هست واکشی میکنه.

#### ساختار جدول ارتباط  پلی مرفیک

برای درک نحوه کار این رابطه، بیایید ساختار پایگاه داده را برای ارتباط پلی مرفیک بررسی کنیم:

	staff
		id - integer
		name - string

	orders
		id - integer
		price - integer

	photos
		id - integer
		path - string
		imageable_id - integer
		imageable_type - string

فیلدهای کلید مورد توجه بر روی جدول `imageable_id`، `photos` و `Imageable_type` هستند. ID مقدار شناسه، در این مثال، staff یا order مربوطه است، و type نام کلاس مدل مربوطه را نگه میدارد. این به ORM کمک میکند هنگام دسترسی به رابطه `imageable` کدام مدل را بازگرداند.

<a name="many-to-many-polymorphic-relations"></a>
### ارتباطهای پلی مرفیک چند به چند

#### ساختار جدول ارتباطهای پلی مرفیک چند به چند

علاوه بر رابطه های پلی مرفیک پیشین، میتوانید رابطه هی پلی مرفیک چند به چند هم تعریف نمایید. برای مثال مدل پست `Post` و ویدئو `Video` در یک بلاگ میتوانند یک رابطه پلی مرفیک با مدل `Tag` اشته باشند. ساختار جدول این رابطه به این شکل تعریف میشود:

	posts
		id - integer
		name - string

	videos
		id - integer
		name - string

	tags
		id - integer
		name - string

	taggables
		tag_id - integer
		taggable_id - integer
		taggable_type - string

در قدم بعد رابطه را بر روی مدل تعریف میکنیم. مدلهای `Post` و `Video` هر دو یک رابطه `morphToMany` با استفاده از متد `tags` دارند:

	class Post extends Model {

		public function tags()
		{
			return $this->morphToMany('App\Tag', 'taggable');
		}

	}

مدل `Tag` برای هر یک از رابطه های خود یک متد تعریف میکند.

	class Tag extends Model {

		public function posts()
		{
			return $this->morphedByMany('App\Post', 'taggable');
		}

		public function videos()
		{
			return $this->morphedByMany('App\Video', 'taggable');
		}

	}

<a name="querying-relations"></a>
## پرس و جو بر روی رابطه ها

#### پرس و جو بر روی رابطه ها هنگام انتخاب

هنگام دسترسی به رکوردهای یک مدل، میتوانید براساس وجود یک رابطه نتایج را محدود نمایید. برای مثال، میتوانید تمامی پستهای بلاگی که دست کم یک کامنت دارند را بازگردانید. برای این کار، میتوانید از متد `has` استفاده کنید:

	$posts = Post::has('comments')->get();

همچنین میتوانید یک اپاتور و تعداد هم مشخص نمایید:

	$posts = Post::has('comments', '>=', 3)->get();

متدهای `has` تودرتو را هم میتوانید با استفاده از "." بسازید:

	$posts = Post::has('comments.votes')->get();

اگر قدرت بیشتری میخواهید، از متدهای `whereHas` و `orWhereHas` برای شرطهای "where" بر روی پرس و جوهای `has` استفاده کنید:

	$posts = Post::whereHas('comments', function($q)
	{
		$q->where('content', 'like', 'foo%');

	})->get();

<a name="dynamic-properties"></a>
### خصوصیتهای پویا

Eloquent امکان دسترسی به رابطه هایتان با استفاده از خصوصیتهای پویا را ایجاد نموده است. Eloquent به طور خودکار رابطه ها را بارگذاری میکند، و به اندازه کافی هوشمند است تا بداند متد `get` را فراخوانی کند (برای رابطه های یک به چند) یا `first` (برای رابطه های یک به یک). سپس با استفاده از یک خصوصیت پویا با همنام رابطه قابل دسترس خواهد بود. برای مثال، در برای مدل `$phone`:

	class Phone extends Model {

		public function user()
		{
			return $this->belongsTo('App\User');
		}

	}

	$phone = Phone::find(1);

به جای اکو کردن ایمیل کاربر به شکل زیر:

	echo $phone->user()->first()->email;

میتوان آن به شکل زیر کوتاه نمود:

	echo $phone->user->email;

> **نکته:** رابطه هایی که نتیجه های زیادی برمیگردانند، نمونه ای از کلاس `Illuminate\Database\Eloquent\Collection` را میسازند.

<a name="eager-loading"></a>
## بارگذاری زودهنگام - Eager Loading

بارگذاری زودهنگام برای پاسخ به مشکل پرس و جوی N + 1 به وجود آمد. برای مثال، یک مدل `Book` مرتبط با مدل `Author` را در نظر بگیرید. رابطه به این شکل تعریف می شود.

	class Book extends Model {

		public function author()
		{
			return $this->belongsTo('App\Author');
		}

	}

حال کد زیر را در نظر بگیرید:

	foreach (Book::all() as $book)
	{
		echo $book->author->name;
	}

این حلقه برای واکشی تمام کتابهای این جدول یک پرس و جو اجرا میکند، و آنگاه یک پرس و جو برای واکشی نویسنده هر کتاب. بنابراین اگر 25 کتاب داریم، 26 پرس و جو اجرا می شود:

خوشبختانه، میتوانیم با استفاده از بارگذاری زودهنگام به شدت تعداد پرس و جوهای اجرا شده را کاهش دهیم. رابطه هایی که باید با استفاده از بارگذاری زودهنگام واکشی شوند را با استفاده از متد `with` مشخص میکنیم:

	foreach (Book::with('author')->get() as $book)
	{
		echo $book->author->name;
	}

در حلقه بالا، تنها دو پرس و جو اجرا می شوند:

	select * from books

	select * from authors where id in (1, 2, 3, 4, 5, ...)

استفاده هوشمندانه از بارگذاری زودهنگام کارایی نرم افزار را به شدت افزایش میدهد:

همچنین میتوانید چندین رابطه را همزمان بارگذاری زودهنگام نمایید:

	$books = Book::with('author', 'publisher')->get();

رابطه های تودرتو را هم میتوانید بارگذاری زودهنگام نمایید:

	$books = Book::with('author.contacts')->get();

در مثال بالا، رابطه `author` زودهنگام بارگذاری میشود، و رابطه `contacts` از نویسندگان هم بارگذاری میشود.

### محدودیتهای بر روی بارگذاری زودهنگام

گاهی ممکن ات بخواهید بر روی بارگذاری زودهنگام محدودیتهایی تعیین نمایید. در ادامه مثالی را میبینید:

	$users = User::with(['posts' => function($query)
	{
		$query->where('title', 'like', '%first%');

	}])->get();
	
در این مثال، ما پستهای کاربر را با استفاده از بارگذاری زودهنام بارگذاری میکنیم، اما شرط محدودیت اینست که عنوان پست شامل کلمه "first" باشد:

البته کلوژرهای بارگذاری زودهنگام به شرطها محدود نیستند. میتوان مرتب سازی هم استفاده نمود:

	$users = User::with(['posts' => function($query)
	{
		$query->orderBy('created_at', 'desc');

	}])->get();

### بارگذاری زودهنگام به شرط نیاز - Lazy Eager Loading

همچنین امکان بارگذاری زودهنگام مدلهای رابطه با استفاده از یک مجموعه مدل موجود وجود دارد. این موضوع زمانی اهمییت پیدا میکند که به صورت پویا تصمیم میگریم مدلهای رابطه واکشی شوند یا خیر، یا در ترکیب با کش:

	$books = Book::all();

	$books->load('author', 'publisher');

همچنین برای اعمال شرط بر روی پرس و جو میتوانید کلوژر به متد بفرستید:

	$books->load(['author' => function($query)
	{
		$query->orderBy('published_date', 'asc');
	}]);

<a name="inserting-related-models"></a>
## درج مدلهای یک رابطه

#### Attaching A Related Model

گاهی نیاز به درج مدلهای وابسته وجود دارد. برای مثال، درج کامنتی برای یک پست. به جای اینکه کلید خارجی `post_id` مدل را دستی مقداردهی کنید، میتوانید کامنت جدید را با استفاده از مدل `Post` پدر مشتقیما درج کنید:

	$comment = new Comment(['message' => 'A new comment.']);

	$post = Post::find(1);

	$comment = $post->comments()->save($comment);

در این مثال، فیلد `post_id` خودکار بر روی کامنت درج شده مقداردهی میشود.

اگر بخواهید چند مدل مرتبط را ذخیره کنید:

	$comments = [
		new Comment(['message' => 'A new comment.']),
		new Comment(['message' => 'Another comment.']),
		new Comment(['message' => 'The latest comment.'])
	];

	$post = Post::find(1);

	$post->comments()->saveMany($comments);

### مرتبط کردن مدل ها (BelongsTo)

هنگام به روزرسانی یک رابطه `belongsTo`، میتوانید از متد `associate` استفاده کنید. این متد کلیدخارجی را بر روی مدل فرزند مقداردهی میکند:


	$account = Account::find(10);

	$user->account()->associate($account);

	$user->save();

### درج مدلهای وابسته (ارتباط چند به چند)

مدلهای رابطه چند به چند را هم میتوانید درج کنید. باز از مدلهای `User` و `Role` به عنوان مثال استفاده میکنیم. به راحتی میتوانیم نقشهای جدید را با استفاده از متد `attach` به کاربر اضاف کنیم:

#### نصب  مدلهای رابطه چند به چند

	$user = User::find(1);

	$user->roles()->attach(1);
	
همچنین میتوانید آرایه ای از خصوصیتها را برای ذخیره بر روی جدول رابطه به عنوان آرگومان به متد بفرستید:

	$user->roles()->attach(1, ['expires' => $expires]);

البته معکوس `attach` عملیات `detach` است:

	$user->roles()->detach(1);

هر دو متد `attach ` و `detach` آراریه ای شناسه ها را به عنوان ورودی دریافت میکنند:

	$user = User::find(1);

	$user->roles()->detach([1, 2, 3]);

	$user->roles()->attach([1 => ['attribute1' => 'value1'], 2, 3]);

#### استفاده از Sync برای ذخیره (attach) مدلهای چند به چند

میتوانید از متد `sync` برای ذخیره (attach) مدلهای وابسته استفاده کرد. متد `sync` آرایه ای از شناسه های جدول پیوت را دریافت میکند. پس از کامل شدن این عملیات، تنها شناسه های موجود در آرایه بر روی جدول میانی مدلها خواهند بود:

	$user->roles()->sync([1, 2, 3]);

#### افزودن داده های پیوت در زمان Sync

همچنین میتوانید برای شناسه های مورد نظر مقادیر دیگری را بر روی جدول پیوت مقداردهی کنید:

	$user->roles()->sync([1 => ['expires' => true]]);

گاهی میخواهید یک مدل وابسته ایجاد کنید و در همان خط آن را attach کنید. برای این کار، میتوانید از متد `save` استفاده کنید:

	$role = new Role(['name' => 'Editor']);

	User::find(1)->roles()->save($role);

در این مثال، مدل جدید `Role` ذخیره می شود و به مدل کاربر `User`  نیز Attach می شود. همچنین میتوانید آرایه ای از خصوصیات را برای قراردادن بر روی جدول میانی در این عملیات به متتد بفرستید:

	User::find(1)->roles()->save($role, ['expires' => $expires]);

<a name="touching-parent-timestamps"></a>
## تغییر تایم استمپ جدول پدر

وقتی مدلی به مدل دیگر تعلق دارد (`belongsTo`)، مانند `Comment` که متعلق به `Post` است. به روزرسانی تایم استمپ پدر پس از به روزرسانی فرزند میتواند مفید باشد. برای مثال، هنگامی که یک مدل `Comment` به روزرسانی می شود، میتوانید فیلد  `updated_at` مدل `Post` پدر آن را هم خودکار به روزرسانی کنید. Eloquent این کار را ساده میکند. تنها کافیست یک خصوصیت `touches` با مقدار نام ارتباط فرزند ایجاد نمایید:

	class Comment extends Model {

		protected $touches = ['post'];

		public function post()
		{
			return $this->belongsTo('App\Post');
		}

	}

حال با به روزرسانی یک `Comment`، فیلد `updated_at` مدل `Post` مربوطه هم به روزرسانی می شود:

	$comment = Comment::find(1);

	$comment->text = 'Edit to this comment!';

	$comment->save();

<a name="working-with-pivot-tables"></a>
## کار با جدولهای پیوت

همانطورکه تا  به حال آموخته اید، کار با رابطه های چند به چند نیازمند یک جداول میانی است. Eloquent راههای کمکی بسیاری برای کار با این جدول ارائه مینماید. برای مثال، فرض کنید شی `User` دارای چندین شی مرتبط `Role` باشد. پس از دسترسی به این رابطه، میتوانیم به جدول میانی یا پیوت دسترسی داشته باشیم:

	$user = User::find(1);

	foreach ($user->roles as $role)
	{
		echo $role->pivot->created_at;
	}

توجه داشته باشید که هر مدل `Role` که واکشی می شود خودکار دارای یک خصوصیت `pivot` است. این خصوصیت دربرگیرنده یک شی مدل از جنس جدول میانی است، و میتواند توسط هر مدل Eloquent دیگر استفاده شود:

به صورت پیش فرض تنها کلیدها برا روی شی پیوت قرار دارند. اگر جدول پیوت خصوصیتهای دیگری هم دارد، باید هنگام تعریف رابطه آنها را مشخص نمایید:

	return $this->belongsToMany('App\Role')->withPivot('foo', 'bar');

حال خصوصیتهای `foo` و `bar` بر روی شی `pivot` برای مدل `Role` قابل دسترس خواهند بود:

اگر میخواهید جدول پیوت شما به طور خودکار تایl استمپ های `updated_at` و `created_at` را به مدیریت نماید، از متد `withTimestamps` بهنگام تعریف رابطه استفاده نمایید:

	return $this->belongsToMany('App\Role')->withTimestamps();

#### حذف رکوردها بر روی جدول پیوت

برای حذف تمامی رکوردهای جدول پیوت برای یک مدل، میتوانید از متد `detach` استفاده کنید:

	User::find(1)->roles()->detach();

توجه داشته باشید که این دستور باعث حذف رکوردها ا زجدول `roles` نمی شود و تنها رکوردهای جدول پیوت را حذف میکند:

#### به روزرسانی یک رکورد از جدول پیوت

گاهی میخواهید جدول پیوت را به روزرسانی کنید اما آن را حذف (detach) نکنید. اگر میخواهید جدول پیوت را بدون حذف به روزرسانی کنید، میتوانید از متد `updateExistingPivot` به شکل زیر استفاده کنید:

	User::find(1)->roles()->updateExistingPivot($roleId, $attributes);

#### تعریف یک مدل پیوت اختصاصی

لاراول همچنین امکان مدلهای پیوت اختصاصی را هم فراهم میکند. برای تعریف مدل اختصاصی، ابتدا مدل "پایه" (Base) که `Eloquent` را اکستند میکند تعریف کنید. دردیگر مدلهای Eloquent، به جای کلاس پایه `Eloquent` از کلاس جدیدی که تعریف کرده اید استفاده نمایید. در مدل پایه خود، تابع زیر که نمونه ای از مدل پیوت اختصاصی شما برمیگرداند، تعریف کنید :

	public function newPivot(Model $parent, array $attributes, $table, $exists)
	{
		return new YourCustomPivot($parent, $attributes, $table, $exists);
	}

<a name="collections"></a>
## کالکشن ها - Collections

تمامی مجموعه های چند نتیجه بازگردانده شده توسط Eloquent، از طریق متد `get` یا یک `relationship`، یک کالکشن بازمیگردانند. این شی اینترفیس `IteratorAggregate` از PHP را ایمپلیمنت میکندبنابراین میتواند مانند یک آرایه در حلقه از آن استفاده کرد. هرچند این شی تعداد بسیار زیادی متدهای کاربردی دیگر هم برای کار با مجموعه های نتیجه دارد.

#### بررسی وجود یک کلید در یک کالکشن

برای مثال، با استفاده از متد `contains` میتوانیم تعیین کنیم آیا یک نتیجه شامل یک کلید اصلی هست یا خیر:

	$roles = User::find(1)->roles;

	if ($roles->contains(2))
	{
		//
	}

کالکشنها را میتوان به آرایه یا JSON نیز تبدیل نمود:

	$roles = User::find(1)->roles->toArray();

	$roles = User::find(1)->roles->toJson();

اگر یک کالکشن cast به string شود آنگاه تبدیل به JSON می شود:

	$roles = (string) User::find(1)->roles;

#### پیمایش کالکشنها

کالکشنهای Eloquent همچنین شامل متدهای مفید بسیاری برای پیمایش و فیلترکردن عناصر درون آنها هستند:

	$roles = $user->roles->each(function($role)
	{
		//
	});

#### فیلترکردن کالکشنها

هنگام فیلترکردن کالکشنها، کالبک استفاده شده به عنوان کالبک [array_filter](http://php.net/manual/en/function.array-filter.php) استفاده می شود.

	$users = $users->filter(function($user)
	{
		return $user->isAdmin();
	});

> **نکته:** وقتی کالکشنی را فیلتر میکنید و آن را به JSON تبدیل میکنید، ابتدا متد `values` را فراخوانی کنید تا کلیدهای آرایه را ریست نمایید.

#### اعمال کالبک برای همه ی عناصر کالکشن

	$roles = User::find(1)->roles;

	$roles->each(function($role)
	{
		//
	});

#### مرتبسازی یک کالکشن با یک مقدار

	$roles = $roles->sortBy(function($role)
	{
		return $role->created_at;
	});

	$roles = $roles->sortByDesc(function($role)
	{
		return $role->created_at;
	});

#### مرتبسازی یک کالکشن با یک مقدار

	$roles = $roles->sortBy('created_at');

	$roles = $roles->sortByDesc('created_at');

#### بازدگرداندن یک نوع کالکشن اختصاصی

گاهی نیازخ خواهید داشت شی کالکشن اختصاصی خود با متدهای مشخص را ایجاد کنید و به عنوان نتیجه بازگردانید. میتوانید این را بر روی مدل Eloquent با (بازنویسی) override متد `newCollection` انجام دهید:

	class User extends Model {

		public function newCollection(array $models = [])
		{
			return new CustomCollection($models);
		}

	}

<a name="accessors-and-mutators"></a>
## Accessors & Mutators

#### تعریف یک  Accessor

Eloquent برای بتدیل خصوصیتهای مدل هنگام مقداردهی و خواندن آنها ارائه مینماید. به سادگی برای تعریف یک accessor متد `fetFooAttribute` را بر روی مدل تعریف کنید. به یاد داشته باشید که متد باید از قوانین نامگذاری کمل کیس پیروی نماید، هر چند نامهای ستونهای پایگاه داده اسنیک کیس باشند:

	class User extends Model {

		public function getFirstNameAttribute($value)
		{
			return ucfirst($value);
		}

	}

در مثال بالا، ستون `first_name` یک accessor دارد. توجه داشته باشید که مقدار خصوصیت به accessor فرستاده شده است.

#### تعریف Mutator

Mutatorها هم به روشی مشابه تعریف می شوند:

	class User extends Model {

		public function setFirstNameAttribute($value)
		{
			$this->attributes['first_name'] = strtolower($value);
		}

	}

<a name="date-mutators"></a>
## Mutators تاریخ

به صورت پیش فرض، Eloquent فیلدهای `created_at` و `updated_at`  را به شی هایی از جنس [Carbon](https://github.com/briannesbitt/Carbon) تبدیل میکند، که تعداد زیادی متدهای مفید ارائه میکند، و کلاس `DateTime` از PHP را اکستند مینماید:

میتوانید مشخص کنید کدام فیلدها خودکار mutate شوند و حتی به طور کامل این mutate را متوقف نمایید. این کار را با بازنویسی (override) متد `getDates` از مدل انجام دهید:

	public function getDates()
	{
		return ['created_at'];
	}

وقتی ستونی به عنوا نتاریخ شناخته می شود، میتوانید مقدار آن را با تایم استمپ UNIX، یک رشته تاریخ (`Y-m-d`)، رشته تاریخ-زمان، و البته نمونه ای کلاس `DateTime`/`Carbon` مقداردهی کنید.

برای غیرفعال کردن mutateهای تاریخ، به سادگی یک آرایه خالی از متد `getDates` بازگردانید:

	public function getDates()
	{
		return [];
	}

<a name="attribute-casting"></a>
### تبدیل نوع خصوصیتها

اگر خصوصیتهایی دارید که میخواهید همیشه به نوع داده های دیگر تبدیل نمایید، میتوانید آن خصوصیت را به خصوصیت `casts` مدل خود بیافزایید. درغیر اینصورت، باید یک mutator برای هر کدام از خصوصیتها تعریف نمایید، که میتواند زمانبر باشد. مثالی از خصوصیت `casts` ارائه شده است:

	/**
	 * The attributes that should be casted to native types.
	 *
	 * @var array
	 */
	protected $casts = [
		'is_admin' => 'boolean',
	];

حال خصوصیت `is_admin` همیشه به بولین cast می شود، حتی اگر مقدار ذخیره شده در پایگاه داده یک مقدار صحیح باشد. دیگر نوع های موجود برای استفاده عبارتند از: `integer` ،`real` ،`float` ،`double` ،`string` ،`boolean` ،`object` و `array`.

تبدیل نوع `array` خصوصا برای کار با ستونهایی که به عنوان JSON سریالایزشده ذخیره شده اند، مفید است. برای مثال، اگر پایگاه داده شما، دارای فیلدی از نوع TEXT باشد که JSON سریالایز شده در آن قرار داشته باشد، افزودن تبدیل نوع `array` به آن باعث تبدیل آن به آرایه PHP در هنگام دسترسی توسط مدل Eloquent می شود:

	/**
	 * The attributes that should be casted to native types.
	 *
	 * @var array
	 */
	protected $casts = [
		'options' => 'array',
	];

حال زمان استفاده از مدل ELoquent:

	$user = User::find(1);

	// $options is an array...
	$options = $user->options;

	// options is automatically serialized back to JSON...
	$user->options = ['foo' => 'bar'];

<a name="model-events"></a>
## رویدادهای مدل

مدلهای Eloquent رویدادهای زیادی ایجاد میکنند. این موضوع امکان انجام عملیات مختلف در قدمهای متفاوتی از چرخه عمر مدل را به شما میدهد: `creating`، `created`  `updating` ،`updated` ،`saving` ،`saved` ،`deleting` ،`deleted` .`restoring` ،`restored`

هرگاه یک عنصر برای اولین بار ذخیره میشود، رویدادهای `creating` و `created` اتفاق می افتند. اگر عنصری جدید نیست، و متد `save` فراخوانی می شود، رویدادهای `updated`/`updating` اتفاق می افتند. در هر دو حالت، رویدادهای `saved`/`saving` اتفاق می افتند.

#### لغو عملیات ذخیره با استفاده از رویدادها

اگر از رویدادهای `updating` ،`creating` ،`saving`، یا `deleting` مقدار `false` بازگردد، عملیات لغو میشود:

	User::creating(function($user)
	{
		if ( ! $user->isValid()) return false;
	});

#### محل ثبت لیسنرهای رویداد

`EventServiceProvider` محل مناسبی برای ثبت رویدادهای مربوط به مدل است. برای مثال: 

	/**
	 * Register any other events for your application.
	 *
	 * @param  \Illuminate\Contracts\Events\Dispatcher  $events
	 * @return void
	 */
	public function boot(DispatcherContract $events)
	{
		parent::boot($events);

		User::creating(function($user)
		{
			//
		});
	}

<a name="model-observers"></a>
## آبزرورهای مدل - Model Observers

برای متمرکزکردن مدیریت رویدادهای مدل، میتوانید یک آبزرور مدل تعریف کنید. یک کلاس آبزرور، میتواند متدهایی متناظر با رویدادهای متنوع مدل داشته باشد. برای مثال، متدهای `creating`، `updating`، `saving` در کنار دیگر نامهای رویداد مدل میتوانند در یک آبزرور تعریف شوند.

برای مثال، یک آبزرور مدل میتواند شکل زیر باشد:

	class UserObserver {

		public function saving($model)
		{
			//
		}

		public function saved($model)
		{
			//
		}

	}

میتوانید شی آبزرور را با استفاده از متد `observe` ثبت کنید:

	User::observe(new UserObserver);

<a name="model-url-generation"></a>
## تولید URL برای مدل

میتوانید به متدهای `route` و یا `action` یک مدل بفرستید، و کلید اصلی آن در URI تولید شده قرار میگیرد. برای مثال:

	Route::get('user/{user}', 'UserController@show');

	action('UserController@show', [$user]);

در این مثال خصوصیت `$user->id` در محل داده `{user}` در URL تولید شده قرارداده می شود. هرچند اگر بخواهید خصوصیت دیگری به جای ID استفاده کنید، متد `getRouteKey را در مدل مربوطه بازنویسی کنید:

	public function getRouteKey()
	{
		return $this->slug;
	}

<a name="converting-to-arrays-or-json"></a>
## تبدیل به آرایه / JSON

#### تبدیل یک مدل به آرایه

هنگام ساختن JSON API، شاید اغلب نیاز داشته باشید مدلها و رابطه ها را به آرایه یا JSON تبدیل کنید. بنابراین Eloquent متدهایی برای این کار دارد. برای تبدیل یک مدل و ارتباطهای بارگذاری شده آن به یک آرائه، میتوانید از متد `toArray` استفاده کنید:

    $user = User::with('roles')->first();

	return $user->toArray();

توجه داشته باشید که تمام کالکشن مدل هم میتواند به آرایه تبدیل شود:

	return User::all()->toArray();

#### تبدیل مدل به JSON

برای تبدیل مدل به JSON، میتوانید از متد `toJson` استفاده کنید:

	return User::find(1)->toJson();

#### اجرای مدل از route

توجه داشته باشید وقتی یک مدل یا کالکش به string تبدیل نوع (cast) می شود، به JSON تبدیل می شود، به این معنی که میتوانید شی های Eloquent را مستقیما از routeهای نرم افزار بازگردانید!

	Route::get('users', function()
	{
		return User::all();
	});

#### جلوگیری از قرارگرفتن خصوصیتها هنگام تبدیل به آرایه یا JSON

گاهی نیاز به محدودکردن خصوصیتهایی از مدل خود دارید که در آرایه یا JSON تولید شده قرار میگیرند، مانند گذرواژه. برای اینکار، خصوصیت `hidden` را در مدل خود تعریف نمایید و خصوصیتهای مورد نظر را به آن بیافزایید:

	class User extends Model {

		protected $hidden = ['password'];

	}

> **نکته:** در هنگام مخفی کردن ارتباطها، از نام **متد** رابطه استفاده کنید نه از نام خصوصیت پویای (dynamic accessor) آن.

همینطور میتونید برای تعریف لیست خصوصیتهایی که میخواید توی تبدیل وجود دشته باشن از خصوصیت `visible` استفاده کنید:

	protected $visible = ['first_name', 'last_name'];

<a name="array-appends"></a>
گاهی نیاز خواهید داشت خصوصیتهایی را در آرایه داشته باشید که ستون متناظری در پایگاه داده ندارند. برای اینکار به سادگی یک accessor برای آن مقدار تعریف کنید:

	public function getIsAdminAttribute()
	{
		return $this->attributes['admin'] == 'yes';
	}

پس از ایجاد accessor، تنها مقدار را به خصوصیت `appends` بر روی مدل ایجاد کردید:

	protected $appends = ['is_admin'];

وقتی خوصیت مورد نظر به لیست `appends` اضاف شد، در هر دو قالب آرایه و JSON مدل قرار خواهد گرفت. خصوصیتهای موجود در آرایه `appends` از تنظیمات `visible` و `hidden` مدل پیروی میکنند.