🎯 Mục tiêu:

MobiX (App A) khi được cài đặt và chạy → lưu dấu hiệu (marker) ra nơi dùng chung

FPT Play, Hi-FPT, FPT Camera (Apps B/C/D) kiểm tra marker đó → biết được MobiX từng tồn tại

Dữ liệu phải:

An toàn (không public rò rỉ)

Khó mất (kể cả gỡ app A)

Không yêu cầu runtime permission

✅ Ta sẽ kết hợp:

External storage: ghi file marker ở vùng /Android/media/

AccountManager: ghi dấu hiệu bằng account hệ thống

Kết hợp này đảm bảo:

App A bị gỡ vẫn còn file marker

User xóa file → vẫn còn account

User xóa account → vẫn còn file

—

🧠 Kiến trúc tổng thể

📱 App A – MobiX

Khi mở lần đầu → thực hiện:

Tạo file marker: /storage/emulated/0/Android/media/fpt.shared/.mobix.installed

Thêm account vào hệ thống: AccountManager với type = com.fpt.mobix.account

📱 App B/C/D – FPT Play, Hi-FPT, FPT Camera

Khi chạy → thực hiện:

Kiểm tra file marker có tồn tại?

Kiểm tra account com.fpt.mobix.account có tồn tại?
→ Nếu 1 trong 2 đúng ⇒ biết MobiX đã từng được cài

—

🛠 Hướng dẫn triển khai chi tiết

App A – Ghi file marker

## 📄 Kotlin: Ghi marker file trên Android

```kotlin
fun writeMarkerFile(context: Context) {
    val dir = File(Environment.getExternalStoragePublicDirectory("Android/media/fpt.shared"))
    if (!dir.exists()) dir.mkdirs()
    val file = File(dir, ".mobix.installed")
    if (!file.exists()) {
        file.writeText("installed")
    }
}
```

---

## 🔐 App A – Ghi Account qua Android AccountManager

### 📄 AndroidManifest.xml

```xml
<manifest ...>
  <application ...>
    ...
    <service
        android:name=".authenticator.AuthenticatorService"
        android:permission="android.permission.BIND_ACCOUNT_AUTHENTICATOR">
        <intent-filter>
            <action android:name="android.accounts.AccountAuthenticator" />
        </intent-filter>
        <meta-data
            android:name="android.accounts.AccountAuthenticator"
            android:resource="@xml/authenticator" />
    </service>
    ...
  </application>
</manifest>
```

### 📁 File: `res/xml/authenticator.xml`

```xml
<account-authenticator
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:accountType="com.fpt.mobix.account"
    android:icon="@mipmap/ic_launcher"
    android:smallIcon="@mipmap/ic_launcher"
    android:label="@string/app_name" />
```

---

### 🧑‍💻 Thêm account bằng Kotlin

```kotlin
fun addMobiXAccount(context: Context) {
    val am = AccountManager.get(context)
    val type = "com.fpt.mobix.account"
    val accounts = am.getAccountsByType(type)
    if (accounts.isEmpty()) {
        val account = Account("MobiX_Installed", type)
        am.addAccountExplicitly(account, null, null)
    }
}
```


App B/C/D – Kiểm tra dấu hiệu

## 📄 Kotlin: Kiểm tra xem thiết bị đã từng cài App MobiX chưa

```kotlin
fun wasMobiXInstalled(context: Context): Boolean {
    val file = File(Environment.getExternalStoragePublicDirectory("Android/media/fpt.shared/.mobix.installed"))
    val hasFile = file.exists()

    val am = AccountManager.get(context)
    val hasAccount = am.getAccountsByType("com.fpt.mobix.account").isNotEmpty()

    return hasFile || hasAccount
}
```

📌 Hàm trên sẽ:
- Trả về `true` nếu tồn tại **file marker** trong bộ nhớ chia sẻ
- Hoặc nếu đã từng thêm **tài khoản hệ thống** cho App MobiX


✅ Nếu return true ⇒ thiết bị đã từng cài MobiX

—

📈 Luồng hoạt động:

App A mở lần đầu:

➜ Ghi file marker

➜ Ghi account marker

App B/C/D chạy:

➜ Kiểm tra file /Android/media/fpt.shared/.mobix.installed

➜ Kiểm tra account com.fpt.mobix.account

➜ Nếu 1 trong 2 có → biết đã từng cài App A (MobiX)

—

📌 Ưu điểm của giải pháp kết hợp:
| Cơ chế         | Ưu điểm                               | Rủi ro                                       |
| -------------- | ------------------------------------- | -------------------------------------------- |
| External file  | Không mất khi gỡ App A                | Có thể bị user xóa                           |
| AccountManager | Rất bền vững, không bị xóa khi gỡ App | Chỉ mất nếu user xóa thủ công trong Settings |

⟶ Kết hợp cả 2: gần như không thể bị mất hoàn toàn.

—

🧠 Mẹo nâng cao:

Mã hóa nội dung file .mobix.installed nếu cần ẩn

Đặt tên account khó hiểu nếu cần bảo mật

Có thể bổ sung userID hash/token trong dữ liệu lưu để liên kết với tài khoản FPT ID

—

✅ Tóm tắt:

App A (MobiX):

→ Khi chạy lần đầu → Ghi dấu hiệu ra file + Ghi account

App B/C/D:

→ Khi chạy → Kiểm tra file và account

→ Nếu có: biết MobiX từng cài trên thiết bị này

—
