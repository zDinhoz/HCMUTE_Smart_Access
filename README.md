# HCM-UTE Smart Access

HCM-UTE Smart Access là hệ thống quản lý ra vào phòng học **A102** sử dụng **thẻ NFC hoặc vân tay**, kết hợp giữa ứng dụng Android, Firebase và bộ điều khiển ESP32.

Ứng dụng Android được phát triển bằng **Kotlin + Jetpack Compose**, sử dụng **Firebase Authentication** để xác thực tài khoản và **Cloud Firestore** để lưu trữ, đồng bộ dữ liệu giữa ứng dụng và ESP32.

---

## 1. Tổng quan hệ thống

Hệ thống gồm ba thành phần chính:

- **Android Application**: giao diện dành cho Student và Admin.
- **Firebase Backend**: xác thực tài khoản, lưu dữ liệu và đồng bộ trạng thái.
- **ESP32_A102**: xử lý NFC, vân tay, LCD, servo cửa và ghi nhận sự kiện truy cập.

```text
┌─────────────────────────┐
│     Android Client      │
│ Kotlin + Jetpack Compose│
└────────────┬────────────┘
             │
             ▼
┌─────────────────────────┐
│ Firebase Authentication │
│    Cloud Firestore      │
└────────────┬────────────┘
             │
             ▼
┌─────────────────────────┐
│       ESP32_A102        │
│   Access Controller     │
└───────┬─────────┬───────┘
        │         │
        ▼         ▼
    RC522 NFC   Fingerprint
     Reader       Sensor
        │         │
        └────┬────┘
             ▼
        Servo / Door
```

Ứng dụng Android **không trực tiếp đọc thẻ NFC hoặc cảm biến vân tay**.

Việc quét và xác thực thực tế được thực hiện tại **ESP32_A102**. Android chủ yếu đảm nhiệm quản lý tài khoản, gửi yêu cầu enrollment, quản trị hệ thống và hiển thị dữ liệu được đồng bộ từ Firebase.

---

## 2. Công nghệ sử dụng

### Android

- Kotlin
- Jetpack Compose
- Material 3
- Single-Activity Architecture
- Stateful Composable Screens
- SharedPreferences
- SnapshotListener

### Firebase

- Firebase Authentication
- Cloud Firestore
- Firestore Security Rules

### Embedded Hardware

- ESP32 WROOM-32
- RC522 NFC Reader
- Fingerprint Sensor
- LCD I2C
- Servo MG90S

---

## 3. Phương thức xác thực

Hệ thống hỗ trợ hai phương thức:

```text
NFC OR Fingerprint
```

NFC và vân tay được sử dụng **độc lập**.

Hệ thống không yêu cầu người dùng phải xác thực cả NFC và vân tay cùng lúc.

---

## 4. Chức năng Student

Student có thể:

- Đăng ký tài khoản.
- Đăng nhập bằng Firebase Authentication.
- Theo dõi trạng thái tài khoản.
- Chờ Admin phê duyệt tài khoản.
- Xem thông tin cá nhân.
- Xem trạng thái phòng A102.
- Theo dõi trạng thái ESP32_A102.
- Theo dõi trạng thái cửa.
- Xem trạng thái NFC.
- Xem trạng thái vân tay.
- Gửi yêu cầu đăng ký NFC.
- Gửi yêu cầu đăng ký vân tay.
- Gửi yêu cầu xóa NFC.
- Gửi yêu cầu xóa vân tay.
- Theo dõi quá trình enrollment.
- Xem lịch sử ra vào.
- Xem số lượt truy cập trong ngày.
- Xem kết quả Granted / Denied.
- Theo dõi thông tin điểm danh.
- Thay đổi giao diện Light / Dark / System.
- Chuyển đổi English / Vietnamese.

---

## 5. Chức năng Admin

Admin có thể:

- Đăng nhập bằng tài khoản Admin.
- Xem Admin Dashboard.
- Theo dõi tổng số Student.
- Theo dõi tài khoản đang chờ phê duyệt.
- Phê duyệt tài khoản Student.
- Theo dõi lịch sử truy cập.
- Theo dõi số lượt Granted / Denied.
- Theo dõi ESP32_A102.
- Theo dõi trạng thái NFC Reader.
- Theo dõi trạng thái Fingerprint Sensor.
- Theo dõi trạng thái cửa.
- Xem credential của từng Student.
- Thu hồi NFC.
- Gửi yêu cầu xóa fingerprint.
- Tạo phiên điểm danh.
- Chỉnh sửa phiên điểm danh.
- Mở phiên điểm danh.
- Đóng phiên điểm danh.
- Xóa phiên điểm danh.
- Xem kết quả điểm danh.

---

## 6. Kiến trúc Android

Ứng dụng hiện được xây dựng theo mô hình:

```text
Single Activity
      │
      ▼
MainActivity
      │
      ▼
currentScreen
      │
      ├── Login
      ├── Register
      ├── Pending Approval
      ├── Student Dashboard
      ├── Student Settings
      │
      ├── Admin Dashboard
      ├── Pending Students
      ├── Access Logs
      ├── Device Management
      ├── Credential Management
      ├── Attendance Management
      └── Settings
```

`MainActivity` đóng vai trò:

- Entry point của ứng dụng.
- Điều phối các màn hình.
- Quản lý `currentScreen`.
- Quản lý theme.
- Quản lý language.
- Kiểm tra Firebase session.
- Xử lý logout.
- Xử lý nút Back.

Phiên bản hiện tại sử dụng **state-based navigation** và chưa triển khai Navigation Compose.

---

## 7. Luồng xác thực tài khoản

```text
Application Start
       │
       ▼
FirebaseAuth.currentUser
       │
       ├── null
       │    │
       │    ▼
       │  Login
       │
       └── User exists
              │
              ▼
         users/{uid}
              │
        ┌─────┼──────────────┐
        │     │              │
        ▼     ▼              ▼
 Admin Active Student Active Student Pending
        │     │              │
        ▼     ▼              ▼
     Admin  Student       Pending
   Dashboard Dashboard    Approval
```

Sau khi Firebase Authentication xác thực người dùng, ứng dụng đọc:

```text
users/{uid}
```

để lấy:

```text
role
status
studentId
fullName
email
```

Từ đó ứng dụng quyết định màn hình phù hợp.

---

## 8. Đồng bộ realtime

Ứng dụng sử dụng:

```kotlin
addSnapshotListener()
```

để theo dõi thay đổi dữ liệu trên Firestore.

Luồng cập nhật:

```text
Firestore thay đổi
        │
        ▼
SnapshotListener
        │
        ▼
Compose State
        │
        ▼
Recomposition
        │
        ▼
UI cập nhật
```

Các listener được đặt trong `DisposableEffect`.

Khi Composable không còn được sử dụng:

```kotlin
onDispose {
    listener.remove()
}
```

Listener sẽ được hủy để tránh tạo nhiều listener trùng nhau.

---

## 9. Cấu trúc Cloud Firestore

Các collection chính:

```text
users/
credentials/
devices/
enrollment_requests/
nfc_lookup/
fingerprint_lookup/
access_logs/
attendance_sessions/
attendance_runtime/
attendance_records/
```

---

### users/{uid}

Lưu thông tin tài khoản:

```text
fullName
studentId
email
role
status
```

Role:

```text
admin
student
```

Status:

```text
pending
active
```

---

### credentials/{uid}

Lưu trạng thái credential:

```text
nfcUid
nfcStatus
fingerprintEnrolled
fingerprintSlot
updatedAt
```

Template vân tay thực tế **không được lưu trên Firestore**.

Template được lưu trực tiếp trong bộ nhớ của cảm biến fingerprint.

Firestore chỉ lưu:

```text
fingerprintEnrolled
fingerprintSlot
```

để tham chiếu.

---

### devices/ESP32_A102

Lưu trạng thái controller tại cửa:

```text
deviceId
status
doorStatus
lastSeen
NFC reader status
fingerprint sensor status
```

---

### enrollment_requests/ESP32_A102

Được sử dụng để trao đổi yêu cầu giữa Android và ESP32.

Một request có thể chứa:

```text
deviceId
userId
studentId
action
status
credentialValue
fingerprintSlot
requestedAt
updatedAt
message
```

Trạng thái request:

```text
waiting
processing
completed
failed
cancelled
```

---

### nfc_lookup/{cardUid}

