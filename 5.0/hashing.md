# Hashing

- [معرفی](#introduction)
- [روش استفاده](#basic-usage)

<a name="introduction"></a>
## معرفی
از فاساد `Hash` در لاراول  برای  برای ذخیره رمز کاربر به روش ( Bcrypt hashing)  استفاده میشود . اگر از کنترلر  `AuthController` استفاده میکنید (بصورت پیشفرض در اپلیکیشن شما وجود دارد) یکی از وظایف این کنترلر هش کردن رمز ورودی کاربران است .


<a name="basic-usage"></a>
## روش استفاده

#### هش کردن پسوورد با روش Bcrypt

	$password = Hash::make('secret');

همچنین میتوانید از تابع کمکی `bcrypt`  استفاده کنید :

	$password = bcrypt('secret');

#### مقایسه یک رشته  با یک رشته هش شده

	if (Hash::check('secret', $hashedPassword))
	{
		// The passwords match...
	}

#### بررسی اینکه آیا پسوورد نیاز به Hash شدن دوباره دارد یا خیر

	if (Hash::needsRehash($hashed))
	{
		$hashed = Hash::make('secret');
	}
