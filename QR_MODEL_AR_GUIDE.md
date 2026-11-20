# Hướng dẫn làm AR quét QR rồi hiện Model 3D chuẩn lên sa bàn (giống video BENA 1:06)

Giải pháp này dùng hoàn toàn **ARFoundation `6.5.0`** (ARCore + ARKit) đã cài sẵn trong dự án `TT-TGOSHI` — **không cần VisionLib, không cần email công ty, chạy được ngay**.

---

## Nguyên lý hoạt động
1. Dán một **Tem QR / Marker** ở vị trí cố định cạnh sa bàn trắng ngoài đời (ví dụ ngay mép trước chân đế sa bàn).
2. Khi camera quét thấy tem QR (`ARTrackedImageManager`), app tự tính ra **tọa độ tâm sa bàn** từ vị trí tem + khoảng cách `Local Position Offset`, rồi khóa chặt vào không gian thực bằng **`ARAnchor` (`TryAddAnchorAsync`)**.
3. **Model 3D CG có màu** được gắn vào `ARAnchor` đó -> Khi bạn xoay vòng quanh sa bàn hoặc che khuất tem QR, mô hình 3D vẫn đứng im chuẩn xác trên sa bàn thật nhờ SLAM của ARCore/ARKit.

---

## Các bước thực hiện trong Unity

### Bước 1: Chuẩn bị ảnh Tem QR để ARCore / ARKit bắt siêu nhạy
> **Mẹo quan trọng:** Nếu dùng ảnh QR đen trắng trơn 100%, một số máy Android (ARCore) có thể đánh giá điểm đặc trưng thấp hoặc khó phân biệt góc xoay.
- Nên thiết kế **1 tấm tem vuông** (ví dụ `10cm x 10cm` hoặc `12cm x 12cm`) gồm: **QR Code + khung viền + tên dự án/logo bất đối xứng** (giống thẻ marker cạnh mô hình trong video BENA).
- In tấm tem đó ra giấy nhám (tránh giấy bóng gây lóa đèn) và đặt/dán cố định cạnh sa bàn.

### Bước 2: Chạy công cụ Setup 1-Click trong Unity
1. Mở [SampleScene.unity](file:///d:/Unity/TT-TGOSHI/Assets/Scenes/SampleScene.unity).
2. Trên thanh menu Unity, chọn **`Tools > TT-TGOSHI > Setup QR Model AR Scene`**.
3. Kéo **Prefab Model 3D** của bạn vào ô *Model 3D Prefab* (tùy chọn), rồi bấm **`2. Setup Scene AR hiện tại (1-Click)`**.
   - Tool sẽ tự gắn `ARTrackedImageManager`, `ARAnchorManager` lên `XR Origin (AR Rig)`, tạo `Assets/QRModelAR/QrMarkerLibrary.asset`, tạo `QR Model AR System` và `QR Model AR HUD`.
4. *(Nếu muốn tắt UI đặt vật thể mẫu có sẵn của template)*: Trong `SampleScene`, tắt các UI/Object Spawner cũ của template nếu không dùng tới để màn hình chỉ hiển thị `QR Model AR HUD`.

### Bước 3: Thêm ảnh Tem QR vào `QrMarkerLibrary`
1. Chọn file `Assets/QRModelAR/QrMarkerLibrary.asset` trong Project window.
2. Bấm **Add Image**:
   - **Texture:** Kéo file ảnh tem QR của bạn vào.
   - **Name:** Đặt tên (ví dụ `SaBan_QR`).
   - **Specify Size:** **Bắt buộc tick chọn** và nhập đúng kích thước thật của tem khi in ngoài đời tính theo **mét** (ví dụ tem in `10cm x 10cm` thì nhập `X = 0.1`, `Y = 0.1`). *Sai kích thước thật sẽ khiến Model 3D bị lệch xa/gần!*

### Bước 4: Khai báo Model 3D, Khoảng cách & Danh sách Texture thay đổi trên AR
Chọn GameObject `QR Model AR System` trong Scene, mở phần **Targets** của `QrModelArAnchorManager`:
- **Reference Image Name:** Nhập đúng `SaBan_QR` (trùng tên trong `QrMarkerLibrary`).
- **Model Prefab:** Kéo Prefab Model 3D CG (tỉ lệ **1:1 ngoài đời tính theo mét**).
- **Local Position Offset:** Khoảng cách từ **tâm tem QR** tới **tâm đặt model 3D** (đơn vị mét):
  - `+X`: sang phải tem QR
  - `+Y`: hướng lên vuông góc mặt giấy
  - `+Z`: hướng lên phía trên của tem QR (nằm dọc mặt bàn nếu đặt tem nằm ngang)
  - *Ví dụ:* Tem QR đặt nằm trên mặt bàn ngay trước sa bàn, tâm sa bàn nằm phía sau cạnh trên tem `25cm` -> nhập `(0, 0, 0.25)`.
- **Local Euler Offset:** Góc xoay thêm nếu trục model khác với hướng tem (ví dụ `(0, 180, 0)`).
- **Texture Variants:** Thêm các phương án texture/vật liệu bạn muốn người dùng chuyển đổi khi xem AR:
  - **Variant Name:** Tên hiển thị trên màn hình (vd: `"Mặt tiền đá trắng"`, `"Mặt tiền kính xanh"`, `"Phương án ban đêm"`).
  - **Texture:** Kéo file ảnh texture (`Texture2D`) vào — app tự áp vào `_BaseMap` (URP) / `_MainTex` qua `MaterialPropertyBlock` mà không làm hỏng asset gốc.
  - **Override Material:** *(Tùy chọn)* Nếu muốn đổi toàn bộ shader/vật liệu (vd: có phát sáng cửa sổ ban đêm), kéo Material vào đây.
  - *(Mẹo nâng cao: Nếu Prefab có nhiều bộ phận và chỉ muốn đổi texture ở phần tường/mặt tiền chứ không đổi phần đế, hãy gắn component `ModelTextureSwitcher` trực tiếp lên Prefab và kéo riêng các `Renderer` cần đổi vào `Target Renderers`.)*

### Bước 5: Build & Căn chỉnh thực tế trên điện thoại
1. Build sang Android (`ARCore`, `IL2CPP + ARM64`) hoặc iOS (`ARKit`).
2. Hướng camera vào tem QR -> sau 5 frame ổn định, app tự khóa `ARAnchor` và phủ lớp Model/Texture lên sa bàn.
3. Thanh **TextureBar (`<` / `>`)** sẽ tự hiện ở phía trên thanh công cụ để người dùng bấm đổi qua lại các Texture ngay trên mô hình đang khóa.
4. Nếu đặt tem hơi lệch tay vài milimet, bấm nút **`Căn chỉnh`** trên màn hình để dịch **`±5mm` (X/Y/Z)** và xoay **`±1°`** — app tự lưu lại thông số cho lần mở sau.
5. Bấm **`Ẩn 3D`** để đối chiếu với sa bàn trắng thật, hoặc **`Quét lại QR`** để khóa lại vị trí.
