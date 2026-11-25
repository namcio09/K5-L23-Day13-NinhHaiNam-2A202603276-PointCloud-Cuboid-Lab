# Báo cáo Day 13 — Ninh Hải Nam, 2A202603276

## 1. Kết luận trước

Mã số 2A202603276, task CVAT `D13-2A202603276` gồm 10 frame: s01f24, s01f41, s02f05, s02f46, s03f04, s03f32, s04f15, s04f33, s05f04, s05f37. Máy Windows, trình duyệt Chrome, CVAT phiên bản 2.81.0. MSSV bạn ngồi cạnh: Chu Tuấn Ngọc - 2A202603234
Toàn bài tôi vẽ được 132 cuboid. Frame ít nhất là s03f32 với 7 hộp, vì cảnh frame này thưa vật và phần lớn vật nằm xa trên 30 m. Phiếu tự kiểm báo 12 trong 132 cuboid còn cảnh báo, chủ yếu là nhóm nhỏ (người đi bộ, xe hai bánh) bị cờ "quá chật" và vài hộp ở rất xa bị cờ kích thước. Tôi tin nhất ở các xe con cự ly gần dưới 20 m vì điểm dày và hộp khớp rõ ở cả ba khung Top/Side/Front. Phần tôi còn yếu nhất là hướng của xe hai bánh và người đi bộ, cùng các vật trên 30 m vì điểm thưa nên khó căn đáy và kích thước.

Cách dùng CVAT 3D của tôi gồm ba việc chính. Thứ nhất là đổi góc nhìn: xoay khung 3D để nhìn từ trên xuống, kéo chuột trái để xoay và chuột phải để dời. Thứ hai là vẽ và chỉnh cuboid: bấm Shape, nháy đúp để đặt hộp, rồi chỉnh ở Top cho tâm và dài rộng, chỉnh ở Side và Front cho chiều cao để đáy chạm đường. Thứ ba là bật Appearance → Cuboid orientation để thấy vạch đỏ `+X`, kéo vạch này dọc thân xe và chỉ về đầu xe, rồi dùng `python scripts/lab.py preview` để xem hộp của mình đè lên ảnh camera.

## 2. Đọc frame (Bước 2)

Hai frame tôi đọc kỹ là s01f24 và s01f41. Ảnh đi kèm là `t1-s01f24-3d`, `t1-s01f24-top`, `t1-s01f41-3d`, `t1-s01f41-top`.

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| s01f24 | Trên ảnh top, phía trước xe là cạnh trên ứng với −y; +x chỉ sang trái, +y chỉ xuống dưới (ra sau) | Đường phẳng, thấp hơn LiDAR khoảng 1,7 m | Khoảng 67 m (một xe con phía sau) | Xe con, xe buýt, xe hai bánh, người đi bộ | Có xe buýt phía sau bên trái và nhiều xe con phía sau; hai bên trái có nhóm xe hai bánh và người đi bộ cách 7–15 m | Một cụm điểm thưa ở phía trước bên trái trên 30 m, chưa rõ là xe hay vật tĩnh |
| s01f41 | Cũng vậy: phía trước là −y ở cạnh trên, +x sang trái, +y xuống dưới | Đường phẳng, thấp hơn LiDAR khoảng 1,7 m | Khoảng 60 m (một xe con phía sau) | Xe con, xe hai bánh, người đi bộ | Phía sau có vài xe con; bên trái có dãy xe hai bánh và người đi bộ cách 10–20 m | Cụm điểm ở bên trái xa trên 20 m, điểm rất thưa nên chưa chắc là người hay vật bên đường |

Ba điều chỉ camera cho biết: màu và kiểu dáng xe để chọn class, đầu hay đuôi xe để đặt mũi tên hướng, và có biển báo hay đèn tín hiệu trong cảnh. Ba điều chỉ LiDAR cho biết: khoảng cách thật theo mét, kích thước thật của vật để so bảng 4.3, và hình bao 3D của vật ở những chỗ camera bị che.

Tám frame còn lại tôi chỉ lướt nhanh:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| s02f05 | −y ở cạnh trên | Có xe buýt lớn bên phải, phía sau nó thưa điểm |
| s02f46 | −y ở cạnh trên | Vật chủ yếu phía sau, điểm thưa |
| s03f04 | −y ở cạnh trên | Nhiều xe con xếp hàng phía trước, điểm dày |
| s03f32 | −y ở cạnh trên | Cảnh thưa vật, ít hộp nhất |
| s04f15 | −y ở cạnh trên | Nhiều xe đi ngược chiều bên phải |
| s04f33 | −y ở cạnh trên | Có xe tải lớn rất xa phía sau |
| s05f04 | −y ở cạnh trên | Xe tải lớn ngay trước mặt, điểm rất dày |
| s05f37 | −y ở cạnh trên | Có xe buýt và xe tải phía sau, xa |