Dùng để ánh xạ:

```text
NFC UID
   │
   ▼
Student / User
```

giúp ESP32 xác định người sở hữu thẻ.

---

### fingerprint_lookup/{slotId}

Dùng để ánh xạ:

```text
Fingerprint Slot
       │
       ▼
Student / User
```

---

### access_logs

Lưu lịch sử ra vào:

```text
studentId
userId
method
result
deviceId
timestamp
```

Method:

```text
NFC
Fingerprint
```

Result:

```text
GRANTED
DENIED
```

---

## 10. Luồng đăng ký NFC

```text
Student
   │
   ▼
ADD NFC
   │
   ▼
Android tạo enrollment request
   │
   ▼
status = waiting
   │
   ▼
ESP32_A102 nhận request
   │
   ▼
status = processing
   │
   ▼
RC522 đọc UID thẻ
   │
   ▼
credentials/{uid}
+
nfc_lookup/{cardUid}
   │
   ▼
status = completed
   │
   ▼
Android nhận Snapshot
   │
   ▼
UI cập nhật NFC LINKED
```

Student không được tự nhập NFC UID vào Firestore.

UID phải được đọc trực tiếp từ RC522 thông qua ESP32.

---

## 11. Luồng đăng ký Fingerprint

```text
Student
   │
   ▼
ADD FINGERPRINT
   │
   ▼
Enrollment Request
   │
   ▼
ESP32_A102
   │
   ▼
Fingerprint Sensor
   │
   ▼
Capture Fingerprint
   │
   ▼
Store Template in Sensor
   │
   ▼
Fingerprint Slot
   │
   ▼
Update Firestore
   │
   ▼
completed
```

Firestore không lưu template sinh trắc học.

---

## 12. Luồng xác thực ra vào

```text
Student
   │
   ▼
NFC / Fingerprint
   │
   ▼
ESP32_A102
   │
   ▼
Credential Validation
   │
   ├── Valid
   │     │
   │     ▼
   │   GRANTED
   │     │
   │     ├── LCD: ACCESS GRANTED
   │     ├── Open Servo
   │     ├── Write Access Log
   │     └── Update Door Status
   │
   └── Invalid
         │
         ▼
       DENIED
         │
         ├── LCD: ACCESS DENIED
         └── Write Access Log
```

ESP32 là thành phần quyết định quyền truy cập tại cửa.

Android không trực tiếp điều khiển servo.

---

## 13. Điều khiển cửa

Khi người dùng xác thực thành công:

```text
Servo Closed
≈ 10°
   │
   ▼
ACCESS GRANTED
   │
   ▼
Servo Open
≈ 90°
   │
   ▼
Wait ~3 seconds
   │
   ▼
Servo Closed
≈ 10°
```

ESP32 đồng thời cập nhật trạng thái cửa lên Firestore.

---

## 14. Attendance

Các collection liên quan:

```text
attendance_sessions/
attendance_runtime/
attendance_records/
```

Luồng cơ bản:

```text
Admin
  │
  ▼
Create Attendance Session
  │
  ▼
Open Session
  │
  ▼
attendance_runtime/ESP32_A102
  │
  ▼
ESP32 nhận session active
  │
  ▼
Student xác thực
  │
  ▼
ACCESS GRANTED
  │
  ▼
Attendance Record
```

Kết quả điểm danh có thể gồm:

```text
present
late
```

Hệ thống sử dụng thông tin session và student để tránh tạo bản ghi trùng.

---

## 15. Firestore Security Rules

Ứng dụng không chỉ kiểm tra quyền ở giao diện.

Firestore Security Rules được sử dụng để kiểm soát quyền đọc và ghi dữ liệu thực tế.

Các nhóm quyền chính:

```text
Student
Admin
ESP32_A102
```

Các hàm kiểm tra bao gồm:

```text
signedIn()
isAdmin()
isActiveStudent()
isA102Device()
```

ESP32_A102 được xác thực bằng Firebase Authentication và đối chiếu với:

```text
devices/ESP32_A102.authUid
```

Cuối ruleset sử dụng nguyên tắc:

```text
Default Deny
```

tức là các thao tác không được cho phép rõ ràng sẽ bị từ chối.

---

