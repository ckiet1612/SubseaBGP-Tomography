# AGENTS.md: Quy tắc làm việc

Tài liệu này áp dụng cho mọi Agent làm việc trong repo SubseaBGP-Tomography.

## 1. Tổng quan dự án

SubseaBGP-Tomography nghiên cứu cách suy luận phân đoạn cáp quang biển có thể gặp sự cố từ dữ liệu định tuyến BGP, độ trễ tối thiểu (minRTT) và thông tin rủi ro hàng hải, địa vật lý. Dự án cũng đề xuất mô phỏng các phương án định tuyến dự phòng SR-TE để đánh giá khả năng duy trì lưu lượng sau sự cố. Đây là phạm vi nghiên cứu và mô phỏng, không phải hệ thống điều khiển mạng sản xuất.

## 2. Nguồn tài liệu tham chiếu

- [Đặc tả yêu cầu hệ thống](docs/REQUIREMENTS.md) là căn cứ cho phạm vi MVP, yêu cầu và tiêu chí nghiệm thu. Với các điểm đã được chuẩn hóa trong quyết định `Q-01` đến `Q-07`, thực hiện theo `REQUIREMENTS.md`; ghi rõ các khác biệt chưa được giải quyết để xác nhận.
- [Kiến trúc hệ thống](docs/ARCHITECTURE.md) là căn cứ cho ranh giới thành phần, hợp đồng dữ liệu, quy trình phát hành và các quyết định kiến trúc MVP. Giữ nguyên phạm vi và tiêu chí nghiệm thu trong `REQUIREMENTS.md` khi chi tiết hóa kiến trúc.
- [Đặc tả dữ liệu](docs/DATA.md) quy định nguồn, lược đồ, phân hoạch, chống rò rỉ và phát hành. [Đặc tả mô hình](docs/MODEL.md) quy định cách dùng dữ liệu cho suy luận, hiệu chuẩn, mô phỏng định tuyến và đánh giá.
- [Giao thức kiểm chứng và nghiệm thu](docs/EVALUATION.md) quy định cách thực thi các cổng nghiệm thu, ma trận kiểm chứng và bằng chứng cần lưu. Khi lập kế hoạch hoặc bàn giao, dùng tài liệu này để truy vết kiểm thử; không coi giao thức hoặc giá trị kỳ vọng là kết quả đã chạy.
- [Đặc tả nghiên cứu](docs/RESEARCH.md) là tài liệu tham chiếu gốc cho mục tiêu, thuật toán, dữ liệu, đánh giá và lộ trình. Hạng mục được mô tả trong đặc tả không mặc nhiên đã được triển khai hoặc xác minh trong repo.
- Đọc và tuân thủ [quy tắc thiết kế giao diện](docs/DESIGN.md) khi làm việc với giao diện.
- [Hướng dẫn tài liệu nghiên cứu](docs/README.md) mô tả cách đọc và bảo toàn bản Markdown. [Bản Word gốc](docs/original-research.docx) dùng để đối chiếu công thức, bảng và ảnh.
- Khi bổ sung hoặc sửa tuyên bố khoa học, đối chiếu nguồn sơ cấp. Nếu đặc tả, dữ liệu hoặc mã nguồn không khớp, ghi rõ chỗ lệch; không âm thầm chọn một cách hiểu nếu việc đó làm đổi phạm vi hoặc kết luận.

## 3. Nguyên tắc làm việc

- Giới hạn thay đổi trong yêu cầu; ưu tiên cách làm và công cụ đã dùng trong repo. Không gộp chỉnh định dạng, tái cấu trúc hay thêm phụ thuộc không cần thiết.
- Bảo toàn thay đổi của người dùng và Agent khác. Không ghi đè, hoàn tác hoặc đưa phần việc ngoài phạm vi vào thay đổi của mình.
- Nêu rõ những giả định ảnh hưởng đến kết quả. Hỏi trước khi quyết định nếu thông tin còn thiếu và lựa chọn có thể làm đổi phạm vi, gây mất dữ liệu hoặc khó hoàn tác.
- Khi một tác vụ khớp với skill trong [`.agents/skills/`](.agents/skills/), đọc `SKILL.md` tương ứng trước khi làm. Dùng các skill `subsea-*` theo phạm vi dữ liệu, mô hình, đánh giá, nghiên cứu hoặc định tuyến; ghi rõ nếu hướng dẫn khác với quyết định đã chốt.

