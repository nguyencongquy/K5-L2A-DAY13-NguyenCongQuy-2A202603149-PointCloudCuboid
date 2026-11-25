# Báo cáo Day 13 — Nguyễn Công Quý, 2A202603149

## 1. Kết luận trước
- **Thông tin chung:** Mã số học viên `2A202603149` · Task CVAT `D13-2A202603149` · 10 frame được giao: `s01f05`, `s01f40`, `s02f29`, `s02f54`, `s03f05`, `s03f41`, `s04f02`, `s04f40`, `s05f04`, `s05f45`.
- **Môi trường làm việc:** macOS (Apple Silicon) · Trình duyệt Google Chrome · Phiên bản CVAT Server `2.74.1` (`cvat-sdk 2.83.0`, Python `3.11`) · Đổi kiểm cùng: `2A202603206`.
- **Tóm tắt kết quả:** Toàn bộ 10 frame có tổng cộng **94 cuboid** dạng `Shape` (phân bố: `s01f05=11`, `s01f40=14`, `s02f29=6`, `s02f54=7`, `s03f05=10`, `s03f41=5`, `s04f02=18`, `s04f40=5`, `s05f04=10`, `s05f45=8`); frame ít nhất có **5 cuboid** (`s03f41`, `s04f40`) do đây là đoạn đường thoáng giữa hai nút giao, ít phương tiện trong bán kính `30.0 m`. Kết quả `selfcheck` ghi nhận **8/94 cuboid** có cảnh báo gợi ý (đều là cờ điểm sát ngoài mép do tán cây/rào chắn cạnh vật thể, `0` hộp rỗng, `0` hộp lơ lửng/chìm, `0` hộp chồng lấn). Phần tự tin nhất là độ cao đáy chạm mặt đường (`ground_gap_m ≈ 0.04 m`), phân loại class, thuộc tính `occluded` và hướng `yaw` trên làn chính; phần khó hơn là ước lượng chiều dài thực của các xe ở xa `35.0–39.0 m` khi chỉ thấy một mặt cản xe.

**Cách thao tác CVAT 3D đã thực hiện:**
1. **Đổi góc nhìn:** Kéo chuột trái trên khung 3D Perspective để xoay nhìn thẳng từ trên xuống (Top-down toàn cảnh), kéo chuột phải để tịnh tiến (pan), lăn con lăn để thu/phóng kiểm tra đủ 360° quanh xe (nháy đúp vào khoảng trống để về góc mặc định).
2. **Vẽ và chỉnh một cuboid:** Chọn công cụ Cuboid → chọn nhãn (`Car`, `Truck`, `Bus`, `Cyclist`, `Pedestrian`) → bấm **Shape** (không dùng `Track`, không dùng `Outside`) và nháy đúp vào cụm điểm; chỉnh tâm, dài, rộng và xoay góc `yaw` duy nhất ở khung **Top**, sau đó chỉnh chiều cao và hạ đáy chạm mặt đường ở khung **Side** và **Front**.
3. **Kiểm hướng và đối chiếu ảnh camera:** Tích bật **Appearance → Cuboid orientation** để căn vạch đỏ ngắn (`+X` của hộp) chỉ đúng về đầu xe/hướng di chuyển, bật `occluded` cho vật bị che một phần, đồng thời chạy `python scripts/lab.py preview` để kiểm tra hình chiếu 3D cuboid trên cả 7 góc camera (`cam01`–`cam09`) và bản đồ BEV `35.0 m`.