## 3. Ba cuboid có hướng dẫn (Bước 3)

Ảnh đi kèm: `t2-a-car`, `t2-b-bus-truck`, `t2-c-ped-cyclist`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | s01f24, #1 | Car | Đúng, yaw 270,7° dọc thân và chỉ về đầu xe | Chạm, z đáy −1,770 m | Đã kiểm đủ ba khung và đối chiếu cam06 (trước-trái) | Đúng: dài 4,28 m, rộng 1,72 m, cao 1,50 m nằm trong khoảng sedan |
| (b) xe buýt | s01f24, #8 | Bus | Đúng, yaw 268,0° | Chạm, z đáy −1,660 m | Đã kiểm đủ ba khung và đối chiếu cam09 | Đúng: dài 10,0 m, rộng 3,0 m, cao 3,06 m trong khoảng xe buýt thường |
| (c) xe hai bánh | s01f24, #13 | Cyclist | Đúng, yaw 47,2° theo hướng đi | Chạm, z đáy −1,601 m | Đã kiểm đủ ba khung và đối chiếu cam07 (trước-phải) | Đúng: dài 1,97 m, rộng 0,75 m, cao 1,57 m trong khoảng xe máy có người |

Trong vòng hiệu chuẩn cuboid của tôi thường lệch cao hơn về trục z so với các bạn, cả lớp chốt guidline như trong repo. Các vật tôi bỏ qua là mật độ điểm quá ít và thưa hoặc bị che không thấy rõ các điểm.
## 4. Bốn hiện tượng chất lượng (Bước 4)

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | s02f05 / `t4-1-bong-che` | Giả thuyết: xe buýt lớn chắn phía sau nên tia LiDAR không tới được vùng khuất | Ngay sau xe buýt gần như trống điểm, trong khi hai bên góc đó vẫn dày điểm | Bật occluded cho vật bị che, cuboid vẫn bao phần suy luận được; vùng trống sau vật là chưa quan sát được chứ không phải đường trống |
| 2. Thưa ở xa | s01f24 / `t4-2-thua-o-xa` | Giả thuyết: mật độ tia LiDAR giảm theo bình phương khoảng cách nên vật trên 30 m chỉ còn vài chục điểm | Một xe con ở khoảng 35 m chỉ còn khoảng 98 điểm, thưa hơn hẳn xe cùng loại ở gần | Chỉ vẽ nếu thấy khoảng 15–20% thể tích hoặc suy luận được từ ảnh camera; không thì bỏ qua và ghi LOW_VISIBILITY |
| 3. Phản xạ thấp hoặc ít điểm bất thường | s02f05 / `t4-3-phan-xa-thap` | Giả thuyết: bề mặt tối màu hoặc kính phản xạ kém nên thu về ít điểm hơn vật cùng tầm | Hai xe con cùng ở khoảng 38 m nhưng một chiếc chỉ có 18 điểm, chiếc kia 175 điểm | Ghi là giả thuyết, không khẳng định; hộp vẽ theo phần suy luận được và đánh dấu occluded nếu bị che |
| 4. Điểm lạ | s01f24 / `t4-4-diem-la` | Giả thuyết: điểm nhiễu hoặc phản xạ bất thường tạo cụm điểm không thuộc vật nào | Có cụm điểm lơ lửng, không nằm trên mặt đường và không gắn với vật nào xung quanh | Không gán thành vật; ghi lại vị trí để báo coach thay vì cố vẽ hộp |

## 5. Số từ cuboid của tôi (Bước 5)

Số lấy từ `submission/D13-2A202603276-bangA.csv`; số điểm lấy từ cột `points_in` của `-tukiem.csv`. Mặt đường khoảng −1,7 m.

| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét |
|---|---|---|---|---|---|---|---|
| s01f24, #1, Car | −1,020 | 1,500 | −1,770 | 6,45 | 6,53 | 270,7 | Chạm đường |
| s03f04, #3, Car | −1,010 | 1,548 | −1,784 | 5,08 | 5,18 | 270,0 | Chạm đường |
| s05f04, #4, Car | −0,827 | 1,688 | −1,671 | 3,29 | 3,40 | 269,6 | Chạm đường |

(a) Cuboid gần nhất là s05f04 #4 với `dist_m` = 3,29 m và r = 3,40 m. Cuboid xa nhất là s04f33 #5 (Truck) với `dist_m` = 72,67 m và r = 72,67 m; hộp này nằm ngoài 50 m nên không được chấm. Hộp xa nhất còn trong vùng chấm là s05f37 #2 (Car) với `dist_m` = 49,16 m và r = 49,17 m.

