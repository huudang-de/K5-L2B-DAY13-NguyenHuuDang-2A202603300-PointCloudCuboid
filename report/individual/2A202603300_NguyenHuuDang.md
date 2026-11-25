# Báo cáo Day 13 — Nguyễn Hữu Đăng, 2A202603300

## 1. Kết luận trước
- **Thông tin chung:** MSSV `2A202603300`, task cá nhân `D13-2A202603300`, 10 frame: `s01f33`, `s01f58`, `s02f07`, `s02f30`, `s03f05`, `s03f59`, `s04f25`, `s04f46`, `s05f07`, `s05f26`. Hệ điều hành: Windows, trình duyệt: Google Chrome, phiên bản CVAT: v2.82.0.
- **Tổng kết kết quả:** Đã vẽ tổng cộng **96 cuboid** trên cả 10 frame (`s01f33` đạt 7 cuboid, các frame khác từ 5–15 cuboid); frame ít nhất là `s03f59` (5 cuboid) do mật độ phương tiện trong phạm vi 50 m thưa hơn. Phiếu tự kiểm (`selfcheck`) ghi nhận 20/96 cuboid có cảnh báo (chủ yếu do nới nhẹ bề rộng để bao trọn hai gương chiếu hậu của xe). Phần tin tưởng nhất là hướng đầu xe (`yaw_deg`) và độ cao đáy chạm mặt đường (`z_bottom`); phần còn yếu nhất là ước lượng chính xác chiều dài các xe đậu ở khoảng cách xa (> 35 m).

- **Cách dùng CVAT:**
  - *Đổi góc nhìn:* Chuột trái giữ và kéo để xoay góc nhìn 3D (xoay thẳng từ trên nóc xuống để nhìn toàn cảnh Top-down view); chuột phải giữ và kéo để tịnh tiến (dời) góc nhìn; lăn con lăn chuột để phóng to/thu nhỏ; nháy đúp vào khoảng trống để reset về góc nhìn mặc định.
  - *Vẽ và chỉnh cuboid:* Chọn công cụ vẽ cuboid ở thanh bên trái, chọn label phù hợp và bấm **Shape** (không dùng Track). Di chuyển hộp theo con trỏ đến cụm điểm rồi nháy đúp để đặt. Dùng khung **Top** để chỉnh tâm, kích thước dài/rộng và góc xoay yaw; dùng khung **Side** và **Front** để chỉnh độ cao và kéo đáy chạm mặt đường (không xoay ở Side/Front).
  - *Kiểm tra hướng và camera:* Bật *Appearance → Cuboid orientation* để hiện vạch đỏ ngắn ($+X$ của hộp) và căn chỉnh chỉ đúng hướng đầu xe. Dùng lệnh `python scripts/lab.py preview` để chiếu kiểm tra cuboid đã vẽ lên ảnh thực của 7 camera.

## 2. Đọc frame (Bước 2)
Hai frame đọc kỹ: `s01f33`, `s01f58`.
Ảnh minh chứng: `t1-s01f33-3d.png`, `t1-s01f33-top.png`, `t1-s01f58-3d.png`, `t1-s01f58-top.png`.

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| s01f33 | Phía trước (-y) hướng lên cạnh trên; +x chỉ sang trái, +y chỉ xuống dưới (phía sau) | Mặt đường phẳng; mặt đường thấp hơn tâm LiDAR khoảng 1,6 m đến 1,7 m (z ≈ -1,65 m) | ~ 40 m | Xe con (Car), người đi bộ / xe hai bánh, cây cối, dải phân cách | Xe con di chuyển phía sau bên phải (cam08), cụm cây cối và vỉa hè bên trái (cam05/cam06) | Cụm điểm thưa ở lề đường bên phải (~38 m) chưa rõ là cột mốc hay thân cây |
| s01f58 | Phía trước (-y) hướng lên cạnh trên; +x chỉ sang trái, +y chỉ xuống dưới (phía sau) | Mặt đường phẳng; mặt đường thấp hơn tâm LiDAR khoảng 1,6 m đến 1,7 m (z ≈ -1,65 m) | ~ 42 m | Xe con (Car), xe tải ở xa, xe máy (Cyclist) | Xe máy chạy cùng chiều bên trái (cam06), xe con bám đuôi phía sau (cam09) | Cụm vài điểm phản xạ tán xạ gần ngọn cây bên lề đường |

