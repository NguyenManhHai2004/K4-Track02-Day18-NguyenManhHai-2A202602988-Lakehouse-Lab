# Reflection — Anti-pattern: small-file problem

Anti-pattern tôi chọn là **small-file problem**: ghi micro-batch liên tục mà không có job bảo trì.

**Vì sao dễ gặp:** log LLM và sự kiện người dùng thường được ghi từng batch nhỏ. Mỗi commit đều đúng nhưng file tích lũy dần. Ở NB6, 200 commit tạo 200 file (trung bình 51.5 KB); mỗi truy vấn phải mở hết các file, còn object storage tính phí theo số request.

**Tác hại đã đo:** compaction giảm 200 → 11 file (18×). Sau Z-order, truy vấn điểm chỉ mở 1/10 file. Ở NB2, thời gian giảm từ 220.5 ms xuống 19.3 ms.

**Cách phòng tránh:**
1. Tăng khoảng trigger của writer để file lớn hơn ngay từ đầu.
2. Lập lịch compaction và Z-order; vacuum với retention ≥ 7 ngày, luôn kèm quét orphan.
3. Tạo checkpoint định kỳ cho log.
4. Theo dõi số file và kích thước trung bình như một metric.

Bảo trì chỉ là cách chữa; nguyên nhân gốc nằm ở writer.

**Sử dụng AI:** tôi dùng Claude Code để chạy và kiểm tra notebook, đổi tên ảnh và soạn nháp phần giải thích; tôi chịu trách nhiệm về số liệu.
