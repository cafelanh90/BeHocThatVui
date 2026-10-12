# Nhà Bé Học — Website học liệu in cho bé

Website tĩnh, giao diện pastel, đáp ứng điện thoại/máy tính, có 18 chủ đề, tìm kiếm/lọc, chọn 10 hoặc 20 trang, chọn phong cách minh họa, giỏ hàng và biểu mẫu đặt hàng. Có thể triển khai miễn phí bằng GitHub Pages.

## Đưa website lên GitHub Pages
1. Đăng nhập hoặc tạo tài khoản tại https://github.com/.
2. Tạo repository tên `TEN-TAI-KHOAN.github.io` (thay bằng tên tài khoản).
3. Upload `index.html`, `style.css`, `script.js`, `README.md`, `.nojekyll` vào thư mục gốc.
4. Vào **Settings → Pages**, chọn **Deploy from a branch**, branch `main`, folder `/(root)`, bấm **Save**.
5. Mở `https://TEN-TAI-KHOAN.github.io/`.

## Chỉnh sửa giá
Mở `script.js`, tìm mảng `PRODUCTS`. `price10` là giá cho 10 trang, đơn vị VND; giá 20 trang được tính gấp đôi. Giá hiện tại chỉ là giá mẫu 1.000đ, 2.000đ, 3.000đ… theo thứ tự chủ đề. Hãy chỉnh lại trước khi bán thật.

## Thêm chủ đề mới
Thêm một đối tượng vào mảng `PRODUCTS` trong `script.js`, trước dấu `];`. Mỗi `id` phải duy nhất:

```js
{id:"nhan-biet-mau", name:"Nhận biết màu sắc", category:"Tư duy",
 description:"Giúp bé nhận biết và phân biệt màu sắc.", price10:19000, badge:"Chủ đề mới"}
```

- `id`: mã duy nhất, không dấu, không khoảng trắng.
- `name`: tên hiển thị.
- `category`: hiện có `Toán học`, `Tập viết`, `Sáng tạo`, `Tư duy`.
- `description`: mô tả ngắn.
- `price10`: giá cho 10 trang.
- `badge`: nhãn nhỏ trên ảnh.
Sau khi thêm, lưu và upload lại `script.js` lên GitHub. Thẻ sản phẩm tự hiển thị. Để thêm bộ lọc nhóm mới, thêm nút như `<button data-cat="Tiếng Anh">Tiếng Anh</button>` vào `index.html` trong phần `id="filters"`, rồi dùng `category:"Tiếng Anh"` cho sản phẩm.

## Thay hình minh họa
Ảnh tạm hiện đặt trong biến `IMAGE_URL` ở đầu `script.js`. Đổi URL ở đó để đổi ảnh dùng chung. Khi có ảnh riêng, upload ảnh vào thư mục `images/` trong repository, thêm `image:"images/ten-anh.jpg"` vào từng sản phẩm; sau đó sửa thẻ ảnh trong hàm `renderProducts()` và `renderCart()` để dùng ảnh riêng. Chỉ dùng ảnh bạn có quyền sử dụng.

## Phong cách hình minh họa
Các lựa chọn hiện có: `Con Vật`, `Công Chúa`, `Bánh Kẹo`, `Kem`, `Phương tiện giao thông`, hoặc `Mix nhiều chủ đề`. Website ghi nhận lựa chọn khách chọn nhưng không tự tạo nội dung file in; chủ shop tự chuẩn bị theo đơn.

## Lưu đơn tự động vào Google Sheets (miễn phí)
Mặc định, website **không tự lưu đơn** vào tài khoản shop: khách điền thông tin xong sẽ thấy nội dung đơn để sao chép thủ công. Để lưu tự động, có thể dùng Google Apps Script:

1. Tạo Google Sheet mới, ví dụ `Đơn hàng Nhà Bé Học`.
2. Mở **Extensions → Apps Script**, dán mã dưới đây:

```javascript
function doPost(e) {
  try {
    var raw = e && e.parameter && e.parameter.payload;
    if (!raw) throw new Error("Missing payload");
    var order = JSON.parse(raw);
    var ss = SpreadsheetApp.getActiveSpreadsheet();
    var sheet = ss.getSheetByName("DonHang");
    if (!sheet) {
      sheet = ss.insertSheet("DonHang");
      sheet.appendRow(["Thời gian nhận","Mã đơn","Tên người nhận","Số điện thoại","Địa chỉ","Ghi chú","Sản phẩm","Số lượng món","Tạm tính (VND)","Chi tiết JSON"]);
      sheet.setFrozenRows(1);
    }
    var items = (order.items || []).map(function(i) {
      return i.product + " | " + i.pages + " trang | " + i.theme + " | SL " + i.quantity + " | " + i.subtotal + "đ";
    }).join("\\n");
    sheet.appendRow([new Date(),order.orderId || "",order.customer?.name || "",order.customer?.phone || "",order.customer?.address || "",order.customer?.note || "",items,order.itemCount || 0,order.total || 0,JSON.stringify(order)]);
    return ContentService.createTextOutput(JSON.stringify({ok:true})).setMimeType(ContentService.MimeType.JSON);
  } catch(err) {
    return ContentService.createTextOutput(JSON.stringify({ok:false,error:String(err)})).setMimeType(ContentService.MimeType.JSON);
  }
}
```

3. Save, chọn **Deploy → New deployment → Web app**. Chọn **Execute as: Me** và cấu hình quyền truy cập theo yêu cầu. Nếu mở công khai, người có URL có thể gửi dữ liệu đến endpoint; không để thông tin bí mật trong mã.
4. Deploy, cấp quyền nếu được hỏi, sao chép URL Web app kết thúc bằng `/exec`.
5. Đầu `script.js`, thay `const GOOGLE_SCRIPT_URL="";` bằng URL `/exec`, rồi upload `script.js` lên GitHub.
6. Gửi đơn thử và kiểm tra tab `DonHang`.

**Lưu ý:** website gửi theo chế độ `no-cors`, trình duyệt không thể xác nhận chắc chắn rằng dữ liệu đã được ghi thành công. Cần thử kỹ trước khi dùng thật. Nếu không muốn cấu hình Apps Script, tiếp tục dùng quy trình sao chép đơn thủ công.

## Quy trình đơn hàng
Khách chọn chủ đề, số trang, phong cách, số lượng; điền tên, số điện thoại, địa chỉ, ghi chú. Shop kiểm tra, liên hệ xác nhận và tự chuẩn bị sản phẩm in. Website chưa xử lý thanh toán, phí giao hàng, tồn kho hoặc tự tạo file in.
