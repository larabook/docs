# کوئری ساز

- [مقدمه](#introduction)
- [دستورات select](#selects)
- [Joins](#joins)
- [دستورات شرطی پیشرفته](#advanced-wheres)
- [اجرای توابع جمعی (Aggregates)](#aggregates)
- [بکارگیری کوئری های توکار (Raw Expression)](#raw-expressions)
- [ایجاد رکورد Insert](#inserts)
- [دستور Update](#updates)
- [حذف رکورد](#deletes)
- [ادغام رکورد ها Union](#unions)
- [قفل Pessimistic](#pessimistic-locking)

<a name="introduction"></a>
## مقدمه
کوئری ساز لاراول یک رابط روان و راحت و ساده برای تولید و اجرای کوئری ها می باشد که میتواند در بیشتر عملیات کار با دیتابیس نیز استفاده شود و برروی تمامی بانکهایی اطلاعاتی که توسط لاراول پشتیبانی میشود قابل اجرا می باشد . 

> **نکته :** کوئری ساز برای نشاندن مقادیر در کوئری ها ، بطور کامل از `PDO parameter binding` استفاده میکند تا وب سایت شما را درمقابل حملات  `SQL injection` حفظ کند و دیگر نیازی نیست که پارامتر های ارسالی به کوئری ساز را از نظر کدهای مخرب چک کنید .

<a name="selects"></a>
## دستورات Select

#### گرفتن تمامی رکورد های جدول

	$users = DB::table('users')->get();

	foreach ($users as $user)
	{
		var_dump($user->name);
	}

#### تکه تکه کردن نتایج حاصل از اجرای کوئری select

	DB::table('users')->chunk(100, function($users)
	{
		foreach ($users as $user)
		{
			//
		}
	});

میتوانید با ارسال مقدار `false` از درون `closure` به اجرای ادامه عملیات خاتمه دهید :

	DB::table('users')->chunk(100, function($users)
	{
		//

		return false;
	});

#### دریافت یک رکورد از کل نتایج

	$user = DB::table('users')->where('name', 'John')->first();

	var_dump($user->name);

#### دریافت یک ستون از کل نتایج

	$name = DB::table('users')->where('name', 'John')->pluck('name');

#### دریافت نتایج بصورت لیست (آرایه)
	$roles = DB::table('roles')->lists('title');
این متد یک آرایه از Title ها را برمیگرداند ، همچنین شما میتوانید مقادیر اندیس ها در آرایه را از طریق پارامتر دوم متد `lists` مشخص کنید :

	$roles = DB::table('roles')->lists('title', 'name');

#### دستور select جهت انتخاب ستون ها

	$users = DB::table('users')->select('name', 'email')->get();

	$users = DB::table('users')->distinct()->get();

	$users = DB::table('users')->select('name as user_name')->get();

#### اضافه کردن یک ستون دیگر با addSelect

	$query = DB::table('users')->select('name');

	$users = $query->addSelect('age')->get();

#### دستورات شرطی

	$users = DB::table('users')->where('votes', '>', 100)->get();

#### دستور Or

	$users = DB::table('users')
	                    ->where('votes', '>', 100)
	                    ->orWhere('name', 'John')
	                    ->get();

#### جستجو رکورد های که مقدار فیلد x آنها بین دامنه مقادیر داده شده باشد : 

	$users = DB::table('users')
	                    ->whereBetween('votes', [1, 100])->get();

#### جستجو رکورد های که مقدار فیلد x آنها بین دامنه مقادیر داده شده  نباشند :

	$users = DB::table('users')
	                    ->whereNotBetween('votes', [1, 100])->get();

#### جستجو رکورد های که مقدار فیلد x آنها محدود به آرایه داده شده باشد :

	$users = DB::table('users')
	                    ->whereIn('id', [1, 2, 3])->get();

	$users = DB::table('users')
	                    ->whereNotIn('id', [1, 2, 3])->get();

#### جستجوی رکوردها با مقادیر null .

	$users = DB::table('users')
	                    ->whereNull('updated_at')->get();

#### دستورات شرط پویا
میتوانید برای تولید دستورات شرطی از عبارات خاصی استفاده کنید تا بتوانید نوع شرط را به روش مفهومی تری بیان نمایید :

	$admin = DB::table('users')->whereId(1)->first();

	$john = DB::table('users')
	                    ->whereIdAndEmail(2, 'john@doe.com')
	                    ->first();

	$jane = DB::table('users')
	                    ->whereNameOrAge('Jane', 22)
	                    ->first();

#### دستورات Order by , Group by, Having

	$users = DB::table('users')
	                    ->orderBy('name', 'desc')
	                    ->groupBy('count')
	                    ->having('count', '>', 100)
	                    ->get();

#### دستور Limit

	$users = DB::table('users')->skip(10)->take(5)->get();

<a name="joins"></a>
## عملیات Join و ضرب دوجدول
برای ضرب دو جدول و Join کردن آنها به شکل زیر عمل کنید :

#### نمونه ساده Join چند جدول با یکدیگر

	DB::table('users')
	            ->join('contacts', 'users.id', '=', 'contacts.user_id')
	            ->join('orders', 'users.id', '=', 'orders.user_id')
	            ->select('users.id', 'contacts.phone', 'orders.price')
	            ->get();

#### عملیات Left Join

	DB::table('users')
		    ->leftJoin('posts', 'users.id', '=', 'posts.user_id')
		    ->get();
همچنین میتوانید عملیات Join پیشرفته تری ایجاد کنید :

	DB::table('users')
	        ->join('contacts', function($join)
	        {
	        	$join->on('users.id', '=', 'contacts.user_id')->orOn(...);
	        })
	        ->get();
میتوانید  از دستورات  "where" یا "orWhere" برای شرط Join خود استفاده کنید، به عنوان مثال علاوه بر  شرط همسان بودن دو فیلد ، فیلد مورد نظر را با یک عدد نیز مقایسه نمایید :

	DB::table('users')
	        ->join('contacts', function($join)
	        {
	        	$join->on('users.id', '=', 'contacts.user_id')
	        	     ->where('contacts.user_id', '>', 5);
	        })
	        ->get();

<a name="advanced-wheres"></a>
## دستور Where پیشرفته

#### تولید کوئری های تو در تو
ممکن است بخواهید دستورات شرطی پیچیده تری را ایجاد کنید یا مثلا از دستور "where exist" برای بررسی موجودیت یک کوئری استفاده نمایید . لاراول برای اجرای اینگونه کوئری ها ، یک راه حل جالب و مفهومی ارائه کرده است :

	DB::table('users')
	            ->where('name', '=', 'John')
	            ->orWhere(function($query)
	            {
	            	$query->where('votes', '>', 100)
	            	      ->where('title', '<>', 'Admin');
	            })
	            ->get();
دستورات بالا نمونه کوئری زیر را تولید میکنند :

	select * from users where name = 'John' or (votes > 100 and title <> 'Admin')

####  بررسی موجودیت یک کوئری (Exists Statements)

	DB::table('users')
	            ->whereExists(function($query)
	            {
	            	$query->select(DB::raw(1))
	            	      ->from('orders')
	            	      ->whereRaw('orders.user_id = users.id');
	            })
	            ->get();

دستورات بالا نمونه کوئری زیر را تولید میکنند :

	select * from users
	where exists (
		select 1 from orders where orders.user_id = users.id
	)

<a name="aggregates"></a>
## اجرای توابع جمعی (Aggregates)
میتوانید نتیجه دستورات `count`, `max`, `min`, `avg`, و `sum` را توسط کوئری ساز (Query Builder) استخراج نمایید . 

	$users = DB::table('users')->count();

	$price = DB::table('orders')->max('price');

	$price = DB::table('orders')->min('price');

	$price = DB::table('orders')->avg('price');

	$total = DB::table('users')->sum('votes');

<a name="raw-expressions"></a>
## بکارگیری از کوئری های توکار (Raw Expression)
برای استفاده از  یک کوئری توکار در کوئری ساز (Query Builder) از متد `DB::raw` استفاده میکنیم . حواستان باشد تنها کاربرد این متد بکارگیری یک کوئری در دل کوئری دیگر است ، و این دستور هرگز کوئری ها را از کد های مخرب (Sql Injection) پاک نمیکند .:

#### نمونه تزریق یک کوئری در یک دستور select

	$users = DB::table('users')
	                     ->select(DB::raw('count(*) as user_count, status'))
	                     ->where('status', '<>', 1)
	                     ->groupBy('status')
	                     ->get();

<a name="inserts"></a>
##  دستورات Insert (ایجاد رکود)

#### ایجاد رکورد جدید در جدول

	DB::table('users')->insert(
		['email' => 'john@example.com', 'votes' => 0]
	);

#### اضافه کردن یک رکورد به جدول همراه با `Auto-Incrementing`
اگر جدول شما دارای یک فیلد Auto-Increment باشد ، میتوانید  توسط متد `insertGetId` برای ایجاد رکورد و دریافت مقدار ID رکورد ایجاد شده استفاده نمایید :

	$id = DB::table('users')->insertGetId(
		['email' => 'john@example.com', 'votes' => 0]
	);

> **نکته :**  اگر از بانک اطلاعاتی PostgreSQL  استفاده میکنید ، برای استفاده از متد `insertGetId` ، نام فیلد کلید اصلی باید  برابر `id` باشد .

#### اضافه کردن چند رکورد به صورت همزمان

	DB::table('users')->insert([
		['email' => 'taylor@example.com', 'votes' => 0],
		['email' => 'dayle@example.com', 'votes' => 0]
	]);

<a name="updates"></a>
## دستور Update

#### ویرایش یک رکود در جدول

	DB::table('users')
	            ->where('id', 1)
	            ->update(['votes' => 1]);

#### افزایش و کاهش مقدار یک فیلد در جدول

	DB::table('users')->increment('votes');

	DB::table('users')->increment('votes', 5);

	DB::table('users')->decrement('votes');

	DB::table('users')->decrement('votes', 5);

میتوانید همزمان فیلد های دیگری را نیز درجدول Update کنید :

	DB::table('users')->increment('votes', 1, ['name' => 'John']);

<a name="deletes"></a>
## حذف رکورد (Delete)

#### حذف یک رکورد از جدول

	DB::table('users')->where('votes', '<', 100)->delete();

#### حذف تمامی رکورد های جدول

	DB::table('users')->delete();

#### خالی کردن یک جدول (Truncating)

	DB::table('users')->truncate();

<a name="unions"></a>
## دستورات Union (ادغام رکورد ها)
لاراول یک راه سریع برای ادغام دو نتیجه کوئری  با یکدیگر ارائه کرده است :

	$first = DB::table('users')->whereNull('first_name');

	$users = DB::table('users')->whereNull('last_name')->union($first)->get();

متد `unionAll` با روشی مشابه با متد  `union` نیز در دسترس می باشد.

<a name="pessimistic-locking"></a>
## قفل Pessimistic 

کوئری ساز ، قابلیت صدور قفل "pessimistic" بر روی دستورات `Select` را دارد.

توسط متد `sharedLock` برای کوئری SELECT قفل "pessimistic" ایجاد کنید :

	DB::table('users')->where('votes', '>', 100)->sharedLock()->get();

و توسط متد lockForUpdate ، هنگام اجرای دستور Select از اجرای Update بر روی آنها جلوگیری نمایید :

	DB::table('users')->where('votes', '>', 100)->lockForUpdate()->get();