**So sánh Camera và LiDAR từ hai frame trên:**
- **3 điều chỉ camera cho biết:**
  1. Màu sắc xe, màu sơn của phương tiện và trang phục của người đi đường.
  2. Biển số xe, chữ viết trên các biển hiệu cửa hàng hoặc biển báo giao thông.
  3. Trạng thái đèn tín hiệu giao thông (xanh/đỏ), đèn phanh/xi-nhan của xe khác, và vạch kẻ sơn trên mặt đường.
- **3 điều chỉ LiDAR cho biết:**
  1. Khoảng cách vật lý chính xác tuyệt đối theo hệ mét $(x, y, z)$ từ xe đến từng bề mặt của vật thể.
  2. Kích thước hình học thực tế (dài, rộng, cao theo mét) độc lập với góc phối cảnh quang học xa gần.
  3. Tầm bao quát không gian 360° đồng nhất, hoạt động ổn định trong bóng tối, không bị chói sáng bởi đèn xe đối diện hoặc lóa nắng như cảm biến camera.

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| s02f07 | Cạnh trên (-y) | Điểm phản xạ mặt đường hơi thưa ở vùng nước hoặc vạch sơn |
| s02f30 | Cạnh trên (-y) | Dải phân cách cứng tạo hàng điểm phản xạ rất rõ nét |
| s03f05 | Cạnh trên (-y) | Bóng che lớn xuất hiện phía sau xe con đi trước |
| s03f59 | Cạnh trên (-y) | Mật độ điểm thưa nhanh ở khoảng cách trên 30 m |
| s04f25 | Cạnh trên (-y) | Xe lớn bên sườn phải che khuất tầm nhìn camera góc rộng |
| s04f46 | Cạnh trên (-y) | Xuất hiện người đi bộ/xe máy di chuyển sát mép đường |
| s05f07 | Cạnh trên (-y) | Cụm điểm phản xạ lạ do tán cây chìa ra lòng đường |
| s05f26 | Cạnh trên (-y) | Xe tải lớn phía trước tạo vùng bóng che hình nón kéo dài |

## 3. Ba cuboid có hướng dẫn (Bước 3)
Ảnh minh chứng: `t2-a-car.png`, `t2-b-bus-truck.png`, `t2-c-ped-cyclist.png`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | s01f58, idx 1 | Car | Đúng (yaw = 270.2°, mũi tên đỏ chỉ đầu xe) | Chạm đường (z_bottom = -1.69 m) | Đã kiểm đủ 3 khung + ảnh cam01 | Đúng (4.43 x 1.81 x 1.57 m) |
| (b) xe buýt hoặc xe tải | s02f07, idx 1 | Bus | Đúng (yaw = 83.8°, xe ngược chiều) | Chạm đường (z_bottom = -1.92 m) | Đã kiểm đủ 3 khung + ảnh camera | Đúng (11.10 x 2.99 x 3.31 m) |
| (c) người đi bộ hoặc xe hai bánh | s01f58, idx 8 | Pedestrian | Đúng (yaw = 273.9°, hướng di chuyển) | Chạm vỉa hè (z_bottom = -1.42 m) | Đã kiểm đủ 3 khung + ảnh camera | Đúng (0.79 x 0.72 x 1.74 m) |

**Ghi nhận hiệu chuẩn và đối tượng nghi vấn:**
- *Vòng hiệu chuẩn:* Các thành viên trong lớp ban đầu có sự lệch nhẹ về chiều dài thân xe khi xe bị che khuất một phần (occluded) và góc xoay yaw ở các xe đỗ nghiêng. Sau khi thống nhất theo guideline: cuboid phải bao trọn cả phần bị che suy đoán được dựa trên kích thước xe tiêu chuẩn, và mũi tên $+X$ bắt buộc phải dọc trục thân xe hướng về đầu xe.
- *Đối tượng bỏ qua vì không đủ cơ sở:* Tại frame `s01f58`, cụm điểm ở lề đường bên phải (~40 m) chỉ có dưới 10 điểm phản xạ rải rác, không nhận diện được hình khối (`LOW_VISIBILITY`), quyết định không đoán và bỏ qua theo đúng guideline.