## 4. Cấu trúc dự án

| Đường dẫn | Trách nhiệm |
| --- | --- |
| `AGENTS.md` | Quy tắc làm việc chung cho Agent. |
| `docs/README.md` | Mục lục tài liệu và hướng dẫn đối chiếu tài liệu nghiên cứu. |
| `docs/REQUIREMENTS.md` | Phạm vi, yêu cầu, tiêu chí nghiệm thu và các quyết định đã chốt. |
| `docs/ARCHITECTURE.md` | Kiến trúc MVP, hợp đồng dữ liệu, cổng phát hành và truy vết kiểm chứng. |
| `docs/DATA.md` | Nguồn, lược đồ, phân hoạch, chống rò rỉ và phát hành dữ liệu. |
| `docs/MODEL.md` | Suy luận, hiệu chuẩn, mô phỏng định tuyến và đối sánh mô hình. |
| `docs/EVALUATION.md` | Giao thức kiểm chứng, ma trận nghiệm thu, trạng thái và hồ sơ tái lập. |
| `docs/RESEARCH.md` | Đặc tả nghiên cứu và kỹ thuật. |
| `docs/DESIGN.md` | Quy tắc thiết kế và triển khai giao diện. |
| `docs/images/` | Ảnh minh họa được tham chiếu từ đặc tả Markdown. |
| `docs/original-research.docx` | Bản Word nguồn để đối chiếu. |
| `.agents/skills/` | Skill cục bộ theo loại công việc, bao gồm các skill chuyên biệt của dự án. |

Repo hiện chưa có mã triển khai, bộ dữ liệu hay bộ kiểm thử của hệ thống. Khi bổ sung thành phần mới, tách rõ trách nhiệm của mã xử lý dữ liệu, mô hình, mô phỏng, giao diện và kiểm thử; cập nhật sơ đồ thư mục khi cấu trúc thực sự thay đổi.

## 5. Tiêu chuẩn lập trình

- Theo quy ước của ngôn ngữ và cấu hình thực tế trong repo. Nếu chưa có, dùng `snake_case` cho hàm và biến Python, `camelCase` cho hàm và biến TypeScript, `PascalCase` cho lớp và kiểu dữ liệu. Dùng tên có nghĩa; nêu đơn vị khi giá trị dễ bị hiểu sai, ví dụ `latency_ms`.
- Tách đọc/ghi dữ liệu và giao diện khỏi thuật toán để có thể kiểm thử riêng. Khai báo rõ kiểu dữ liệu, đơn vị và điều kiện đầu vào tại ranh giới giữa các thành phần. Không bỏ qua lỗi hoặc tự thay bằng giá trị mặc định làm sai kết quả.
- Dùng công cụ định dạng, lint và kiểm tra kiểu đã được cấu hình. Khai báo phụ thuộc mới trong tệp quản lý phụ thuộc và cập nhật lockfile tương ứng nếu có.

## 6. Tiêu chuẩn nghiên cứu khoa học

- Phân biệt kết quả đo thực tế, dữ liệu bán tổng hợp, kết quả mô phỏng và giả định. Không bịa số liệu, trích dẫn hoặc tuyên bố đã tái lập kết quả khi chưa chạy thực nghiệm; ghi rõ giới hạn và độ bất định.
- Ghi nguồn, giấy phép (nếu có), phiên bản hoặc ngày lấy dữ liệu, quy trình tiền xử lý, cách chia tập, cấu hình chạy, phiên bản mã và seed khi có ngẫu nhiên. Lưu lệnh chạy và đầu ra cần thiết để tái lập.
- Tuân thủ giao thức chống rò rỉ dữ liệu trong đặc tả. Không dùng tập kiểm thử để chọn mô hình, ngưỡng hay tham số. Khi so sánh với mốc đối chứng hoặc loại bỏ thành phần, dùng cùng giao thức đánh giá và nêu rõ khác biệt nếu không thể giữ nguyên điều kiện.

