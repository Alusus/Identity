# Identity
[[English]](readme.md)

<div dir=rtl>

مكتبة لتمكين ربط تطبيقات مـنصة_ويب مع خدمات المصادقة العامة، مثل خدمة جوجل للمصادقة التي تمكن المستخدمين
من تسجيل الدخول باستخدام حسابهم على جيميل. صممت لتدعم خدمات مصادقة متعددة لكنها في الوقت الحالي تدعم خدمة
جوجل فقط.

## الاستخدام

تفترض هذه الخطوات أن لديك بالفعل مشروعًا على منصة الويب وتريد إضافة المصادقة إليه.

* أضف المكتبة إلى مشروعك:

```
اشمل "مـحا"؛
مـحا.اشمل_ملف("Alusus/Identity"، "هـوية.أسس")؛
```

<div dir=ltr>

```
import "Apm";
Apm.importFile("Alusus/Identity");
```

</div>

* انشئ تعريفا لتطبيقك في خدمة جوجل من خلال زيارة الموقع https://console.cloud.google.com/apis/ ثم
  اضف معرف OAuth 2.0 هذا التطبيق عبر خيار Credentials في القائمة. انسخ قيمتي clientId و clientSecret منها.

* عبر خيار OAuth consent screen حدد الScopes المطلوبة للتطبيق. ستحتاج على الأقل لopenid و email و profile.

* أنشئ مفتاح RSA لاستخدامه في توثيق معرفات المصادقة وخزنه في ملف ضمن مشروعك. يجب أن يعتمد المفتاح ترميز RSA 256.

* هيئ واجهة REST للمكتبة باستدعاء دالة `هيئ_واجهة_رست` واعطائها اسم ملف مفتاح RSA الخاص بالإضافة إلى مغلفة تستدعى
  عند تسجيل الدخول لإنشاء حساب للمستخدم وإرجاع معرفه الخاص.

```
هـوية.هيئ_واجهة_رست("rsakey"، مغلفة (جواب: هـوية.جـواب_مصادقة): نـص {
    // استخدم بيانات المستخدم المضمنة في `جواب` لإيجاد حساب المستخدم في قاعدة بيانات المشروع، أو إنشاء قيد
    // جديد له إن كان المستخدم جديدًا. هذه المغلفة يجب أن ترجع معرفًا فريدًا للمستخدم.
    أرجع معرف_المستخدم؛
})؛
```

<div dir=ltr>

```
Identity.setupRestApi("rsakey", closure (res: Identity.AuthenticationResponse): String {
    // Use user info from res to lookup the user record in the project's DB, or to create a new
    // record if this is a new user. This closure should return the user's unique id.
    return userId;
}
```

</div>

* أضف مزود جوجل لقائمة مزودي المكتبة:

```
عرف معرف_تطبيق_جوجل: "<قيمة clientId المأخوذة من خدمة مصادقة جوجل>"؛
عرف كلمة_سر_جوجل: "<قيمة clientSecret المأخوذة من خدمة مصادقة جوجل>"؛
هـوية.أضف_مزودا(هـوية.مـزود_جوجل(نـص(معرف_تطبيق_جوجل)، نـص(كلمة_سر_جوجل)))؛
```

<div dir=ltr>

```
def clientId: "<the value of clientId from Google's dashbaord>";
def clientSecret: "<the value of clientSecret from Google's dashboard>";
Identity.addProvider(Identity.GoogleProvider(String(clientId), String(clientSecret)));
```

</div>

* أضف زر تسجيل الدخول باستخدام جوجل إلى موقعك:

```
عرف عنوان_إعادة_التحويل: "العنوان ضمن موقعك الذي سيُرسل المستخدم إليه بعد المصادقة لإتمام تسجيل الدخول"؛
هـوية.كـبسة_مصادقة_جوجل(نـص(معرف_تطبيق_جوجل)، نـص(عنوان_إعادة_التحويل)، 0).{
    // تخصيص الطراز.
}
```


<div dir=ltr>

```
def redirectUrl: "the URL within your site to send the user to after authorizing to complete the login";
Identity.GoogleAuthenticationButton(String(clientId), String(redirectUrl), 0).{
    // Customize styles.
}
```

</div>

المعطى الثالث لهذا المركب يحدد ما إن كان المطلوب نمط الزر الداكن أو الفاتح. القيمة 0 تشير للنمط الفاتح
بينما تشير القيمة 1 إلى النمط الداكن.

* في صفحة إكمال تسجيل الدخول (الصفحة التي سينتقل المستخدم إليها بعد المصادقة) أضف التالي لإكمال عملية
  المصادقة وتسجيل الدخول:

```
إذا انتظر[ثـنائي](هـوية.أكمل_المصادقة(عنوان_إعادة_التحويل)) {
    // اكتملت عملية تسجيل الدخول.
    // نزيل معطيات جوجل من شريط العنوان بنقل المستخدم إلى صفحة أخرى (قد تكون نفس هذه الصفحة).
    نـافذة.النموذج.ادفع_مسارا(عنوان_إعادة_التحويل)؛
    // الآن تسجيل الدخول مكتمل ويمكن الآن إكمال عمل التطبيق.
} وإلا {
    // فشلت عملية المصادقة، أو أنه لم تكن هناك عملية مصادقة (لا توجد معطيات المصادقة في شريط العنوان).
}
```

<div dir=ltr>

```
if await[Bool](Identity.finalizeAuthentication(redirectUrl)) {
    // Authentication is complete.
    // Remove auth params from the URL bar by redirecting the user to another page, or to the same page.
    Window.instance.pushLocation(redirectUrl);
    // Login is complete now, and we can proceed with the app.
} else {
    // Authentication failed, or there were no auth params in the URL.
}
```

</div>

* للتأكد من حالة تسجيل الدخول وتجديد رمز الولوج تلقائيًا (باستخدام رمز التجديد، أي refresh token):

```
عرف داخل: ثـنائي = انتظر[ثـنائي](هـوية.تحقق_من_حالة_المصادقة())؛
إذا ليس داخل {
    // أظهر صفحة تسجيل الدخول.
}
```

<div dir=ltr>

```
def loggedIn: Bool = await[Bool](Identity.checkAuthenticationStatus());
if not loggedIn {
    // Show login screen.
}
```

</div>

### استخدام رمز الولوج للعمليات الموثقة

عند استدعاء منفذ موثق (authenticated) في الخادم تحتاج لتضمين رمز الولوج في طلبك. طريقة تضمين الرمز
عائد لك (يمكن إضافته للترويسة على سبيل المثال). يمكن الحصول على رمز الولوج من الزبون باستدعاء دالة
`هات_رمز_الولوج` (`getAccessToken`):

```
عرف رمز_الولوج: نـص = هات_رمز_الولوج()؛
```

<div dir=ltr>

```
def accessToken: String = getAccessToken();
```

</div>

بعد استلام رمز الولوج من جهة الخادم تحتاج للتحقق منه باستخدام دالة `تحقق_من_رمز_الولوج` (`validateAccessToken`).

### تسجيل الخروج

يمكن تسجيل خروج المستخدم بإزالة بيانات المصادقة من المتصفح، كما يلي:

```
أزل_بيانات_المصادقة()؛
```

<div dir=ltr>

```
clearAuthenticationData();
```

</div>

## الدالات والأصناف

### صادق (authenticate)

```
دالة صادق(اسم_المزود: نـص، رمز_المصادقة: نـص، عنوان_إعادة_التوجيه: نـص): لـا_مضمون[جـواب_مصادقة]؛
```

<div dir=ltr>

```
function authenticate(providerName: String, code: String, redirectUri: String): Possible[AuthenticationResponse];
```

</div>

دالة لإكمال عملية المصادقة من جهة الخادم. تُستدعى هذه الدالة داخليًا من قبل منافذ البيانات التابعة
للمكتبة.

* `اسم_المزود` (`providerName`): اسم المزود الذي يُستخدم لعملية المصادقة (على سبيل المثال: `google`).
* `رمز_المصادقة` (`code`): رمز المصادقة الذي يوفره مزود الخدمة للصفحة التي يُعاد توجيه المستخدم إليها
  عقب تصريحه بالموافقة.
* `عنوان_إعادة_التوجيه` (`redirectUri`): عنوان الصفحة التي يُعاد توجيه المستخدم إليها عقب تصريح الموافقة.

### جدد_المصادقة (reauthenticate)

```
دالة جدد_المصادقة(اسم_المزود: نـص، رمز_التجديد: نـص): لـا_مضمون[جـواب_مصادقة]؛
```

<div dir=ltr>

```
function reauthenticate(providerName: String, refreshToken: String): Possible[AuthenticationResponse];
```

</div>

دالة لإنشاء رمز ولوج خارجي جديد باستخدام رمز التجديد. تُستخدم هذه الدالة من قبل منافذ البيانات
التابعة للمكتبة.

* `اسم_المزود` (`providerName`): اسم المزود الذي يُستخدم لعملية المصادقة (على سبيل المثال: `google`).
* `رمز_التجديد` (`refreshToken`): رمز التجديد الذي يوفره مزود الخدمة عند عملية المصادقة الأولية.

