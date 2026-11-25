# Báo cáo Day 13 — Hoàng Trọng Trà, 2A202603342

## 1. Kết luận trước

- MSSV **2A202603342**, task `D13-2A202603342`, job CVAT #28. Môi trường kiểm là Windows 11, trình duyệt Chrome, máy chủ CVAT 2.74.1. Chưa có MSSV bạn ngồi cạnh để ghi.
- 10 frame gồm `s01f26`, `s01f52`, `s02f19`, `s02f46`, `s03f02`, `s03f21`, `s04f12`, `s04f50`, `s05f27` và `s05f48`.

CVAT đã lưu **79 cuboid** gồm 76 Car, 1 Truck, 1 Bus và 1 Cyclist. `s02f46` và `s05f48` có ít hộp nhất, cùng 5 hộp. Riêng `s05f48` có 4 hộp trong phạm vi 50 m vì một hộp ở 75,60 m. Phiếu tự kiểm gắn cờ **5/79 hộp**. Tôi tin hơn vào kích thước và cao độ đáy của các xe gần có nhiều điểm. Tôi còn chưa chắc về lớp và hướng đầu xe ở một số cụm xa hoặc bị che.

- Tôi dùng góc nhìn từ trên để xác định trục, sau đó xoay hoặc dời khung Perspective để nhìn hai bên và phía sau.
- Tôi chọn **Shape**, nháy đúp để đặt hộp, chỉnh tâm và yaw ở **Top**, rồi chỉnh đáy và chiều cao ở **Side** và **Front**.
- Tôi bật **Cuboid orientation** để xem vạch đỏ chỉ đầu xe. Lệnh `python scripts/lab.py preview` dùng để chiếu hộp lên ảnh camera khi đã có Bảng A.

## 2. Đọc frame (Bước 2)

Hai frame đọc kỹ. Ảnh `t1-s01f26-3d.jpg`, `t1-s01f26-top.png`, `t1-s01f52-3d.jpg` và `t1-s01f52-top.png`. Hai ảnh Top có đánh dấu phía trước, +x và +y.

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| s01f26 | Phía trước (−y) chếch xuống trái, +x chếch xuống phải, +y chếch lên phải. | Tương đối phẳng gần xe, thấp hơn LiDAR khoảng 1,69 m. | Khi đọc frame ban đầu, thấy cụm giống thân xe ở sau-phải khoảng 35 m. | Các cụm thuôn thấp có dáng xe con. | Cụm sau-phải khoảng 6 m, phần điểm thấy được trải ngang khoảng 3,4 × 1,6 m. | Cụm trước-trái khoảng 48 m chỉ thấy một phần nhỏ. |
| s01f52 | Phía trước (−y) chếch lên trái, +x chếch xuống trái, +y chếch xuống phải. | Tương đối phẳng gần xe, thấp hơn LiDAR khoảng 1,77 m. | Khi đọc frame ban đầu, thấy cụm giống thân xe ở trước-phải khoảng 26 m. | Các cụm có dáng xe con, trong đó có một cụm trước-trái khoảng 8 m. | Một cụm phía sau gần trục giữa khoảng 6 m và một cụm bên trái khoảng 5 m. | Cụm dài sau-trái khoảng 44 m, chưa phân biệt được xe lớn với vật cố định. |

**Camera** cho thấy vùng chiếu trước-trái ở `s01f26` tối trên cam01 và cam06. Ở cùng frame, vùng sau-phải khoảng 6 m trên cam09 có sắc lam và mảng gần trắng. Ở `s01f52`, cam06 cho thấy vùng bên trái khoảng 5 m tối hơn vùng trước-trái khoảng 8 m. Các màu này mô tả vùng ảnh tương ứng, nên tôi chưa dùng chúng để kết luận màu thân xe.

**LiDAR** cho phép ước lượng độ cao mặt đường dưới gốc LiDAR ở cả hai frame, đo độ trải ngang 3,4 × 1,6 m và phần điểm cao khoảng 1,4 m của cụm sau-phải ở `s01f26`, đồng thời tách được hai cụm bên trái và phía sau ở `s01f52` theo khoảng cách tính bằng mét.

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| s02f19 | Chếch lên trái | Cụm dài khoảng 42 m còn thưa điểm. |
| s02f46 | Bên trái | Cụm lớn trước-trái khoảng 25 m chưa rõ lớp. |
| s03f02 | Bên phải | |
| s03f21 | Chếch lên trái | Một hộp có nhiều điểm sát ngoài mép. |
| s04f12 | Bên phải | Hộp Bus ở xa có chiều rộng cần kiểm lại. |
| s04f50 | Bên trái | Có vài cụm nhỏ chưa phân biệt được người với vật ven đường. |
| s05f27 | Chếch lên phải | |
| s05f48 | Bên trái | Các dải điểm ven đường dễ lẫn với thân xe ở xa. |

## 3. Ba cuboid có hướng dẫn (Bước 3)