## 2. Đọc frame (Bước 2)
Hai frame đọc kỹ đầu tiên là `s01f05` và `s01f40`. Ảnh chụp lưu tại: `t1-s01f05-3d.png`, `t1-s01f05-top.png`, `t1-s01f40-3d.png`, `t1-s01f40-top.png` (trên hai ảnh `-top` đã đánh dấu trực tiếp tại tâm LiDAR hướng phía trước `-y`, trục `+x` sang trái và trục `+y` ra sau).

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| `s01f05` | Trên ảnh `t1-s01f05-top.png`, phía trước xe (`-y`) hướng chéo lên góc trên-trái (ngược vạch gốc xanh lá); trục `+x` (vạch đỏ, bên trái xe) chỉ chéo xuống góc dưới-trái, trục `+y` (vạch xanh lá, phía sau) chỉ chéo xuống góc dưới-phải | Khá phẳng trên làn chính, thấp hơn tâm LiDAR khoảng `1.70 m` (`z ≈ -1.66 m` đến `-1.85 m`) | Khoảng `39.0–45.0 m` với ô tô trên làn thẳng | `Car`, `Truck`, hàng cây dải phân cách | Phía trái-sau có xe con màu xanh (`x = 4.129 m, y = 3.410 m, dist_m = 5.36 m`); phía sau có các xe con ở `y ≈ 17.2–26.6 m` và xe tải bên trái-sau (`dist_m = 21.63 m`) | Cụm điểm lẫn vào dải bồn cây phân cách ở `x ≈ 7.5 m, y ≈ -24.0 m` và dãy xe đạp dựng sát bụi cây bên phải (`x ≈ -8.0 m`) bị che tán lá không đủ cơ sở tách riêng |
| `s01f40` | Trên ảnh `t1-s01f40-top.png`, phía trước xe (`-y`) hướng sang cạnh trái (hơi chếch xuống dưới); trục `+x` (vạch đỏ, bên trái xe) chỉ xuống cạnh dưới-phải, trục `+y` (vạch xanh lá, phía sau) chỉ sang cạnh phải-trên | Tương đối phẳng ở làn ô tô (`z ≈ -1.77 m`), phần làn xe hai bánh/vỉa hè bên phải cao hơn lòng đường khoảng `0.35–0.50 m` (`z ≈ -1.19 m` đến `-1.40 m`) | Khoảng `32.0–44.0 m` dọc trục đường chính | `Car`, `Cyclist`, `Pedestrian` | Ngay sát trái-trước có xe con màu trắng (`x = 3.004 m, y = -1.109 m, dist_m = 3.20 m`); dọc làn phải (`x ≈ -9.0 m` đến `-11.1 m`) có nhiều người đi xe hai bánh và người đi bộ cả phía trước lẫn phía sau | Một số cụm điểm nhỏ cách `> 38.0 m` gần dải phân cách giữa và dưới tán cây bên phải chỉ còn vài điểm rời rạc, không phân biệt chắc chắn là người đi bộ hay cọc tiêu |

**Đối chiếu giữa 7 Camera và LiDAR trên `s01f05`, `s01f40`:**
- **3 điều chỉ Camera cho biết:** (1) Màu sắc thân xe, vạch sơn phân làn liền/đứt và tín hiệu đèn giao thông/đèn hậu phanh đỏ; (2) Phân biệt chính xác xe hai bánh đang có người ngồi lái (`Cyclist`) hay xe hai bánh dựng trống không người bên lề (không gán nhãn); (3) Nhận diện mặt trước (lưới tản nhiệt, kính lái) so với mặt sau xe ở tầm xa để chốt chiều mũi tên `yaw` không bị ngược `180.0°`.
- **3 điều chỉ LiDAR cho biết:** (1) Tọa độ tâm `(x, y, z)` và khoảng cách ngang thực tế chính xác tới từng mét (`dist_m`) để xét ngưỡng `30.0 m` và `50.0 m`; (2) Kích thước hình học 3D thực (`length, width, height` tính bằng `m`) và độ cao mặt đường cục bộ (`z ≈ -1.70 m`) không bị biến dạng phối cảnh xa-gần như ảnh 2D; (3) Cấu trúc không gian 360° liên tục bao gồm cả các khe mù hẹp giao thoa giữa hai góc quạt camera hông.