## 16. Cấu trúc source code

```text
HCMUTESmartAccess/
│
├── app/
│   ├── src/
│   │   └── main/
│   │       │
│   │       ├── AndroidManifest.xml
│   │       │
│   │       ├── java/com/hcmute/smartaccess/
│   │       │   │
│   │       │   ├── MainActivity.kt
│   │       │   ├── LoginScreen.kt
│   │       │   ├── RegisterScreen.kt
│   │       │   ├── PendingApprovalScreen.kt
│   │       │   ├── StudentHomeScreen.kt
│   │       │   │
│   │       │   ├── AdminDashboardScreen.kt
│   │       │   ├── PendingStudentsScreen.kt
│   │       │   ├── AccessLogsScreen.kt
│   │       │   ├── DeviceManagementScreen.kt
│   │       │   ├── CredentialManagementScreen.kt
│   │       │   ├── AttendanceManagementScreen.kt
│   │       │   │
│   │       │   ├── SettingsScreen.kt
│   │       │   ├── AppLanguage.kt
│   │       │   ├── ClassroomConfig.kt
│   │       │   │
│   │       │   └── ui/theme/
│   │       │       ├── Color.kt
│   │       │       ├── Theme.kt
│   │       │       └── Type.kt
│   │       │
│   │       └── res/
│   │           ├── drawable/
│   │           ├── mipmap-*/
│   │           ├── values/
│   │           └── xml/
│   │
│   └── build.gradle.kts
│
├── gradle/
├── build.gradle.kts
├── settings.gradle.kts
├── gradle.properties
├── gradlew
└── gradlew.bat
```

---

## 17. Vai trò các file chính

| File | Chức năng |
|---|---|
| `MainActivity.kt` | Entry point, session check và điều phối màn hình |
| `LoginScreen.kt` | Đăng nhập Firebase Authentication |
| `RegisterScreen.kt` | Đăng ký Student |
| `PendingApprovalScreen.kt` | Theo dõi tài khoản chờ duyệt |
| `StudentHomeScreen.kt` | Student Dashboard |
| `AdminDashboardScreen.kt` | Admin Dashboard |
| `PendingStudentsScreen.kt` | Phê duyệt Student |
| `AccessLogsScreen.kt` | Lịch sử ra vào |
| `DeviceManagementScreen.kt` | Theo dõi ESP32 và sensor |
| `CredentialManagementScreen.kt` | Quản lý NFC và fingerprint |
| `AttendanceManagementScreen.kt` | Quản lý attendance |
| `SettingsScreen.kt` | Theme và language |
| `ClassroomConfig.kt` | Cấu hình Room / Device |
| `AppLanguage.kt` | Hỗ trợ English / Vietnamese |

---

# 18. Video Demo

## Đăng ký tài khoản Student

