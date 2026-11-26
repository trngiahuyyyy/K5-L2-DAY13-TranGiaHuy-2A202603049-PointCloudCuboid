# Báo cáo Day 13 — Trần Gia Huy, 2A202603049

## 1. Kết luận trước

- **MSSV:** `2A202603049`
- **Task CVAT:** `D13-2A202603049`
- **10 frame được giao:** `s01f24`, `s01f49`, `s02f29`, `s02f53`, `s03f22`, `s03f41`, `s04f40`, `s04f56`, `s05f05`, `s05f24`
- **Môi trường:** Windows · Trình duyệt: chrome · Phiên bản CVAT: 2.82.1
- **MSSV bạn ngồi cạnh:** 
- **Tổng quan kết quả:** Tổng 98 cuboid (Car 78, Cyclist 12, Pedestrian 5, Bus 2, Truck 1). Frame đầu `s01f24` có 13 cuboid; frame ít nhất là `s03f41` với 4 cuboid (dưới mức 5 của yêu cầu, tôi ghi ở §6c). `selfcheck` cờ 9 cuboid trên tổng 98. Phần tôi tin nhất: class và hướng của xe con cùng chiều phía trước, vì cụm điểm rõ và ảnh camera xác nhận được đầu xe. Phần còn yếu nhất: vật xa 30–50 m và xe tối màu ít điểm, nơi kích thước và đáy chủ yếu dựa vào suy luận.

**Cách dùng CVAT (3 dòng):**
- Đổi góc nhìn: kéo chuột để xoay khung 3D, cuộn chuột để phóng to/thu nhỏ, dùng các nút mũi tên và nhóm phím U I O J K L ở góc khung để dời và xoay góc nhìn; đưa về nhìn từ trên xuống để xác định phía trước (−y), +x, +y.
- Vẽ và chỉnh cuboid: chọn công cụ vẽ cuboid 3D, chọn Shape và đúng class, nháy đúp vào vật để đặt hộp, rồi chỉnh vị trí, kích thước, xoay trong ba khung Top, Side, Front cho hộp sát cụm điểm và đáy chạm mặt đường.
- Kiểm hướng và đối chiếu camera: bật Cuboid orientation để thấy vạch đỏ `+X` chỉ đầu xe, đối chiếu ảnh cam01–cam09 để phân biệt đầu và đuôi xe, rồi chạy `python scripts/lab.py preview` để xem cuboid chiếu lên 7 ảnh camera.

## 2. Đọc frame (Bước 2)

Hai frame đọc kỹ (`s01f24`, `s01f49`). Ảnh: `t1-s01f24-3d.png`, `t1-s01f24-top.png`, `t1-s01f49-3d.png`, `t1-s01f49-top.png` (ảnh nhìn từ trên xuống có đánh dấu phía trước `−y`, `+x`, `+y`).

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| `s01f24` | -y | phẳng, thấp hơn LiDAR ~1,7 m | 27 m | car | Phía sau: xe trắng sát đuôi xe thu dữ liệu (cam09). Hai bên: một hộp dài màu vàng (xe tải/xe buýt) ở bên cạnh, xe máy và người ở vạch qua đường (cam05–cam08) | Các mảng điểm dày ở rìa cảnh là nhà, cây, hàng rào nên không đọc thành vật |
| `s01f49` | -y | phẳng, thấp hơn LiDAR ~1,7 m | 47 m | Car, Cyclist | Phía sau: xe trắng đi sau (cam09). Hai bên: các xe con chạy song song hai làn, hai vật nhỏ màu xanh lá (xe hai bánh) gần xe thu dữ liệu | Cụm điểm thưa ở xa hai đầu đường, quá ít điểm để chắc là vật |

