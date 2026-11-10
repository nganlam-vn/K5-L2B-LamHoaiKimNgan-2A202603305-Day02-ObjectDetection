# Mini guideline — bài làm cá nhân

> Điền nội dung này trước khi bắt đầu gán nhãn. Giữ lại các quyết định sau khi self-QC và rework.

## Định nghĩa lớp (Thứ tự schema: car=0, van=1, truck=2, bus=3)

| ID | Lớp | Định nghĩa của bạn | Ví dụ / Điểm phân biệt |
| :---: | --- | --- | --- |
| 0 | `car` | Xe con, sedan, hatchback, SUV, CUV, taxi, xe bán tải (pickup). | Taxi và xe bán tải (pickup cabin đôi/lửng) thuộc `car`, không tính là `truck`. |
| 1 | `van` | Xe van, minivan, xe chở khách cỡ nhỏ/vừa (dưới 16 chỗ), xe giao hàng thùng kín liền khối (như Transit, HiAce, Solati). | Minivan chở gia đình hoặc van giao hàng thuộc `van`; xe buýt lớn thuộc `bus`; xe con thông thường thuộc `car`. |
| 2 | `truck` | Xe tải chở hàng thùng rời, xe đầu kéo, xe ben, xe tải chuyên dụng cỡ trung và lớn. | Xe tải có cabin tách biệt với thùng hàng; các loại xe bán tải cabin nhỏ thuộc `car`. |
| 3 | `bus` | Xe buýt công cộng, xe khách liên tỉnh cỡ lớn, xe buýt trường học. | Xe buýt lớn nhiều hàng ghế thuộc `bus`; các dòng minibus nhỏ/van chở hàng thuộc `van`. |

## Quy tắc chung

- **Box bám sát vật thể:** Vẽ bounding box 2D ôm sát phần thân xe nhìn thấy được (tight bounding box), không lấy dư mặt đường, bóng đổ của xe, phần bầu trời hoặc lấn sang xe bên cạnh.
- **Vật thể bị che khuất (Occluded):** Vẫn gán box quanh phần thân xe nhìn thấy được nếu phần đó còn đủ nhận diện rõ loại xe; hai xe chồng lên nhau thì vẽ 2 box riêng biệt cho từng xe ôm sát phần nhìn thấy của mỗi xe.
- **Vật thể bị cắt bởi rìa ảnh (Truncated):** Vẽ box bao quanh phần nhìn thấy và cạnh box chạm sát đúng mép ảnh, tuyệt đối không tự vẽ phỏng đoán phần thân xe nằm ngoài khung hình.
- **Vật thể rất nhỏ/mơ hồ:** Chỉ gán khi phần nhìn thấy đủ để nhận diện chắc chắn loại phương tiện (thường kích thước tối thiểu $\ge 12 \times 10$ pixels); nếu quá nhỏ, mờ căm hoặc chỉ là một đốm sáng/vệt màu không thể xác định loại xe thì bỏ qua, không tự đoán.

## Ghi lại quyết định cho các ca mơ hồ

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
| --- | --- | --- | --- |
| `drive_001.jpg` | Xe ô tô trắng bên phải bị cột đèn che ngang thân; dãy xe con đỗ dọc lề trái bị lùm cây che khuất một phần. | Gán 1 box `car` (0) cho mỗi xe quanh phần nhìn thấy. | Xe đang đỗ vẫn gán theo loại xe; vật thể bị che khuất một phần nhưng nhận diện rõ lớp `car` vẫn gán box ôm sát phần nhìn thấy. |
| `drive_002.jpg` | Xe minivan màu bạc sát rìa phải và xe van trắng ở làn bên trái dễ nhầm với `car` (SUV) hoặc `bus`; các chấm xe rất nhỏ ở xa tít ngã tư. | Gán `van` (1) cho 2 xe minivan/van; không gán các chấm xe quá nhỏ ở xa. | Schema phân biệt minivan/van chở hàng hoặc người thuộc `van`; vật thể quá xa/mơ hồ không xác định chắc lớp thì không tự đoán. |
| `drive_004.jpg` | Xe ô tô đỏ ở góc dưới bên phải bị cắt ngang bởi mép dưới ảnh; trong cảnh đêm có nhiều người đi bộ, xe máy, xe đạp qua đường. | Gán `car` (0) cho xe đỏ với cạnh box chạm đúng mép ảnh; không gán người đi bộ, xe máy, xe đạp. | Vật thể bị cắt khung hình vẫn gán box chạm đúng rìa ảnh; người đi bộ và phương tiện 2 bánh nằm ngoài 4 lớp của schema. |
| `drive_014.jpg` | Xe lớn ở rìa phải bị cắt cạnh và xe ở giữa đường dễ gây phân vân giữa xe tải thùng kín (`truck`) và xe buýt chở khách (`bus`). | Gán `truck` (2) cho xe thùng chở hàng; xác định rõ mục đích chở hàng để gán đúng 1 box duy nhất. | Schema quy định xe chở hàng thùng kín thuộc `truck`, phân biệt với `bus` chở khách; mỗi vật thể chỉ nhận 1 box duy nhất. |
| `drive_024.jpg` | Phương tiện di chuyển ở làn giữa có phần thân sau dạng thùng lửng, dễ phân vân giữa xe bán tải (pickup) thuộc `car` hay xe tải (`truck`). | Gán `car` (0) nếu là xe bán tải/cabin đôi; chỉ gán `truck` (2) nếu là xe tải chở hàng chuyên dụng thùng rời. | Schema nêu rõ: "xe bán tải/cabin nhỏ thuộc car, xe tải chở hàng/xe đầu kéo thuộc truck". |
| `drive_038.jpg` | Cảnh ban đêm có ánh đèn pha chói lóa, các xe ở xa và xe ở rìa trái mép ảnh bị cắt ngang thân dễ nhầm giữa car/van/truck. | Gán `car` (0) ôm sát phần nhìn thấy chạm mép ảnh; các vệt sáng đèn pha xa xôi không rõ hình dạng xe thì bỏ qua không gán. | Tuân thủ quy tắc box cắt rìa ảnh và chỉ gán khi nhìn thấy đủ chi tiết để xác định lớp tin cậy. |
