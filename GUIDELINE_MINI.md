# Mini guideline — bài làm cá nhân

> Điền nội dung này trước khi bắt đầu gán nhãn. Giữ lại các quyết định sau khi self-QC.

## Định nghĩa lớp

| Lớp | Định nghĩa của bạn | Ví dụ/điểm phân biệt |
| --- | --- | --- |
| `car` | Xe con cá nhân: sedan, SUV, hatchback, taxi, xe bán tải cỡ nhỏ (cabin đơn). Dùng cho xe đậu, xe đang di chuyển, kể cả bị che một phần. | Phân biệt: nhỏ hơn xe van/truck, không có thùng hàng lớn, không phải xe chở khách nhiều chỗ. Mô tô/xe máy **không** gán. |
| `truck` | Xe tải chở hàng có thùng riêng hoặc xe đầu kéo (semi-truck). Trọng tải thường lớn, buồng lái tách biệt rõ với thùng hàng. | Phân biệt: thùng hàng cao/dài hơn cabin; xe bán tải nhỏ 2 cửa vẫn là `car`. |
| `bus` | Xe buýt chở khách: minibus ≥16 chỗ, xe buýt thành phố, xe khách liên tỉnh. Khung xe dài, có nhiều cửa sổ hành khách liên tiếp. | Phân biệt: thân xe dài đồng nhất, hàng cửa sổ liên tục; không phải van hay xe bán tải. |
| `van` | Xe van/minivan chở hàng hoặc chở người ≤15 chỗ. Thân hộp vuông vức, cửa hậu lớn, cửa trượt bên sườn thường có. | Phân biệt: nhỏ hơn bus, cao hơn car; không có thùng hàng tách biệt như truck. |

## Quy tắc chung

- **Box bám sát vật thể** nghĩa là: vẽ cạnh box chạm mép ngoài phần thân xe nhìn thấy được; không lấy thêm đường, nền trời hay phần xe khác vào trong box.
- **Vật thể bị che khuất:** vẫn gán box quanh phần nhìn thấy được nếu đủ để xác định lớp (ví dụ thấy đầu xe hoặc 1/3 thân xe là đủ). Nếu chỉ còn mảnh rất nhỏ không xác định được lớp thì bỏ qua và ghi vào bảng quyết định dưới.
- **Vật thể bị cắt bởi rìa ảnh:** box chạm đúng rìa ảnh tại cạnh bị cắt; không đoán phần nằm ngoài ảnh; chỉ bao phần nhìn thấy trong frame.
- **Vật thể rất nhỏ/mơ hồ:** nếu không phân biệt được lớp (car/van/truck) thì bỏ qua; nếu phân biệt được thì gán box dù nhỏ; ghi trường hợp vào bảng quyết định.

## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
| --- | --- | --- | --- |
| `drive_...` | | | |
| `drive_...` | | | |
| `drive_...` | | | |
