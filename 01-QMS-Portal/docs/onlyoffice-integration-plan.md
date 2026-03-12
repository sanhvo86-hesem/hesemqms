# Phương án tích hợp OnlyOffice để đạt độ giống Word cao

## Kết luận ngắn
- Nếu yêu cầu “giống Word tối đa” (layout, shape, table, compatibility cao), cần tích hợp **OnlyOffice Document Server**.
- cPanel không phù hợp để chạy Document Server trực tiếp (thường cần Docker/Linux service), nên triển khai mô hình tách lớp.

## Kiến trúc đề xuất
- `qms.hesem.com.vn` (cPanel hiện tại):
  - giữ portal + API nghiệp vụ + lưu trữ file.
  - đóng vai trò WOPI host / file host + auth.
- `docs.hesem.com.vn` (VPS riêng):
  - chạy OnlyOffice Document Server.
  - cấu hình JWT secret + allowlist domain portal.

## Luồng tích hợp
1. Người dùng mở tài liệu trong portal.
2. Portal gọi API tạo `editorConfig` + token JWT ngắn hạn.
3. Frontend nhúng OnlyOffice iframe/editor bằng config ký số.
4. User sửa nội dung trực tiếp trong OnlyOffice.
5. OnlyOffice callback về endpoint portal để lưu version mới.
6. Portal cập nhật workflow/draft/review như hiện tại.

## Endpoint backend cần bổ sung (đề xuất)
- `POST /api.php?action=oo_config`
  - Input: doc code, mode (view/edit), user info.
  - Output: fileUrl, callbackUrl, documentKey, jwt token.
- `POST /api.php?action=oo_callback`
  - Nhận trạng thái save từ OnlyOffice.
  - Tải file mới về lưu vào thư mục version chuẩn QMS.
- `GET /api.php?action=oo_download&doc=...`
  - Endpoint cấp file cho Document Server (có token).

## Bảo mật bắt buộc
- JWT giữa portal và Document Server (secret mạnh, rotate định kỳ).
- Callback verify chữ ký + IP allowlist.
- Token file URL hết hạn ngắn (5-15 phút).
- Chặn truy cập trực tiếp file nhạy cảm không qua token.
- Audit log: ai mở/sửa/lưu/khi nào, phiên nào.

## Tương thích workflow hiện tại
- Giữ logic trạng thái `draft/in_review/approved` ở portal.
- OnlyOffice chỉ thay lớp soạn thảo.
- Mỗi lần callback save -> tạo version manifest giống cơ chế hiện tại.

## Lộ trình triển khai thực tế

## Pha A - PoC (1-2 tuần)
- Dựng VPS + Document Server.
- Tích hợp 1 loại tài liệu pilot.
- Kiểm tra callback save + version.

## Pha B - Pilot hạn chế (2 tuần)
- Bật cho nhóm QA/IT và 10-20 tài liệu trọng điểm.
- Đo tính ổn định, tốc độ, lỗi compatibility.

## Pha C - Rollout (2-4 tuần)
- Mở rộng toàn bộ nhóm tài liệu Office.
- Giữ fallback mở bằng viewer cũ khi Document Server lỗi.

## Năng lực hạ tầng tối thiểu (khuyến nghị)
- VPS 4 vCPU / 8GB RAM / SSD 100GB (pilot nhỏ).
- SSL chuẩn, backup hằng ngày.
- Monitoring CPU/RAM/disk + callback failure alerts.

## Rủi ro & phương án giảm thiểu
- Rủi ro: server docs quá tải.
  - Giảm thiểu: scale VPS, limit session, queue retry callback.
- Rủi ro: callback thất bại mất bản lưu.
  - Giảm thiểu: retry có idempotency key + alert ngay.
- Rủi ro: mismatch quyền sửa/xem.
  - Giảm thiểu: token mode theo role từ workflow portal.

## Tiêu chí đạt “Word-like cao”
- 95%+ tài liệu .docx mở/chỉnh/saved không lỗi bố cục lớn.
- Shape/table/numbering hành vi nhất quán với Word.
- Người dùng nghiệp vụ không cần học lại thao tác cơ bản.
