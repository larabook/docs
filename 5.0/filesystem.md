# سیستم فایل / ذخیره سازی ابری

- [آشنایی](#introduction)
- [پیکربندی](#configuration)
- [کاربرد پایه](#basic-usage)
- [سیستم فایل اختصاصی](#custom-filesystems)

<a name="introduction"></a>
## آشنایی

لاراول امکانات فایل سیستم بسیار مفیدی را با اتکا به کتابخانه [Flysystem](https://github.com/thephpleague/flysystem) که یک بسته PHP است ارائه میدهد. این کتابخانه توسط Frank de Jonge تهیه شده است. استفاده لاراول از Flysystem درایورهای ساده ای برای کار با سیستم فایل محلی، Amazon S3، و فضای ذخیره سازی ابری Rackspace فراهم می آورد. یکسان بودن API برای همه این گزینه ها سوئیچ بین آنها را ساده میکند.

<a name="configuration"></a>
## پیکربندی

فایل پیکربندی سیستم فایل، در مسیر `config/filesystems.php` قرار دارد. در این فایل میتوانید تمامی دیسکهایتان را پیکربندی نمایید. هر دیسک نماینده یک درایور ذخیره سازی است، پیکربندی نمونه برای هر درایور پشتیبانی شده در فایل پیکربندی ارائه شده است. بنابراین، به سادگی میتوانید با تغییر مقادیر، تنظیمات پیکربندی و مجوزهای دسترسی مربوط به فضای ذخیره سازی خود را جایگزین کنید!

پیش از استفاده از درایورهای S3 و Rackspace، باید بسته های متناسب را با استفاده از کامپوزر نصب کنید:

- Amazon S3: `league/flysystem-aws-s3-v2 ~1.0`
- Rackspace: `league/flysystem-rackspace ~1.0`

البته، هر تعداد دیسک را که بخواهید میتوانید تنظیم کنید و یا میتوانید چندین دیسک با درایور مشابه داشته باشید.

هنگامی که از یک درایور `local` استفاده میکنید، توجه داشته باشید تمامی عملیات فایلی که انجام میدهید نسبت به دایرکتوری `root` که در فایل پیکربندی معرفی کرده اید انجام می شود. به صورت پیش فرض، این مقدار برابر با دایرکتوری `storage/app` مقداردهی شده است. بنابراین، متد زیر فایل را در آدرس `storage/app/file.txt` ذخیره مینماید:

	Storage::disk('local')->put('file.txt', 'Contents');

<a name="basic-usage"></a>
## کاربرد ابتدایی

برای کار با هر یک از دیسکهای پیکربندی شده می توانید از فاساد `Storage` استفاده نمایید. به جای آن، میتوانید کانترکت `Illuminate\Contracts\Filesystem\Factory` را بر روی هر کلاسی که توسط [service container](/docs/%7B%7Bversion%7D%7D/container) لاراول شناخته می شود، تایپ هینت کنید.

#### بازیابی یک دیسک مشخص

	$disk = Storage::disk('s3');

	$disk = Storage::disk('local');

#### تعیین وجود یک فایل

	$exists = Storage::disk('s3')->exists('file.jpg');

#### فراخوانی متدها بر روی دیسک پیش فرض

	if (Storage::exists('file.jpg'))
	{
		//
	}

#### بازیابی محتوای یک فایل

	$contents = Storage::get('file.jpg');

#### مقداردهی محتوای فایل

	Storage::put('file.jpg', $contents);

#### افزودن به ابتدای یک فایل

	Storage::prepend('file.log', 'Prepended Text');

#### افزودن به انتهای یک فایل

	Storage::append('file.log', 'Appended Text');

#### حذف یک فایل

	Storage::delete('file.jpg');

	Storage::delete(['file1.jpg', 'file2.jpg']);

#### کپی یک فایل در یک مکان جدید

	Storage::copy('old/file1.jpg', 'new/file1.jpg');

#### جابجایی یک فایل به یک مکان جدید

	Storage::move('old/file1.jpg', 'new/file1.jpg');

#### گرفتن اندازه یک فایل

	$size = Storage::size('file1.jpg');

#### خواندن آخرین زمان ویرایش یک فایل (UNIX)

	$time = Storage::lastModified('file1.jpg');

#### خواندن تمامی فایلهای یک دایرکتوری

	$files = Storage::files($directory);

	// Recursive...
	$files = Storage::allFiles($directory);

#### خواندن تمامی دایرکتوریهای درون یک دایرکتوری

	$directories = Storage::directories($directory);

	// Recursive...
	$directories = Storage::allDirectories($directory);

#### ایجاد یک دایرکتوری

	Storage::makeDirectory($directory);

#### حذف یک دایرکتوری

	Storage::deleteDirectory($directory);

<a name="custom-filesystems"></a>
## سیستمهای فایل اختصاصی

ترکیب لاراول با Flysystem درایورهایی بسیاری ارائه میکند؛ هر چند Flysystem محدود به این فایل سیستمها نیست و مبدلهایی برای سیستمهای ذخیره سازی بسیار دیگری ارائه می دهد. میتوانید برای استفاده از آنها در نرم افزارهای لاراول خود درایورهای اختصاصی تولید نمایید. نگران نباشید، چندان سخت نیست!

برای ایجاد یک سیستم فایل جدید شما باید یک service provider مانند `DropboxFilesystemServiceProvider` ایجاد نمایید. در متد `boot` مربوط به provider باید نمونه ای از کانترکت `Illuminate\Contracts\Filesystem\Factory` را تزریق نمایید و متد `extend` را برای نمونه تزریق شده فراخوانی کنید. در روشی دیگر میتوانید از متد `extend` فاساد `Disk` استفاده کنید.

آرگومان اول متد `extend` نام درایور است، و آرگومان دوم کلوژری است که متغیرهای `$app` و `$config` را دریافت میکند. به کلوژر resolver باید نمونه ای از کلاس `League\Flysystem\Filesystem` ارسال کنید.

> **نکته:** متغیر $config حاوی تمامی مقادیر تعریف شده در `config/filesystems.php` برای دیسک مشخص شده است.

#### مثال Dropbox

	<?php namespace App\Providers;

	use Storage;
	use League\Flysystem\Filesystem;
	use Dropbox\Client as DropboxClient;
	use League\Flysystem\Dropbox\DropboxAdapter;
	use Illuminate\Support\ServiceProvider;

	class DropboxFilesystemServiceProvider extends ServiceProvider {

		public function boot()
		{
			Storage::extend('dropbox', function($app, $config)
			{
				$client = new DropboxClient($config['accessToken'], $config['clientIdentifier']);

				return new Filesystem(new DropboxAdapter($client));
			});
		}

		public function register()
		{
			//
		}

	}