Ảnh mẫu `t2-a-car.jpg`, `t2-b-bus-truck.jpg` và `t2-c-ped-cyclist.jpg`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | s01f26, 1 | Car | 268,1°, gần hướng −y. Chưa xác nhận đầu xe trên camera. | Đáy cao hơn mặt đường tại chỗ 0,06 m. | Ảnh T2-a hiển thị Top, Side và Front. Chưa chốt hướng bằng camera. | 4,730 × 2,020 × 1,551 m. Chiều rộng vượt khoảng sedan/hatchback. Chiều cao thấp hơn khoảng SUV/minivan. Chưa chốt kiểu thân xe. |
| (b) xe buýt hoặc xe tải | s04f12, 4 | Truck | 89,9°, gần hướng +y. Chưa xác nhận đầu xe trên camera. | Đáy thấp hơn mặt đường tại chỗ 0,09 m. | Ảnh T2-b hiển thị Top, Side và Front. Chưa chốt kiểu xe bằng camera. | 4,760 × 2,329 × 2,260 m. Dài và cao dưới khoảng xe tải nhẹ, rộng hơn khoảng bán tải. Cần kiểm lại kiểu xe. |
| (c) người đi bộ hoặc xe hai bánh | s01f52, 10 | Cyclist | 277,0°, gần −y và phù hợp với chiều dịch chuyển qua các lượt quét liền kề. | Đáy gần trùng mặt đường ước lượng tại chỗ, chênh 0,00 m sau khi làm tròn. | Ảnh T2-c hiển thị Top, Side, Front và cam07. Hướng lấy từ chuyển động LiDAR. | 1,620 × 0,600 × 1,770 m. Cả ba chiều nằm trong khoảng xe đạp có người. |

Vòng hiệu chuẩn chưa có số đo đối chiếu với bạn khác hoặc ghi chép kết luận của lớp. Khi rà nhãn, tôi dùng vạch đỏ để kiểm đầu xe và so đáy với mặt đường ngay tại vị trí vật.

Các cụm bỏ qua vì chưa đủ cơ sở gồm cụm thấp trước-trái khoảng 23 m và cụm đứng sau-phải khoảng 19 m ở `s01f26` (`UNCERTAIN_CLASS`), cụm xa khoảng 42 m ở `s02f19` (`LOW_VISIBILITY`), cụm lớn trước-trái khoảng 25 m ở `s02f46` (`UNCERTAIN_CLASS`), và ba cụm nhỏ trước-trái 14 m, trước-phải 22 m, sau-trái 31 m ở `s04f50` (`UNCERTAIN_CLASS`). Ở `s05f48`, các cụm dài trước-phải 19 m, trước-trái 23 m và 43 m, sau-trái 34 m vẫn khó tách khỏi dải ven đường. Cụm sau-trái khoảng 43 m chỉ lộ một mặt đứng và cần xem tiếp cam05/cam09 (`UNCERTAIN_CLASS`, `LOW_VISIBILITY`).

## 4. Bốn hiện tượng chất lượng (Bước 4)

Nguyên nhân trong bảng là giả thuyết. Nguồn frame được ghi cùng tên ảnh chụp từ CVAT.

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | s02f00, `t4-1-bong-che.jpg`, task chung `D13-HC485087`. | Xe buýt gần có thể chặn tia LiDAR. | Vùng sau xe buýt thưa rõ so với các vùng liền kề. | Vật bị che một phần chỉ gán khi còn đủ căn cứ suy luận. Bật `Occluded` khi xác định được vật bị che. |
| 2. Thưa ở xa | s01f26, `t4-2-thua-o-xa.jpg`, task cá nhân. | Ở xa, các tia cách nhau rộng hơn trên bề mặt vật. | Car idx 10 cách 35,72 m có 89 điểm. Car idx 1 ở 7,07 m có 1.927 điểm. | Ưu tiên phần thân còn đọc được. Đối chiếu camera hoặc ghi `LOW_VISIBILITY` nếu thiếu căn cứ gán. |
| 3. Phản xạ thấp / ít điểm bất thường | s01f52, `t4-3-phan-xa-thap.jpg`, task cá nhân. | Bề mặt phản xạ yếu, góc tới hoặc che khuất có thể làm giảm tín hiệu. | Car idx 5 ở 22,12 m có 89 điểm, trong khi Car idx 4 ở 21,70 m có 283 điểm. Điểm bên trong idx 5 cũng có intensity thấp hơn trong phép kiểm cục bộ. | So với vật ở cùng tầm và kiểm camera trước khi suy ra kích thước hoặc nguyên nhân. |
| 4. Điểm lạ | s02f35, `t4-4-diem-la.jpg`, task chung `D13-HC485087`. | Có thể do phản xạ gián tiếp hoặc sai số đo. | Cụm rời phía sau-trái ở khoảng 29 m, có cao độ khoảng −13,3 m so với LiDAR, sâu hơn mặt đường xung quanh. | Bỏ qua cụm điểm lạ khi gán vật trên mặt đường. |

## 5. Số từ cuboid của tôi (Bước 5)

Số liệu lấy từ `submission/D13-2A202603342-bangA.csv` và `submission/D13-2A202603342-tukiem.csv`. `points_in` là số điểm trong hộp. Tôi dùng mặt đường tại vị trí hộp để kiểm z đáy vì cao độ mặt đường thay đổi giữa các vùng của frame.

| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| s03f02, 6, Car | −1,098 m | 1,637 m | −1,917 m | 3,76 m | 3,92 m | 271,6° | Đáy thấp hơn mặt đường ước lượng tại chỗ 0,14 m. |
| s05f48, 3, Car | −1,536 m | 1,640 m | −2,356 m | 29,55 m | 29,59 m | 89,5° | Đáy gần trùng cao độ mặt đường ước lượng tại chỗ. |
| s03f21, 8, Car | −0,698 m | 1,280 m | −1,338 m | 42,90 m | 42,90 m | 311,9° | Chưa đủ điểm mặt đường quanh hộp để kết luận. |

(a) Hộp gần nhất là `s03f02` idx 6 với `dist_m` 3,76 m và r 3,92 m. Hộp xa nhất đã lưu là `s05f48` idx 4 với `dist_m` 75,60 m và r 75,61 m, ngoài phạm vi 50 m. Hộp xa nhất trong phạm vi là `s03f21` idx 8 với `dist_m` và r đều làm tròn thành 42,90 m.

(b) Car `s01f52` idx 6 có kích thước 4,380 × 1,670 × 1,510 m. Dài và cao trong khoảng sedan/hatchback. Rộng thấp hơn mốc 1,70 m khoảng 0,03 m, có thể do ít điểm ở mép chìa ra hoặc hộp còn hẹp. Cyclist `s01f52` idx 10 có kích thước 1,620 × 0,600 × 1,770 m, nằm trong khoảng xe đạp có người và chứa 306 điểm LiDAR.

(c) Hộp `s01f52` idx 6 nằm trước LiDAR và có yaw 270,0°, phù hợp với hướng cùng chiều. Hộp `s05f48` idx 3 có yaw 89,5°, phù hợp với hướng ngược chiều suy ra từ các lượt quét liền kề.

Mật độ theo khoảng cách (4 cuboid ở các `dist_m` khác nhau):

| Cuboid (frame, idx, class) | dist_m | Số điểm (`points_in`) |
|---|---|---|
| s03f02, 6, Car | 3,76 m | 2.202 |
| s01f26, 4, Car | 15,28 m | 765 |
| s05f48, 3, Car | 29,55 m | 187 |
| s01f26, 10, Car | 35,72 m | 89 |

Trong bốn mẫu này, hộp xa 35,72 m cách LiDAR khoảng 9,5 lần hộp gần 3,76 m và còn xấp xỉ 4% số điểm. Xu hướng giảm theo khoảng cách giống đường cong bài giảng. Các mốc tham chiếu tương ứng là 1.968, 718, 214 và 72 điểm cho các dải 0–10, 10–20, 20–30 và 30–40 m. Che khuất và góc nhìn cũng ảnh hưởng đến số điểm của từng xe.

Hai hộp đã sửa khi rà hình học là `s01f52` idx 6, đưa yaw về 270,0° và dời tâm cho sát thân xe, và `s05f48` idx 1, thu chiều rộng từ khoảng 2,39 m còn 2,25 m và chiều cao từ khoảng 2,35 m còn 2,29 m. Tôi ghi lại năm cờ đang còn để kiểm trên CVAT. `s01f26` idx 15, `s01f52` idx 9 và `s02f46` idx 4 cần kiểm lại kích thước hoặc lớp. `s03f21` idx 2 cần phân biệt 269 điểm sát mép với điểm của vật bên cạnh. `s04f12` idx 7 cần kiểm chiều rộng 4,864 m và khả năng hộp bao vào cụm cạnh bên.

Đổi kiểm với bạn bên cạnh chưa có bằng chứng xác nhận. Hai số có thể đối chiếu trên Bảng A là `dist_m` 3,76 m của `s03f02` idx 6 và `yaw_deg` 270,0° của `s01f52` idx 6.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc

Nhãn 3D dùng mét để mô hình học kích thước và vị trí của vật trong không gian, vì cùng một xe có thể chiếm số pixel khác nhau ở các khoảng cách khác nhau. Vật thưa điểm cần quy tắc riêng vì phần thân quan sát được có thể chỉ là một mặt, nên vị trí, kích thước và hướng đều khó suy ra hơn.

- Tôi có sử dụng Codex trong quá trình đọc hiểu tài liệu hướng dẫn.
- Nội dung chính là cách đọc point cloud và áp dụng guideline gán nhãn 3D cho các trường hợp khó.
- Tôi dùng kết quả để đối chiếu với bản lưu CVAT và ghi lại những trường hợp còn cần xác nhận.

Tôi còn chưa chắc về những lớp và hướng đầu xe chưa được đối chiếu bằng camera. Ở `s05f48`, idx 5 đã có nhãn Car nhưng chưa có ước lượng mặt đường tại chỗ. Idx 4 vượt phạm vi 50 m, nên chỉ có 4 hộp trong phạm vi ở frame này. Tôi chưa có thông tin xác nhận lỗi phát hiện sau hạn.
