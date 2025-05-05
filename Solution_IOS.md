🎯 Mục tiêu: Đánh dấu thiết bị đã từng:

Cài đặt App A: MobiX

Và đăng nhập thành công (ví dụ: tài khoản FPT ID)

→ Sau đó: các app khác như FPT Play, Hi-FPT, FPT Camera có thể nhận biết được thiết bị đã cài và đăng nhập MobiX thông qua dữ liệu chia sẻ (shared storage)

✅ Trên iOS, giải pháp an toàn và phù hợp nhất chính là:

→ Keychain Sharing thông qua Access Group

—

📦 Tổng quan giải pháp

| Nền tảng      | Cơ chế                                                     | Ghi chú                                |
| ------------- | ---------------------------------------------------------- | -------------------------------------- |
| iOS           | Shared Keychain Access Group                               | App cùng Group có thể truy cập dữ liệu |
| MobiX (App A) | Ghi 1 key vào Keychain Access Group sau khi user đăng nhập |                                        |
| App B/C/D     | Đọc key đó từ Keychain (nếu có) để xác định                |                                        |

—

🧱 Kiến trúc hoạt động:

Tất cả apps: MobiX, FPT Play, Hi-FPT, FPT Camera

Phải cùng dùng chung 1 Access Group (ví dụ: group.com.fpt.shared)

Cùng bật Keychain Sharing trong Capability

App MobiX (khi đăng nhập thành công)

Ghi key FPTInstalled=YES vào Keychain (Access Group shared)

Các App B/C/D

Đọc key từ Keychain Access Group

Nếu key = YES → biết user từng đăng nhập MobiX

—
🛠 Cách triển khai chi tiết

Bước 1: Cấu hình Access Group (chung cho tất cả apps)

Vào Apple Developer → Certificates, Identifiers & Profiles

Chọn mỗi app → Capabilities → bật Keychain Sharing

Thêm Access Group: group.com.fpt.shared (hoặc tên tùy công ty)

Ghi nhớ Team ID của bạn (ví dụ: 9A4QK2FDL3)

## 🧭 Bước 2: App MobiX (ghi marker khi đăng nhập thành công)

```swift
import Security

func saveMobiXMarkerToKeychain() {
    let account = "FPTInstalled"
    let value = "YES"
    let accessGroup = "9A4QK2FDL3.group.com.fpt.shared"

    let query: [String: Any] = [
        kSecClass as String: kSecClassGenericPassword,
        kSecAttrAccount as String: account,
        kSecValueData as String: value.data(using: .utf8)!,
        kSecAttrAccessGroup as String: accessGroup
    ]

    SecItemDelete(query as CFDictionary) // Xóa trước nếu đã có
    SecItemAdd(query as CFDictionary, nil)
}
```


---

## 🧭 Bước 3: App FPT Play, Hi-FPT, FPT Camera (đọc marker)

```swift
func checkIfMobiXWasInstalled() -> Bool {
    let account = "FPTInstalled"
    let accessGroup = "9A4QK2FDL3.group.com.fpt.shared"

    let query: [String: Any] = [
        kSecClass as String: kSecClassGenericPassword,
        kSecAttrAccount as String: account,
        kSecAttrAccessGroup as String: accessGroup,
        kSecReturnData as String: kCFBooleanTrue!,
        kSecMatchLimit as String: kSecMatchLimitOne
    ]

    var item: CFTypeRef?
    let status = SecItemCopyMatching(query as CFDictionary, &item)

    if status == errSecSuccess {
        if let data = item as? Data,
           let value = String(data: data, encoding: .utf8),
           value == "YES" {
            return true
        }
    }

    return false
}
```

—

🧠 Gợi ý định danh thêm (nếu cần):

Ghi thêm userID hoặc token mã hóa trong Keychain để xác thực sâu hơn

Có thể kết hợp cả UserDefaults (không chia sẻ) với Keychain (chia sẻ) để tối ưu xác minh

—

🛡 Ưu điểm giải pháp:
| Ưu điểm                   | Chi tiết                                       |
| ------------------------- | ---------------------------------------------- |
| Bảo mật cao               | Dữ liệu trong Keychain được mã hóa, OS quản lý |
| Bền vững                  | Không mất khi app bị xoá rồi cài lại           |
| Không bị user xóa dễ dàng | Không hiển thị qua Settings hay Files app      |
| Cross-app                 | Các app cùng Access Group đều đọc được         |

—

⚠️ Lưu ý:

Tất cả apps phải cùng Team ID (hoặc do cùng công ty quản lý trong Apple Developer)

Access Group phải được add trong entitlements (.entitlements file) của từng app

Dữ liệu ghi vào Keychain nên đặt tên "account" và "service" rõ ràng (vd: account=FPTInstalled, service=mobix)

—

✅ Tóm tắt:

App MobiX:

Khi user đăng nhập thành công

Ghi key "FPTInstalled" = "YES" vào Keychain Access Group shared

App FPT Play / Hi-FPT / FPT Camera:

Đọc key đó từ Access Group

Nếu tồn tại và bằng "YES" ⇒ biết thiết bị từng đăng nhập MobiX

—
✅ Ảnh minh họa:
![image](https://github.com/user-attachments/assets/435ad692-057b-427a-96dd-cad6cd07b092)
