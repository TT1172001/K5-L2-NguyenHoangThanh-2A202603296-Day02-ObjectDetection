# Mini guideline — bài làm cá nhân

> Điền nội dung này trước khi bắt đầu gán nhãn. Giữ lại các quyết định sau khi self-QC.

## Định nghĩa lớp

| Lớp | Định nghĩa của bạn | Ví dụ/điểm phân biệt |
| --- | --- | --- |
| `car` | Xe chở người cỡ nhỏ: sedan, hatchback, SUV, taxi, xe bán tải, kể cả xe cảnh sát dáng sedan/SUV | Taxi vàng-xanh, SUV đen; thân thấp, có nắp ca-pô và cốp tách khỏi khoang khách |
| `truck` | Xe chở hàng có thùng hoặc thiết bị phía sau cabin: xe tải thùng, xe ben, xe bồn, xe cứu hộ (cẩu kéo) | Cabin tách khỏi thùng hàng; xe tải xanh `drive_009`, xe ben `drive_040`, xe cứu hộ `drive_014` |
| `bus` | Xe buýt, xe buýt khớp nối, xe buýt hai tầng, xe khách, minibus chở khách nhiều hàng ghế | Dài, nhiều cửa sổ dọc thân, biển số tuyến; minibus kem `drive_012` |
| `van` | Xe van/minivan một khối (không tách ca-pô rõ, mái cao), kể cả MPV chở người | Van bạc/trắng kiểu Wuling, van đỏ IVECO `drive_014`; ngắn hơn bus, cao và vuông hơn car |

## Quy tắc chung

- Box bám sát vật thể nghĩa là: bốn cạnh chạm mép phần thân xe nhìn thấy (gồm gương, bánh xe); không lấy bóng, vệt nước phản chiếu trên mặt đường ướt hay xe bên cạnh.
- Vật thể bị che khuất: vẽ box quanh phần nhìn thấy nếu vẫn nhận ra lớp; mỗi xe một box riêng, không gộp hai xe chồng nhau.
- Vật thể bị cắt bởi rìa ảnh: kéo box chạm đúng rìa ảnh, không đoán phần nằm ngoài khung.
- Vật thể rất nhỏ/mơ hồ: không gán xe nhỏ hơn box nhỏ nhất trong nhãn `eval` (khoảng 310 px² trên ảnh 640×640) và không gán cụm xe dày đặc, mờ, dính vào nhau ở cuối đường; không gán xe ba bánh, xe máy, xe đạp, người.

## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
| --- | --- | --- | --- |
| `drive_014` | Xe đỏ IVECO mái cao ở mép phải, nằm giữa van lớn và minibus | `van` | Một khối, chỉ một hàng cửa sổ ngắn, không dài như minibus chở khách; cần giảng viên xác nhận |
| `drive_030` | MPV bạc 7 chỗ (dáng Buick GL8) giữa dòng xe | `van` | Schema xếp minivan chở người vào `van`; mái cao, thân một khối, khác sedan/SUV |
| `drive_014` | Xe cứu hộ cảnh sát trắng có cần cẩu phía sau | `truck` | Cabin tách khỏi phần thiết bị chở/kéo, đúng nhóm xe chở hàng; không phải xe con |
| `drive_003` | Cụm xe rất nhỏ, mờ dưới gầm cầu ở cuối đường | Không gán | Không tách được từng xe để xác định lớp; nhãn `eval` cũng không gán các cụm như vậy |
| `drive_024` | Xe ba bánh chở người có mui | Không gán | Không thuộc bốn lớp của schema |
