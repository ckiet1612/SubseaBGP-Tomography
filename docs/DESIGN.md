# Quy tắc thiết kế giao diện

Áp dụng cho mọi màn hình và thay đổi giao diện của SubseaBGP-Tomography. Ưu tiên để người dùng đọc dữ liệu, so sánh kịch bản và thao tác mà không bị phân tán.

## Màu sắc

- Không dùng gradient cho nền, nút, chữ, biểu đồ hoặc hiệu ứng trang trí. Dùng màu phẳng và giữ độ tương phản đủ để đọc.
- Khi màu biểu thị mức độ hoặc trạng thái, dùng các nhóm màu rời rạc nhất quán. Kèm nhãn, biểu tượng hoặc chú giải; không truyền đạt thông tin chỉ bằng màu.

## Bố cục và card

- Dùng bố cục mở với khoảng trắng, đường phân cách, bảng hoặc danh sách làm mặc định. Không bọc cả màn hình, từng mục hay từng chỉ số trong card chỉ để chia bố cục.
- Chỉ dùng card khi cần phân biệt các mục độc lập, thường là các mục lặp lại có nội dung hoặc thao tác riêng. Không lồng card trong card.

## Nội dung giao diện

- Dùng nhãn ngắn, cụ thể và nhất quán. Không thêm dòng phụ (subtext) nếu chỉ lặp lại tiêu đề, nhãn hoặc giá trị đang hiển thị.
- Chỉ thêm lời giải thích khi cần để tránh hiểu sai dữ liệu, xử lý lỗi hoặc đưa ra quyết định. Giữ đơn vị, chú giải và thông tin về độ bất định khi chúng ảnh hưởng đến cách đọc kết quả.
- Không dùng slogan, lời trích dẫn trang trí hoặc câu dẫn không phục vụ thao tác hay diễn giải dữ liệu.

## Component và chuyển động

- Dùng [coss ui](https://coss.com/ui) cho các component giao diện. Trước khi viết mới, kiểm tra component đã có trong dự án, sau đó tra thư viện và kết hợp các component sẵn có. Không tự viết lại component có sẵn.
- Khi chuyển động giúp người dùng nhận ra thay đổi trạng thái, dùng [Animate UI](https://animate-ui.com/) cho hiệu ứng phù hợp. Không tự viết hiệu ứng có sẵn trong thư viện. Nếu không có hiệu ứng phù hợp, ưu tiên bỏ chuyển động.
- Không thêm animation chỉ để trang trí hoặc cản trở việc đọc dữ liệu. Tôn trọng `prefers-reduced-motion`; mọi thao tác và thông tin phải còn dùng được khi giảm hoặc tắt chuyển động.

## Rà soát trước khi bàn giao

- Không có gradient hay card thừa; bỏ mọi subtext hoặc lời trích dẫn không giúp người dùng hiểu hoặc thao tác.
- Component và hiệu ứng dùng nguồn quy định ở trên, không trùng với thứ đã có. Kiểm tra khả năng đọc, thao tác bằng bàn phím và chế độ giảm chuyển động khi thay đổi có liên quan.
