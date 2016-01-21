# درستی سنجی - Validation

- [کاربرد پایه](#basic-usage)
- [درستی سنجی کنترلر](#controller-validation)
- [درستی سنجی درخواستهای فرم](#form-request-validation)
- [کار با پیغامهای خطا](#working-with-error-messages)
- [پیغامهای خطا و viewها](#error-messages-and-views)
- [قوانین درستی سنجی موجود](#available-validation-rules)
- [افزودن قوانین متناسب با شرایط](#conditionally-adding-rules)
- [پیغامهای خطای اختصاصی شده](#custom-error-messages)
- [قوانین درستی سنجی اختصاصی شده](#custom-validation-rules)

<a name="basic-usage"></a>
## کاربرد پایه

لاراول یک ابزار ساده برای درستی سنجی داده و بازیابی پیامهای درستی سنجی با استفاده از کلاس `Validator` ارائه میدهد.

#### مثالهای ابتدایی از درستی سنجی

	$validator = Validator::make(
		['name' => 'Dayle'],
		['name' => 'required|min:5']
	);

داده ای که میخواهید درستی سنجی شود را به عنوان اولین آرگومان به متد `make` بفرستید. آرگومان دوم قوانین درستی سنجی مورد استفاده برای این کار هستند.

#### استفاده از آرایه ها برای مشخص کردن قوانین

قوانین را میتوانید با استفاده از "خط عمودی یا |" و یا عناصر مجزای یک آرایه بفرستید.

	$validator = Validator::make(
		['name' => 'Dayle'],
		['name' => ['required', 'min:5']]
	);

#### درستی سنجی چندین فیلد

	$validator = Validator::make(
		[
			'name' => 'Dayle',
			'password' => 'lamepassword',
			'email' => 'email@example.com'
		],
		[
			'name' => 'required',
			'password' => 'required|min:8',
			'email' => 'required|email|unique:users'
		]
	);

پس از ایجاد یک نمونه از کلاس `Validator` میتوانید از متدهای `fails` (یا `pass`) برای انجام درستی سنجی استفاده کنید.

	if ($validator->fails())
	{
		// The given data did not pass validation
	}

در صورتی که درستی اطلاعات تایید نشود، میتوانید پیامهای خطا را با استفاده از کلاس بازیابی کنید.

	$messages = $validator->messages();

همچنین با استفاده از متد `failed` میتوانید قانونهایی که اعث رد درستی سنجی شده اند را هم بازایابی نمایید. این متد پیامها را بازنمیگرداند:

	$failed = $validator->failed();

#### درستی سنجی فایلها

کلاس `Validator` قانونهای بسیاری برای درستی سنجی فایلها ارائه می نماید، مانند `size`، `mimes`. برای درستی سنجی فایلها میتوانید آنها را به همراه دیگر اطلاعات به ارزیابی کننده اطلاعات بفرستید.

### پستهای بعد از درستی سنجی

امکان انجام عملیاتهای زنجیروار پس از انجام درستی سنجی نیز وجود دارد. این امکان شما را قادر میسازد، درستی سنجیهای بیشتری انجام دهید، و همچنین امکان افزودن پیامهای خطای بیشتر به مجموعه پیغامها فراهم میکند. برای شروع، از متد `after` بر روی نمونه کلاس درستی سنجی استفاده کنید:

	$validator = Validator::make(...);

	$validator->after(function($validator)
	{
		if ($this->somethingElseIsInvalid())
		{
			$validator->errors()->add('field', 'Something is wrong with this field!');
		}
	});

	if ($validator->fails())
	{
		//
	}

به هر تعداد کالبک `after` که بخواهید میتوانید بیافزایید.

<a name="controller-validation"></a>
## درستی سنجی کنترلر

مسلما ایجاد دستی نمونه ای از `Validator` هربار که میخواهید داده ای را درستی سنجی کنید کاردشواری است. نگران نباشید، گزینه های دیگری هم دارید! کلاس پایه `App\Http\Controllers\Controller` در لاراول از تریت `ValidatesRequest` استفاده میکند. این تریت یک روش ساده برای درستی سنجی درخواستهای HTTP ارائه مینماید. نحوه استفاده از آن در ادامه آورده شده:

	/**
	 * Store the incoming blog post.
	 *
	 * @param  Request  $request
	 * @return Response
	 */
	public function store(Request $request)
	{
		$this->validate($request, [
			'title' => 'required|unique|max:255',
			'body' => 'required',
		]);

		//
	}

در صورت تایید درستی، کد شما به روال طبیعی اجرا ادامه میدهد. و در صورت رد درستی اکسپشن `Illuminate\Contracts\Validation\ValidationException` ایجاد میشود. این اکسپشن به صورت خودکار ایجاد میشود و کاربر به مکان پیشین خود ریدایرکت میشود. پیغامهای خطای ایجاد شده هم به session اضاف می شوند.

اگر درخواست ایجاد شده از نوع AJAX باشد، ریدایرکت ایجاد نمیشود. به جای آن یک پاسخ HTTP با کد وضعیت 422 حاوی پیامهای درستی سنجی در قالب JSON به مرورگر فرستاده می شود.

برای مثال، در اینجا کد معادل که دستی نوشته شده است ارائه شده است:

	/**
	 * Store the incoming blog post.
	 *
	 * @param  Request  $request
	 * @return Response
	 */
	public function store(Request $request)
	{
		$v = Validator::make($request->all(), [
			'title' => 'required|unique|max:255',
			'body' => 'required',
		]);

		if ($v->fails())
		{
			return redirect()->back()->withErrors($v->errors());
		}

		//
	}

### اختصاصی سازی قالب خطاهای درستی سنجی

اگر میخواهید قالب خطاهای درستی سنجی ارائه شده هنگام رد درستی را تغییر داده، اختصاصی کنید، `formatValidationErrors` را در کنترلر پایه بازنویسی نمایید. فراموش نکنید باید کلاس `Illuminate\Validation\Validator` را در بالای کلاس وارد نمایید:

	/**
	 * {@inheritdoc}
	 */
	protected function formatValidationErrors(Validator $validator)
	{
		return $validator->errors()->all();
	}

<a name="form-request-validation"></a>
## درستی سنجی درخواستهای فرم

برای سناریوهای پیچیده تر درستی سنجی، میتوانید یک "درخواست فرم" ایجاد نمایید. درخواستهای فرم کلاسهای درخواست اختصاصی هستند که منطق درستی سنجی در آنها تعریف شده است. برای ایجاد کلاس درخواست فرم، از فرمان آرتیزان `make:request` استفاده کنید:

	php artisan make:request StoreBlogPostRequest

کلاس ایجاد شده در دایرکتوری `app/Http/Requests` قرارخواهد گرفت. میتوانید قوانین درستی سنجی را به متد `rules` اضافه کنید:

	/**
	 * Get the validation rules that apply to the request.
	 *
	 * @return array
	 */
	public function rules()
	{
		return [
			'title' => 'required|unique|max:255',
			'body' => 'required',
		];
	}

حال، قوانین درستی سنجی چگونه اجرا می شوند؟ تمامی کاری که باید انجام دهید اعلام این موضوع در متد کنترلرتان است:

	/**
	 * Store the incoming blog post.
	 *
	 * @param  StoreBlogPostRequest  $request
	 * @return Response
	 */
	public function store(StoreBlogPostRequest $request)
	{
		// The incoming request is valid...
	}

درستی سنجی درخواست فرم ایجاد شده پیش از اجرای کنترلر می شود. با این کار کد و منطق کنترلر شما با منطق درستی سنجی ترکیب نمیشود. درستی سنجی پیش از این انجام شده است.

در صورت تایید نشدن درستی، یک ریدایرکت برای فریتادن کاربر به مکان پیشین ایجاد می شود. خطاها نیز به session افزوده می شوند و برای نمایش در دسترس هستند. در صورتی که درخواست از نوع AJAX باشد، یک پاسخ HTTP با کد وضعیت 422 حاوی پیامهای درستی سنجی در قالب JSON به مرورگر فرستاده می شود.

### Authorizing Form Requests

کلاس درخواست فرم شامل یک متد `authorize` هم میباشد. در این متد، میتوانید امکان به روزرسانی یک منبع توسط کاربر مشخص را بررسی نمایید. برای مثال، اگر یک کاربر میخواهد کامنت یک پست بلاگ  را به روزرسانی کند، باید بررسی شود آیا کامنت را خود او گذاشته است؟ یا هر منطق دیگری. برای مثال: 

	/**
	 * Determine if the user is authorized to make this request.
	 *
	 * @return bool
	 */
	public function authorize()
	{
		$commentId = $this->route('comment');

		return Comment::where('id', $commentId)
                      ->where('user_id', Auth::id())->exists();
	}

به فراخوانی متد `route` در مثال بالا توجه نمایید. این متد به شما امکان دسترسی به پارامترهای URI تعریف شده بر روی روت فراخوانی شده را می دهد، مانند پارامتر `{comment}` در مثال پایین:

	Route::post('comment/{comment}');

اگر متد `authorize` مقدار `false` برگرداند، یک پاسخ HTTP با کد وضعیت 403 به طو رخودکار بازگردانده می شود و متد کنترلر اجرا نمی شود.

اگر برنامه ای برای منطق احراز هویت در بخش دیگری از برنامه دارید، کافیست مقدار خروجی متد `authorize` را `true` بازگردانید:

	/**
	 * Determine if the user is authorized to make this request.
	 *
	 * @return bool
	 */
	public function authorize()
	{
		return true;
	}

### اختصاصی سازی قالب خطاها

اگر میخواهید قالب خطاهای درستی سنجی که پس از رد درستی در session قرار میگیرند را تغییر داده، اختصاصی نمایید، بر روی درخواست پایه (`App\Http\Requests\Request`) مقدار `formatErrors` را بازنویسی نمایید. فراموش نکنید کلاس `Illuminate\Validation\Validator` را به کلاس خود وارد نمایید:

	/**
	 * {@inheritdoc}
	 */
	protected function formatErrors(Validator $validator)
	{
		return $validator->errors()->all();
	}

<a name="working-with-error-messages"></a>
## کار با پیغامهای خطا

پس از فراخوانی متد `messages` بر روی نمونه `Validator`، نمونه ای از کلاس `MessageBag` دریافت خواهید نمود، که دارای تعداد زیادی متد برای کار با پیامهای خطاست.

#### بازیابی اولین پیام خطا برای یک فیلد

	echo $messages->first('email');

#### بازیابی تمامی پیامهای خطا برای یک فیلد

	foreach ($messages->get('email') as $message)
	{
		//
	}

#### بازیابی تمامی پیامهای خطا برای تمامی فیلدها

	foreach ($messages->all() as $message)
	{
		//
	}

#### بررسی وجود پیام برای یک فیلد

	if ($messages->has('email'))
	{
		//
	}

#### بازیابی یک پیام خطا با یک قالب

	echo $messages->first('email', '<p>:message</p>');

> **نکته:** به صورت پیش فرض پیامها با استفاده از قالب سازگار با بوت استرپ (Bootstrap) قالب بندی شده اند.

#### بازیابی تمامی پیامهای خطا با یک قالب مشخص

	foreach ($messages->all('<li>:message</li>') as $message)
	{
		//
	}

<a name="error-messages-and-views"></a>
## پیامهای خطا و viewها

پس از انجام درستی سنجی، باید راه سریعی برای انتقال پیامها به view داشته باشید. این کار به راحتی توسط لاراول انجام می شود. routeهای زیر را به عنوان مثال در نظر بگیرید:

	Route::get('register', function()
	{
		return View::make('user.register');
	});

	Route::post('register', function()
	{
		$rules = [...];

		$validator = Validator::make(Input::all(), $rules);

		if ($validator->fails())
		{
			return redirect('register')->withErrors($validator);
		}
	});

توجه داشته باشید هنگامی که درستی رد می شود، با استفاده از متد `withErrors` نمونه ای از کلاس `Validator` را به Redirect میفرستیم. این متد پیامهای خطا را به session اضاف میکند تا در درخواست بعدی قابل استفاده باشند.

توجه کنید نیازی به بایندکردن پیامهای خطا به view در روت GET نیست. دلیل این موضوع چک کردن داده های session توسط لاراول هست، و در صورتی که چیزی پیدا بشه در view قابل دسترسی خواهد بود. **بنابراین توجه به این نکته که متغیر `$errors` در تمام viewها و در تمام درخواستها وجود خواهد داشت**، به شما اطمینان خاطر میدهد که متغیر `$errors` همیشه وجود دارد و بی دغدغه میتوانید از آن استفاده کنید. متغیر `$errors` نمونه ای از کلاس `MessageBag` است.

بنابراین پس از ریدایرکت، میتوانید از متغیر `$errors` که خودکار به session اضافه شده است در view استفاده کنید:

	<?php echo $errors->first('email'); ?>

### بسته های خطای نامگذاری شده - Named Error Bags

اگر چندین فرم بر روی یک صفحه دارید، میتوانید برای `MessageBag` مربوط به خطاها نام تعیین کنید. این کار به شما امکان بازیابی پیامهای خطای مربوط به هر فرم را میدهد. این کار را به سادگی با فرستادن یک نام به آرگومان دوم `withErrors` انجام دهید:

	return redirect('register')->withErrors($validator, 'login');

با نامگذاری میتوانید با استفاده از آن به نمونه کلاس `MessageBag` از متغیر `$errors` دسترسی داشته باشید:

	<?php echo $errors->login->first('email'); ?>

<a name="available-validation-rules"></a>
## قوانین درستی سنجی موجود

در ادامه لیستی از قوانین درستی سنجی موجود و کاربردهای آنها ارائه شده است:

- [مورد پذیرش - Accepted](#rule-accepted)
- [URL فعال - Active URL](#rule-active-url)
- [پس از  (تاریخ) - After (Date)](#rule-after)
- [آلفا - Alpha](#rule-alpha)
- [Alpha Dash](#rule-alpha-dash)
- [Alpha Numeric](#rule-alpha-num)
- [آرایه - Array](#rule-array)
- [پیش از (تاریخ) - Before (Date)](#rule-before)
- [بیش از - Between](#rule-between)
- [Boolean](#rule-boolean)
- [تاییدشده - Confirmed](#rule-confirmed)
- [تاریخ - Date](#rule-date)
- [قالب تاریخ - Date Format](#rule-date-format)
- [اختلاف - Different](#rule-different)
- [رقمها - Digits](#rule-digits)
- [رقمهای بین - Digits Between](#rule-digits-between)
- [E-Mail](#rule-email)
- [وجود داشتن( پایگاه داده) - Exists (Database)](#rule-exists)
- [عکس (فایل) - Image (File)](#rule-image)
- [In](#rule-in)
- [Integer](#rule-integer)
- [IP Address](#rule-ip)
- [Max](#rule-max)
- [MIME Types](#rule-mimes)
- [Min](#rule-min)
- [Not In](#rule-not-in)
- [Numeric](#rule-numeric)
- [Regular Expression](#rule-regex)
- [Required](#rule-required)
- [Required If](#rule-required-if)
- [Required With](#rule-required-with)
- [Required With All](#rule-required-with-all)
- [Required Without](#rule-required-without)
- [Required Without All](#rule-required-without-all)
- [Same](#rule-same)
- [Size](#rule-size)
- [String](#rule-string)
- [Timezone](#rule-timezone)
- [Unique (Database)](#rule-unique)
- [URL](#rule-url)

<a name="rule-accepted"></a>
#### پذیرفته شده - accepted

فیلدی که درsتی سنجی می شود باید دارای مقادیر _yes_، _no_، _1_، یا _true_ باشد. این قانون برای درستی سنجی پذیرفتن "قوانین خدمات  - Terms of Service" کاربرد دارد.

<a name="rule-active-url"></a>
#### active_url

فیلدی که درستی سنجی می شود باید براساس تابع `checkdnsrr` در PHP باید یک URL صحیح باشد.

<a name="rule-after"></a>
#### after:_date_

فیلدی که درستی سنجی می شود باید حاوی مقداری پس از تاریخ ارائه شده باشد. تاریخها به تابع `strtotime` از PHP فرستاده می شوند.

<a name="rule-alpha"></a>
#### alpha

فیلدی که مورد درستی سنجی قرار میگیرد باید تمام حاوی کاراکترهای حروف باشد.

<a name="rule-alpha-dash"></a>
#### alpha_dash

فیلد مورد نظرسنجی میتواند حاوی کاراکترهای عددی و حرف، همچنین خط تیره "-" و همینطور "_" باشد.

<a name="rule-alpha-num"></a>
#### alpha_num

فیلد مورد درستی سنجی باید تمام حاوی کاراکترهای عدی-رقمی باشد.

<a name="rule-array"></a>
#### array

فیلد مورد درستی سنجی باید از نوع آرایه باشد.

<a name="rule-before"></a>
#### before:_date_

فیلدی که درستی سنجی میشود باید تاریخی پیش از تاریخ داده شده باشد. تاریخها به تابع `strtotime` از PHP فرستاده می شوند.

<a name="rule-between"></a>
#### between:_min_,_max_

فیلدی که درستی سنجی می شود باید مقداری بین یک مقدار _min_ و _max_ ارائه شده داشته باشد. stringها، اعداد، و فایلها مشابه قانون `size` درستی سنجی می شوند.

<a name="rule-boolean"></a>
#### boolean

فیلدی که درستی سنجی می شود باید امکان تبدیل نوع (cast) به بولین را داشته باشد. ورودیهای مورد قبول عبارتند از `true`، `false`، `1`، `0`، `"1"`، `"0"`.

<a name="rule-confirmed"></a>
#### confirmed

فیلدی که درستی سنجی می شود باید یک فیلد مشابه `foo_confirmation` داشته باشد. برای مثال، اگر فیلدی که درستی سنجی می شود `password` باشد، یک فیلد `password_confirmation` متناظر باید در ورودی موجود باشد.

<a name="rule-date"></a>
#### date

با توجه به تابع `strtotime` در PHP فیلد مورد درستی سنجی باید یک تاریخ معتبر باشد.

<a name="rule-date-format"></a>
#### date_format:_format_

فیلد مورد درستی سنجی باید با _format_ ارائه شده همخوانی داشته باشد. این فرمت با توجه به تابع `date_parse_from_format` در PHP تعیین می شود.

<a name="rule-different"></a>
#### different:_field_

_field_ داده شده باید با فیلدی که درستی سنجی می شود متفاوت باشد.

<a name="rule-digits"></a>
#### digits:_value_

فیلد مورد درستی سنجی باید "عددی" باشد و باید طولی دقیق به اندازه _value_ داشته باشد.

<a name="rule-digits-between"></a>
#### digits_between:_min_,_max_

فیلدی که درستی سنجی می شودباید طولی بین _min_ و _max_ داشته باشد.

<a name="rule-email"></a>
#### email

فیلدی که درستی سنجی می شود باید قالب ایمیل داشته باشد.

<a name="rule-exists"></a>
#### exists:_table_,_column_

فیلدی که درستی سنجی می شود باید در جدول ارائه شده موجود باشد.

#### استفاده ابتدایی از قانون exists

	'state' => 'exists:states'

#### تعیین یک نام ستون اختصاصی

	'state' => 'exists:states,abbreviation'

میتوان شرطهای بیشتری که مانند عبارات "where" در پرس و جو عمل میکند مشخص نمود:

	'email' => 'exists:staff,email,account_id,1'

ارائه مقدار `NULL` به عنوان عبارت "where" یک مقدار پایگاه داده را با مقدار `NULL` مقایسه میکند:

	'email' => 'exists:staff,email,deleted_at,NULL'

<a name="rule-image"></a>
#### image

فایلی که درستی سنجی می شود باید عکس (jpeg ،png ،bmp، gif، svg) باشد.

<a name="rule-in"></a>
#### in:_foo_,_bar_,...

فیلدی که درستی سنجی می شود باید در لیست مقادیر ارائه شده موجود باشد.

<a name="rule-integer"></a>
#### integer

فیلدی که درستی سنجی می شود باید مقدار integer داشته باشد.

<a name="rule-ip"></a>
#### ip

فیلدی که درستی سنجی می شود، باید در قالب یک آدرس IP باشد.

<a name="rule-max"></a>
#### max:_value_

فیلدی که درستی سنجی می شود باید از مقدار بیشینه _value_ کمتر یا با آن مساوی باشد. stringها، اعداد (numeric)، مشابه روش گفته شده در مورد قانون [`size`](#rule-size) درستی سنجی می شوند.

<a name="rule-mimes"></a>
#### mimes:_foo_,_bar_,...

فایلی که درستی سنجی می شود باید نوع MIME متناسب با آنچه در لیست پسوندها آورده شده است داشته باشد.

#### کاربرد ابتدایی قوانین MIME

	'photo' => 'mimes:jpeg,bmp,png'

<a name="rule-min"></a>
#### min:_value_

فیلدی که درستی سنجی می شود باید مقدار کمینه _value_ را دارا باشد. stringها، اعداد (numeric)، مشابه روش گفته شده در مورد قانون [`size`](#rule-size) درستی سنجی می شوند.

<a name="rule-not-in"></a>
#### not_in:_foo_,_bar_,...

فیلدی که درستی سنجی می شود نباید در لیست مقادیر ارائه شده قرار داشته باشد.

<a name="rule-numeric"></a>
#### numeric

فیلدی که درستی سنجی می شود باید مقداری عددی داشته باشد.

<a name="rule-regex"></a>
#### regex:_pattern_

فیلدی که درستی سنجی می شود باید با الگوی regular expression همخوانی داشته باشد.

**نکته** هنگام استفاده از الگوی Regex باید به جای جدا کردن قوانین با استفاده از پایپ "|" باید آنها را در آرایهارائه دهید. این اتفاق زمانی که regular expression شامل پایپ باشد اهمییت پیدا میکند.

<a name="rule-required"></a>
#### required

فیلدی که درستی سنجی می شود باید در ورودیها موجود باشد.

<a name="rule-required-if"></a>
#### required_if:_field_,_value_,...

در صورتی که مقدار _field_ برابر با مقدار _any_ باشد، فیلدی که درستی سنجی می شود باید در ورودیها موجود باشد.

<a name="rule-required-with"></a>
#### required_with:_foo_,_bar_,...

تنها در صورتی که هر یک از فیلدهای مشخص شده دیگر وجود داشته باشند، فیلدی که درستی سنجی می شود باید در ورودیها موجود باشد.

<a name="rule-required-with-all"></a>
#### required_with_all:_foo_,_bar_,...

تنها در صورتی که تمامی فیلدهای مشخص شده دیگر وجود داشته باشند، فیلدی که درستی سنجی می شود باید در ورودیها موجود باشد.

<a name="rule-required-without"></a>
#### required_without:_foo_,_bar_,...

تنها در صورتی که هر یک از فیلدهای مشخص شده دیگر وجود نداشته باشند، فیلدی که درستی سنجی می شود باید در ورودیها موجود باشد.

<a name="rule-required-without-all"></a>
#### required_without_all:_foo_,_bar_,...

تنها در صورتی که تمامی فیلدهای مشخص شده دیگر وجود نداشته باشند، فیلدی که درستی سنجی می شود باید در ورودیها موجود باشد.

<a name="rule-same"></a>
#### same:_field_

فیلد مشخص شده باید با فیلدی که در حال درستی سنجی است یکسان باشد.

<a name="rule-size"></a>
#### size:_value_

فیلدی که درستی سنجی می شود باید اندازه ای برابر با مقدار _value_ داده شده داشته باشد. برای داده های string این نکته به تعداد کاراکترها اشاره میکند. برای داده های عددی، مقدار _value_ به مقدار عدد صحیح داده شده اشاره میکند. برای فایلها _size_ به اندازه فایل برحسب کیلوبایت اشاره میکند.

<a name="rule-string"></a>
#### string

فیلدی که درستی سنجی می شود باید باید از جنس string باشد.

<a name="rule-timezone"></a>
#### ناحیه زمانی - timezone

فیلدی که درستی سنجی می شود باید براساس تابع `timezone_identifiers_list` در PHP شناسه ناحیه زمانی درستی باشد.

<a name="rule-unique"></a>
#### unique:_table_,_column_,_except_,_idColumn_

فیلدی که درستی سنجی می شود باید در جدول پایگاه داده ارائه شده یکتا باشد. در صورتی که گزینه `column` مشخص نشده باشد، نام فلید استفاده می شود.

گاهی نیاز دارید برای پرس و جوهای انجام شده توسط validator یک اتصال اختصاصی تعریف کنید. همانطور که پیش از این نمایش داده شد، قراردادن `unique:users` به عنوان یک قانون درستی سنجی، برای پرس و جوی پایگاه داده از اتصال پیش رض استفاده میکند. برای استفاده از اتصال پایگاه داده اختصاصی، از روش زیر استفاده نمایید:

	$verifier = App::make('validation.presence');

	$verifier->setConnection('connectionName');

	$validator = Validator::make($input, [
		'name' => 'required',
		'password' => 'required|min:8',
		'email' => 'required|email|unique:users',
	]);

	$validator->setPresenceVerifier($verifier);

#### استفاده ابتدایی از قوانین یکتایی

	'email' => 'unique:users'

#### تعیین یک نام اختصاصی برای ستون

	'email' => 'unique:users,email_address'

#### اعمال قانون یکتایی بر تمام موارد جز یک ID مشخص شده

	'email' => 'unique:users,email_address,10'

#### افزودن شرطهای بیشتر

میتوانید شرطهای بیشتری را بیافزایید که به عنوان عبارات "where" به پرس و جو افزوده می شوند.

	'email' => 'unique:users,email_address,NULL,id,account_id,1'

در قانون بالا، تنها ردیفهایی با مقدار `account_id` برابر با `1` در بررسی یکتایی قرار خواهند گرفت.

<a name="rule-url"></a>
#### url

فیلدی که درستی سنجی می شود باید با قالب URL باشد.

> **Note:** This function uses PHP's `filter_var` method.

<a name="conditionally-adding-rules"></a>
## افزودن مشروط قوانین

گاهی، میخواهید درستی سنجی **تنها** در صورتی انجام شود که فیلد مورد نظر درآرایه ورودی ها موجود باشد. برای رسیدن به این هدف، یک قانون `sometimes` به لیست قوانین بیافزایید:

	$v = Validator::make($data, [
		'email' => 'sometimes|required|email',
	]);

در مثال بالا، فیلد `email` تنها در صورتی که در آرایه `$data` وجود داشته باشد، درستی سنجی می شود.

#### درستی سنجی شرطی پیچیده

گاهی وجود یک فیلد در صورتی الزامیست که مقدار فیلدی دیگر از 100 بزرگتر باشد. یا مقدار دو فیلد در صورتی که فیلدی مشخص وجود داشته باشد، باید با مقداری برابری کند. افزودن این قوانین درستی سنجی نباید دردسر زیادی ایجاد کند. ابتدا یک نمونه از `Validator` با قوانین ثابتی که هیچگاه تغییر نمیکنند بسازید:

	$v = Validator::make($data, [
		'email' => 'required|email',
		'games' => 'required|numeric',
	]);

فرض کنید نرم افزار وب ما برای کلکسیونرهای بازی باشد. در صورتی که یک کلکسیونر در سرویس ما ثبت نام نماید و بیش از 100 بازی داشته باشد، از آنها میخواهیم دلیل جمع آوری این تعداد بازی را توضیح دهد. برای مثال، شاید یک فروشگاه فروش بازی را اداره می کنند، یا تنها از جمع آوری بازیها لذت میبرند. برای افزودن مشروط این نیازمندیها، میتوانیم از متد `sometimes` بر روی نمونه کلاس `Validator` استفاده کنیم:

	$v->sometimes('reason', 'required|max:500', function($input)
	{
		return $input->games >= 100;
	});

آرگومان اولی که به متد `sometimes` فرستاده می شود نام فیلدی است که درستی آن مشروط سنجیده می شود. آرگومان دوم قوانینی هستند که میخواهیم بیافزاییم. اگر `Closure` فرستاده شده به عنوان آرگومان سوم، مقدار خروجی `true` داشته باشد، قوانین افزوده می شوند. این متد ساخت قوانین درستی سنجی شرطی پیچیده را بسیار ساده میکند. حتی میتوانید قوانین شرظی را برای چندین فیلد همزمان بیافزایید:

	$v->sometimes(['reason', 'cost'], 'required', function($input)
	{
		return $input->games >= 100;
	});

> **نکته:** پارامتر ورودی `$input` که به `Closure` میفرستید، نمونه ای از کلاس `Illuminate\Support\Fluent` است و به عنوان یک شی برای دسترسی به ورودی و فایلها استفاده می شود.

<a name="custom-error-messages"></a>
## پیامهای خطای اختصاصی شده

در صورت نیاز میتوانید به جای پیامهای خطای پیش فرض از پیامهای خطای اختصاصی شده استفاده نمایید. راههای بسیاری برای مشخص کردن پیامهای اختصاصی شده وجود دارد.

#### فرستادن پیامهای اختصاصی به Validator

	$messages = [
		'required' => 'The :attribute field is required.',
	];

	$validator = Validator::make($input, $rules, $messages);

> **نکته:** فضانگهدار `:attribute` با نام فیلدی که درستی سنجی بر روی آن انجام میشود جایگزین خواهد شد. شما نیزمیتوانید از فضانگهدارهای دیگر در پیامهای درستی سنجی استفاده کنید:

#### فضانگهدارهای دیگر درستی سنجی

	$messages = [
		'same'    => 'The :attribute and :other must match.',
		'size'    => 'The :attribute must be exactly :size.',
		'between' => 'The :attribute must be between :min - :max.',
		'in'      => 'The :attribute must be one of the following types: :values',
	];

#### ارائه یک پیام اختصاصی برای یک متغیر

گاهی میخواهید تنها برای یک فیلد پیام خطای اختصاصی ایجاد کنید:

	$messages = [
		'email.required' => 'We need to know your e-mail address!',
	];

<a name="localization"></a>
#### ارائه پیام خطای اختصاصی در فایل زبان

برخی مواقع به جای فرستادن پیامهای اختصاصی به `Validator` میخواهید پیام خطا را در یک فایل پیامهای زبان ارائه کنید. برای این کار، پیامها را به آرایه `custom` در فایل زبان `resources/lang/xx/validation.php` بیافزایید.

	'custom' => [
		'email' => [
			'required' => 'We need to know your e-mail address!',
		],
	],

<a name="custom-validation-rules"></a>
## قوانین درستی سنجی اختصاصی

#### ثبت یک قانون درستی سنجی اختصاصی

لاراول قوانین درستی سنجی کاربردی دارد؛ هرچند، شاید نیاز داشته باشید قوانین خود را تعریف کنید. یک راه برای ثبت و معرفی قوانین درستی سنجی استفاده از متد `Validator::extend` میباشد:

	Validator::extend('foo', function($attribute, $value, $parameters)
	{
		return $value == 'foo';
	});

کلوژر اختصاصی validator سه آرگومان دریافت میکند: نام `$attribute` که درستی آن بررسی می شود، مقدار `$value`، و آرایه ای از `$parameters` که به قانون فرستاده می شود.

میتوانید به جای فرستادن کلوژر به متد `extend`، کلاس و متد بفرستید:

	Validator::extend('foo', 'FooValidator@validate');

توجه داشته باشید که باید برای قوانین اختصاص یک پیام خطا هم تعریف نمایید. این کار را همزمان با تعریف قانون از طریق ارائه آرایه ی پیامها و یا با افزودن یک پیام به فایل زبان انجام دهید.

#### اکستند کلاس Validator

به جای استفاده از کلوژرهای کلابک برای توسعه Validator، میتوانید از خود کلاس Validator را اکستند کنید. برای این کار، یک کلاس Validator که کلاس `Illuminate\Validation\Validator` را اکستند میکند ایجاد کنید. با افزودن پیشوند `validate` به متدها میتوانید متدهای درستی سنجی بسازید:
	<?php

	class CustomValidator extends \Illuminate\Validation\Validator {

		public function validateFoo($attribute, $value, $parameters)
		{
			return $value == 'foo';
		}

	}

#### ثبت معرفی کننده Validator اختصاصی

در قدم بعد باید Validator اختصاصی ایجاد شده را ثبت و معرفی کنید:

	Validator::resolver(function($translator, $data, $rules, $messages)
	{
		return new CustomValidator($translator, $data, $rules, $messages);
	});

وقتی یک قانون درستی سنجی اختصاصی میسازید، میتوانید برای پیامهای خطا فضانگهدارهایی قرار دهید. این کار را با افزودن یک تابع `replaceXXX` به validator انجام دهید.

	protected function replaceFoo($message, $attribute, $rule, $parameters)
	{
		return str_replace(':foo', $parameters[0], $message);
	}
	
اگر میخواهید بدون اکستند کردن کلاس `Validator` یک جایگزین گننده پیام اختصاصی ایجاد کنید، میتوانید از متد `Validator::replacer` استفاده کنید:

	Validator::replacer('rule', function($message, $attribute, $rule, $parameters)
	{
		//
	});