## 4. Bốn hiện tượng chất lượng (Bước 4)
Nguyên nhân được ghi dưới dạng **giả thuyết**. Các frame tham khảo từ task hiệu chuẩn chung `D13-HC485087` đều ghi rõ nguồn.

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | `s02f00` / `t4-1-bong-che.png` / task chung `D13-HC485087` | Giả thuyết: Thân xe buýt cỡ lớn nằm chắn toàn bộ chùm tia laser LiDAR hướng về phía sau nó | Vùng phía sau xe buýt gần như trống rỗng hoàn toàn không có điểm, trong khi hai bên sườn xe điểm vẫn rất dày | Bật thuộc tính `occluded`, cuboid vẫn phải bao trọn phần kích thước xe suy luận được chứ không cắt ngắn theo phần điểm nhìn thấy |
| 2. Thưa ở xa | `s02f30` / `t4-2-thua-o-xa.png` / task cá nhân `D13-2A202603300` | Giả thuyết: Góc phân kỳ giữa các tia LiDAR khiến khoảng cách giữa các điểm quét tăng mạnh theo khoảng cách | Xe ở khoảng 46 m (`idx 8`) chỉ còn khoảng 72 điểm phản xạ rời rạc thành vài vệt ngang | Chỉ vẽ khi nhận diện được ít nhất 15–20% thể tích vật hoặc có ảnh camera xác thực; nếu quá ít điểm không rõ hình dạng thì bỏ qua, ghi `LOW_VISIBILITY` |
| 3. Phản xạ thấp / ít điểm bất thường | `s01f58` / `t4-3-phan-xa-thap.png` / task cá nhân `D13-2A202603300` | Giả thuyết: Xe màu tối hoặc bề mặt kính hấp thụ/khúc xạ tia hồng ngoại thay vì phản xạ thẳng về cảm biến | Thân xe con màu tối và vùng kính có rất ít điểm hoặc điểm có cường độ phản xạ yếu hơn hẳn các xe sáng màu cùng tầm | Không khẳng định nguyên nhân, ghi nhận giả thuyết và đối chiếu ảnh camera để căn cuboid đúng kích thước tiêu chuẩn |
| 4. Điểm lạ | `s02f35` / `t4-4-diem-la.png` / task chung `D13-HC485087` | Giả thuyết: Phản xạ đa đường (multipath) qua mặt kính/mặt đường ướt hoặc nhiễu cảm biến tạo điểm ảo | Xuất hiện cụm ~46 điểm lơ lửng sâu bên dưới mặt đường khoảng 11 m (ở $z \approx -13{,}5\text{ m}$) tại tầm ~29 m | Tuyệt đối không gán nhãn thành vật thể hay vẽ cuboid cho các cụm điểm nhiễu này |

## 5. Số từ cuboid của tôi (Bước 5)
Số liệu lấy từ `submission/D13-2A202603300-bangA.csv` và cột `points_in` của `submission/D13-2A202603300-tukiem.csv`.

| Cuboid (frame, idx, class) | z (m) | h (m) | z đáy = z − h/2 (m) | dist_m (m) | r = √(x²+y²+z²) (m) | yaw_deg (°) | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| s01f58, idx 1, Car | -0.903 | 1.574 | -1.690 | 2.97 | 3.11 | 270.2° | Chạm đường chuẩn (lệch 0.01 m so với -1.7 m) |
| s01f33, idx 1, Car | -0.965 | 1.581 | -1.755 | 9.15 | 9.20 | 271.9° | Chạm đường chuẩn (lệch 0.055 m so với -1.7 m) |
| s02f30, idx 9, Car | -0.989 | 1.319 | -1.649 | 34.75 | 34.76 | 91.8° | Chạm mặt đường tại vị trí xa (~35 m); `selfcheck` báo cao hơn mặt đường 0.48 m do đường tại đó dốc nhẹ |

- **(a) Cuboid gần nhất và xa nhất:**
  - Gần nhất: `s01f58, idx 1 (Car)` với `dist_m = 2.97 m`, khoảng cách 3D `r = 3.11 m`.
  - Xa nhất: `s01f33, idx 5 (Car)` với `dist_m = 63.67 m`, khoảng cách 3D `r = 63.67 m`.
- **(b) So sánh kích thước với bảng 4.3:**
  - Người đi bộ (`s01f58, idx 8, Pedestrian`): `l = 0.79 m`, `w = 0.72 m`, `h = 1.74 m` $\rightarrow$ Hoàn toàn nằm trong khoảng chuẩn của Người lớn (`l: 0.4–0.9 m`, `w: 0.5–0.9 m`, `h: 1.50–1.95 m`).
  - Xe con (`s01f58, idx 1, Car`): `l = 4.43 m`, `w = 1.81 m`, `h = 1.57 m` $\rightarrow$ Hoàn toàn nằm trong khoảng chuẩn của Sedan/Hatchback (`l: 3.8–5.0 m`, `w: 1.70–1.90 m`, `h: 1.40–1.60 m`).