## 7. Quy trình phát triển

1. **Phân tích:** Đọc yêu cầu, mã hoặc tài liệu liên quan; xác định đầu ra và tiêu chí nghiệm thu từ hiện trạng repo.
2. **Lập kế hoạch:** Với việc nhiều bước hoặc tác động nhiều thành phần, chia thành các phần có thể kiểm tra độc lập; xác định thứ tự và rủi ro trước khi sửa.
3. **Triển khai và kiểm thử:** Sửa trong phạm vi đã xác định, thêm kiểm thử phù hợp với hành vi thay đổi và cập nhật tài liệu liên quan.
4. **Bàn giao:** Xem lại diff, chạy các kiểm tra liên quan và báo kết quả theo mục 13 và 14.

## 8. Phối hợp đa Agent

- Chỉ làm song song khi có thể tách đầu ra và quyền sửa tệp. Trước khi giao việc, chốt phạm vi, tệp được sửa, đầu ra, tiêu chí nghiệm thu và nhánh hoặc worktree nếu cần cách ly.
- Agent nhận việc báo tệp đã sửa, giao diện hoặc giả định đã đổi, lệnh kiểm tra và vướng mắc. Không tự mở rộng sang tệp Agent khác đang phụ trách; báo trùng phạm vi trước khi tiếp tục.
- Agent điều phối đọc diff, giải quyết xung đột dựa trên ý định của từng thay đổi và chạy lại kiểm tra sau tích hợp. Không coi báo cáo của Agent khác là bằng chứng kiểm tra tích hợp.

## 9. Kiểm thử và xác minh

- **Phần mềm:** Thêm kiểm thử hồi quy khi sửa lỗi và kiểm tra đường đi chính, trường hợp biên, đầu vào không hợp lệ khi đổi hành vi. Chạy kiểm thử tích hợp nếu thay đổi hợp đồng giữa các thành phần.
- **Dữ liệu:** Kiểm tra lược đồ, đơn vị, mốc thời gian, giá trị thiếu, bản ghi trùng, tính tách biệt giữa các tập và các chốt chống rò rỉ liên quan trước khi huấn luyện.
- **Mô hình và mô phỏng:** Đánh giá bằng chỉ số trong đặc tả phù hợp với thay đổi, trên tập độc lập. Kiểm tra độ bao phủ hiệu chuẩn khi đổi mô hình bất định; kiểm tra bảo toàn luồng và giới hạn năng lực liên kết khi đổi bộ giải định tuyến.
- **Tài liệu:** Kiểm tra tiêu đề, bảng, đường dẫn tương đối, ảnh tham chiếu và cú pháp Markdown/LaTeX bị ảnh hưởng. Chỉ báo kiểm tra đạt khi đã chạy hoặc đối chiếu; nêu rõ mục không áp dụng hoặc chưa kiểm tra được.

## 10. Quản lý phiên bản

1. **Trước khi sửa:** Chạy `git status --short --branch` và xem diff đã stage hoặc chưa stage để nhận diện thay đổi có sẵn.
2. **Nhánh:** Sau khi repo có commit đầu tiên, dùng nhánh riêng từ `main` cho mỗi phần việc, ví dụ `docs/working-rules`, `feat/route-model` hoặc `fix/data-parser`. Không tự đổi nhánh nếu có thể ảnh hưởng tới việc đang làm dở. Commit khởi tạo có thể thực hiện trên `main` khi được yêu cầu.
3. **Stage:** Chỉ stage tệp thuộc phần việc bằng `git add -- <đường-dẫn>`. Xem `git diff --cached` và chạy `git diff --cached --check` trước khi commit. Không stage bí mật, `.env` chứa thông tin nhạy cảm, `.DS_Store`, bộ nhớ đệm, đầu ra sinh tự động hoặc dữ liệu lớn chưa được duyệt.
4. **Thời điểm commit:** Nên commit khi một thay đổi độc lập đã hoàn tất, diff đã được xem lại và các kiểm tra liên quan đã đạt. Ví dụ: sửa xong một lỗi kèm test hồi quy hoặc hoàn tất một cập nhật tài liệu. Commit trước khi chuyển sang phần việc khác, không chờ gom tất cả vào cuối nhánh.
5. **Thông điệp commit:** Mỗi commit chỉ có một mục đích. Dùng dạng `type: mô tả ngắn`, với `feat`, `fix`, `docs`, `refactor`, `test`, `chore` khi phù hợp. Ví dụ: `docs: define repository working rules`.
6. **Push và PR:** Sau commit khởi tạo, tích hợp vào `main` qua PR. Ghi mục đích, phạm vi và kết quả kiểm tra; xử lý các kiểm tra bắt buộc và phản hồi review trước khi merge. Agent chỉ commit, push, mở PR hoặc merge khi người dùng yêu cầu hành động tương ứng.
7. **Lịch sử và xung đột:** Xem chênh lệch với upstream trước khi tích hợp. Không rebase hoặc viết lại lịch sử nhánh dùng chung. Không dùng `git reset --hard`, `git clean -fd` hay force-push nếu chưa được đồng ý rõ ràng. Nếu chưa rõ ý định của một bên khi có xung đột, hỏi trước khi chọn cách giải quyết.