**Tám frame lướt còn lại:**

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ / điểm đáng chú ý (nếu có) |
|---|---|---|
| `s02f29` | Cạnh trên khi xoay chuẩn hướng `cam01` (`-y` hướng lên, `+x` sang trái) | Có hàng rào kim loại phân cách sát bên trái (`x ≈ 1.9 m`); xuất hiện một số điểm nhiễu âm dưới mặt đất (`z < -3.5 m`) |
| `s02f54` | Cạnh trên (`-y` hướng lên, `+x` sang trái) | Có cụm **207 điểm lạ dưới mặt đường** xuống tới `z ≈ -7.9 m` (thấp hơn mặt đường hơn `6.0 m`); hàng rào trắng che nửa dưới các xe làn ngược chiều |
| `s03f05` | Cạnh trên (`-y` hướng lên, `+x` sang trái) | Tường rào công trình cao bên trái (`x ≈ 4.8 m`) tạo vùng bóng che khuất hoàn toàn phía sau tường; 2 làn xe dừng sát nhau cùng chiều `-y` |
| `s03f41` | Cạnh trên (`-y` hướng lên, `+x` sang trái) | Cảnh thoáng giữa hai nút giao (5 vật thể rõ trong `30.0 m`), có hàng rào trắng tách làn chính với đường gom bên phải |
| `s04f02` | Cạnh trên (`-y` hướng lên, `+x` sang trái) | Mật độ đông nhất (`18` cuboid), bên phải (`x ≈ -15.0 m` đến `-29.0 m`) có khu vực trạm dịch vụ nhiều xe con và người đi bộ; xe MPV tối màu sát phải (`4.51 m`) tạo bóng che lớn |
| `s04f40` | Cạnh trên (`-y` hướng lên, `+x` sang trái) | Hàng rào phân cách màu vàng bên trái (`x ≈ 1.7 m`) mảnh và dài dễ gây nhầm nếu không đối chiếu `cam01`/`cam06` |
| `s05f04` | Cạnh trên (`-y` hướng lên, `+x` sang trái) | Ngay sát phía sau đuôi xe (`x = -0.083 m, y = 10.625 m`) có xe tải lớn chỉ lộ mặt cabin phẳng trước tia LiDAR; dải cây thấp phân cách ở `x ≈ 5.3 m` |
| `s05f45` | Cạnh trên (`-y` hướng lên, `+x` sang trái) | Nút giao ngã tư lớn: các xe con và xe tải ben công trình ở nhánh phải (`x ≈ -27.3 m` đến `-34.0 m`) dừng chờ đèn đỏ hướng về giao lộ (`yaw ≈ 11.0°–13.0°`), nhánh trái có xe tải hướng ngược lại (`yaw = 192.0°`) |

## 3. Ba cuboid có hướng dẫn (Bước 3)
Ảnh chụp minh họa lưu tại: `t2-a-car.png`, `t2-b-bus-truck.png`, `t2-c-ped-cyclist.png`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) Xe con gần nhất | `s01f05`, `idx=1` | `Car` | Đúng (`yaw_deg = 269.0°`, cùng chiều `-y` khớp đầu xe trên `cam05`/`cam06`) | Chạm đường (`z_bottom = -1.738 m`, `ground_gap_m = 0.04 m`) | Đã kiểm đủ 3 chiếu + `preview` | Khớp Sedan Bảng 4.3 (`4.156 m × 1.860 m × 1.467 m`) |
| (b) Xe buýt hoặc xe tải | `s01f05`, `idx=7` | `Truck` | Đúng (`yaw_deg = 89.0°`, đi ngược chiều `+y` bên kia dải phân cách khớp `cam05`/`cam09`) | Chạm đường (`z_bottom = -1.802 m`, `ground_gap_m = 0.04 m`) | Đã kiểm đủ 3 chiếu + `preview` | Khớp Xe tải hộp nhẹ Bảng 4.3 (`5.788 m × 2.174 m × 2.702 m`) |
| (c) Người đi bộ hoặc xe hai bánh | `s01f40`, `idx=3` | `Cyclist` | Đúng (`yaw_deg = 261.0°`, di chuyển dọc làn xe hai bánh bên phải khớp `cam07`/`cam08`) | Chạm đường (`z_bottom = -1.190 m` trên nền làn phụ cao, `ground_gap_m = 0.04 m`) | Đã kiểm đủ 3 chiếu + `preview` | Khớp Xe máy có người Bảng 4.3 (`1.920 m × 0.780 m × 1.660 m`) |