### هات_بيانات_المستخدم (getUserInfo)

```
دالة هات_بيانات_المستخدم(اسم_المزود: نـص، رمز_الولوج_الخارجي: نـص): لـا_مضمون[جـواب_بيانات_المستخدم]؛
```

<div dir=ltr>

```
function getUserInfo(providerName: String, extAccessToken: String): Possible[UserInfoResponse];
```

</div>

تجلب بيانات المستخدم من مزود الخدمة.

* `اسم_المزود` (`providerName`): اسم المزود الذي يُستخدم لعملية المصادقة (على سبيل المثال: `google`).
* `رمز_الولوج_الخارجي` (`extAccessToken`): رمز ولوج المستخدم التابع لمزود الخدمة.

### تحقق_من_رمز_الولوج (validateAccessToken)

```
دالة تحقق_من_رمز_الولوج(
    رمز_الولوج: نـص، اسم_المزود: سند[نـص]، رمز_الولوج_الخارجي: سند[نـص]، معرف_المستخدم: سند[نـص]
): ثـنائي؛
```

<div dir=ltr>

```
function validateAccessToken(
    token: String, providerName: ref[String], extAccessToken: ref[String], userId: ref[String]
): Bool
```

</div>

ترجع هذه الدالة قيمة صحيحة (1) إن كان رمز الولوج صالحًا، وبعكسه ترجع 0.

* `رمز_الولوج` (`token`): رمز الولوج المطلوب التحقق منه.
* `اسم_المزوج` (`providerName`): في حال كان الرمز صالحًا يتم تحديد قيمة هذا المتغير باسم المزود الذي
  يتبع له رمز الولوج. حاليًا المكتبة تدعم مزود جوجل فقط، لذلك ستكون قيمة هذا المتغير `google`.
* `رمز_الولوج_الخارجي` (`extAccessToken`): في حال كان الرمز صالحًا يحدد في هذا المعطى رمز الولوج
  الخارجي (التابع لخدمة جوجل على سبيل المثال). يمكن استخدام رمز الولوج الخارجي في التواصل مع الخدمة
  الخارجية (أي، مع واجهة جوجل البرمجية).
* `معرف_المستخدم` (`userId`): المعرف الفريد للمستخدم، والذي ترجعه المغلفة الممررة لدالة
  `هيئ_واجهة_رست` (`setupRestApi`).

### بـيانات_توكن (TokenInfo)

```
صنف بـيانات_توكن {
    عرف رمز_الولوج: نـص؛
    عرف صلاحية_رمز_الولوج: صـحيح[64] = 0؛
    عرف رمز_التجديد: نـص؛
    
    عملية هذا~هيئ(رمز_الولوج: نـص، صلاحية_رمز_الولوج: صـحيح[64]، رمز_التجديد: نـص)؛
}
```

<div dir=ltr>

```
class TokenInfo {
    def accessToken: String;
    def accessTokenExpiry: Int[64] = 0;
    def refreshToken: String;

    handler this~init(accessToken: String, accessTokenExpiry: Int[64], refreshToken: String);
}
```

</div>

### بـيانات_مستخدم (UserInfo)

```
صنف بـيانات_مستخدم {
    عرف المعرف: نـص؛
    عرف الاسم: نـص؛
    عرف البريد_الإلكتروني: نـص؛
    
    عملية هذا.هيئ(المعرف: نـص، الاسم: نـص، البريد_الإلكتروني: نـص)؛
}
```

<div dir=ltr>

```
class UserInfo {
    def id: String;
    def name: String;
    def email: String;

    handler this~init(id: String, name: String, email: String);
}
```

</div>

### جـواب_مصادقة (AuthenticationResponse)

```
صنف جـواب_مصادقة {
    عرف اسم_المزود: نـص؛
    @حقنة عرف بيانات_التوكن: بـيانات_توكن؛
    @حقنة عرف بيانات_المستخدم: بـيانات_مستخدم؛
}
```

<div dir=ltr>

```
class AuthenticationResponse {
    def providerName: String;
    @injection def tokenInfo: TokenInfo;
    @injection def userInfo: UserInfo;
}
```

</div>

### جـواب_بيانات_المستخدم (UserInfoResponse)

```
صنف جـواب_بيانات_المستخدم {
    عرف اسم_المزود: نـص؛
    @حقنة عرف بيانات_المستخدم: بـيانات_مستخدم؛
}
```

<div dir=ltr>

```
class UserInfoResponse {
    def providerName: String;
    @injection def userInfo: UserInfo;
}

```

</div>

</div>