- **(c) Góc hướng `yaw_deg`:**
  - Xe cùng chiều phía trước (`s01f58, idx 1, Car`): `yaw_deg = 270.2°` (rất sát mốc `270°`).
  - Xe đi ngược chiều (`s02f07, idx 8, Car`): `yaw_deg = 86.1°` (rất sát mốc `90°`).

Mật độ theo khoảng cách (4 cuboid `Car` ở các `dist_m` khác nhau):

| Cuboid (frame, idx, class) | dist_m (m) | Số điểm (`points_in`) |
|---|---|---|
| s05f07, idx 3, Car | 4.9 m | 5005 |
| s05f07, idx 7, Car | 15.0 m | 832 |
| s04f25, idx 4, Car | 25.8 m | 208 |
| s02f30, idx 8, Car | 46.4 m | 72 |

- **Quy luật mật độ và tự kiểm:**
  - *Quy luật:* Khi khoảng cách tăng gấp khoảng 3 lần (từ `4.9 m` lên `15.0 m`), số điểm LiDAR trong hộp giảm khoảng 6 lần (từ `5005` xuống `832` điểm); lên `25.8 m` còn `208` điểm và đến `46.4 m` chỉ còn `72` điểm — bám rất sát đường cong suy giảm trung vị trên slide bài giảng (khoảng 1968 điểm ở 0–10 m, 718 ở 10–20 m, 214 ở 20–30 m, 72 ở 30–40 m).
  - *Các cuboid đã sửa sau `preview` / `selfcheck`:* (1) Sửa `s05f07 #1 Cyclist` từ trạng thái bị xoay ngang (`length < width`) thành dọc theo hướng xe (`2.67 x 1.24 m`); (2) Kéo đáy `s05f07 #13 Car` xuống mặt đường (trước đó bị lơ lửng `0.70 m`); (3) Nâng nhẹ đáy `s04f25 #13 Car` (trước đó hơi chìm `0.30 m`).
  - *Các cảnh báo giữ nguyên:* Giữ nguyên một số cảnh báo bề rộng `width = 2.31–2.45 m` ở các xe SUV/bán tải cỡ lớn (`s02f30 #7`, `s04f25 #9`) vì cần bao trọn cả hai gương chiếu hậu chìa ra ngoài thân xe theo đúng guideline; giữ nguyên cảnh báo `ground_gap_m = 0.48 m` ở `s02f30 #9 Car` vì thực tế đáy hộp đã bám sát mặt đường dốc cục bộ tại khoảng cách xa ~35 m.
- **Đổi kiểm:** Đã đối chiếu trực tiếp hai con số từ Bảng A (`s01f58 #1 Car`: `dist_m = 2.97 m`, `yaw_deg = 270.2°`; `s05f07 #3 Car`: `dist_m = 4.94 m`, `points_in = 5005`), số liệu giữa giao diện CVAT và file CSV khớp nhau 100%.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc
- **(a) Ý nghĩa đối với mô hình học máy:**
  - *Vì sao nhãn 3D dùng mét thay vì pixel:* Mô hình nhận thức 3D của xe tự hành cần biết chính xác khoảng cách vật lý, kích thước thật và quỹ đạo trong không gian thực để ra quyết định phanh/tránh né an toàn, điều mà ảnh 2D theo pixel bị biến dạng phối cảnh không thể cung cấp trực tiếp.
  - *Vì sao vật thưa điểm cần quy tắc riêng:* Nếu người gán nhãn tự đoán mò hình dạng và hướng cho những cụm chỉ có vài điểm rời rạc ở xa, nhãn sẽ bị nhiễu nặng và sai lệch, làm mô hình học phải các mẫu sai (hallucination / false positive).
- **(b) Cách dùng AI:**
  - *Công cụ:* Trợ lý AI lập trình Antigravity.
  - *Câu hỏi chính:* Hỏi cách thiết lập môi trường, cấu trúc hệ tọa độ VF, giải thích các thông số Bảng A/selfcheck và hỗ trợ chạy các lệnh `scripts/lab.py`.
  - *Cách dùng kết quả:* Dùng để kiểm tra cú pháp lệnh, đối chiếu số liệu CSV và chuẩn hóa trình bày báo cáo; toàn bộ cuboid đều tự quan sát và vẽ trực tiếp trên CVAT local, tuyệt đối không tải ảnh camera hay dữ liệu VF lên hệ thống AI ngoài.
- **(c) Điều còn chưa chắc:** Một số xe đậu sát thành hàng ở khoảng cách xa (> 35 m) có độ phân giải điểm thấp nên khó tách biệt ranh giới chính xác tuyệt đối giữa phần đuôi xe trước và đầu xe sau.