- **Ghi nhận từ vòng hiệu chuẩn & chuẩn hóa cách hiểu guideline:** Hai sai lệch phổ biến nhất khi vẽ 3D cuboid là: (1) chỉ bao phần điểm LiDAR nhìn thấy của xe ở xa mà quên suy luận phần thân bị tự che khuất (làm hộp ngắn còn `2.5–3.0 m` và tâm lệch gần LiDAR), và (2) nhầm xe hai bánh dựng trống trên vỉa hè thành `Cyclist`. Nguyên tắc chuẩn theo guideline: bám sát mặt nhìn thấy gần LiDAR nhưng mở rộng chiều khuất theo kích thước thực kết hợp đối chiếu camera, bật `occluded` khi bị vật khác hoặc rào chắn che một phần, và không gán xe hai bánh không có người lái.
- **Các vật bỏ qua vì không đủ cơ sở:**
  1. `s01f05` (`x ≈ -8.0 m, y ≈ 8.0–12.0 m`, bên phải sau bụi cây): dãy xe đạp chia sẻ dựng trên vỉa hè không có người ngồi lái → không gán nhãn theo đúng mục 1 `GUIDELINE.md`.
  2. `s01f40` (`x ≈ -8.0 m, y ≈ -33.0 m`, `dist_m ≈ 34.0 m`): cụm điểm lưa thưa dưới `15 điểm` bị che khuất nặng sau hàng xe và tán cây (`LOW_VISIBILITY`, thấy dưới `15%` thể tích và ảnh camera bị khuất) → bỏ qua, không đoán mò.
  3. `s03f41` (`x ≈ -4.1 m, y ≈ 46.7 m`, `dist_m ≈ 46.9 m`): cụm chỉ còn `7 điểm` LiDAR ở rìa gần `50.0 m` phía sau (`UNCERTAIN_SCOPE` / `LOW_VISIBILITY`, dưới ngưỡng `15–20%` thể tích) → bỏ qua để giữ chất lượng nhãn.