(b) Một người đi bộ ở s01f24 #3 có dài 0,53 m, rộng 0,70 m, cao 1,90 m, nằm trong khoảng người lớn của bảng 4.3 (dài 0,4–0,9 m, rộng 0,5–0,9 m, cao 1,50–1,95 m). Một xe con ở s03f04 #5 có dài 4,94 m, rộng 2,31 m, cao 2,19 m; chiều dài và chiều cao vẫn trong khoảng SUV/van (dài 4,4–5,3 m, cao 1,65–2,10 m gần sát), nhưng chiều rộng 2,31 m vượt khoảng 1,80–2,10 m của SUV/van khoảng 0,2 m. Tôi nghĩ đây là xe cỡ lớn hoặc hộp bao hơi rộng do gương và thân xe, nên giữ nguyên.

(c) Một xe cùng chiều phía trước là s01f24 #1 có `yaw_deg` = 270,7°, gần 270° như mong đợi. Một xe đi ngược chiều là s01f24 #18 có `yaw_deg` = 91,3°, gần 90°, đúng chiều ngược lại.

Mật độ theo khoảng cách:

| Cuboid (frame, idx, class) | dist_m | Số điểm (`points_in`) |
|---|---|---|
| s05f04, #4, Car | 3,29 | 5650 |
| s03f04, #4, Car | 12,18 | 866 |
| s04f15, #8, Car | 27,59 | 161 |
| s01f24, #11, Car | 46,96 | 58 |

Quy luật: khoảng cách tăng khoảng hai lần thì số điểm còn lại giảm khoảng ba đến sáu lần, vì mật độ tia LiDAR giảm theo bình phương khoảng cách. So với đường cong ở slide bài giảng cho xe con (khoảng 1968 điểm ở 0–10 m, 718 ở 10–20 m, 214 ở 20–30 m, 72 ở 30–40 m), số của tôi ở cự ly gần cao hơn vì hộp bao cả thân xe lớn, còn ở 30–50 m thấp hơn một chút, cùng xu hướng giảm nhanh theo khoảng cách.

Sau khi chạy `preview` và `selfcheck`, tôi đã sửa hai cuboid: xoá hộp s05f37 #7 (Truck) vì hộp rỗng hoàn toàn 0 điểm và đáy chìm 0,45 m, và hạ chiều cao hộp s05f04 #2 (Car) từ 2,52 m xuống 1,54 m cho khớp thân xe. Các cảnh báo tôi giữ nguyên có lý do: nhóm Pedestrian ở s01f24 #3, #4, #17 và s01f41 #14 bị cờ "quá chật" vì đó là hai người đứng sát nhau, điểm lẫn vào nhau chứ không phải hộp đặt sai; hộp s01f24 #22 Cyclist dài 0,91 m ở 34,5 m chỉ có 40 điểm nên tôi giữ theo quy tắc vật thưa; các hộp trên 50 m như s04f15 #11, #12 và s05f37 #5 bị cờ kích thước nhưng nằm ngoài vùng chấm nên tôi không chỉnh.

Đổi kiểm: tôi đã đưa báo cáo mục 5 cho một bạn cùng lớp xem và bạn chọn hai con số là z đáy của s01f24 #1 (−1,770 m) và `dist_m` của cuboid xa nhất trong vùng chấm (49,16 m) để đối chiếu với bảng A mở trên máy tôi; cả hai khớp. Tôi cũng kiểm ngược lại hai con số trong bài của bạn ấy.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc

(a) Nhãn 3D dùng mét vì model học tương quan không gian thật giữa vật và xe, thứ mà pixel không giữ được khi ảnh đổi tỉ lệ và góc nhìn, và mét là đơn vị chung để ghép LiDAR với camera. Vật thưa điểm cần quy tắc riêng vì càng xa thì số điểm càng ít, nếu vẽ tràn lan sẽ dạy model những hộp sai kích thước ở vùng xa nơi dữ liệu vốn không đủ.

(b) Dùng AI: công cụ Command Code (trợ lý lập trình). Câu hỏi chính: cách chạy các lệnh `lab.py`, cách dùng giao diện CVAT 3D, cách đọc các cột trong `bangA.csv` và `tukiem.csv`, và nhờ dựng bản nháp bố cục báo cáo từ số liệu đã có. Kết quả: tôi dùng để sửa lỗi thao tác và dựng khung trình bày, sau đó tự kiểm lại số với bảng A và tự viết lại nhận xét; AI không vẽ cuboid thay tôi.

(c) Điều còn chưa chắc: hướng của vài xe hai bánh và người đi bộ ở cự ly trên 20 m vì điểm thưa nên khó xác định đầu xe; đáy của nhóm vật nhỏ ở xa dễ lệch do mặt đường ước từ ít điểm; và một cụm điểm lạ ở s01f24 tôi chưa xác định được nguyên nhân, chỉ ghi là giả thuyết.