[![YouTube](https://img.shields.io/badge/YouTube-Watch_Demo-red?logo=youtube)](https://youtu.be/G3_Ac4GDy2A)

https://youtu.be/G3_Ac4GDy2A

---

## Admin duyệt tài khoản Student

[![YouTube](https://img.shields.io/badge/YouTube-Watch_Demo-red?logo=youtube)](https://youtu.be/rPZDO_zdbOs)

https://youtu.be/rPZDO_zdbOs

---

## Student đăng ký NFC và vân tay

[![YouTube](https://img.shields.io/badge/YouTube-Watch_Demo-red?logo=youtube)](https://youtu.be/lbneo9DQlTE)

https://youtu.be/lbneo9DQlTE

---

## Student xóa NFC và vân tay

[![YouTube](https://img.shields.io/badge/YouTube-Watch_Demo-red?logo=youtube)](https://youtu.be/264U0LPDeXI)

https://youtu.be/264U0LPDeXI

---

## Admin tạo buổi điểm danh

[![YouTube](https://img.shields.io/badge/YouTube-Watch_Demo-red?logo=youtube)](https://youtu.be/vrL58Vwm7Xg)

https://youtu.be/vrL58Vwm7Xg

---

## 19. Android Permissions

Ứng dụng sử dụng:

```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
```

Ứng dụng Android không cần quyền NFC vì NFC được xử lý trực tiếp bởi RC522 kết nối với ESP32.

---

## 20. Cài đặt project

Clone repository:

```bash
git clone https://github.com/YOUR_USERNAME/HCMUTE-Smart-Access.git
```

Đi vào thư mục project:

```bash
cd HCMUTE-Smart-Access
```

Sau đó:

1. Mở project bằng Android Studio.
2. Kiểm tra Firebase configuration.
3. Sync Gradle.
4. Chờ Android Studio tải dependencies.
5. Kết nối điện thoại Android hoặc mở Emulator.
6. Run module `app`.

---

## 21. Firebase Configuration

Project sử dụng:

```text
Firebase Authentication
Cloud Firestore
```

Firebase configuration được đặt tại:

```text
app/google-services.json
```

Nếu sử dụng Firebase project khác, cần tải `google-services.json` tương ứng từ Firebase Console và đặt vào thư mục `app/`.

> Không nên commit file chứa thông tin nhạy cảm hoặc signing key lên repository public.

---

## 22. Build APK

### Android Studio

```text
Build
→ Build Bundle(s) / APK(s)
→ Build APK(s)
```

Debug APK được tạo tại:

```text
app/build/outputs/apk/debug/app-debug.apk
```

### Gradle trên Windows

```cmd
gradlew.bat assembleDebug
```

---

## 23. Yêu cầu chạy

- Android 8.0 hoặc mới hơn.
- Có kết nối Internet.
- Firebase project được cấu hình hợp lệ.
- Để kiểm thử toàn bộ chức năng cần ESP32_A102.
- RC522 và Fingerprint Sensor cần được kết nối đúng với ESP32.
- Servo cửa cần được kết nối và cấp nguồn phù hợp.

---

## 24. Hạn chế hiện tại

Phiên bản hiện tại là prototype chủ yếu dành cho:

```text
Room A102
ESP32_A102
```

Một số hạn chế:

- Chưa hỗ trợ đồng thời nhiều phòng học.
- Chưa hỗ trợ nhiều controller ESP32 trong cùng một hệ thống hoàn chỉnh.
- Enrollment hiện sử dụng một request document chính cho ESP32_A102.
- Chưa tối ưu cho nhiều enrollment request đồng thời.
- Android hiện chưa tách ViewModel / Repository.
- Logic Firebase vẫn còn nằm trực tiếp trong một số Composable Screen.
- Chưa có trusted backend riêng.
- Trạng thái online của ESP32 chưa có heartbeat hoàn chỉnh.
- Chưa thực hiện đầy đủ load test.
- Chưa đo latency định lượng trong nhiều điều kiện mạng khác nhau.

---

## 25. Hướng phát triển

Các hướng cải tiến:

- Refactor Android theo MVVM + Repository.
- Sử dụng Navigation Compose.
- Sử dụng StateFlow để quản lý UI State.
- Sử dụng Hilt khi số lượng dependency tăng.
- Chuyển SharedPreferences sang DataStore khi cần.
- Hỗ trợ nhiều phòng.
- Hỗ trợ nhiều ESP32 controller.
- Chuyển enrollment sang queue theo `requestId`.
- Bổ sung heartbeat và `lastSeen`.
- Bổ sung trusted backend hoặc Cloud Functions.
- Bổ sung Firebase App Check.
- Bổ sung push notification.
- Mở rộng kiểm thử mất mạng / reconnect.
- Kiểm thử nhiều request đồng thời.
- Hoàn thiện bản Release APK / AAB.

---

## 26. Project Information

**Project:** HCM-UTE Smart Access  
**Course:** Phát triển ứng dụng di động  
**Platform:** Android  
**Language:** Kotlin  
**UI:** Jetpack Compose  
**Backend:** Firebase  
**Embedded Controller:** ESP32  
**Authentication:** NFC / Fingerprint  
**Target Room:** A102  

---

## Báo cáo đồ án

Báo cáo đầy đủ của đề tài **HCM-UTE Smart Access – Hệ thống quản lý ra vào phòng học sử dụng vân tay và thẻ NFC**:

[📄 Xem báo cáo HCM-UTE Smart Access](docs/Report_Final_Di_động.pdf)

---

## HCM-UTE Smart Access

**Android + Firebase + ESP32 + NFC + Fingerprint**

Smart classroom access management system for Room A102.