## 4. Bốn hiện tượng chất lượng (Bước 4)
Ảnh chụp lưu tại: `t4-1-bong-che.png`, `t4-2-thua-o-xa.png`, `t4-3-phan-xa-thap.png`, `t4-4-diem-la.png`.

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | `s04f02` / `t4-1-bong-che.png` / Task cá nhân `D13-2A202603149` | **Giả thuyết:** Tia laser phát ra theo phương xuyên tâm từ nóc xe bị thân chiếc MPV cỡ lớn ở rất gần bên phải (`idx=1`, `dist_m = 4.51 m`, cao `1.828 m`) chắn hoàn toàn đường truyền thẳng tới vùng không gian phía sau nó | Phía sau lưng hộp `s04f02 #1` xuất hiện một hình quạt tối trống rỗng hoàn toàn không có điểm mặt đường lẫn điểm vật thể, dù hai bên rìa quạt điểm vẫn dày đặc | Vùng trống sau vật chắn là vùng **chưa quan sát được** chứ không phải đường trống; các vật phía sau bị che một phần phải bật thuộc tính `occluded = 1` và bao trọn phần thể tích suy luận được |
| 2. Thưa ở xa | `s01f05` / `t4-2-thua-o-xa.png` / Task cá nhân `D13-2A202603149` | **Giả thuyết:** Các chùm tia LiDAR 40 tia phân kỳ theo góc phương vị và góc ngẩng khi truyền đi xa, khiến khoảng cách giữa hai tia quét liên tiếp giãn rộng theo khoảng cách `dist_m` | Chiếc xe con `s01f05 #10` ở `dist_m = 36.25 m` chỉ còn `66 điểm` và `#11` ở `dist_m = 38.87 m` chỉ còn `90 điểm` (so với `2568 điểm` của xe `#1` ở `5.36 m`), chỉ còn vài vệt cắt ngang đuôi xe | Áp dụng quy tắc `15–20%`: nếu kết hợp ảnh camera xác nhận rõ loại xe và hình dáng đầu/đuôi thì vẽ cuboid đủ kích thước tham chiếu Bảng 4.3; nếu quá thưa dưới `15%` và camera mờ thì bỏ qua và ghi `LOW_VISIBILITY` |
| 3. Phản xạ thấp / ít điểm bất thường | `s05f04` / `t4-3-phan-xa-thap.png` / Task cá nhân `D13-2A202603149` | **Giả thuyết:** Bề mặt sơn tối màu hoặc kính chắn gió/kính sườn đặt nghiêng có thể hấp thụ năng lượng tia hồng ngoại hoặc phản xạ gương lệch ra hướng khác thay vì tán xạ ngược về đầu thu LiDAR | Trên `s05f04`, xe con tối màu `#7` ở `dist_m = 23.69 m` chỉ thu được `83 điểm` và khuyết hẳn mảng kính, ít hơn hẳn so với xe sáng màu ở cùng cự ly (`#8` ở `23.95 m` có `262 điểm`) | Không thu hẹp chiều cao hay chiều dài hộp theo phần kính bị khuyết điểm; phải đối chiếu ảnh camera và mặt đường để dựng hộp bao trọn cả phần nóc kính, đồng thời chỉ ghi nhận nguyên nhân dưới dạng giả thuyết |
| 4. Điểm lạ | `s02f35` / `t4-4-diem-la.png` / Mượn từ task chung `D13-HC485087` *(trên frame cá nhân `s02f54` cũng có 207 điểm ở `z = -3.5 m` đến `-7.9 m`)* | **Giả thuyết:** Hiện tượng phản xạ đa đường (multipath reflection) khi tia laser đập vào bề mặt kính/kim loại bóng hoặc mặt đường rồi phản xạ tiếp trước khi về đầu thu, làm thời gian bay (ToF) kéo dài khiến tọa độ tính ra chìm sâu xuống dưới lòng đất | Khi xoay nghiêng khung 3D nhìn xuống dưới mặt phẳng đường của `s02f35`, thấy rõ cụm khoảng `46 điểm` lơ lửng biệt lập ở `z ≈ -13.5 m` (thấp hơn mặt đường khoảng `11.0 m`, cách xe `≈ 29.0 m`) | Tuyệt đối **không gán nhãn thành vật thể** và không kéo đáy cuboid tụt theo các điểm nhiễu âm dưới mặt đất |

## 5. Số từ cuboid của tôi (Bước 5)
Số liệu trích xuất trực tiếp từ `submission/D13-2A202603149-bangA.csv` và cột `points_in` của `submission/D13-2A202603149-tukiem.csv`:

| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| `s01f40`, `idx=1`, `Car` *(gần nhất)* | `-0.999 m` | `1.550 m` | `-1.774 m` | `3.20 m` | `3.35 m` | `271.0°` | Chạm đường chuẩn (`ground_gap_m = 0.04 m`, mặt đường quanh xe `≈ -1.81 m`) |
| `s03f05`, `idx=2`, `Pedestrian` | `-0.825 m` | `1.680 m` | `-1.665 m` | `9.79 m` | `9.82 m` | `270.0°` | Chạm đường/vỉa hè chuẩn (`ground_gap_m = 0.04 m`, lệch `< 0.05 m` so với mốc `-1.70 m`) |
| `s01f05`, `idx=11`, `Car` *(xa nhất)* | `-0.864 m` | `1.650 m` | `-1.689 m` | `38.87 m` | `38.88 m` | `271.0°` | Chạm mặt đường chuẩn (`ground_gap_m = 0.04 m`, sát mốc `-1.70 m`) |

- **(a) Khoảng cách gần nhất và xa nhất:**
  - Cuboid **gần nhất** toàn bài là `s01f40`, `idx=1` (`Car` tại `x = 3.004 m, y = -1.109 m, z = -0.999 m`): khoảng cách ngang `dist_m = 3.20 m`, khoảng cách 3D tới tâm hộp `r = 3.35 m` (chênh `0.15 m` do ở cự ly rất gần thành phần độ cao `|z| = 0.999 m` đóng góp rõ vào cạnh huyền 3D).
  - Cuboid **xa nhất** toàn bài là `s01f05`, `idx=11` (`Car` tại `x = 5.268 m, y = 38.511 m, z = -0.864 m`): khoảng cách ngang `dist_m = 38.87 m`, khoảng cách 3D `r = 38.88 m` (ở xa gần `39.0 m`, `r` và `dist_m` chỉ chênh `0.01 m` vì `|z| ≪ dist_m`).