- **3 điều chỉ camera cho biết (từ 2 frame trên):** (1) màu xe, ví dụ nhiều xe trắng ở cam01 và cam09; (2) loại vật và đầu/đuôi xe, ví dụ người đi xe máy ở cam07, cam08; (3) biển báo, đèn tín hiệu, vạch kẻ đường và giàn biển trên cao (cam03 ở `s01f49`).
- **3 điều chỉ LiDAR cho biết (từ 2 frame trên):** (1) khoảng cách và kích thước theo mét của từng vật; (2) vật ở phía sau và hai bên, ngoài hình quạt của cam01, nhờ phủ 360°; (3) độ cao mặt đường và đáy vật (z ≈ −1,7 m) cùng hình dạng 3D mà ảnh 2D không cho.

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| `s02f29` | -y | |
| `s02f53` | -y | |
| `s03f22` | -y | |
| `s03f41` | -y | |
| `s04f40` | -y | |
| `s04f56` | -y | |
| `s05f05` | -y | |
| `s05f24` | -y | |

## 3. Ba cuboid có hướng dẫn (Bước 3)

Ảnh: `t2-a-car.png`, `t2-b-bus-truck.png`, `t2-c-ped-cyclist.png`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | `s01f24`, idx 2 | Car | Đúng: yaw 272,1° ≈ 270° (cùng chiều, xe ở phía trước, y = −5,68 m) | Chạm: z đáy −1,830 m, `ground_gap_m` −0,01 | Có: hộp ôm sát cụm điểm ở Top, Side, Front | l 4,705, w 2,134, h 1,805 m: l và h nằm trong khoảng; w nhỉnh hơn khoảng SUV (1,80–2,10) khoảng 0,03 m, giả thuyết do hộp gồm cả gương |
| (b) xe buýt hoặc xe tải | `s01f24`, idx 6 | Bus | Đúng: yaw 270,4° ≈ 270° | Chạm: z đáy −1,730 m, `ground_gap_m` −0,05 | Có: Top, Side, Front khớp cụm điểm | l 9,25 m thấp hơn khoảng xe buýt thường (10,0–12,5); w 3,105 m cao hơn khoảng 2,50–2,60; h 3,061 m trong khoảng. Hai chiều lệch nên tôi còn nghi ngờ, ghi ở §6c |
| (c) người đi bộ hoặc xe hai bánh | `s01f24`, idx 8 | Cyclist | Đúng: yaw 37,4°, xe máy đang băng ngang vạch qua đường sang trái (+x) nên không ở 270° | Chạm: z đáy −1,638 m, `ground_gap_m` 0,09 | Có: kiểm với ảnh cam07, cam08 thấy người đi xe máy | l 2,219, w 0,890, h 1,677 m: cả ba chiều nằm trong khoảng xe máy có người (1,8–2,3 × 0,70–0,95 × 1,40–1,80) |

- **Vòng hiệu chuẩn:** 
- **Các vật bỏ qua vì không đủ cơ sở:** frame 5 (s03f41), cam sau, 2 Xe con đỏ và trắng , LOW_VISIBILITY.

## 4. Bốn hiện tượng chất lượng (Bước 4)

Nguyên nhân ghi là **giả thuyết**. Frame mượn ghi rõ nguồn (task chung `D13-HC485087`, hoặc MSSV của bạn cho mượn).

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | 0 / `t4-1-bong-che.png` / `s01f24` | Giả thuyết: vật ở gần che tia LiDAR nên phía sau nó không có điểm | Hộp lớn gần như trống, chỉ lác đác vài điểm trong khi vùng xung quanh vẫn có điểm | Vùng trống là chưa quan sát được, không phải đường trống. Nếu có vật thì bật occluded và cuboid bao phần suy luận được |
| 2. Thưa ở xa | 0 / `t4-2-thua-o-xa.png` / `s01f24` | Giả thuyết: LiDAR 40 tia, khoảng cách giữa các tia dãn ra khi xa nên vật chỉ nhận được vài điểm | Các điểm rời rạc, xếp thành hàng theo tia quét, không dựng được hình vật | Vẽ nếu thấy khoảng 15–20% thể tích hoặc suy luận được; không thì bỏ qua và ghi LOW_VISIBILITY |
| 3. Phản xạ thấp / ít điểm bất thường | 0 / `t4-3-phan-xa-thap.png` / `s01f24` | Giả thuyết: thân xe tối màu hoặc kính hấp thụ, phản xạ tia kém | Hộp xe chỉ có vài chục điểm thưa, ít hơn hẳn xe cùng tầm | Không khẳng định nguyên nhân. Dựa vào ảnh camera để xác định class và kích thước, vẽ nếu đủ cơ sở, nếu không thì ghi LOW_VISIBILITY |
| 4. Điểm lạ | 0 / `t4-4-diem-la.png` / `s01f24` | Giả thuyết: nhiễu đo hoặc phản xạ nhiều đường, chưa rõ nguyên nhân | Điểm lẻ loi ở chỗ không có vật, không thuộc cụm nào | Không gán thành vật, không vẽ cuboid cho điểm lạ |

