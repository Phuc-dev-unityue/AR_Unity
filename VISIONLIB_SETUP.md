# Hướng dẫn tích hợp VisionLib (Edge-Based Model Tracking) + ARFoundation

Dự án `TT-TGOSHI` đang sử dụng **Unity 6 (`6000.3.28f1`)** và **ARFoundation `6.5.0`** (ARCore + ARKit).
Vì VisionLib là SDK thương mại (có bản Trial miễn phí), bạn cần tải bộ cài từ Visometry và gắn vào dự án theo các bước dưới đây.

---

## 1. Tải VisionLib SDK 4.1+ (hỗ trợ Unity 6) & Trial License
1. Đăng ký/đăng nhập tại trang chủ VisionLib (`https://visionlib.com/`).
2. Xin cấp **Trial License** — lưu ý chọn quyền **`AllowInjectingImage`** (bắt buộc để dùng chung với ARFoundation).
3. Tải các gói trong trang download của VisionLib (**phiên bản 4.1 trở lên** để tương thích Unity 6 & URP 17):
   - `VisionLib.SDK-Unity-*.tgz`
   - `Extensions/VisionLib.SDK.ARFoundation-Unity-*.tgz`
   - `Extensions/VisionLib.SDK.ARFoundation.Examples-Unity.unitypackage` *(khuyên dùng để lấy prefab mẫu `VLXRTracking`)*

---

## 2. Cài đặt vào Unity (`TT-TGOSHI`)
1. Mở **Window > Package Manager** → bấm **`+` > Add package from tarball...**:
   - Thêm `VisionLib.SDK-Unity-*.tgz`
   - Thêm `VisionLib.SDK.ARFoundation-Unity-*.tgz`
2. Import file `VisionLib.SDK.ARFoundation.Examples-Unity.unitypackage` qua **Assets > Import Package > Custom Package...** để tham khảo scene mẫu `ARFoundationModelTracking`.
3. Chép file license bạn nhận được vào đúng đường dẫn và đổi tên thành:
   - `Assets/StreamingAssets/VisionLib/license.xml`

---

## 3. Đưa file 3D sa bàn trắng (`.obj` / `.stl`) vào dự án
1. Chép file `.obj` hoặc `.stl` của mô hình vật lý vào:
   - `Assets/StreamingAssets/VisionLib/Models/ScaleModel.obj`
2. Kiểm tra file cấu hình đã tạo sẵn tại:
   - `Assets/StreamingAssets/VisionLib/WhiteScaleModelTracking.vl`
   - Nếu file của bạn là `.stl` hoặc tên khác, sửa dòng `"modelURI": "project-dir:Models/ScaleModel.obj"`.
   - Kiểm tra `"metric"`: đặt `"m"` nếu xuất theo mét, hoặc `"mm"` nếu xuất theo milimet đúng với kích thước thật ngoài đời của sa bàn.
   - Đã cấu hình sẵn cho **sa bàn trắng trơn**: `"useColor": false`, `"extendibleTracking": true` (kết hợp SLAM của ARFoundation), `lineGradientThreshold: 15`, `laplaceThreshold: 0.5`.

---

## 4. Dựng Scene AR kết hợp VisionLib + Model CG có màu
1. Mở scene mẫu ARFoundation trong gói VisionLib Examples (hoặc tạo scene mới với `ARSession`, `XROrigin` và prefab `VLXRTracking` của VisionLib).
2. Trỏ component Tracking Configuration của VisionLib tới:
   - Config `.vl`: `WhiteScaleModelTracking.vl`
   - License: `license.xml`
3. Tạo một GameObject gắn **TrackingAnchor** (`VLTrackingAnchor`) của VisionLib cho sa bàn:
   - Đặt **Model CG có màu** (`ColoredCG`) làm con (child) của `TrackingAnchor` — đảm bảo cùng gốc tọa độ, hướng trục và tỉ lệ 1:1 với file `.obj`/`.stl`.
   - Gắn `ScaleModelOverlayController` (`Assets/Scripts/ARModelOverlay/ScaleModelOverlayController.cs`) lên `TrackingAnchor`, kéo `ColoredCG` vào ô `Colored Cg Root`.
   - Trong sự kiện Tracking của `TrackingAnchor`, nối sự kiện nhận diện được (`Tracked`) vào `ScaleModelOverlayController.OnTrackingFound()` và mất tracking (`Lost`) vào `ScaleModelOverlayController.OnTrackingLost()`.

---

## 5. Lưu ý thực tế khi test sa bàn trắng (như ảnh Mansion AR)
- **Tương phản nền:** Đặt sa bàn trắng trên mặt bàn tối/sẫm màu (tránh mặt bàn trắng hoặc bóng gương) để viền sa bàn nổi rõ.
- **Khởi tạo 360° không cần căn khung:** Với `initPose` mặc định, bạn hướng camera vào góc nhìn tương ứng để khóa lần đầu; nếu muốn tự động nhận diện từ **mọi góc 360°** ngay lập tức không cần căn hướng, hãy dùng **VisionLib AutoInit (WorkSpaces)** để tạo file học góc cạnh trước khi build app.
- **Build Mobile:** Trong *Project Settings > Player*, dùng **IL2CPP + ARM64** cho Android (ARCore) hoặc cấu hình Camera Usage Description cho iOS (ARKit).