- **(b) Đối chiếu kích thước với Bảng 4.3:**
  - Với người đi bộ `s03f05`, `idx=2` (`Pedestrian`): `length = 0.860 m, width = 0.706 m, height = 1.680 m` — cả 3 chiều đều nằm gọn trong khoảng Người lớn Bảng 4.3 (`0.4–0.9 m × 0.5–0.9 m × 1.50–1.95 m`). Ngoài ra, với xe hai bánh `s05f45`, `idx=5` (`Cyclist`: `1.920 m × 0.939 m × 1.660 m`), chiều rộng `0.939 m` vượt ngoài khoảng Xe đạp có người (`0.50–0.70 m`) vì đây là xe máy tay ga có gương/tay lái rộng và người lái chống chân tại vạch dừng, phù hợp nhóm Xe máy có người (`0.70–0.95 m`).
  - Với xe con `s01f05`, `idx=6` (`Car`): `length = 4.292 m, width = 1.860 m, height = 1.944 m` — nếu so với dòng Sedan/hatchback (`height: 1.40–1.60 m`) thì chiều cao `1.944 m` nằm ngoài khoảng Sedan vì thực tế trên ảnh camera đây là xe SUV/MPV gầm cao, hoàn toàn khớp dòng SUV/minivan của Bảng 4.3 (`4.4–5.3 m × 1.80–2.10 m × 1.65–2.10 m`).
- **(c) Kiểm tra góc `yaw_deg` theo chiều giao thông:**
  - Xe con đi cùng chiều phía trước mặt xe thu dữ liệu `s01f05`, `idx=2` (`x = 2.198 m, y = -14.512 m`) có `yaw_deg = 269.0°` — rất sát giá trị lý thuyết `270.0°` (hướng `-y`).
  - Xe con đi ngược chiều bên kia dải phân cách `s01f40`, `idx=6` (`x = 8.963 m, y = -16.421 m`) có `yaw_deg = 90.0°` (và `s01f40`, `idx=4` có `yaw_deg = 91.0°`) — khớp sát giá trị lý thuyết `90.0°` (hướng `+y`).

**Mật độ điểm theo khoảng cách (4 cuboid `Car` ở 4 dải cự ly):**

| Cuboid (frame, idx, class) | dist_m | Số điểm (`points_in`) |
|---|---|---|
| `s01f05`, `idx=1`, `Car` | `5.36 m` | `2568 điểm` |
| `s01f05`, `idx=2`, `Car` | `14.68 m` | `619 điểm` |
| `s01f40`, `idx=9`, `Car` | `24.04 m` | `227 điểm` |
| `s01f05`, `idx=11`, `Car` | `38.87 m` | `90 điểm` |

