# قالبها - Templates

- [کتابخانه ساخت قالب Blade](#blade-templating)
- [دیگر ساختارهای کنترل Blade](#other-blade-control-structures)

<a name="blade-templating"></a>
## کتابخانه ساخت قالب Blade

Blade یک موتور ساده و در عین حال قدرتمند ایجاد قالب است که همراه لاراول ارائه می شود. برخلاف قالبهای کنترلر، کار Blade برپایه _ارث بری قالبها_ و _بخشها_ بنا شده است. تمامی قالبهای Blade باید از پسوند `blade.php.` استفاده کنند.

#### تعریف یک لی اوت(layout) مبتنی بر Blade

	<!-- Stored in resources/views/layouts/master.blade.php -->

	<html>
		<head>
			<title>App Name - @yield('title')</title>
		</head>
		<body>
			@section('sidebar')
				This is the master sidebar.
			@show

			<div class="container">
				@yield('content')
			</div>
		</body>
	</html>

#### نحوه استفاده از لی اوت Blade

	@extends('layouts.master')
	
	@section('title', 'Page Title')

	@section('sidebar')
		@@parent

		<p>This is appended to the master sidebar.</p>
	@stop

	@section('content')
		<p>This is my body content.</p>
	@stop

اگر به viewها دقت کنید یک لی اوت Blade را `extend` میکند. با این کار میتوانید بخشهای مشخص شده جایگزین کنید. در viewهایی که این لی اوت را استفاده میکنند، محتوای لی اوت با استفاده از راهنمای (directive) تعریف شده `parent@@` قابل دسترس است. در این بخشها میتوان محتوای لی اوت را با بخشهایی مانند منوکنار (side bar)، و یا فوتر جانمایی کرد.

گاهی که از تعریف شدن یک بخش اطمینان ندارید، میتوانید یک مقدار پیش فرض به راهنمای `yield@` بفرستید. مقدار پیش فرض را میتوانید به عنوان پارامتر دوم بفرستید:

	@yield('section', 'Default Content')

<a name="other-blade-control-structures"></a>
## دیگر ساختارهای کنترلی Blade

#### چاپ داده

	Hello, {{ $name }}.

	The current UNIX timestamp is {{ time() }}.

#### چاپ داده پس از بررسی وجود

گاهی میخواهید داده ای را چاپ نمایید اما از مقداردهی شدن آن اطمینان ندارید. به سادگی میتوانید روش زیر را استفاده کنید:

	{{ isset($name) ? $name : 'Default' }}

Blade به جای استفاده از عبارت ترنری (ternary) بالا، امکان استفاده از ساختار زیر را فراهم میکند:

	{{ $name or 'Default' }}

#### نمایش متن خام در آکولاد

اگر میخواهید یک رشته متن را که در آکولاد قراردارد نمایش دهید، میتوانید با استفاده از علامت `@` پیش از متن، از تفسیر آکولادها توسط Blade جلوگیری کنید:

	@{{ این متن توسط Blade پردازش نمیشود }}

اگر میخواهید داده های درون متن هنوز نمایش داده شوند از قالب زیر استفاده کنید:

	Hello, {!! $name !!}.

> **نکته:** هنگام چاپ محتوایی که توسط کاربران نرم افزار ارائه شده اند بسیار مراقب باشید. همیشه برای نمایش موجودیتهای HTML درون متن از جفت-آکولاد استفاده کنید.

#### عبارات if

	@if (count($records) === 1)
		I have one record!
	@elseif (count($records) > 1)
		I have multiple records!
	@else
		I don't have any records!
	@endif

	@unless (Auth::check())
		You are not signed in.
	@endunless

#### حلقه های تکرار

	@for ($i = 0; $i < 10; $i++)
		The current value is {{ $i }}
	@endfor

	@foreach ($users as $user)
		<p>This is user {{ $user->id }}</p>
	@endforeach

	@forelse($users as $user)
		<li>{{ $user->name }}</li>
	@empty
		<p>No users</p>
	@endforelse

	@while (true)
		<p>I'm looping forever.</p>
	@endwhile

#### استفاده از زیرviewها

	@include('view.name')

میتوانید آرایه ای از داده به view استفاده شده بفرستید:

	@include('view.name', ['some' => 'data'])

#### بازنویسی بخشها

برای بازنویسی کل یک بخش، میتوانید از عبارت `overwrite` استفاده کنید:

	@extends('list.item.container')

	@section('list.item.content')
		<p>This is an item of type {{ $item->type }}</p>
	@overwrite

#### نمایش خطوط زبان

	@lang('language.line')

	@choice('language.line', 1)

#### کامنتها

	{{-- این بخش در HTML نهایی نخواهد بود --}}

