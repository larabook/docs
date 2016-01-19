# Migration ها و داده ریزی (seeding)

- [مقدمه](#introduction)
- [ساختن یک Migrations](#creating-migrations)
- [اجرای Migration ها](#running-migrations)
- [بازگردانی Migration ها  به عقب (rollback)](#rolling-back-migrations)
- [داده ریزی در بانک (Database Seeding)](#database-seeding)

<a name="introduction"></a>
## مقدمه
Migration ها همانند سیستم های کنترل سورس کد SVN,Git ، ولی برای بانک اطلاعاتی تعبیه شده اند و به تیم کمک میکند تا تغییرات بانک اطلاعاتی را همانند سورس کد پروژه Commit کنند و اعضای تیم همیشه یک نسخه بروز از دیتابیس را برروی سیستم خود داشته باشند . Migration ها بطور کلی از  [Schema Builder](/docs/{{version}}/schema)  استفاده مینمایند تا براحتی بتوانند تغییرات را بر روی ساختار (Schema) بانک اطلاعاتی اعمال و مدریرت کنند .

<a name="creating-migrations"></a>
## ساخت Migration 

برای ساخت یک Migration ، دستور `make:migration` را در Artisan CLI وارد نمایید :

	php artisan make:migration create_users_table
سپس Migration شما در مسیر `database/migrations` تولید خواهد شد . فریموورک برای نامگذاری فایل های Migration از یک Timestamp استفاده مینماید که ترتیب اجرای آنها را مشخص میکند .

برای تعیین اینکه Migration شما بر روی کدام جدول اعمال شود ، پارامتر `--table`  را به آخر دستور آرتیسان خود اضافه کنید . واگر Migration شما قرار است یک جدول جدید درون بانک اطلاعاتی ایجاد نماید ، `--create` را به پارامتر های خود اضافه نمایید :

	php artisan make:migration add_votes_to_users_table --table=users

	php artisan make:migration create_users_table --create=users

<a name="running-migrations"></a>
## اجرای Migration ها

#### دستور زیر تمامی Migration ها را بر روی دیتابیس اجرا میکند

	php artisan migrate

> **نکته:** اگر بواسطه اجرای دستور فوق یک خطای "class not found" دریافت کردید ، ابتدا دستور  `composer dump-autoload` را در خط فرامان اجرا کنید .

### اجرای بی چون و چرا Migration ها
گاهی اوقات Migration های شما نقشی مخرب دارند به این معنی که ممکن است اطلاعات بانک اطلاعاتی شما را از بیین ببرند ، به منظور جلوگیری از اجرای آنها دربانک اطلاعاتی (بانک اطلاعاتی سیستم در مرحله production)  شما باید اجرای عملیات را تایید نمایید ، و اگر میخواهید تمامی عملیات بدون هیچ پرسشی از شما اجرا شود  `--force` را به پارامتر اجرای Migration اضافه نمایید :

	php artisan migrate --force

<a name="rolling-back-migrations"></a>
## بازگردانی  به عقب (rollback)

#### بازگردانی تغییرات به قبل از اجرای آخرین عملیات Migration 

	php artisan migrate:rollback

#### بازگردانی تمامی Migration ها

	php artisan migrate:reset

#### بازگردانی تمامی Migration ها و اجرای دوباره آنها

	php artisan migrate:refresh

	php artisan migrate:refresh --seed

<a name="database-seeding"></a>
## داده ریزی در بانک اطلاعاتی (Database Seeding)
لاراول همچنین از یک روش ساده برای پرکردن اطلاعات تستی در جداول شما استفاده میکنند . تمامی کلاس های `seed` شما در مسیر `database/seeds` ذخیره میشوند ، میتوانید برای کلاس های `seed` از هر نامی استفاده نمایید اما از قواعد نامگذاری لاراول نیز باید پیروی کنید . مانند `UserTableSeeder` و غیره .. ،همچنین به صورت پیشفرض یک کلاس `DatabaseSeeder`  برای شما تعریف شده است . از متد `call` این کلاس میتوانید برای فراخوانی دیگر کلاسهای `seed` تان و کنترل ترتیب اجرای آنها استفاده نمایید .

#### یک نمونه از کلاس Seed 

	class DatabaseSeeder extends Seeder {

		public function run()
		{
			$this->call('UserTableSeeder');

			$this->command->info('User table seeded!');
		}

	}

	class UserTableSeeder extends Seeder {

		public function run()
		{
			DB::table('users')->delete();

			User::create(['email' => 'foo@bar.com']);
		}

	}

برای اجرای عملیات داده ریزی از دستور `db:seed` در  Artisan CLI استفاده نمایید :

	php artisan db:seed
دستور  `db:seed` به صورت پیشفرض کلاس  `DatabaseSeeder` را اجرا میکند ، که به موجب آن دیگر کلاس های `seed` معرفی شده در آن اجرا میشود . میتوانید از پارامتر `--class` برای اجرای یک کلاس `seed` مشخص استفاده کنید  تا تنها آن کلاس برای شما اجرا شود :

	php artisan db:seed --class=UserTableSeeder
همچنین میتوانید توسط دستور `migrate:refresh` در آرتیسان برای rollback  و اجرای دوباره تمامی Migration  ها  به همراه عملیات داده ریزی (seeding) استفاده نمایید :

	php artisan migrate:refresh --seed
