# ایمیل (Mail)

- [تنظیمات](#configuration)
- [کاربرد ابتدایی](#basic-usage)
- [افزودن پیوست درلحظه - Inline Attachment](#embedding-inline-attachments)
- [صف کردن ایمیل ها](#queueing-mail)
- [ایمیل و توسعه محلی](#mail-and-local-development)

<a name="configuration"></a>
## تنظیمات

لاراول یک API ساده و خوش ساخت برپایه کتابخانه معروف [SwiftMailer](http://swiftmailer.org) ارائه می نماید. فایل تنظیمات ایمیل `config/mail.php` است، و گزینه هایی برای تنظیم میزبان SMTP، پورت، و اطلاعات دسترسی و همینطور یک متغیر عمومی `from` برای تمامی ایمیل های فرستاده شده توسط این کتابخانه در خود دارد. شما میتوانید از هر سرور SMTP که بخواهید استفاده کنید. اگر میخواهید از تابع `mail` در PHP برای فرستادن ایمیل استفاده کنید، باید مقدار متغیر `driver` درفایل تنظیمات را به `mail` تغییر دهید. یک درایور `sendmail` هم برای استفاده وجود دارد.

### درایورهای API

لاراول همچنین درایورهایی برای APIهای HTTP با نامهای Mailgun و Mandrill ارائه می دهد. این APIها اغلب از سرورهای SMTP ساده تر و سریعتر هستند. هردوی این درایورها برای کار نیاز به نصب یک کتابخانه HTTP با نام Guzzle 5 دارند. با افزودن خط زیر به فایل `composer.json` میتوانید کتابخانه Guzzle 5 را به پروژه خود بیافزایید:

	"guzzlehttp/guzzle": "~5.0"

#### درایور Mailgun

برای استفاده از درایور Mailgun در فایل تنظیمات `config/mail.php` مقدار متغیر `driver` را با `mailgun` مقداردهی کنید. در قدم بعد اگر فایل تنظیمات `config/services.php` وجود ندارد آن را ایجاد کنید. از وجود متغیرهای زیر در آن مطمئن شوید:

	'mailgun' => [
		'domain' => 'your-mailgun-domain',
		'secret' => 'your-mailgun-key',
	],

#### درایور Mandrill

برای استفاده از درایور Mandrill در فایل تنظیمات `config/mail.php` مقدار متغیر `driver` را با `mandrill` مقداردهی کنید. در قدم بعد اگر فایل تنظیمات `config/services.php` وجود ندارد آن را ایجاد کنید. از وجود متغیرهای زیر در آن مطمئن شوید:

	'mandrill' => [
		'secret' => 'your-mandrill-key',
	],

### درایور لاگ

اگر مقدار متغیر `driver` در فایل تنظیمات `config/mail.php` با `log` مقداردهی شده باشد، تمامی ایمیل ها در فایلهای لاگ نوشته خواهند شد، و به هیچ یک از مخاطبین فرستاده نمی شوند. این کار برای عیب یابی و بررسی محتوای محلی و سریع مناسب است.

<a name="basic-usage"></a>
## کاربرد ابتدایی

متد `Mail::send` میتواند برای فرستادن ایمیل استفاده شود:

	Mail::send('emails.welcome', ['key' => 'value'], function($message)
	{
		$message->to('foo@example.com', 'John Smith')->subject('Welcome!');
	});

آرگومان اول فرستاده شده به متد `send` نام view استفاده شده به عنوان بدنه ایمیل است. آرگومان دوم داده ای است که باید به view فرستاده شود، و اغلب به عنوان یک آرایه ارائه می شود. داده ها از طریق متغیر `$key` در view قابل دسترس خواهند بود. آرگومان سوم کلوژری است که به شما امکان مشخص کردن گزینه های مختلف در مورد پیام ایمیل را میدهد.

> **نکته:** همیشه یک متغیر `$message` به view مربوط به ایمیل فرستاده می شود و امکان افزودن پیوست در محل را به وجود می آورد. بنابراین بهتر است از فرستادن متغیر `message` خودداری کنید.

همینطور میتوانید علاوه بر view در قالب HTML، از view با متن ساده استفاده کنید:

	Mail::send(['html.view', 'text.view'], $data, $callback);

یا میتوانید با استفاده از کلیدهای `html` یا `text` تنها یک نوع view مشخص نمایید:

	Mail::send(['text' => 'view'], $data, $callback);

میتوانید گزینه های دیگری مانند رونوشت، یا پیوست هم برای ایمیل خود مشخص نمایید:

	Mail::send('emails.welcome', $data, function($message)
	{
		$message->from('us@example.com', 'Laravel');

		$message->to('foo@example.com')->cc('bar@example.com');

		$message->attach($pathToFile);
	});

هنگامی که فایلی را به پیام پیوست میکنید، میتوانید نوع MIME و/یا نام نمایش داده شده را هم مشخص نمایید:

	$message->attach($pathToFile, ['as' => $display, 'mime' => $mime]);

اگر میخواهید به جای یک ایمیل کامل یک string ساده بفرستید، میتوانید از متد `raw` استفاده کنید:

	Mail::raw('Text to e-mail', function($message)
	{
		$message->from('us@example.com', 'Laravel');

		$message->to('foo@example.com')->cc('bar@example.com');
	});

> **نکته:** نمونه پیام فرستاده شده به کلوژر `Mail::send` کلاس SwiftMailer را اکستند میکند. این موضوع امکان فراخوانی هر متدی بر روی آن کلاس برای ساخت پیام ایمیلتان ایجاد میکند.

<a name="embedding-inline-attachments"></a>
## پیوست درمحل - Inline Attachment

پیوست عکسها به بدنه ایمیلها کار دشواری است؛ هرچند، لاراول روشی ساده برای پیوست تصویر به ایمیل و بازیابی CID مناسب ارائه مینماید.

#### قراردادن تصویر در View مربوط به ایمیل

	<body>
		Here is an image:

		<img src="<?php echo $message->embed($pathToFile); ?>">
	</body>

#### قراردادن داده خام در View ایمیل

	<body>
		Here is an image from raw data:

		<img src="<?php echo $message->embedData($data, $name); ?>">
	</body>

توجه داشته باشید که متغیر `$message` همیشه با استفاده از فاساد `Mail` به view مربوط به ایمیل فرستاده می شود.

<a name="queueing-mail"></a>
## صف بندی ایمیلها

#### قراردادن یک ایمیل در صف

از آنجاکه فرستادن ایمیل میتواند به شدت زمان پاسخ نرم افزار شما را افزایش دهد، بسیاری از برنامه نویسان ترجیح میدهند ایمیلها را برای اینکه در زمان مناسب و پشت صحنه فرستاده شوند، در صف قرار دهند. لاراول این کار را با استفاده از [API یکتای صف](/docs/%7B%7Bversion%7D%7D/queues) ساده میکند. برای قراردادن ایمیل در صف، به سادگی متد `queue` از فاساد `Mail` را استفاده کنید:

	Mail::queue('emails.welcome', $data, function($message)
	{
		$message->to('foo@example.com', 'John Smith')->subject('Welcome!');
	});

همچنین میتوانید با استفاده از متد `later` مدت زمانی که میخواهید فرستادن ایمیل را به تاخیر بیندازید مشخص کنید:

	Mail::later(5, 'emails.welcome', $data, function($message)
	{
		$message->to('foo@example.com', 'John Smith')->subject('Welcome!');
	});

اگر میخواهید یک صف مشخص را برای افزودن پیام مشخص کنید، میتوانید از متدهای `queueOn` و `laterOn` استفاده کنید:

	Mail::queueOn('queue-name', 'emails.welcome', $data, function($message)
	{
		$message->to('foo@example.com', 'John Smith')->subject('Welcome!');
	});

<a name="mail-and-local-development"></a>
## Mail و توسعه محلی

هنگامی که در حال توسعه نرم افزاری با امکان ارسال ایمیل هستید، غیرفعال کردن فرستادن ایمیل از ماشین محلی شما میتواند مفید باشد. برای انجام این کار، میتوانید متد `Mail::pretend` را استفاده کنید یا مقدار گزینه `pretend` را در فایل `config/mail.php` را با `true` مقداردهی کنید. هنگامی که فرستنده ایمیل در حالت `pretend` باشد، پیام به جای فرستاده شدن در فایلهای لاگ نرم افزار نوشته میشود.

اگر میخواهید واقعا ایملیهای تستی را ببینید، از سرویسی مانند [Mailtrap](https://mailtrap.io) استفاده کنید.