## 11. Bảo mật và quyền riêng tư

- Dùng quyền truy cập tối thiểu. Không ghi khóa API, token, mật khẩu hay dữ liệu cá nhân vào Git, log hoặc báo cáo. Dùng cơ chế quản lý bí mật của môi trường; tệp cấu hình mẫu chỉ chứa giá trị giả.
- Kiểm tra giấy phép và điều khoản chia sẻ trước khi tải, lưu hoặc công bố dữ liệu bên ngoài. Dùng dữ liệu đã ẩn danh hoặc dữ liệu giả lập cho ví dụ và kiểm thử khi có thông tin nhạy cảm.
- Ưu tiên bộ dữ liệu công khai và mô phỏng cục bộ. Không tự chạy đo chủ động hoặc thay đổi cấu hình trên hạ tầng thật ngoài phạm vi được phép. Nếu phát hiện bí mật bị lộ, thông báo mà không lặp lại giá trị.

## 12. Quản lý tài liệu

- Cập nhật tài liệu trong cùng thay đổi khi lệnh chạy, cấu hình, kiến trúc, lược đồ dữ liệu hoặc giao thức đánh giá thay đổi. Dùng đường dẫn tương đối và kiểm tra tệp đích.
- Khi sửa bản Markdown nghiên cứu, giữ nguyên ý nghĩa của công thức LaTeX, bảng và trích dẫn. Đối chiếu bản Word khi cần; đánh dấu đoạn nguồn thiếu hoặc không đọc được thay vì tự điền. Không sửa bản Word gốc trừ khi được yêu cầu.

## 13. Tiêu chí hoàn thành

- Đầu ra đáp ứng tiêu chí nghiệm thu đã thống nhất, diff đã được rà soát và không chứa thay đổi ngoài phạm vi.
- Các kiểm tra bắt buộc liên quan đã đạt, tài liệu và hướng dẫn tái lập cần thiết đã cập nhật. Nếu một kiểm tra bắt buộc thất bại hoặc chưa chạy được, báo phần việc chưa được xác minh thay vì gọi là hoàn thành.
- Báo cáo bàn giao nêu tệp bị ảnh hưởng, lệnh hoặc cách kiểm tra và kết quả thực tế; ghi rõ giới hạn còn lại. Nêu trạng thái commit hoặc PR nếu nhiệm vụ có thao tác Git.

## 14. Quy tắc giao tiếp

- Mặc định trao đổi với người dùng bằng tiếng Việt, trừ khi được yêu cầu ngôn ngữ khác. Giữ nguyên tên API, đường dẫn, lệnh và thông báo lỗi cần đối chiếu.
- Với việc kéo dài, cập nhật ngắn gọn những gì đã xác nhận, việc đang làm và trở ngại ảnh hưởng tới tiến độ. Không báo một kiểm tra đã chạy hoặc một kết quả đã được xác nhận nếu chưa có bằng chứng.
- Khi cần quyết định từ người dùng, nêu câu hỏi cụ thể, điều đã thử và tác động của quyết định. Khi bàn giao, tóm tắt thay đổi, kết quả kiểm tra và điểm còn mở.