## 5. Số từ cuboid của tôi (Bước 5)

Số liệu lấy từ `submission/D13-2A202603049-bangA.csv`; số điểm lấy từ cột `points_in` của `submission/D13-2A202603049-tukiem.csv`. Mặt đường khoảng −1,7 m (z đáy lệch hơn 0,3 m là lơ lửng hoặc chìm).

| Cuboid (frame, idx, class) | z (m) | h (m) | z đáy = z − h/2 (m) | dist_m (m) | r = √(x²+y²+z²) (m) | yaw_deg (°) | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| `s01f24`, 2, Car | −0,927 | 1,805 | −1,830 | 6,57 | 6,63 | 272,1 | Chạm đường: thấp hơn −1,7 m khoảng 0,13 m, `ground_gap_m` −0,01 |
| `s01f24`, 6, Bus | −0,200 | 3,061 | −1,730 | 21,91 | 21,91 | 270,4 | Chạm đường: lệch 0,03 m, `ground_gap_m` −0,05 |
| `s03f22`, 3, Car | −1,003 | 1,844 | −1,925 | 17,17 | 17,20 | 92,8 | Chìm: z đáy chỉ lệch 0,225 m so với −1,7 m nhưng `ground_gap_m` −0,63 (bị cờ). Giả thuyết: mặt đường quanh hộp cao hơn −1,7 m (dốc hoặc lề) |

- **(a) Cuboid gần nhất và xa nhất:** Gần nhất: `s05f05`, idx 1, Car, `dist_m` 3,41 m, r 3,52 m. Xa nhất: `s04f40`, idx 8, Car, `dist_m` 48,54 m, r 48,55 m. Hai giá trị r và `dist_m` gần như bằng nhau vì |z| nhỏ so với khoảng cách ngang.
- **(b) So kích thước với bảng 4.3:** Người đi bộ `s01f24` idx 10: l 0,517, w 0,566, h 1,732 m, cả ba chiều trong khoảng người lớn (0,4–0,9 × 0,5–0,9 × 1,50–1,95). Xe con `s05f05` idx 1: l 4,764, w 2,123, h 1,738 m. l nằm trong khoảng; w vượt 1,70–1,90 của sedan và nhỉnh hơn khoảng SUV (1,80–2,10) 0,02 m; h vượt khoảng sedan (1,40–1,60) nhưng nằm trong khoảng SUV (1,65–2,10). Giả thuyết: đây là SUV và hộp gồm cả gương chìa ra.
- **(c) Góc hướng `yaw_deg`:** Xe cùng chiều phía trước `s01f24` idx 2: 272,1° (gần 270°). Xe ngược chiều `s02f29` idx 1 (y = −14,65 m, phía trước): 90,7° (gần 90°). Cả hai đúng quy ước.

Mật độ theo khoảng cách (4 cuboid ở các `dist_m` khác nhau):

