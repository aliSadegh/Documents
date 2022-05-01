# HDFS
## معماری
		- HDFS برای نرم افزارهایی مناسب است که پردازش دسته ای (batch) دارند بجای پردازش های تعاملی با کاربر.
	- برای فایل های با حجم زیاد مناسب است. مثلا بیشتر از 1 گیگ.
	- بهینه شده برای حالتی که یک بار فایل ساخته و نوشته شود و بارها خوانده شود. برای حالت هایی که فایل دائما بروز میشود مناسب نیست.
	- انتقال عملیات های مختلف نرم افزار یه کنار دیتا، کم هزینه تر است از حالتی که دیتا منتقل بشن به کنار عملیات ها و بعد دوباره دیتاها منتقل بشن به فضای ذخیره سازی (Moving Computation is Cheaper than Moving Data)
	- به گونه ای طراحی شده که بتواند با پلتفرم های مختلف به خوبی کار کند.
	- NameNode and DataNode:
		○ NameNode وظیفه مدیریت کردن دیتا نود ها و عملیات باز کردن و بستن و تغییر نام فایل ها و فولدرها هست و همچینن نگاشت بلاک ها در دیتا نود ها را دارد.
		○ DataNode وظیفه خواندن و نوشتن بلاک ها را بر عهده دارد.
	- HDFS سافت لینک و هارد لینک را پشتیبانی نمیکنه
	- سایز بلاک و تعداد کپی های بلاک میتواند به ازای هر فایل تغییر شود.
	- NameNode به صورت دوره ای Heartbeat و Blockreport از دیتا نود دریافت میکند، هارتبیت مشخص میکنه که دیتانود زنده هست و داره کار میکنه، و بلاک ریپورت اطلاعات تمام بلاک های داخل دیتا نود را مشخص میکند.
	- Hadoop Rack Awareness:
		○ هدوپ برای افزایش سرعت خواندن و همچنین حفظ دیتا در شرایط خرابی، کپی هایی از بلاک ها میسازد ولی از طرفی این کپی ها سرعت نوشتن را کم میکند برای اینکه سرعت نوشتن خیلی پایین نیاد، هدوپ از روشی جالب استفاده میکند. دو سوم بلاک های کپی را در یک رک نگه داری میکند و یک سوم دیگر را در رکی دیگر، اینطوری هم سرعت نوشتن حفظ میشود و هم سرعت خواندن در شرایط خرابی و هم اینکه اگر یک رک تماما از کلاستر خارج بشود، باز هم دیتا در رکی دیگر وجود دارد.
	- در ابتدای اجرای NameNode، NameNode به حالت سیف مود میرود، در این حالت هیچ کپی کردن از بلاک ها اتفاق نمیافته، بلاک ریپورت ها از دیتا نود ها گرفته میشه و چک میشه که تمامی بلاک ها حداقل تعداد کپی رو رعایت کرده باشن، بعد از تموم شدن این بررسی، حالت سیف مود تموم میشه و اگر نیازی باشه که از بلاک هایی کپی گرفته بشه تا به حداقل تعداد برسه، این عملیات انجام میشه
	- فایلی به نام EditLog در نیم نود وجود دارد که شامل همه اتفاقاتی است که در کلاستر برای فایلها افتاده، هر اتفاقی که در فایل ها میافتد و یا فایلی ساخته میشود یک رکورد به Editlog اضافه میشود.
	- تمام نگاشت های بلاک ها به فایل ها، مشخصات فایل ها و تمامی اطلاعات مربوط به فایل سیستم ها در فایلی به نام FSImage در نود نیم به صورت فایل ذخیره میشود.
	- بعد از هر بار بالا اومدن نیم نود یا به صورت دوره زمانی(CheckPoint)، Editlog خوانده شده و تغییرات آن به FSImage اضافه میشود و editlog خالی میشود
	- ارتباط بین نیم نود و دیتا نود از طریق RPC که بر روی TCP است، برقرار میشود. نیم نود هیچ وقت ارتباط با دیتانود را ایجاد نمیکنه بلکه فقط به درخواست دیتانود پاسخ میده (و همین شرایط برای کلاینت وجود دارد)
	-  دیتانود هارت بیت به نیم نود میفرسته و اگر نیم نود تا 10 دقیقه(به صورت پیشفرض) هارت بیت از دیتا نود نگیره، اونو مرده فرض میکنه و شروع میکنه کپی کردن بلاک هایی که در اون دیتا نود بود، در بقیه کلاستر
	- متادیتا در HDFS اهمیت بالایی داره و اگر متادیتا که در نیم نود وجود داره به مشکل بخوره، کل کلاستر پایین میاد، برای همین میتونیم از متد Distributed Editlog استفاده کنیم (Journal)
	- HDFS دارای سطل زباله هست، به این صورت که فایلهایی که از طریق FS Shell حذف میشود ابتدا به پوشه سطل زباله رفته و بعد از مدتی که تاریخ انقضای آن بگذرد حذف میشود (کاربر میتواند از امکان سطل زباله استفاده نکند)
