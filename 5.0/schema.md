# سازنده اسکیما - Schema Builder

- [آشنایی](#introduction)
- [ایجاد و حذف جدولها](#creating-and-dropping-tables)
- [افزودن ستونها](#adding-columns)
- [تغییر ستونها](#changing-columns)
- [تغییرنام ستونها](#renaming-columns)
- [حذف ستونها](#dropping-columns)
- [بررسی وجود ستون](#checking-existence)
- [افزودن ایندکس](#adding-indexes)
- [کلیدهای خارجی](#foreign-keys)
- [حذف ایندکس](#dropping-indexes)
- [حذف تایم استمپ و حذف های موقت](#dropping-timestamps)
- [موتورهای ذخیره سازی - Storage Engines](#storage-engines)

<a name="introduction"></a>
## آشنایی

کلاس `Schema` لاراول روشی مستقل از پایگاه داده برای انجام تغییرات بر روی جدولها ارائه مینماید. این کلاس با تمامی پایگاه داده هایی که توسط لاراول پشتیبانی میشوند به خوبی کارمیکند، و برای تمامی این سیستمها یک واسط توسعه یکسان دارد.

<a name="creating-and-dropping-tables"></a>
## ایجاد و حذف جداول

برای ایجاد یک جدول پایگاه داده جدید، متد `Schema::create` استفاده می شود:

	Schema::create('users', function($table)
	{
		$table->increments('id');
	});

آرگومان اولی که به متد `create` فرستاده می شود، اسم جدول است، و آرگومان دوم یک `Closure` است. این کلوژر یک شی `Blueprint` میپذیرد که برای تعریف جدول می توان از آن استفاده کرد.

برای تغییر نام یک جدول پایگاه داده، میتوان از متد `rename` استفاده کرد:

	Schema::rename($from, $to);

برای مشخص کردن اتصال پایگاه داده مورد استفاده در انجام عملیات، از متد `Schema::connection` استفاده کنید:

	Schema::connection('foo')->create('users', function($table)
	{
		$table->increments('id');
	});

برای حذف یک جدول، میتوانید از متد `Schema::drop` استفاده کنید:

	Schema::drop('users');

	Schema::dropIfExists('users');

<a name="adding-columns"></a>
## افزودن ستونها

برای به روزرسانی یک جدول، از متد `Schema::table` استفاده میکنیم:

	Schema::table('users', function($table)
	{
		$table->string('email');
	});

schema builder نوع ستونهای متفاوتی را معرفی میکند که در زمان ساخت جدولها میتوان از آنها استفاده کرد:

دستور | تعریف
------------- | -------------
`$table->bigIncrements('id');`  |  افزایش مقدار ID با استفاده از معادل "big integer"
`$table->bigInteger('votes');`  |  معادل BIGINT برای جدول
`$table->binary('data');`  |  معادل BLOB جدول
`$table->boolean('confirmed');`  |  معادل BOOLEAN جدول
`$table->char('name', 4);`  |  معدل CHAR در جدول
`$table->date('created_at');`  |  معادل DATE در جدول
`$table->dateTime('created_at');`  |  معدل DATETIME برای جدول
`$table->decimal('amount', 5, 2);`  |  معادل DECIMAL با مقدار صحیح و اعشار
`$table->double('column', 15, 8);`  |  معادل DOUBLE با اعشار، در مجموع 15 رقم با 8 رقم اعشار
`$table->enum('choices', ['foo', 'bar']);` | معادل ENUM در جدول
`$table->float('amount');`  |  معادل FLOAT در جدول
`$table->increments('id');`  |  افزایش ID در جدول
`$table->integer('votes');`  |  معادل INTEGER در جدول
`$table->json('options');`  |  معادل JSON در جدول
`$table->jsonb('options');`  |  معادل JSONB در جدول
`$table->longText('description');`  |  معادل LONGTEXT در جدول
`$table->mediumInteger('numbers');`  |  معادل MEDIUMINT در جدول
`$table->mediumText('description');`  |  معادل MEDIUMTEXT در جدول
`$table->morphs('taggable');`  | ستون INTEGER با عنوان 'taggable_id' و STRING با عنوان  'taggable_type' به جدول اضاف میکند
 `$table->nullableTimestamps();`  |  مشابه متد timestamp() با این تفاوت که امکان  NULL بودن مقدار وجود دارد
  `$table->smallInteger('votes');`  |  معادل SMALLINT در جدول
`$table->tinyInteger('numbers');`  |  معادل TINYINT در جدول
`$table->softDeletes();`  | ستون **deleted\_at** را برای امکان حذف موقت به جدول می افزاید
`$table->string('email');`  |  معادل ستون VARCHAR در جدول
`$table->string('name', 100);`  |  معادل ستون VARCHAR با امکان ارائه اندازه
`$table->text('description');`  |  معادل TEXT در جدول
`$table->time('sunrise');`  |  معادل TIME در جدول
`$table->timestamp('added_on');`  |  معادل TIMESTAMP در جدول
`$table->timestamps();`  |  ستونهای **created\_at** و **updated\_at** را به جدول می افزاید
`$table->rememberToken();`  |  ستون remember_token با نوع VARCHAR(100) و امکان NULL بودن می افزاید
`->nullable()`  |  اشاره میکند که ستون امکان NULL بودن را هم دارد.
`->default($value)`  |  یک مقدار پیشفرض برای ستون تعیین میکند.
`->unsigned()`  |  نوع INTEGER را به UNSIGNED تغییر میدهد.

#### استفاده از after در MySQL

اگر از پایگاه داده MySQL استفاده میکنید، میتوانید از متد `after` برای مشخص کردن ترتیب ستونها استفاده کنید:

	$table->string('name')->after('email');

<a name="changing-columns"></a>
## تغییر ستونها

**نکته:** پیش از تغییر ستون، مطمئن باشید وابستگی `doctrine/dbal` را به فایل `composer.json` افزوده اید.

گاهی نیاز به تغییر یک ستون از جدول دارید. برای مثال، گاهی نیاز به افزایش طول string یک ستون دارید. متد `change` این کار را به سادگی انجام میدهد! برای مثال، اندازه ستون `name` را از 25 به 50 افزایش میدهیم:

	Schema::table('users', function($table)
	{
		$table->string('name', 50)->change();
	});

همچنین میتوان وضعیت یک ستون را به `Nullable` تغییر داد:

	Schema::table('users', function($table)
	{
		$table->string('name', 50)->nullable()->change();
	});

<a name="renaming-columns"></a>
## تغییر نام یک ستون

برای تغییر نام ستون، میتوانید از متد `renameColumn` در Schema builder استفاده کنید. پیش از تغییر نام ستون، وابستگی `doctorine/dbal` را به فایل `composer.json` بیافزایید.

	Schema::table('users', function($table)
	{
		$table->renameColumn('from', 'to');
	});

> **نکته:** تغییر نام در جدولی با ستونی از جنس `enum` در حال حاضر پشتیبانی نمیشود.

<a name="dropping-columns"></a>
## حذف ستونها

برای حذف یک ستون، میتوانید از متد `dropColumn` در Schema builder استفاده کنید. پیش از حذف یک ستون وابستگی `doctorine/dbal` را به فایل `composer.json` بیافزایید.

#### حذف یک ستون از یک جدول

	Schema::table('users', function($table)
	{
		$table->dropColumn('votes');
	});

#### حذف چند ستون از جدول

	Schema::table('users', function($table)
	{
		$table->dropColumn(['votes', 'avatar', 'location']);
	});

<a name="checking-existence"></a>
## بررسی وجود

#### بررسی وجود یک جدول

میتوانید برای بررسی وجود یا عدم وجود یک جدول یا ستون از متدهای `hasTable` یا `hasColumn` استفاده کنید:

	if (Schema::hasTable('users'))
	{
		//
	}

#### بررسی وجود یک ستون

	if (Schema::hasColumn('users', 'email'))
	{
		//
	}

<a name="adding-indexes"></a>
## افزودن ایندکس

Schema builder چندین نوع ایندکس از چند نوع ایندکس پشتیبانی میکند. دو راه برای افزودن آنها وجود دارد. راه اول، میتوانید آن را مستقیما هنگام معرفی ستون تعریف کنید، یا در مرحله ای مجزا آنها را بیافزایید:
	$table->string('email')->unique();

یا میتوانید ایندکس را در خطی مجزا بیافزایید. در ادامه لیستی از نوع ایندکسهای موجود ارائه شده اند:

Command  | Description
------------- | -------------
`$table->primary('id');`  |  افزودن کلید اصلی
`$table->primary(['first', 'last']);`  |  افزودن کلید ترکیبی
`$table->unique('email');`  |  افزودن ایندکس یکتا
`$table->index('state');`  |  افزودن ایندکس پایه ای

<a name="foreign-keys"></a>
## کلید خارجی

لاراول امکان افزودن محدودیت کلید خارجی به جداول را نیز فراهم مینماید:

	$table->integer('user_id')->unsigned();
	$table->foreign('user_id')->references('id')->on('users');

در این مثال، تعریف کرده ایم که ستون `user_id` به ستون `id` از جدول `users` اشاره میکند. پیش از اینکار از ساخته شدن کلید خارجی مطمئن شوید.

همچنین برای عملیاتهای "on update" و "on delete" میتوانید با پارامترهایی که میفرستید شرایط را هم تعیین نمایید:

	$table->foreign('user_id')
          ->references('id')->on('users')
          ->onDelete('cascade');

برای حذف یک کلید خارجی، میتوانید از متد `dropForeign` استفاده کنید. قرارداد نامگذاری استفاده شده برای کلیدهای خارجی دقیقا مشابه آنچه در نامگذاری ایندکسها استفاده میشود است.

	$table->dropForeign('posts_user_id_foreign');

> **نکته:** هنگام ایجاد یک کلید خارجی که به یک Integer با افزایش مقدار خودکار اشاره دارد، به یاد داشته باشید همیشه ستون کلید خارجی را `unsigned` قرار دهید.

<a name="dropping-indexes"></a>
## حذف ایندکسها

برای حذف ایندکس باید نام آن را مشخص نمایید. لاراول به صورت پیش فرض یک نام منطقی به ایندکس انتصاب میدهد. این کار با ترکیب نام جدول، نام ستونی که در ایندکس آمده، و نوع ایندکس مشخص می شود. در ادامه چند مثال را میبینید:

فرمان | توصیف
------------- | -------------
`$table->dropPrimary('users_id_primary');`  |  حذف یک کلید اصلی از جدول "users"
`$table->dropUnique('users_email_unique');`  |  حذف یک ایندکس یکتا از جدول "users"
`$table->dropIndex('geo_state_index');`  |  حذف یک ایندکس ابتدایی از جدول "geo"

<a name="dropping-timestamps"></a>
## حذف تایم استمپها و حذفهای موقت

برای حذف ستونهای از جنس `timestamps`، `nullableTimestamps`، softDeletes میتوانید از متدهای زیر استفاده کنید:

فرمان | توصیف
------------- | -------------
`$table->dropTimestamps();`  |  حذف ستونهای **created\_at** و **updated\_at** از جدول
`$table->dropSoftDeletes();`  |  حذف ستون **deleted\_at** از جدول

<a name="storage- engines"></a>
## موتورهای ذخیره سازی

برای تعیین موتور ذخیره سازی برای یک جدول، مقدار خصوصیت `engine` را بر روی schema builder مقداردهی کنید:

	Schema::create('users', function($table)
	{
		$table->engine = 'InnoDB';

		$table->string('email');
	});