| Cuboid (frame, idx, class) | dist_m (m) | Số điểm (`points_in`) |
|---|---|---|
| `s05f05`, 1, Car | 3,41 | 5381 |
| `s03f22`, 2, Car | 12,57 | 861 |
| `s02f53`, 1, Car | 23,03 | 251 |
| `s02f29`, 9, Car | 46,12 | 66 |

- **Quy luật mật độ:** Xa gấp khoảng 13,5 lần (3,41 m lên 46,12 m) thì số điểm còn khoảng 1/80 (5381 xuống 66), nhanh hơn 1/d nhưng chậm hơn 1/d² (≈ 1/180). Trung vị của 78 xe con trong bảng A theo bin là 2047 (0–10 m), 770 (10–20 m), 293 (20–30 m), 88 (30–50 m); mỗi bin xa hơn còn khoảng 0,3–0,4 lần, sát đường cong slide (1968, 718, 214, 72).
- **Tự kiểm (`preview` / `selfcheck`):** Phiếu tự kiểm hiện có 9 cờ: `s01f24` idx 1 (w 2,45 m ngoài khoảng Car), `s01f24` idx 11 (Pedestrian, 67 điểm sát mép), `s01f49` idx 6 (w 2,60 m, h 2,48 m ngoài khoảng Car), `s01f49` idx 9 (chìm 0,35 m), `s02f29` idx 7 (chìm 0,29 m), `s03f22` idx 3 (chìm 0,63 m), `s03f22` idx 8 (Bus l 5,72 m), `s04f56` idx 9 (Cyclist l 2,59 m), `s05f05` idx 11 (chìm 0,31 m).
- **Đổi kiểm với bạn ngồi cạnh:** 

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc

- **(a) Ý nghĩa từ phía model học trên nhãn:**
  - Vì sao nhãn 3D đo bằng mét chứ không phải pixel: model dùng nhãn này để học vị trí, kích thước và hướng thật của vật trong không gian, từ đó tính được khoảng cách và lập kế hoạch lái. Pixel thay đổi theo camera và khoảng cách, còn mét thì không, nên một nhãn đo bằng mét dùng được cho mọi cảm biến và mọi khoảng cách.
  - Vì sao vật thưa điểm cần quy tắc riêng: nếu đoán hộp cho vật gần như không có điểm thì nhãn chứa nhiễu, model học theo thứ không có bằng chứng. Quy tắc 15–20% và LOW_VISIBILITY giữ nhãn chỉ gồm những gì có cơ sở, và những ca bỏ qua được ghi lại thay vì bị đoán cho xong.
- **(b) Cách dùng AI:**
  - Công cụ: Antigravity.
  - Câu hỏi chính: đọc các tài liệu của lab, tóm tắt những mục còn thiếu trong báo cáo.
  - Cách dùng kết quả: Tự đối chiếu lại với CVAT, bảng A và phiếu tự kiểm, và tự điền các số liệu từ hai file CSV.
- **(c) Điều còn chưa chắc / lỗi phát hiện sau hạn:**
  - `s03f41` chỉ có 4 cuboid, thiếu 1 so với mức 5 của yêu cầu; tổng 98 cuboid vẫn đạt mức tối thiểu 53.
  - `s01f24` idx 6 (Bus): w 3,105 m và l 9,25 m lệch khỏi khoảng 2,50–2,60 × 10,0–12,5 của bảng 4.3; chưa chắc hộp có bao dư hoặc xe có phải xe buýt thường không.
  - `s03f22` idx 8 (Bus) ở 46,34 m chỉ có 92 điểm và l 5,72 m: class Bus hay Truck chưa chắc (UNCERTAIN_CLASS), vật quanh ngưỡng 50 m (UNCERTAIN_SCOPE).
  - Bốn cuboid Car bị cờ chìm (`s01f49` idx 9, `s02f29` idx 7, `s03f22` idx 3, `s05f05` idx 11): chưa rõ đáy sai hay mặt đường quanh hộp cao hơn −1,7 m (giả thuyết).
