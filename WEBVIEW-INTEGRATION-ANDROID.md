# اتصال Android WebView — باز کردن مستقیم پیامک

نسخه وب V61 سه مسیر را برای پیامک امتحان می‌کند:

1. اگر Bridge با نام `Android.openSms(number, body)` وجود داشته باشد، از آن استفاده می‌کند.
2. در مرورگرهای معمولی از لینک `sms:` استفاده می‌کند.
3. اگر WebView اجازه این کار را ندهد، کاربر می‌تواند شماره را با یک لمس کپی کند.

برای حل قطعی داخل Android WebView، Bridge زیر پیشنهاد می‌شود.

## Kotlin

```kotlin
class WebAppBridge(private val context: Context) {
    @JavascriptInterface
    fun openSms(number: String, body: String) {
        val intent = Intent(Intent.ACTION_SENDTO).apply {
            data = Uri.parse("smsto:$number")
            putExtra("sms_body", body)
            addFlags(Intent.FLAG_ACTIVITY_NEW_TASK)
        }
        context.startActivity(intent)
    }
}
```

و هنگام راه‌اندازی WebView:

```kotlin
webView.settings.javaScriptEnabled = true
webView.addJavascriptInterface(WebAppBridge(this), "Android")
```

همچنین برای لینک‌های خارجی و schemeهای سیستم:

```kotlin
webView.webViewClient = object : WebViewClient() {
    override fun shouldOverrideUrlLoading(
        view: WebView?,
        request: WebResourceRequest?
    ): Boolean {
        val uri = request?.url ?: return false
        val scheme = uri.scheme.orEmpty().lowercase()

        if (scheme == "sms" || scheme == "smsto") {
            val intent = Intent(Intent.ACTION_SENDTO, uri)
            startActivity(intent)
            return true
        }

        if (scheme == "tel") {
            startActivity(Intent(Intent.ACTION_DIAL, uri))
            return true
        }

        return false
    }
}
```

## نکته امنیتی

Bridge فقط متدهای لازم را با `@JavascriptInterface` در اختیار صفحه قرار دهد. اگر WebView محتوای اینترنتی ناشناس هم باز می‌کند، Bridge را فقط برای دامنه/محتوای مورد اعتماد فعال کنید.
