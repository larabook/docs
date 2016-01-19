# رمزنگاری 

- [معرفی](#introduction)
- [روش استفاده](#basic-usage)

<a name="introduction"></a>
## معرفی
در لاراول امکان رمزنگاری به روش AES از طریق  افزونه php با نام  Mcrypt  وجود دارد .
<a name="basic-usage"></a>
## روش استفاده

#### رمزکردن یک رشته

	$encrypted = Crypt::encrypt('secret');

> **نکته :** حتما طول مقدار `key` در فایل `config/app.php`  باید برابر 16, 24, یا 32  کاراکتر باشد . در غیر اینصورت , رمز نگاری امن نخواهد بود !.

#### رمز گشایی یک رشته

	$decrypted = Crypt::decrypt($encryptedValue);

#### تظیم  Cipher و حالت
برای تنظیم کردن Cipher و حالت رمزنگاری از متد های زیر استفاد ه کنید :

	Crypt::setMode('ctr');

	Crypt::setCipher($cipher);
