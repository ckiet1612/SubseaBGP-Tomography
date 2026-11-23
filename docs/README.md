# SubseaBGP-Tomography — Tài liệu dự án

## Đặc tả dự án

- [REQUIREMENTS.md](REQUIREMENTS.md): phạm vi MVP, yêu cầu, tiêu chí nghiệm thu và quyết định `Q-01` đến `Q-07`.
- [ARCHITECTURE.md](ARCHITECTURE.md): ranh giới thành phần, hợp đồng dữ liệu, luồng thực thi, cổng phát hành và quyết định kiến trúc MVP.
- [DATA.md](DATA.md): nguồn, tiền xử lý, lược đồ, phân hoạch, chống rò rỉ và phát hành dữ liệu.
- [MODEL.md](MODEL.md): suy luận LP-GISTA, hiệu chuẩn, mô phỏng định tuyến và đánh giá mô hình.
- [EVALUATION.md](EVALUATION.md): giao thức kiểm chứng, ma trận nghiệm thu và bằng chứng tái lập; không phải báo cáo kết quả đã chạy.
- [DESIGN.md](DESIGN.md): quy tắc thiết kế và triển khai giao diện.
- [RESEARCH.md](RESEARCH.md): đặc tả nghiên cứu và kiến trúc tham chiếu. Với các điểm đã chuẩn hóa trong `Q-01` đến `Q-07`, áp dụng `REQUIREMENTS.md`.

## Đối chiếu tài liệu nghiên cứu

- `images/`: 4 ảnh minh họa được sử dụng trong Markdown. Không còn ảnh công thức toán học riêng lẻ.
- `original-research.docx`: tài liệu Word gốc để đối chiếu.

**Công thức toán học:** 128 ảnh công thức (ký hiệu và phương trình) được đối chiếu với ảnh nhúng trong DOCX gốc và chuyển sang LaTeX: `$...$` cho công thức nội dòng, `$$...$$` cho phương trình riêng. Có 3 ảnh công thức vốn đã bị cắt trong DOCX, được chép lại phần nhìn thấy và đánh dấu phần thiếu bằng `\ldots` cùng chú thích để đối chiếu. Không tự suy đoán phần công thức bị mất. Các ảnh công thức đã được loại khỏi thư mục `images/`; chúng vẫn có trong tài liệu Word gốc để đối chiếu.

Mở `RESEARCH.md` bằng trình xem Markdown hỗ trợ LaTeX (ví dụ GitHub). Giữ thư mục `images/` cùng cấp với `RESEARCH.md` để các ảnh hiển thị đúng.