- **Quy luật mật độ & đối chiếu bài giảng:** Khi khoảng cách `dist_m` tăng gấp khoảng `2.7–3.0` lần (từ `5.36 m` lên `14.68 m`, hoặc từ `14.68 m` lên `38.87 m`), số điểm LiDAR trên xe con giảm mạnh còn khoảng `1/4` đến `1/7` (xấp xỉ tỷ lệ nghịch bình phương khoảng cách `1 / dist_m²` do độ phân kỳ góc cả theo phương ngang lẫn phương đứng). Bốn con số thực đo (`2568 → 619 → 227 → 90 điểm` qua các khoảng `0–10 m`, `10–20 m`, `20–30 m`, `30–40 m`) bám rất sát đường cong trung vị ở slide bài giảng (`1968 → 718 → 214 → 72 điểm`).
- **Tự kiểm và chỉnh sửa sau `preview` / `selfcheck`:**
  - **Các cuboid đã sửa sau khi kiểm tra `preview` và `selfcheck`:** (1) Tại `s05f45`, sửa nhãn chiếc xe ben công trình màu xanh ở nhánh phải từ `Bus` về đúng `Truck` và xoay hộp xe tải xanh ở nhánh trái (`idx=3`) về đúng góc đường ngang `192.0°`; (2) Tại `s03f05`, xoay hộp xe con bị khuất ở làn trái (`idx=9`) từ `182.0°` về đúng `270.0°` dọc làn cùng chiều, bổ sung các xe con gần hàng rào ở `s02f29`, `s02f54` kèm bật `occluded = 1` và xóa hộp nhiễu lùm cây ở `s05f04`.
  - **Các cảnh báo giữ nguyên kèm lý do:** 8 cờ `điểm sát ngay ngoài mép` còn lại (`s01f05 #4`, `s01f40 #2`, `s01f40 #5`, `s02f54 #1`, `s03f05 #9`, `s03f41 #4`, `s04f02 #15`, `s05f04 #6`) được giữ nguyên sau khi kiểm tra lại point cloud vì các điểm nằm trong lớp vỏ `0.25 m` bên ngoài hộp thực chất thuộc về tán cây phân cách, hàng rào kim loại sát mép làn đường, trụ mái che trạm dịch vụ hoặc cây xanh vỉa hè sát người đi bộ; nếu nới rộng hộp để bao các điểm đó thì kích thước hộp sẽ bị phình sai kích thước thật của vật thể.
- **Đổi kiểm chéo:** Đã thực hiện đổi kiểm trực tiếp trên màn hình với học viên mã số `2A202603206` đối chiếu hai con số ở Mục 5 là `z_bottom = -1.774 m` (của `s01f40 #1`, kiểm từ `z = -0.999 m` và `height = 1.550 m`) và `dist_m = 38.87 m` (của `s01f05 #11`, kiểm từ `x = 5.268 m, y = 38.511 m`): kết quả cả hai con số đều **khớp chính xác** với `D13-2A202603149-bangA.csv` (chỉ đối chiếu trên màn hình, không gửi file CSV).

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc
- **(a) Ý nghĩa của nhãn đối với mô hình học máy 3D:**
  1. **Vì sao nhãn 3D phải đo bằng mét (`m`) thay vì pixel:** Mô hình phát hiện vật thể 3D cho xe tự hành cần dự đoán trực tiếp vị trí, kích thước vật lý bất biến theo khoảng cách và vận tốc thực trong không gian thế giới thực để hệ thống lập kế hoạch chuyển động (motion planning) tính khoảng cách phanh an toàn và bề rộng lách tránh; đơn vị pixel trên ảnh 2D bị co giãn theo phép chiếu phối cảnh và phụ thuộc tiêu cự từng ống kính nên không thể biểu diễn hình học 3D nhất quán quanh 360°.
  2. **Vì sao vật thưa điểm cần quy tắc ngưỡng riêng (`15–20%` thể tích / `LOW_VISIBILITY`):** Nếu ép gán nhãn phỏng đoán lên các cụm chỉ có `2–4` điểm quá thưa ở xa mà không đủ bằng chứng hình thái, bộ dữ liệu huấn luyện sẽ chứa nhãn nhiễu sai tâm/sai hướng khiến mô hình học nhầm đặc trưng nhiễu nền và tăng tỷ lệ báo động giả (false positives); ngược lại, có ngưỡng rõ ràng giúp ranh giới giám sát (supervision boundary) của hàm mất mát nhất quán.
- **(b) Dùng AI:** Không dùng.
- **(c) Điều còn chưa chắc:**
  - Ở các xe con cách xa trên `34.0–39.0 m` như `s01f05 #10, #11`, tia LiDAR chỉ quét trúng mặt cản gần cảm biến (`66–90 điểm`), nên chiều dài `length` phải dựa một phần vào kích thước chuẩn của dòng xe kết hợp ảnh camera; vị trí tâm dọc trục thân xe có thể dao động nhẹ `±0.20 m` so với kích thước thực của từng mẫu xe cụ thể.
