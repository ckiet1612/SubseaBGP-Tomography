# Giao thức Kiểm chứng và Nghiệm thu SubseaBGP-Tomography

Tài liệu này xác lập **giao thức kiểm chứng, tiêu chí nghiệm thu, ma trận truy vết và mẫu hồ sơ tái lập** cho toàn bộ hệ thống nghiên cứu – mô phỏng **SubseaBGP-Tomography** để triển khai và thực thi ở các bước sau. Đây là **đặc tả giao thức kiểm chứng**, **không phải báo cáo kết quả thực nghiệm**.

> **Hiện trạng kho mã tại thời điểm lập giao thức:**
> - Trong kho mã hiện mới có các tài liệu đặc tả dưới `docs/`, quy tắc làm việc [AGENTS.md](../AGENTS.md) và các skill chuyên môn dưới `.agents/skills/`; **chưa có mã nguồn, bộ dữ liệu thô hay bán tổng hợp, mô hình ONNX, tệp `ReleaseManifest`, bản dựng tĩnh hay bất kỳ bài kiểm thử tự động nào được chạy**.
> - **Không có mục nào trong tài liệu này mang trạng thái `ĐẠT`.** Toàn bộ các giá trị kỳ vọng của fixture LP, ví dụ số học tính tỷ lệ và mẫu `run_provenance.json` chỉ là **đặc tả kiểm thử / ví dụ minh họa**, tuyệt đối không phải kết quả đã chạy.
> - Mọi đường dẫn đầu ra hoặc bước tái lập cho phần mã chưa triển khai được đánh dấu rõ là **dự kiến khi triển khai**, không coi là tệp hay lệnh đã tồn tại trong repo.

---

## 1. Nguồn căn cứ, phạm vi và giới hạn diễn giải

### 1.1. Thứ bậc tài liệu tham chiếu và skill áp dụng

| Cấp độ ưu tiên | Tài liệu / Skill | Phạm vi quyết định |
| --- | --- | --- |
| **1. Quy tắc kho mã** | [AGENTS.md](../AGENTS.md) | Quy tắc làm việc, tiêu chuẩn nghiên cứu khoa học, kiểm thử, bảo mật và quản lý phiên bản. |
| **2. Phạm vi & Cổng nghiệm thu** | [REQUIREMENTS.md](REQUIREMENTS.md) | Phạm vi MVP, 54 yêu cầu (`FR`, `DATA`, `ML`, `NFR`, `UX`), tiêu chí nghiệm thu và bộ quyết định đã chốt **`Q-01`–`Q-07`** cùng cổng giai đoạn sau **`O-01`–`O-03`**. |
| **3. Kiến trúc, Đo lường & Truy vết** | [ARCHITECTURE.md](ARCHITECTURE.md) *(đặc biệt §1.6, §4, §8.2, §9.1, §10)* | Hợp đồng dữ liệu, máy chuẩn đo hiệu năng M1, `ADR-01`–`ADR-06`, 6 nhóm kiểm chứng **`V-*`** và ma trận truy vết thành phần. |
| **4. Giao thức dữ liệu & mô hình đã duyệt** | [DATA.md](DATA.md) *(§4–§6)* và [MODEL.md](MODEL.md) *(§3–§7)* | Cấu hình khởi đầu `C3`, 5 chốt chống rò rỉ, quy tắc đóng băng $b_m$, hàm mất mát, quy tắc chọn $\tau_{\mathrm{cut}}$, hiệu chuẩn Split Conformal, chấm điểm, fixture LP, tiêu chí `V-FALLBACK` và hồ sơ `run_provenance.json`. |
| **5. Ràng buộc giao diện** | [DESIGN.md](DESIGN.md) | Tiêu chuẩn màu phẳng (0 gradient), bố cục mở (0 lồng card), chữ tối giản, bắt buộc dùng `coss ui` và `Animate UI`, bàn phím và `prefers-reduced-motion`. |
| **6. Tài liệu nghiên cứu đối chiếu** | [RESEARCH.md](RESEARCH.md) và [README.md](README.md) | Nguồn đối chiếu công thức, ngữ cảnh khoa học và mốc tham chiếu nguyên mẫu cũ. **Không lấy số liệu cũ từ chú thích `[1]` (`unknown_url`) làm kết quả tái lập** (`Q-06`). |
| **7. Skill kiểm chứng cục bộ** | [subsea-evaluation](../.agents/skills/subsea-evaluation/SKILL.md), [subsea-anti-leakage](../.agents/skills/subsea-anti-leakage/SKILL.md), [subsea-research-integrity](../.agents/skills/subsea-research-integrity/SKILL.md), [subsea-routing-verification](../.agents/skills/subsea-routing-verification/SKILL.md) | Kỷ luật chấm điểm đồng nhất, chặn công bố khi thiếu bằng chứng chống rò rỉ, phân định bản chất dữ liệu và hậu kiểm độc lập bộ giải LP. |

### 1.2. Phân tách bốn cổng nghiệm thu độc lập và phân kỳ MVP vs GĐ2/GĐ3

Hệ thống tách cứng **bốn cổng nghiệm thu độc lập** — vượt qua một cổng không thay thế cho cổng khác và không dùng ngưỡng của cổng này để đánh giá cổng kia:

1. **Cổng 1 — Đường cơ sở bán tổng hợp (`ML-010`, `Q-04`):** Nghiệm thu định lượng trên tập `200 test` khóa trước của bộ kiểm chuẩn bán tổng hợp xây mới (`24` phân đoạn, `48` kênh) theo đúng `Q-04`, đối sánh `B1`/`B2`/`A1`/`A2`/`Full`.
2. **Cổng 2 — Phát lại và hồi quy sự cố lịch sử (`ML-011`, `Q-04` — *HIỆN BỊ CHẶN*):** Kiểm chứng trên $\ge 3$ sự cố lịch sử độc lập trong khung `test`, `100%` truy vết nguồn sơ cấp, `0` rò rỉ, áp chế độ chuyển giao `ADR-05` (`prior_mode = uniform_w1`). **Tuyệt đối không áp các ngưỡng định lượng của bộ bán tổng hợp (`Hit@1 ≥ 90%`, `PR-AUC > 0,85`, v.v.) lên tập lịch sử cỡ nhỏ hoặc thiếu nhãn phân đoạn.**
3. **Cổng 3 — Bộ giải quy hoạch tuyến tính dự phòng (`ML-012`, `V-LP`):** Kiểm chứng bảo toàn luồng, giới hạn dung lượng, trần SRLG thích nghi, đối sánh đường ngắn nhất theo `FR-012` và 6 bộ fixture mô phỏng bắt buộc.
4. **Cổng 4 — Hiệu năng, tài nguyên và giao diện trên máy chuẩn M1 (`NFR-003`, `V-PERF-UX`):** Kiểm chứng 7 ngưỡng hiệu năng/bộ nhớ/tải mạng trên máy chuẩn `MacBook Pro M1` bằng workload phát triển (không dùng `200 test` khóa), cùng độ đúng đường dự phòng `WASM` (`V-FALLBACK`) và tiêu chuẩn `DESIGN.md`.

**Tách bạch đường chạy MVP khỏi Giai đoạn 2 / Giai đoạn 3 và cổng `O-01`–`O-03` (`Q-01`, `REQUIREMENTS.md §3.2, §9.5 B`):**

| Phân kỳ | Thành phần thuộc phạm vi | Cổng mở rộng / Điều kiện kích hoạt |
| --- | --- | --- |
| **Giai đoạn 1 — Đường chạy MVP** *(trọng tâm nghiệm thu hiện tại)* | Đường ống ngoại tuyến `C1`–`C5` (5 chốt chống rò rỉ, kiểm chuẩn bán tổng hợp `C3`, LP-GISTA 12 lớp kèm `B1`/`B2`/`A1`/`A2`, quét $\tau_{\mathrm{cut}}$ trên `val_tau`, Split Conformal trên `200 cal`, tắt Mohr–Coulomb, chạy `uniform_w1` cho lịch sử); ứng dụng trình duyệt tĩnh `C6`–`C9` cho **Luồng 2** (phát lại 5 phút tách 3 lớp) và **Luồng 3** (cắt cáp giả định & LP 3 pha + đối sánh đường ngắn nhất); mặc định `public_demo` chỉ gồm fixture độc lập + trình tải nguồn gốc + mô hình ONNX FP32. | Không còn câu hỏi mở chặn thiết kế MVP; riêng **nghiệm thu `ML-011`** (và phần phát lại $\ge 3$ sự cố lịch sử thật của `FR-010`) đang **`BỊ CHẶN`** chờ xác minh dữ liệu thực tế cho $\ge 3$ ca khung `test`. |
| **Giai đoạn 2** *(ngoài đường chạy MVP)* | Luồng 1 BGP trực tiếp (`FR-004`, `FR-005`), Luồng 4 quy hoạch hạ tầng 2030 (`FR-013`), tích hợp đầy đủ bộ đệm & logic loại trừ IODA v2 / Cloudflare Radar (`FR-007`), đường ống định kỳ GFW 4Wings AIS/SAR (`FR-008`), huấn luyện bộ mã hóa tiên nghiệm với LOBO (`ML-002`), kích hoạt số hạng phạt vật lý Mohr–Coulomb (`ML-008`). | **`O-01`** (`DATA-009`): xác minh điều khoản & ký duyệt `manifest` từng tệp cho nguồn hạn chế (GFW 4Wings, RIPE RIS/Atlas, CAIDA, PeeringDB, IODA, Radar, gói lịch sử).<br>**`O-02`** (`ML-008`): xác minh dữ liệu thí nghiệm cắt trực tiếp thoát nước theo phân đoạn (`kPa`, `radian`, $\ge 3$ mức ứng suất hữu hiệu) và mô hình $\sigma'_{n,e}$.<br>**`O-03`** (`U3`, `U4`): xác nhận nhu cầu từ người dùng vận hành NOC / quy hoạch thực tế. |
| **Giai đoạn 3** *(ngoài đường chạy MVP)* | Giám sát trôi dạt hàng tuần ($\mathrm{JSD} \ge 0{,}10$, `bin` cố định, $\ge 100$ cửa sổ hợp lệ và $\ge 80\%$ dữ liệu dự kiến/tuần; 2 tuần liên tiếp hoặc đổi tô-pô xác nhận $\Rightarrow$ sinh ứng viên $(\mathbf{R}_t, \hat{q}_{0{,}90})$ duyệt thủ công qua PR — `ML-014`, `Q-07`). | Phụ thuộc đường cơ sở đã duyệt ở MVP và luồng thu thập định kỳ ở GĐ2. |
| **Ngoài phạm vi mọi giai đoạn** | Điều khiển mạng sản xuất, đẩy cấu hình BGP/SRv6/MPLS/IGP (`NFR-008`), phát đo chủ động trái phép, truy cập OTDR/PFE nội bộ, công bố GeoParquet gộp đa giấy phép (`DATA-007`), tự động xuất bản mô hình không qua PR. | Bị loại trừ tuyệt đối trong mọi kiểm chứng. |

### 1.3. Phân loại bản chất dữ liệu và ranh giới diễn giải bắt buộc

Mọi tệp dữ liệu, bảng kiểm chứng và báo cáo đánh giá bắt buộc gắn đúng một trong bốn nhãn `data_provenance_class` (`DATA-002`, [ARCHITECTURE.md](ARCHITECTURE.md) §4.1):

- **`observed_measurement` (dữ liệu đo thực tế):** Bản tin BGP lưu trữ (RouteViews, RIPE RIS MRT), phép đo tích hợp/neo RIPE Atlas, bản chụp CAIDA/PeeringDB, GEBCO_2026, USGS, nhãn sự cố lịch sử xác minh, metadata IODA/Radar nội bộ.
- **`stylized_derived` (dẫn xuất sơ đồ hóa):** Hình học tuyến cáp và CLS từ Submarine Cable Map v3 (kèm bổ sung `VSTN`, `TEA`), ma trận $\mathbf{L}_{\mathrm{SRLG}}$ và ma trận trọng số tiên nghiệm $\mathbf{R}_t$.
- **`synthetic_calibrated_baseline` (bán tổng hợp — đường cơ sở mới):** Bộ kiểm chuẩn 24 phân đoạn / 48 kênh do `C3` tự sinh làm đường cơ sở thực nghiệm mới (`Q-06`), **không gọi là tái lập số liệu `[1]` (`unknown_url`)**.
- **`simulated_assumption` (giả định mô phỏng):** Ma trận nhu cầu giả định (`82,0 Tbps`, `10 Tbps`, `12 & 8 Tbps`), thao tác cắt cáp giả định trong Luồng 3 và toàn bộ đầu ra lưu lượng/MLU/độ trễ của bộ giải LP và đối sánh đường ngắn nhất.

**Hai giới hạn diễn giải bất biến (`AGENTS.md §1`, `REQUIREMENTS.md §1.2`, `MODEL.md §1.1`):**

1. **Suy luận cắt lớp KHÔNG tự xác nhận đứt cáp vật lý:** Mọi giá trị $\hat{x}_e(t), \hat{p}_e(t)$ và tập $\mathcal{C}_{0{,}90}(t)$ chỉ là giả thuyết xác suất về vùng nghi suy giảm do bốn giới hạn gốc: (a) chuyển mạch bảo vệ quang tầng 1 DWDM/OTN đồng hành lang ($< 50\ \mathrm{km}$, $\Delta\mathrm{minRTT} < 0{,}5\ \mathrm{ms}$ nhỏ hơn sai số $9{,}5\%$ của `minRTT` RIPE Atlas) gây âm tính giả từ xa; (b) sự cố cáp trục đất liền nội địa hoặc trạm trung chuyển ra CLS gây dương tính giả; (c) hình học TeleGeography là đường sơ đồ hóa và $(\Omega, \Phi_{\mathrm{PoP}})$ chỉ là trọng số tiên nghiệm (`FR-002`, `Q-02`); (d) xác nhận đứt cáp vật lý đòi hỏi OTDR/PFE nội bộ, thông cáo nhà vận hành hoặc khảo sát của tàu sửa cáp.
2. **Bộ giải LP KHÔNG điều khiển mạng sản xuất:** Toàn bộ bài toán SR-TE/SRv6 là mô phỏng nghiên cứu ngoại tuyến/trình duyệt; tuyệt đối không tồn tại kênh đẩy cấu hình lên thiết bị thật (`NFR-008`).

### 1.4. Đối chiếu điểm lệch giữa tài liệu nguồn và các chi tiết cần xác nhận trước khi chạy

Theo `AGENTS.md §2`, không tự đặt ngưỡng nghiệm thu mới và không âm thầm chọn cách hiểu làm thay đổi kết luận. Bảng dưới ghi nhận các điểm lệch đã được chuẩn hóa cũng như các chi tiết chấm điểm kỹ thuật cần khóa trước khi mở `D_test`:

| STT | Điều khoản liên quan | Khác biệt / Khoảng trống phát hiện | Tác động đến kiểm chứng | Xử lý trong giao thức & Câu hỏi cần xác nhận |
| --- | --- | --- | --- | --- |
| 1 | [RESEARCH.md](RESEARCH.md) §8.2 (Tuần 4) vs [REQUIREMENTS.md](REQUIREMENTS.md) `ML-010`, `Q-04` và [MODEL.md](MODEL.md) §6.1 | `RESEARCH.md §8.2` nêu độ phủ trong dải `[88,5%; 93,0%]` và sai lệch ONNX `< 0,5%`; trong khi `Q-04` (`ML-010`) chốt **độ phủ `≥ 88,5%`** (không có chặn trên `93,0%`) và **giảm `PR-AUC < 0,005`** nếu lượng tử hóa; còn `MODEL.md §6.1` chốt tương đương FP32 `WebGPU`/`WASM` là `max_abs_diff ≤ 1e-4` kèm khớp quyết định. | Nếu áp chặn trên `93,0%` hoặc dùng sai tiêu chí lượng tử hóa/FP32 sẽ làm lệch tiêu chí đạt `ML-010` và `V-FALLBACK`. | **Áp dụng đúng `Q-04` và `MODEL.md §6.1`** (căn cứ cấp trên): độ phủ `≥ 88,5%` kèm kích thước tập TB `≤ 3`, `ΔPR-AUC_quant < 0,005` (nếu lượng tử hóa), FP32 `max_abs_diff ≤ 1e-4` + khớp quyết định. |
| 2 | [RESEARCH.md](RESEARCH.md) §6.1 vs [MODEL.md](MODEL.md) §3.1, §5.2, §7.2 | `RESEARCH.md §6.1` nêu chọn ngưỡng `F1` tối ưu trên tập hiệu chuẩn; `MODEL.md §5.2, §7.2` đã duyệt chấm `F1` phụ tại đúng ngưỡng Conformal $1 - \hat{q}_{0{,}90}$ đã khóa, **không tối ưu $\tau_{\mathrm{F1}}$**. | Tránh làm ô nhiễm vai trò đơn mục đích của `cal` (chỉ $\hat{q}_{0{,}90}$) và `val_tau` (chỉ $\tau_{\mathrm{cut}}$). | **Áp dụng `MODEL.md §5.2, §7.2`**: không quét $\tau_{\mathrm{F1}}$. |
| 3 | [RESEARCH.md](RESEARCH.md) §2, §3.6, §4.1, §5.1–§5.2, §6.2, §8.1 vs [REQUIREMENTS.md](REQUIREMENTS.md) §9.4 (`Q-01`–`Q-07`) & [ARCHITECTURE.md](ARCHITECTURE.md) §1.6, §9.1 (`ADR-01`–`ADR-06`) & [DATA.md](DATA.md) §6.2 | Đã chuẩn hóa 6 điểm lớn trong `REQUIREMENTS.md §9.4` (3 công thức cắt trong DOCX, LP 3 pha có $\ell_d$, cấm GeoParquet gộp đa giấy phép, đường cơ sở mới thay vì tái lập `[1]`, GPU logic `< 150 MB` & trung vị `≥ 55 FPS`, JSD duyệt PR), 4 điểm chi tiết trong `ARCHITECTURE.md §1.6` (phân hoạch `train_fit`/`val_tau`/`200 cal`/`200 test`, đường ngắn nhất hòa `µs` & chia tỷ lệ nghẽn, bản đồ 2D phẳng WebGL2 không Three.js/3D/hạt, `external_flags` nội bộ), `ADR-01`–`ADR-06` (không bắt buộc DVC/MLflow, tệp phân mảnh tĩnh trước, ONNX FP32 trước, `ΔR_θ=0` chuyển giao), và đính chính giấy phép Radar API `CC BY-NC 4.0` (chú thích `[88]` của `RESEARCH.md` trỏ nhầm bài arXiv `2306.03825v2`). | Tránh triển khai/kiểm chứng nhầm theo mô tả tham chiếu cũ của `RESEARCH.md`. | **Áp dụng đầy đủ các quyết định đã chốt**; không dùng số liệu `[1]` làm kết quả tái lập. |
| 4 | [ARCHITECTURE.md](ARCHITECTURE.md) §4.2 (dòng 240) vs [MODEL.md](MODEL.md) §2.3 (dòng 109) | Khi `valid_channel_count == 0`, trước đây `ARCHITECTURE.md §4.2` ghi `conformal_set_segment_ids = []`, trong khi `MODEL.md §2.3` ghi `[]` *(hoặc `null` ở chế độ chưa hiệu chuẩn)*. | Gây sai khác kiểm thử hợp đồng `V-CONTRACT` nếu không chuẩn hóa trường hợp `uncalibrated_transfer` kèm `valid_channel_count == 0`. | **Đã chốt (`D-01`):** Khi `valid_channel_count == 0`, trả `null` cho `uncalibrated_transfer` và `[]` cho `calibrated_90`. Cả hai giữ `status = insufficient_observations`, mọi phân đoạn là `unobservable_na`, và không được tính là âm tính đúng trong `Distractor FAR` (`null` nghĩa là không có tập Conformal được hiệu chuẩn; `[]` trong trường hợp này không có nghĩa là an toàn). |
| 5 | [MODEL.md](MODEL.md) §5.2–§5.3 (`Hit@1`, `Hit@3`, `MRR`, `ECE`, CI, ca lỗi không khả kiến) | `MODEL.md §5` trước đây chưa chốt tường minh quy tắc phá hòa xếp hạng, số bin `ECE`, phương pháp CI 95% và mẫu số cổng `Q-04` khi có phân đoạn lỗi `unobservable_na`. | Có thể làm thay đổi nhẹ giá trị `Hit@1`, `MRR`, `ECE`, biên khoảng tin cậy và mẫu số độ phủ nếu thay đổi sau khi nhìn `test`. | **Đã chốt (`D-02` — khóa trước `T4.5`, áp dụng đồng nhất mọi cấu hình):**<br>- **Xếp hạng:** điểm chưa làm tròn + phá hòa bi quan (trong nhóm đồng điểm, mọi phân đoạn không lỗi đứng trước các phân đoạn lỗi); đa cáp SRLG: `Hit@k` cần ít nhất 1 lỗi thật trong top-$k$, `MRR` dùng hạng của lỗi thật đầu tiên, độ phủ Conformal vẫn yêu cầu chứa toàn bộ lỗi thật.<br>- **ECE:** `10` bin đều trên $[0, 1]$, trái đóng phải mở, bin cuối chứa `1`; chỉ tính các cặp có nhãn và khả kiến.<br>- **CI:** Wilson hai phía 95% cho độ phủ, kèm tử số và mẫu số; cổng `≥ 88,5%` áp dụng cho **ước lượng điểm**, không áp dụng cho cận dưới CI.<br>- **Mẫu số:** dùng toàn bộ ca có lỗi (`n_fault_total`) cho `Hit@k` và độ phủ ở cổng `Q-04`, đồng thời báo riêng trên các ca có tất cả lỗi thật khả kiến; ca đa cáp còn 1 lỗi khả kiến trong top-$k$ vẫn có thể đạt `Hit@k`, nhưng chỉ 1 lỗi không khả kiến cũng đủ làm ca đó trượt độ phủ; `PR-AUC` giữ mẫu số theo cặp có nhãn và khả kiến, không gán `0` cho `NA`. |
| 6 | [MODEL.md](MODEL.md) §4.2, §6.1 (`V-LP` & `V-FALLBACK` dung sai đối chiếu nghiệm LP) | `MODEL.md §6.1` chốt `max_abs_diff ≤ 1e-4` cho $(\hat{\mathbf{x}}, \hat{\mathbf{p}})$ của ONNX FP32 và `MODEL.md §4.2` chốt $\varepsilon_{\mathrm{lex}} = 10^{-9}$ giữa 3 pha LP, nhưng cần quy tắc hậu kiểm khả thi LP (`V-LP`) và đối chiếu nghiệm đồng tối ưu giữa hai backend (`V-FALLBACK`). | Khi có nhiều nghiệm `f_k` đồng tối ưu trên cùng `(L*, u*, RTT*)`, so sánh trực tiếp từng `f_k` có thể báo sai dù tổng mục tiêu khớp. | **Đã duyệt nguyên tắc & bộ dung sai đề xuất (`D-03` — chưa chạy, xác minh trên fixture phát triển trước khi khóa `T5.3` và nhánh LP của `T8.2`):**<br>- Hậu kiểm tuyệt đối đề xuất: `1e-7 Tbps` cho phần dư bảo toàn nhu cầu, giới hạn dung lượng, trần SRLG và biên luồng; `1e-8` cho biên của $u$; đường qua liên kết cắt bị loại hoặc có luồng đúng `0`; kiểm riêng ràng buộc khóa pha và **không dùng dung sai hậu kiểm chung để nới $\varepsilon_{\mathrm{lex}} = 10^{-9}$** (xác minh cấu hình thực tế HiGHS vốn có dung sai khả thi mặc định `1e-7` ở cả hai backend).<br>- Với nghiệm đồng tối ưu, **không buộc từng $f_k$ giống nhau**: hai backend phải độc lập vượt hậu kiểm trên cùng đầu vào, khớp trạng thái/cờ rời rạc và `weighted_rtt_ms = null` + `NO_SERVED_FLOW` khi không có lưu lượng phục vụ, rồi đối chiếu 3 mục tiêu (`total_dropped_tbps`: `1e-6 Tbps`, $u^*$: `1e-7` không thứ nguyên, $\sum_k \tau_k^{\mathrm{RTT}} f_k$: `1e-4 Tbps·ms`). |

---

## 2. Bộ kiểm chuẩn bán tổng hợp (`C3`) và Giao thức chống rò rỉ (`V-LEAKAGE`)

### 2.1. Cấu hình khởi đầu đã duyệt của bộ kiểm chuẩn bán tổng hợp (`C3`)

Bộ kiểm chuẩn bán tổng hợp xây mới dùng để thiết lập **đường cơ sở thực nghiệm mới** (`synthetic_calibrated_baseline`, `Q-06`), **không gọi là tái lập số liệu cũ của `[1]`**. Toàn bộ thông số dưới đây tuân thủ đúng cấu hình khởi đầu đã được phê duyệt tại [DATA.md](DATA.md) §4.1, §6.2 và [MODEL.md](MODEL.md) §3.1:

- **Đồ thị kiểm chuẩn:** `24` phân đoạn truyền dẫn (`22` phân đoạn cáp biển quanh Việt Nam, eo biển Luzon, Singapore, Biển Đỏ, Nhật Bản và `2` hành lang cáp đất liền `VSTN` cùng tuyến hướng bắc) và `48` kênh đo liên miền.
- **Phân hoạch bốn tập độc lập theo `incident_group_id`:** Giữ nguyên vẹn `400` biến cố gồm `200 cal + 200 test`; **tuyệt đối không rút mẫu từ `cal` hay `test` sang `train`**. Sinh riêng phần `train` bằng cùng quy trình và tách cứng thành `train_fit` và `val_tau`:

| Tập phân hoạch | Tổng số biến cố | Đứt đơn (`55%`) | Đa cáp SRLG 2–3 cáp (`30%`) | Nhiễu BGP–IXP phi vật lý (`15%`) | Che khuất BGP trong nhóm đứt đơn |
| --- | ---: | ---: | ---: | ---: | --- |
| **`train_fit`** | `1.600` | `880` | `480` | `240` | Xác suất `25%` theo Bernoulli/ngẫu nhiên khóa `seed`; **bắt buộc báo số thực tế phát sinh** |
| **`val_tau`** | `400` | `220` | `120` | `60` | Xác suất `25%`; **bắt buộc báo số thực tế phát sinh** |
| **`cal`** | `200` | `110` | `60` | `30` | Xác suất `25%`; **bắt buộc báo số thực tế phát sinh** |
| **`test`** | `200` | `110` | `60` | `30` | Xác suất `25%`; **bắt buộc báo số thực tế phát sinh** |
| **Tổng cộng** | **`2.400`** | **`1.320`** | **`720`** | **`360`** | Báo riêng số thực tế từng tập trong `split_manifest` |

- **Ràng buộc khóa cứng trước khi sinh dữ liệu:**
  1. **Thứ tự và tỷ lệ cố định:** đứt đơn / đa cáp SRLG (2–3 cáp) / nhiễu phi vật lý theo đúng tỷ lệ `55% / 30% / 15%` trên cả bốn tập.
  2. **Không chồng lấp nhóm sự cố:** Mọi biến thể của cùng một sự cố mang cùng `incident_group_id` phải nằm trọn trong đúng một tập; giao cắt `incident_group_id` giữa mọi cặp tập phải bằng `0`.
  3. **Khóa cấu hình và `seed`:** Khóa toàn bộ tệp cấu hình sinh, `seed` riêng cho từng tập (`train_fit`, `val_tau`, `cal`, `test`) và ghi nhận trong `run_provenance.json` **trước khi chạy `C3`**.
  4. **Lưu ý về cỡ mẫu:** Quy mô `1.600 / 400 / 200 / 200` là **cấu hình khởi đầu đã duyệt để triển khai `C3`/`C4`, không phải kết luận thống kê về cỡ mẫu tối ưu**.

### 2.2. Kỷ luật cô lập vai trò bốn tập và quy tắc huấn luyện / ngưỡng / hiệu chuẩn

| Tập | Nhiệm vụ duy nhất được phép | Quy tắc thuật toán bắt buộc (`MODEL.md §3`) | Hành vi bị cấm tuyệt đối |
| --- | --- | --- | --- |
| **`train_fit`** (`1.600`) | Khớp mô hình $\Theta$, chuẩn hóa đặc trưng và đóng băng đường cơ sở $b_m$. | - Khớp toàn bộ $\Theta = \{\{\eta_k, \lambda_k, \gamma_k\}_{k=0}^{11}, \Delta\mathbf{R}_\theta, a_{\mathrm{cal}}, b_{\mathrm{cal}}\}$ với ràng buộc dương cho $\eta_k, \lambda_k, \gamma_k, a_{\mathrm{cal}}$.<br>- Hàm mất mát **Weighted BCE trên logits** $z_e = a_{\mathrm{cal}}\, x_e^{(12)} - b_{\mathrm{cal}}$, **chỉ tính trên các cặp `(biến cố, phân đoạn)` có nhãn và khả kiến ($c_{\mathrm{obs}}(e) \ge 1$)**.<br>- Trọng số lớp âm $w_- = 1$; trọng số lớp dương $w_+ = N_{\text{âm}} / N_{\text{dương}}$ (kiểm tra $N_{\text{dương}} > 0$), trong đó **$N_{\text{âm}}, N_{\text{dương}}$ chỉ được đếm trên tập cặp có nhãn và khả kiến của `train_fit`**.<br>- Đóng băng $b_m = \mathrm{minRTT}_m^{\mathrm{base}}$ và tham số chuẩn hóa hoàn toàn trên `train_fit`. | Rút mẫu từ `cal`/`test`; trộn `incident_group_id`; đếm $N_{\text{âm}}/N_{\text{dương}}$ trên tập khác; dùng điểm `train_fit` làm điểm nghiệm thu. |
| **`val_tau`** (`400`) | **Chỉ** chọn ngưỡng cắt cứng tự động $\tau_{\mathrm{cut}}$. | - Quét lưới $\tau \in [0{,}50;\, 0{,}99]$ bước `0,01` trên `val_tau`.<br>- Chọn ngưỡng đạt **recall phát hiện đứt hoàn toàn cao nhất** thỏa mãn **tỷ lệ kịch bản có ít nhất 1 liên kết không đứt bị cắt nhầm `< 5%`**.<br>- Nếu nhiều ngưỡng cùng đạt recall cao nhất $\Rightarrow$ **chọn ngưỡng cao hơn**.<br>- **Tắt cắt tự động** (`auto_cut_policy.enabled = false`, `tau_cut = null`, ghi rõ `disabled_reason`) nếu thiếu nhãn đứt hoàn toàn hoặc mọi $\tau \in [0{,}50;\, 0{,}99]$ đều gây `≥ 5%` kịch bản cắt nhầm. | Cập nhật $\Theta$, khớp $a_{\mathrm{cal}}/b_{\mathrm{cal}}$, đóng băng $b_m$, ước lượng $\hat{q}_{0{,}90}$, hay tối ưu thêm $\tau_{\mathrm{F1}}$. |
| **`cal`** (`200`) | **Chỉ** ước lượng phân vị Split Conformal $\hat{q}_{0{,}90}$ riêng cho từng cấu hình. | - Trên cùng `200` bản ghi `cal`, dùng các **ca có lỗi đủ điều kiện** ($\mathcal{F}_i \neq \varnothing$ và khả kiến) để tính điểm bất phù hợp $s_i = 1 - \min_{e \in \mathcal{F}_i} \hat{p}_{i,e}$.<br>- Lấy phân vị $\hat{q}_{0{,}90}$ tại bậc $\frac{\lceil (n_{\mathrm{fault,cal}} + 1)(1 - \alpha) \rceil}{n_{\mathrm{fault,cal}}}$ với $\alpha = 0{,}10$.<br>- Thực hiện **hiệu chuẩn riêng từng cấu hình** (`B1`, `B2`, `A1`, `A2`, `Full`) trên đúng cùng tập `200 cal` đó. | Huấn luyện $\Theta$ (kể cả $a_{\mathrm{cal}}, b_{\mathrm{cal}}$), đóng băng $b_m$, chọn kiến trúc/siêu tham số, quét $\tau_{\mathrm{cut}}$ hay tối ưu $\tau_{\mathrm{F1}}$. |
| **`test`** (`200`) | Đánh giá độc lập **đúng một lần duy nhất ở chế độ chỉ đọc**. | - **Tháo hoàn toàn tệp `D_test` khỏi tiến trình `train_and_calibrate`** (kiểm tra tệp không tồn tại / không được gắn trong môi trường huấn luyện và hiệu chuẩn).<br>- Khóa cứng toàn bộ trọng số, $b_m$, $\tau_{\mathrm{cut}}$, $\hat{q}_{0{,}90}$, cấu hình và `run_provenance.json` trước khi gọi tiến trình `evaluate_locked_test` đúng `1` lần (`locked_test_access_count == 1`). | Chỉnh sửa bất kỳ tham số/ngưỡng nào sau khi mở `test`; mở lại `test` nhiều lần; **dùng `D_test` để chọn runtime trình duyệt** (`ADR-01`–`ADR-04`). |

### 2.3. Giao thức kiểm chứng tự động 5 chốt chống rò rỉ và bất biến mô hình (`V-LEAKAGE`)

`V-LEAKAGE` là **cổng chặn bắt buộc** cho mọi lượt huấn luyện, hiệu chuẩn và công bố điểm ([DATA.md](DATA.md) §4.2, [subsea-anti-leakage](../.agents/skills/subsea-anti-leakage/SKILL.md)). **Tiêu chí đạt duy nhất là `0` vi phạm trên mọi chốt thuộc phạm vi lượt chạy; nếu thiếu bằng chứng xác minh của bất kỳ chốt nào, bắt buộc đánh dấu lượt chạy `BLOCKED` và chặn công bố mọi điểm số mô hình.**

| Chốt / Kiểm tra (`V-LEAKAGE`) | Phạm vi áp dụng | Ca dương (hợp lệ — phải vượt qua) | Ca âm (cố tình vi phạm — phải bị chặn/báo lỗi) | Bằng chứng tự động bắt buộc |
| --- | --- | --- | --- | --- |
| **Chốt 1 (`ML-001`):** Chia tiến thời gian, vùng đệm `±72 giờ` & gom nhóm sự cố | MVP (`Must`) | 1. Lịch sử: chuỗi chia đúng khung `UTC` (`train`: `2021-01-01..2024-06-30`, `cal`: `2024-07-01..2025-06-30`, `test`: `2025-07-01..2026-11-15`), $\max(t_{\mathrm{train}}) < \min(t_{\mathrm{cal}})$ và $\max(t_{\mathrm{cal}}) < \min(t_{\mathrm{test}})$, mọi mẫu âm nằm ngoài $[t_0-72\text{h}, t_0+72\text{h}]$ và $[t_{\mathrm{repair}}-72\text{h}, t_{\mathrm{repair}}+72\text{h}]$.<br>2. Bán tổng hợp: `1.600/400/200/200` giữ đủ tỷ lệ `55%/30%/15%`, `0` trùng `incident_group_id`. | 1. Cố tình đưa 1 cửa sổ ở `2024-07-01T00:00:00Z` vào `train`, hoặc 1 mẫu âm tại $t_0 + 71\text{h}59\text{p}$ / $t_{\mathrm{repair}} - 72\text{h}00\text{p}$ $\Rightarrow$ phải chặn.<br>2. Cố tình để 1 `incident_group_id` xuất hiện ở cả `train_fit` và `val_tau`/`cal`/`test`, hoặc rút 10 mẫu từ `cal` sang `train` $\Rightarrow$ phải chặn. | `split_manifest`, biên `min/max UTC` từng tập, `embargo_violation_count = 0`, `incident_group_overlap_count = 0`, kiểm định cỡ mẫu `1600/400/200/200`. |
| **Chốt 2 (`ML-002`):** LOBO 4 lưu vực & cấm tọa độ thô ở đường tiên nghiệm GĐ2 | Đường ống tiên nghiệm GĐ2 (`E3`); kiểm thử đơn vị bộ chia & lược đồ có thể chạy sớm, **không áp lên đường chạy LP-GISTA MVP không có Hazard Encoder** | Bộ chia tạo đủ 4 lượt không giao nhau trên 4 lưu vực (*Biển Đông & Eo biển Luzon; Biển Đỏ & Ấn Độ Dương; Tây Phi & Đại Tây Dương; Biển Baltic & Địa Trung Hải*) với `0` `segment_id` giao cắt giữa train/val từng lượt; lược đồ $\mathbf{z}_e$ chỉ gồm đặc trưng vật lý hành lang, không có cột tọa độ thô. | 1. Cố tình gán 1 phân đoạn xuất hiện đồng thời ở train và val trong một lượt LOBO $\Rightarrow$ phải chặn.<br>2. Cố tình chèn cột `lat_deg` hoặc `lon_deg` (hoặc tọa độ tâm phân đoạn thô) vào $\mathbf{z}_e$ $\Rightarrow$ bộ kiểm lược đồ phải từ chối. | Báo cáo 4 lượt LOBO (`basin_overlap_count = 0`), danh sách cột lược đồ $\mathbf{z}_e$ đã xác thực `0` cột tọa độ thô. |
| **Chốt 3 (`ML-003`):** Mặt nạ trễ AIS `96 giờ` / SAR `120 giờ` & loại tàu sửa cáp | **Kiểm thử logic đơn vị bắt buộc trong MVP (`Must`)**; nguồn GFW 4Wings bật ở GĐ2 sau `O-01` | Fixture hàng hải chứa bản ghi AIS có mốc $\le \min(t, t_0) - 96\text{h}$, SAR $\le \min(t, t_0) - 120\text{h}$ thuộc tàu thương mại/đánh cá thông thường $\Rightarrow$ giữ đúng 100% bản ghi hợp lệ. | Fixture chứa bản ghi AIS trong $(\min(t, t_0) - 96\text{h}, t_0]$, bản ghi SAR trong $(\min(t, t_0) - 120\text{h}, t_0]$, hoặc bản ghi phân lớp tàu sửa cáp / dịch vụ công trình ngầm (dù ở trước hay sau $t_0$) $\Rightarrow$ phải lọc sạch 100%; nếu lọt vào đặc trưng thì bộ kiểm `V-LEAKAGE` phải báo `FAIL`. | Nhật ký kiểm thử đơn vị Chốt 3 (`ais_lag_violations = 0`, `sar_lag_violations = 0`, `repair_vessel_retained_count = 0`). |
| **Chốt 4 (`ML-004`):** Vòng đời RFS & bản chụp `< t` (gồm hồi quy `2023-02` và `2026-08-25`) | MVP (`Must`) | 1. Mốc **`2023-02`**: `SMW-3` có `in_service = true`, còn `ADC`, `SJC2`, `VSTN` có `in_service = false` và `r_weight = 0`.<br>2. Mốc **`2026-08-25`**: `SMW-3` có `in_service = false` (`r_weight = 0`), còn `ADC`, `SJC2`, `VSTN` có `in_service = true`.<br>3. Mọi `source_cutoffs_utc < effective_at_utc` và mọi mẫu RTT có $\tau \le$ `window_end_utc`. | 1. Cố tình nạp `GraphSnapshot` tại `2023-02` để `ADC`/`VSTN` có `r_weight > 0`, hoặc tại `2026-08-25` để `SMW-3` có `in_service = true` $\Rightarrow$ phải chặn.<br>2. Nạp bản chụp CAIDA/PeeringDB có `published_at_utc == effective_at_utc` hoặc `> effective_at_utc`, hoặc mẫu RTT tại $\tau >$ `window_end_utc` $\Rightarrow$ phải chặn. | Kết quả 2 ca hồi quy `2023-02` & `2026-08-25`, kiểm tra `future_cutoff_violations = 0`, `future_rtt_sample_violations = 0`. |
| **Chốt 5 (`ML-005`, `ML-006`):** Cô lập vai trò tập, tháo `D_test` và chọn $\tau_{\mathrm{cut}}$ trên `val_tau` | MVP (`Must`) | 1. Tiến trình `train_and_calibrate` chạy thành công khi tệp `D_test` bị tháo hoàn toàn khỏi đĩa/môi trường.<br>2. `b_m`, chuẩn hóa và tỷ số $N_{\text{âm}}/N_{\text{dương}}$ chỉ đọc từ `train_fit`.<br>3. Bộ chọn $\tau_{\mathrm{cut}}$ trên `val_tau` chọn đúng ngưỡng cao hơn khi hòa recall dưới trần `< 5%`.<br>4. `evaluate_locked_test` mở `D_test` chỉ đọc đúng `1` lần sau khi khóa tham số. | 1. Cố tình để tiến trình huấn luyện truy cập đường dẫn `D_test`, hoặc dùng `cal` để chọn $\tau_{\mathrm{cut}}$ / khớp $b_m$, hoặc dùng `val_tau` cập nhật $\Theta$ $\Rightarrow$ phải chặn.<br>2. Ca biên $\tau_{\mathrm{cut}}$: (a) mọi ngưỡng gây `≥ 5%` kịch bản cắt nhầm và (b) thiếu nhãn đứt hoàn toàn $\Rightarrow$ bắt buộc tắt (`enabled = false`, `tau_cut = null`); nếu vẫn trả ngưỡng số là `FAIL`.<br>3. Cố tình ghi `locked_test_access_count > 1` hoặc dùng `D_test` trong benchmark M1 $\Rightarrow$ phải chặn. | Log chạy `train_and_calibrate` với `D_test` vắng mặt, 3 ca unit test biên $\tau_{\mathrm{cut}}$, `run_provenance.json` có `locked_test_access_count = 0` (lúc train) và `1` (lúc chấm cuối). |
| **Kiểm hỗ trợ định tuyến (`ML-009`, `ADR-05`):** Mặt nạ $\mathbf{M}_{\mathrm{mask}}$ & chiếu $\Pi_\Delta$ | MVP (`Must`) | Sau huấn luyện (chế độ `in_profile`) và ở chế độ chuyển giao `zero_delta_r_transfer` ($\Delta\mathbf{R}_\theta = \mathbf{0}$): `100%` vị trí $R_{me}(t) = 0$ giữ $\tilde{R}_{me}(t) = 0$; mọi $\tilde{R}_{me} \ge 0$; mọi hàng có $\mathcal{S}_m \neq \varnothing$ có tổng bằng `1` trong sai số dấu phẩy động; $\hat{\mathbf{x}}, \hat{\mathbf{p}} \in [0, 1]^E$. | Cố tình tiêm nhiễu $\Delta R_{me} \neq 0$ tại vị trí $R_{me} = 0$ (cạnh phi vật lý) hoặc bỏ phép chiếu đơn hình $\Pi_\Delta$ $\Rightarrow$ phải phát hiện `non_physical_edge_count > 0` và báo `FAIL`. | Báo cáo kiểm tra bất biến `ML-009` (`non_physical_edge_count = 0`, sai số tổng hàng đơn hình, biên $[0,1]^E$). |
| **Khóa tắt Mohr–Coulomb (`ML-007`):** Cấm suy $(c_e,\varphi_e)$ từ GEBCO & cấm gán $S_e=1$ sai | MVP (`Must`) | Cấu hình huấn luyện MVP khóa cứng `mohr_coulomb_enabled = false`; kiểm tra tĩnh mã nguồn xác nhận không có hàm ánh xạ độ sâu GEBCO $\to (c_e, \varphi_e)$ và không sinh $S_e(t) = 1$ từ các khoảng thời gian không có báo cáo sự cố. | Cố tình bật `mohr_coulomb_enabled = true` khi chưa có tệp dữ liệu thí nghiệm cắt trực tiếp đã duyệt `O-02`, hoặc truyền giá trị mặc định $(c_e, \varphi_e)$ / suy từ GEBCO $\Rightarrow$ phải chặn ngay trước khi huấn luyện. | Báo cáo kiểm tra cấu hình (`mohr_coulomb_enabled = false`) và kết quả kiểm tra tĩnh mã nguồn/fixture âm tính. |

---

## 3. Giao thức chấm bộ bán tổng hợp (`ML-010`) và phát lại lịch sử (`ML-011`)

### 3.1. Đối sánh năm cấu hình (`B1`, `B2`, `A1`, `A2`, `Full`) trên điều kiện đồng nhất

Toàn bộ 5 cấu hình đối sánh (và bản lượng tử hóa nếu phát hành) bắt buộc chạy trên **chính xác cùng `GraphSnapshot` đóng băng, cùng `split_manifest` (`1.600 / 400 / 200 / 200`), cùng bộ `random_seeds` và cùng mã nguồn chấm điểm** ([MODEL.md](MODEL.md) §5.1, [subsea-evaluation](../.agents/skills/subsea-evaluation/SKILL.md)):

| Mã | Cấu hình | Chế độ tiên nghiệm (`prior_mode`) | Mục đích khoa học |
| --- | --- | --- | --- |
| **`B1`** | Heuristic đồng rút tuyến BGP ($\mathbf{R}_t^\top \tilde{\mathbf{y}}^{\mathrm{BGP}}$) | Không dùng tiên nghiệm | Mốc cơ sở đơn tầng kiểm chứng mức suy giảm do cộng tuyến hành lang và báo động giả trước nhiễu BGP phi vật lý. |
| **`B2`** | Cảm biến nén L1 đồng nhất (60 vòng lặp ISTA cổ điển, chỉ BGP) | Không dùng tiên nghiệm (`w_e = 1`) | Tách riêng đóng góp của giải ngược thưa cổ điển khi chưa trải phẳng và chưa có cổng $\Delta\mathrm{minRTT}$. |
| **`A1`** | LP-GISTA 12 lớp cắt bỏ cổng độ trễ $\Delta\mathrm{minRTT}$ | `synthetic_benchmark_prior` | Cô lập đóng góp của cổng vật lý chéo tầng $g_m(t)$ đối với `Distractor FAR` và nhóm đứt đơn bị che khuất BGP (`25%`). |
| **`A2`** | LP-GISTA 12 lớp cắt bỏ tiên nghiệm hàng hải – địa vật lý | **`uniform_w1`** ($w_e(t) = 1$) | Cô lập đóng góp của vector trọng số phạt thưa thích nghi $\mathbf{w}(t)$ trong việc tách cáp song song cộng tuyến cao và thu hẹp $\lvert \mathcal{C}_{0{,}90} \rvert$. |
| **`Full`** | SubseaBGP-Tomography đầy đủ (LP-GISTA 12 lớp + cổng $\Delta\mathrm{minRTT}$ + $\mathbf{L}_{\mathrm{SRLG}}$ + $\mathbf{w}(t)$ + Split Conformal) | **`synthetic_benchmark_prior`** *(trong benchmark bán tổng hợp)* | Mô hình mục tiêu nghiệm thu cổng `ML-010` trên đường cơ sở thực nghiệm mới *(lưu ý: `synthetic_benchmark_prior` chỉ áp dụng trong bộ kiểm chuẩn bán tổng hợp `C3`; phát lại lịch sử MVP dùng `uniform_w1` theo `DATA.md §2.4`)*. |

- Mỗi cấu hình (`B1`, `B2`, `A1`, `A2`, `Full`) tự hiệu chuẩn phân vị Split Conformal $\hat{q}_{0{,}90}$ riêng trên **cùng tập `200 cal`** để bảo đảm đối sánh công bằng về độ phủ, kích thước tập và `Distractor FAR`.
- **Cấm hoàn toàn dùng chỉ số `Accuracy` tổng thể** ở mọi bảng báo cáo do mất cân bằng lớp nghiêm trọng trên đồ thị (`ML-010`, `RESEARCH.md §6.1`).

### 3.2. Cổng nghiệm thu `Q-04` trên `200 test` và quy tắc bản lượng tử hóa (`ML-010`)

Trên tập `200 test` khóa trước (`110` đứt đơn + `60` đa cáp SRLG = `170` ca có lỗi; `30` ca nhiễu phi vật lý), mô hình `Full` phải đạt **đồng thời** toàn bộ các cổng sau:

| Chỉ số (`Q-04`, `ML-010`) | Ngưỡng nghiệm thu bắt buộc | Dấu bất đẳng thức | Tham chiếu cũ `[1]` *(chỉ đối chiếu, KHÔNG phải mục tiêu tái lập)* |
| --- | --- | --- | --- |
| **`Hit@1`** | **`≥ 90%`** | Không nhỏ hơn (`>=`) | *`92,4%` (`unknown_url`)* |
| **`Hit@3`** | **`≥ 95%`** | Không nhỏ hơn (`>=`) | *`98,3%` (`unknown_url`)* |
| **`PR-AUC`** | **`> 0,85`** | Lớn hơn nghiêm ngặt (`>`) | *`0,879` (`unknown_url`)* |
| **`Distractor FAR`** | **`< 5%`** | Nhỏ hơn nghiêm ngặt (`<`) | *`3,6%` (`unknown_url`)* |
| **Độ phủ Conformal thực nghiệm** *(trên ca có lỗi)* | **`≥ 88,5%`** *(mức danh định `90%`, $\alpha = 0{,}10$)* | Không nhỏ hơn (`>=`) | *`89,5%` (`unknown_url`)* |
| **Kích thước tập trung bình $\lvert \mathcal{C}_{0{,}90} \rvert$** | **`≤ 3` phân đoạn** | Không lớn hơn (`<=`) | *`2,02` (`unknown_url`)* |
| **Mức giảm `PR-AUC` khi lượng tử hóa** *(chỉ áp dụng nếu phát hành `fp16` / `int8_mixed`)* | **`ΔPR-AUC = PR-AUC_FP32 - PR-AUC_quant < 0,005`** | Nhỏ hơn nghiêm ngặt (`<`) | *Không áp dụng cho MVP mặc định `FP32` (`ADR-02`)* |

- **Điều kiện phát hành bản lượng tử hóa:** Mặc định MVP phát hành **ONNX FP32** (`ADR-02`). Chỉ khi phát hành thêm biến thể lượng tử hóa (`fp16` hoặc `int8_mixed`) mới kích hoạt cổng $\Delta\mathrm{PR\text{-}AUC}_{\mathrm{quant}} < 0{,}005$, đồng thời biến thể lượng tử hóa vẫn phải giữ đạt toàn bộ các cổng `Q-04` còn lại (độ phủ `≥ 88,5%`, `Distractor FAR < 5%`, v.v.) và ghi tường minh `delta_pr_auc_vs_fp32` vào `ReleaseManifest.model_artifact` và `run_provenance.json`.

### 3.3. Định nghĩa chi tiết từng chỉ số, đơn vị chấm và cách xử lý thiếu dữ liệu

Mọi chỉ số dưới đây tuân thủ [MODEL.md](MODEL.md) §5.2–§5.3. Báo cáo bắt buộc ghi đồng thời **mẫu số tổng** và **mẫu số khả kiến/chấm được** — nghiêm cấm loại ngầm ca `NA` khỏi mẫu số.

| Chỉ số | Vai trò | Đơn vị chấm | Tử số / Mẫu số & Công thức | Xử lý `NA` / Không khả kiến & Ghi chú bắt buộc |
| --- | --- | --- | --- | --- |
| **`Hit@1`**, **`Hit@3`** | Cổng `Q-04` | Biến cố có lỗi ($\mathcal{F}_i \neq \varnothing$) | $\mathrm{Hit@}k = \frac{\sum_{i \in \mathcal{I}_{\mathrm{fault}}} \mathbb{I}\left(\mathrm{Top}_k(\hat{\mathbf{p}}_i) \cap \mathcal{F}_i \neq \varnothing\right)}{n_{\mathrm{fault,total}}}$, xếp hạng giảm dần theo điểm chưa làm tròn $\hat{p}_{i,e}$ trên các phân đoạn khả kiến với **phá hòa bi quan** (`D-02`: trong nhóm đồng điểm, mọi phân đoạn không lỗi đứng trước các phân đoạn lỗi); đa cáp SRLG cần ít nhất 1 lỗi thật trong top-$k$. | Cổng `Q-04` dùng mẫu số tổng `n_fault_total` (`170` ca), đồng thời báo riêng trên các ca có tất cả lỗi thật khả kiến (`n_fault_all_obs`). Phân đoạn `unobservable_na` không được xếp vào top-$k$; ca đa cáp còn 1 lỗi khả kiến trong top-$k$ vẫn có thể đạt `Hit@k`. |
| **`PR-AUC`** | Cổng `Q-04` | Cặp `(biến cố, phân đoạn)` có nhãn và khả kiến | Diện tích dưới đường cong Precision–Recall tính theo phương pháp bậc thang (*Average Precision*) trên toàn bộ các cặp `(i, e)` có nhãn $y_{i,e} \in \{0, 1\}$ và $c_{\mathrm{obs}}(i, e) \ge 1$ trong `200 test` (bao gồm cả ca lỗi lẫn ca nhiễu để phản ánh độ tinh khiết trước nhiễu). | Loại tường minh các cặp `unobservable_na` khỏi đường cong (**không gán điểm `0` cho `NA`**) và bắt buộc báo riêng `n_pair_obs / n_pair_total` (trên $200 \times 24 = 4.800$ cặp). |
| **Độ phủ Conformal thực nghiệm** | Cổng `Q-04` | Biến cố có lỗi ($\mathcal{F}_i \neq \varnothing$) | $\widehat{\mathrm{Cov}} = \frac{1}{n_{\mathrm{fault,total}}}\sum_{i \in \mathcal{I}_{\mathrm{fault}}} \mathbb{I}\left(\mathcal{F}_i \subseteq \mathcal{C}_{0{,}90}(\mathbf{y}_i)\right)$ với $\mathcal{C}_{0{,}90}(\mathbf{y}_i) = \{e : c_{\mathrm{obs}}(i,e) \ge 1,\ \hat{p}_{i,e} \ge 1 - \hat{q}_{0{,}90}\}$. **Bao phủ phải chứa toàn bộ tập lỗi thực tế $\mathcal{F}_i$** (với đứt đa cáp SRLG 2–3 cáp, thiếu dù chỉ 1 phân đoạn là không bao phủ). | Bắt buộc báo dạng **`k / n (p%)`** kèm **khoảng tin cậy Wilson hai phía 95%** (`D-02`; cổng **`≥ 88,5%` áp dụng cho ước lượng điểm, không áp dụng cho cận dưới CI**). **Phân biệt rõ:** ngưỡng `≥ 88,5%` là cổng kiểm thử hữu hạn mẫu, còn mức `90%` ($\alpha = 0{,}10$) là bảo chứng xác suất biên danh định dưới giả thiết hoán đổi được — không tuyên bố đạt `90%` trước khi hiệu chuẩn đã chạy và xác minh. Chỉ 1 phân đoạn lỗi `unobservable_na` cũng đủ làm ca đó trượt độ phủ trên mẫu số cổng `n_fault_total`; báo thêm kết quả trên các ca có tất cả lỗi thật khả kiến (`n_fault_all_obs`). |
| **Kích thước tập TB $\lvert \mathcal{C}_{0{,}90} \rvert$** | Cổng `Q-04` | Biến cố | $\overline{\lvert \mathcal{C}_{0{,}90} \rvert} = \frac{1}{n} \sum_i \lvert \mathcal{C}_{0{,}90}(\mathbf{y}_i) \rvert$. | **Bắt buộc báo tách riêng hai con số:** (1) trên các ca có lỗi ($\mathcal{F}_i \neq \varnothing$) và (2) trên toàn bộ `200 test` (ghi rõ số ca `insufficient_observations` bị loại khỏi trung bình hoặc có tập rỗng do `NA`, không ẩn mẫu số). |
| **`Distractor FAR`** | Cổng `Q-04` | Biến cố nhiễu phi vật lý ($\mathcal{F}_i = \varnothing$) | $\mathrm{FAR}_{\mathrm{dist}} = \frac{k_{\mathrm{FA}}}{n_{\mathrm{dist,scorable}}}$, trong đó mỗi biến cố nhiễu tính **đúng một lần** và là báo động giả nếu **tồn tại ít nhất một cửa sổ chấm khóa trước** thỏa $\mathcal{C}_{0{,}90}(t) \neq \varnothing$. | $n_{\mathrm{dist,scorable}} = 30 - n_{\mathrm{dist,NA}}$. **Bắt buộc báo riêng `n_dist_NA / 30` và tuyệt đối không tính ca `NA` là âm tính đúng** (kể cả khi `calibrated_90` trả `[]` ở `insufficient_observations`). *(Ví dụ số học minh họa — **không phải kết quả thực nghiệm**: nếu `n_dist_scorable = 30`, thì `1/30 = 3,33% < 5%`, còn `2/30 = 6,67% ≥ 5%`.)* |
| **`MRR`** | Chỉ số phụ | Biến cố có lỗi ($\mathcal{F}_i \neq \varnothing$) | $\mathrm{MRR} = \frac{1}{n_{\mathrm{fault,total}}}\sum_{i \in \mathcal{I}_{\mathrm{fault}}} \frac{1}{\min_{e \in \mathcal{F}_i} \mathrm{rank}_i(e)}$ dùng hạng của lỗi thật đầu tiên với điểm chưa làm tròn và phá hòa bi quan (`D-02`, đóng góp `0` nếu mọi lỗi thật `unobservable_na`). | Không đặt cổng chặn riêng; báo kèm trên `n_fault_all_obs` để đối chiếu chất lượng xếp hạng giữa `B1`/`B2`/`A1`/`A2`/`Full`. |
| **`ECE`** | Chỉ số phụ | Cặp `(biến cố, phân đoạn)` có nhãn và khả kiến | $\mathrm{ECE} = \sum_{b=1}^{10} \frac{\lvert \mathcal{B}_b \rvert}{N_{\mathrm{obs}}} \left\lvert \mathrm{acc}(\mathcal{B}_b) - \mathrm{conf}(\mathcal{B}_b) \right\rvert$ trên **`10` bin đều trên $[0, 1]$**, trái đóng phải mở $[0{,}1(b-1), 0{,}1b)$ với bin cuối chứa `1` ($[0{,}9, 1{,}0]$) (`D-02`), so sánh xác suất dự đoán trung bình với tỷ lệ lỗi thực tế trong mỗi bin. | Chỉ tính trên cặp có nhãn và khả kiến; báo rõ `B = 10` và `N_obs`. |
| **`F1`** | Chỉ số phụ | Cặp `(biến cố, phân đoạn)` có nhãn và khả kiến | Dự đoán dương tính $\hat{y}_{i,e} = \mathbb{I}(\hat{p}_{i,e} \ge 1 - \hat{q}_{0{,}90})$ tại **đúng ngưỡng Conformal $1 - \hat{q}_{0{,}90}$ đã khóa**; $\mathrm{F1} = \frac{2\mathrm{TP}}{2\mathrm{TP}+\mathrm{FP}+\mathrm{FN}}$. | **Tuyệt đối không tối ưu thêm ngưỡng $\tau_{\mathrm{F1}}$ riêng** trên `val_tau`, `cal` hay `test` (`MODEL.md §5.2, §7.2`). Báo đủ `TP / FP / FN` và số cặp `unobservable_na`. |

### 3.4. Yêu cầu báo cáo đầy đủ về mẫu số, lỗi bỏ sót, báo động giả và trạng thái `NA`

Mọi báo cáo kết quả chạy thực tế sau này trên bộ kiểm chuẩn bán tổng hợp bắt buộc kèm 4 khối kiểm toán sau (mọi con số chỉ xuất hiện sau khi chạy thật; nếu đặt ví dụ cấu trúc trong tài liệu phải ghi nhãn rõ **`[VÍ DỤ MINH HỌA — KHÔNG PHẢI KẾT QUẢ]`**):

1. **Bảng mẫu số tổng và mẫu số khả kiến:** Ghi rõ cho từng tập (`train_fit`, `val_tau`, `cal`, `test`): tổng số biến cố theo 3 nhóm (`55%/30%/15%`), **số thực tế ca đứt đơn bị che khuất BGP** (phát sinh theo xác suất `25%`), số ca/cửa sổ `ok`, `degraded_partial_na`, `insufficient_observations`, số cặp `(biến cố, phân đoạn)` `unobservable_na`, và `n_dist_NA / 30`.
2. **Bảng đối sánh 5 cấu hình (`B1`, `B2`, `A1`, `A2`, `Full`) + bản lượng tử hóa (nếu có):** Ghi đủ 6 chỉ số cổng `Q-04` (kèm tử số/mẫu số và CI 95% cho độ phủ), kích thước tập tách riêng (ca lỗi vs toàn bộ `200 test`), 3 chỉ số phụ (`MRR`, `ECE`, `F1` tại $1 - \hat{q}_{0{,}90}$), giá trị $\hat{q}_{0{,}90}$ riêng của từng cấu hình và trạng thái `auto_cut_policy` ($\tau_{\mathrm{cut}}$ hoặc `disabled`).
3. **Danh mục phân tích ca bỏ sót (*False Negatives*):** Tách rõ bỏ sót do che khuất bảo vệ quang tầng 1, bỏ sót do cộng tuyến cáp song song, và bỏ sót do mất khả kiến (`unobservable_na`).
4. **Danh mục phân tích báo động giả (*False Alarms*):** Tách rõ báo động giả từ nhiễu mặt phẳng điều khiển BGP/IXP và báo động giả do nhiễu hạ tầng nội địa nối ra CLS, kèm `window_id` kích hoạt $\mathcal{C}_{0{,}90}(t) \neq \varnothing$.

### 3.5. Giao thức phát lại và hồi quy sự cố lịch sử (`ML-011` — HIỆN `BỊ CHẶN`)

#### Trạng thái hiện tại và phân loại danh sách ứng viên (`DATA.md §4.1, §6.2`)

- **Trạng thái cổng `ML-011`:** **`BỊ CHẶN`** — danh sách sự cố nghiệm thu chính thức **chưa được khóa** vì kho mã chưa tải và xác minh thực tế dữ liệu BGP/RIPE Atlas của từng ứng viên.
- **Ba ứng viên khung `test` (`2025-07-01..2026-11-15`) — CHỈ LÀ ỨNG VIÊN, CHƯA KHÓA:**
  1. *Biển Đỏ (`2025-09-06`):* Cloudflare ghi nhận tác động mạng và dẫn thông cáo PTCL; mốc `05:45 UTC` là mốc gián đoạn dịch vụ Azure — **tuyệt đối không dùng làm mốc đứt cáp $t_0$ đã xác nhận**.
  2. *Đông Đài Loan (`2025-12-27`):* MODA xác nhận 6 hệ thống cáp gặp sự cố liên tiếp sau động đất; mốc USGS `15:05:55 UTC` là thời điểm động đất — **tuyệt đối không dùng làm mốc hỏng của từng tuyến cáp**.
  3. *Việt Nam (`2026-08-25`):* Nguồn công khai ghi nhận lỗi phát sinh trên `SJC2` và `AAE-1`, trong khi `AAG` tồn đọng lỗi trước đó và `IA` có gián đoạn — **tuyệt đối không gán chung một $t_0$ cho cả bốn tuyến khi chưa có xác nhận độc lập từng tuyến**.
- **Hai sự cố năm `2024`** (*Biển Đỏ `2024-02-24`*, *Tây Phi `2024-03-14`*): Rơi vào khung thời gian `train` (`2021-01-01..2024-06-30`) $\Rightarrow$ **chỉ dùng cho phát triển, kiểm thử hồi quy phát triển hoặc minh họa nội bộ; nghiêm cấm đưa vào tập nghiệm thu `ML-011`**.

#### Điều kiện bắt buộc để gỡ chặn và khóa danh sách `ML-011`

Danh sách nghiệm thu `ML-011` chỉ được phép chuyển từ `BỊ CHẶN` sang `CHƯA CHẠY` (đủ điều kiện thực thi) khi kiểm tra thực tế trên dữ liệu sơ cấp xác nhận đồng thời **cả 6 điều kiện** cho **ít nhất `3` sự cố độc lập nằm trọn trong khung `test` (`2025-07-01..2026-11-15`)**:

1. **Đủ dữ liệu đo liên quan:** Có đủ bản ghi BGP (RouteViews / RIPE RIS MRT) liên quan đến các ASN bị ảnh hưởng VÀ chuỗi đo RIPE Atlas (`ping`/`traceroute` *built-in*/*anchoring*) liên quan trong cửa sổ phát lại (cùng bản kê tường minh tỷ lệ `NA` / `MISSING_BM`). *Việc tệp BGP thô tồn tại trên kho lưu trữ không tự đồng nghĩa có đủ dữ liệu phát lại.*
2. **`100%` truy vết nguồn sơ cấp:** Mỗi sự cố có đầy đủ `source_citation` từ nguồn sơ cấp/công bố xác minh được.
3. **Độ chính xác mốc thời gian `UTC`:** Giữ đúng độ chính xác thời gian mà nguồn sơ cấp xác nhận; **không** thay giờ đứt cáp bằng giờ động đất hay giờ gián đoạn dịch vụ đám mây và **không** gán chung $t_0$ cho các tuyến chưa được xác nhận đồng thời.
4. **`GraphSnapshot` đóng băng trước cửa sổ chấm:** `source_cutoffs_utc < effective_at_utc` và trạng thái vòng đời RFS (`in_service`) chính xác tại thời điểm sự cố (`ML-004`).
5. **Phân định cấp nhãn và trạng thái `NA`:** Ghi rõ `label_granularity` (`system` vs `segment`; khi là `system` bắt buộc `confirmed_segment_ids = null`, không tự suy đoán phân đoạn) và kê đủ toàn bộ trạng thái `NA`.
6. **`0` vi phạm chống rò rỉ (`V-LEAKAGE`):** Đạt toàn bộ các chốt chống rò rỉ liên quan trước khi chấm.

#### Giao thức chạy phát lại lịch sử MVP theo `ADR-05`

- **Chế độ suy luận & tiên nghiệm:** Bắt buộc chạy ở chế độ chuyển giao **`inference_mode = zero_delta_r_transfer`** ($\Delta\mathbf{R}_\theta = \mathbf{0}$, giữ nguyên mặt nạ $\mathbf{M}_{\mathrm{mask}}$ và phép chiếu đơn hình $\Pi_\Delta$) với **`prior_mode = uniform_w1`** ($w_e(t) = 1$, `hazard_prob = null`, `prior_na_reason = NO_APPROVED_PRIOR`).
- **Hiệu chuẩn Conformal:** Khi chưa có tập hiệu chuẩn độc lập tương thích với lưu vực/tô-pô đích, bắt buộc đặt `conformal_calibration_status = uncalibrated_transfer`, `calibration_quantile_q = null`, `nominal_coverage_target = null`, `conformal_set_segment_ids = null` và **tuyệt đối không gán bảo chứng độ phủ `90%`**.
- **Quy tắc đóng băng đường cơ sở $b_m$ (`DATA.md §2.3`, `MODEL.md §2.3`):** Đóng băng $b_m = \mathrm{minRTT}_m^{\mathrm{base}}$ theo cặp `(channel_id, path_regime)` từ **cửa sổ sạch `60 phút` gần nhất có `≥ 2` mẫu hợp lệ nằm trong `train_fit` lịch sử**, **không sau mốc `2024-06-30`** và đã **loại bỏ toàn bộ khoảng sự cố cùng vùng đệm `±72 giờ` quanh $(t_0, t_{\mathrm{repair}})$**. Nếu thiếu đường cơ sở hợp lệ $\Rightarrow$ trả **`NA` (`MISSING_BM`)**; **tuyệt đối không mượn cửa sổ ngay trước ca thuộc `test` để bù** *(chủ ý chấp nhận `MISSING_BM` đối với tuyến RFS sau `2024-06-30` như `ADC`, `SJC2`, `VSTN` hoặc chế độ đường đi mới phát sinh giai đoạn `2025–2026`)*.
- **Phát lại bước `5 phút` và tách 3 lớp (`FR-010`):** Bước thời gian cố định `300s`, trên giao diện và báo cáo phải tách bạch tuyệt đối: **(1) tín hiệu đo đầu vào + `NA`**, **(2) đầu ra suy luận mô hình**, và **(3) thông cáo xác nhận sự cố** (`IncidentAnnouncement` tuyệt đối không đi vào `C7`).
- **Tiêu chí đánh giá lịch sử (`Q-04`):** Báo cáo trung thực thứ hạng/xác suất định vị tách riêng giữa nhóm `label_granularity = segment` và `label_granularity = system`, kèm bảng kê đầy đủ kênh/phân đoạn `NA` và `MISSING_BM`; **không áp ngưỡng định lượng của bộ bán tổng hợp lên lịch sử**.

---

## 4. Kiểm chứng hợp đồng, bộ giải LP, phát hành, dự phòng và hiệu năng M1

### 4.1. Kiểm chứng hợp đồng dữ liệu ngoại tuyến (`V-CONTRACT`)

Chạy hoàn toàn ngoại tuyến không cần trình duyệt hay kết nối mạng (`NFR-009`), xác thực đầy đủ **5 nhóm hợp đồng tại [ARCHITECTURE.md](ARCHITECTURE.md) §4** (`GraphSnapshot`, `SignalWindow`, `InferenceResult`, `SimulationRequest` / `SimulationResult`, `ReleaseManifest`) cùng tệp tách riêng **`IncidentAnnouncement`**:

| Hạng mục kiểm chứng (`V-CONTRACT`) | Điều kiện kiểm thử dương tính & âm tính | Tiêu chí đạt bắt buộc |
| --- | --- | --- |
| **SemVer (`schema_version`)** | Dương: `1.0.0` khớp và `1.0.1` (`PATCH`) đọc bình thường; `1.1.0` (`MINOR` cao hơn, không yêu cầu reader cao hơn) đọc lõi kèm cảnh báo.<br>Âm: lệch `MAJOR` (`2.0.0`) hoặc `requires_min_reader_version` vượt phiên bản bên đọc. | Từ chối nạp ngay lập tức với thông báo lỗi nêu rõ phiên bản kỳ vọng và thực tế cho 100% ca không tương thích. |
| **Thời gian `UTC`, khóa & đơn vị** | Dương: mọi mốc dạng `ISO-8601 UTC` kết thúc bằng `Z`; bước `SignalWindow` đúng `300s`; cửa sổ RTT `60 phút`; đủ khóa liên kết (`snapshot_id`, `segment_id`, `channel_id`, `srlg_group_id`, `demand_id`, `path_id`, `window_id`, `request_id`, `model_version`, `run_id`) và hậu tố đơn vị (`_km`, `_m`, `_ms`, `_tbps`, `_pct`, `_mw`, `_kpa`, `_rad`, `_deg`).<br>Âm: mốc thiếu `Z` hoặc dùng `+07:00`; cửa sổ `299s`/`301s`; `length_km ≤ 0`; `capacity_tbps < 0`; `lat_deg = 95`; trùng ID khóa chính. | Chặn 100% bản ghi sai lược đồ, sai đơn vị, lệch múi giờ, sai độ dài cửa sổ hoặc trùng khóa định danh. |
| **Bốn nhãn bản chất dữ liệu (`DATA-002`)** | Kiểm tra trường `data_provenance_class` trên mọi hợp đồng và báo cáo chỉ thuộc tập đóng `{observed_measurement, stylized_derived, synthetic_calibrated_baseline, simulated_assumption}`. | `0` giá trị ngoài tập đóng; `routing_weight_disclaimer`, `interpretation_disclaimer` và `interpretation_warning` hiện diện đầy đủ, không rỗng. |
| **Ngữ nghĩa `NA` tách biệt khỏi `0` (`DATA-006`)** | Dương: `y_bgp = 0` khi `bgp_active_pairs_count > 0` không rút tuyến; `y_rtt = 0` khi `rtt_sample_count ≥ 2`, $b_m > 0$ và $\widehat{\mathrm{minRTT}} \le b_m$; `J = 0` khi hợp tập cơ sở rỗng; `r_weight = 0` khi `in_service = false` hoặc trượt $\Psi_{\mathrm{SoL}}$; $\psi_{\mathrm{shelf}} = 0$ với cáp đất liền; $\mathrm{PGA} = 0$ khi không có động đất $\mathrm{Mw} \ge 4{,}0$.<br>Âm: thiếu $b_m$, $b_m \le 0$, thiếu PoP, `< 2` mẫu RTT, mẫu số BGP `= 0`, `collector_gap`, hoặc `total_served_tbps == 0`. | Mọi trường thiếu điều kiện trả `null` kèm đúng mã trong tập đóng `na_reason` (`MISSING_BM`, `MISSING_POP`, `INSUFFICIENT_RTT_SAMPLES`, `NON_POSITIVE_BM`, `ZERO_BGP_DENOMINATOR`, `COLLECTOR_GAP`, `OUT_OF_SERVICE`, `NO_APPROVED_PRIOR`, `EXTERNAL_SOURCE_UNAVAILABLE`, và `NO_SERVED_FLOW` cho `weighted_rtt_ms`). **`0` trường hợp tự thay `NA` bằng `0` hay mặc định qua cổng $\Psi_{\mathrm{SoL}}$.** |
| **Tiền xử lý lõi `C1` & đồ thị `C2`** | Kiểm thử lọc BGP Class A/B, khử trùng bản tin, lọc reset phiên và công thức $\tilde{y}_m^{\mathrm{BGP}} = (W_m + P_m)/2$ (`FR-003`, `DATA-005`); đặc tả thay thế `formula-33`, cổng $g_m(t)$ ($\kappa=10{,}0, \tau_{\mathrm{SoL}}=0{,}12, \beta_{\mathrm{BGP}}=0{,}45$) với 3 ca biên (`FR-006`); USGS `eventtype=earthquake`, $\mathrm{Mw} \ge 4{,}0$, $\le 20.000$ sự kiện (`DATA-005`); $\mathbf{A}^{\mathrm{SRLG}}$ đối xứng ($=1$ khi chung eo biển, $>0$ khi $\le 25\ \mathrm{km}$, `FR-001`); bổ sung `VSTN`/`TEA`, vùng đệm hành lang và kiểm chéo rDNS trước $\Psi_{\mathrm{SoL}}$ (`DATA-003`). | 100% ca đơn vị lõi `C1`/`C2` đạt khớp công thức đã chốt mà không cần trình duyệt hay mạng. |
| **Ranh giới `external_flags` (`FR-007`)** | Chạy `C7` hai lần trên cùng `SignalWindow` nội bộ: (a) `external_flags = null` và (b) `external_flags` chứa cờ IODA/Radar bất kỳ; đồng thời kiểm tra `SignalWindow` xuất cho kênh `public_demo`. | Đầu ra $(\hat{\mathbf{x}}, \hat{\mathbf{p}}, \mathcal{C}_{0{,}90})$ của mô hình MVP **giống hệt nhau (sai khác `0`)** bất kể `external_flags`; thiếu dữ liệu IODA/Radar ghi `null` + `EXTERNAL_SOURCE_UNAVAILABLE`; kênh `public_demo` bỏ hoàn toàn trường này. |
| **Cô lập `IncidentAnnouncement` (`FR-010`)** | Kiểm tra lược đồ đầu vào của `C7` (`Inference Worker`) và đường truyền từ `C6`/`C9` sang `C7`; kiểm tra quy tắc `label_granularity == "system"` $\Rightarrow$ `confirmed_segment_ids == null`. | `IncidentAnnouncement` lưu ở tệp độc lập, **tuyệt đối không xuất hiện trong chữ ký hàm hay payload truyền vào `C7`**. |

### 4.2. Kiểm chứng hồi quy bộ giải LP và đối sánh đường ngắn nhất (`V-LP`, `ML-012`)

Chạy ngoại tuyến bằng lõi `C8` (Node/HiGHS WASM hoặc Python/HiGHS đối chiếu) theo [MODEL.md](MODEL.md) §4 và [subsea-routing-verification](../.agents/skills/subsea-routing-verification/SKILL.md):

- **Thứ tự giải tuần tự 3 pha bắt buộc (`FR-012`, `Q-03`):**
  1. **Pha 1:** cực tiểu hóa tổng lưu lượng rớt $L^* = \min \sum_{d} \ell_d$.
  2. **Pha 2:** cố định $\sum_d \ell_d \le L^* + \varepsilon_{\mathrm{lex}}$ ($\varepsilon_{\mathrm{lex}} = 10^{-9}$), cực tiểu hóa $u^* = \min u \in [0, 1]$ (đặt $u^* = 0$ nếu $L^* = \sum_d D_d$).
  3. **Pha 3:** cố định thêm $u \le \min(1, u^* + \varepsilon_{\mathrm{lex}})$, cực tiểu hóa tổng độ trễ gia quyền $\sum_d \sum_k \tau_k^{\mathrm{RTT}} f_k$.
- **Hậu kiểm độc lập bắt buộc (không coi trạng thái trả về từ HiGHS là đủ bằng chứng khả thi; bộ dung sai đề xuất `D-03` chưa chạy — xác minh trên fixture phát triển trước khi khóa `T5.3` và nhánh LP của `T8.2`):**
  1. Không âm & biên luồng (`1e-7 Tbps`): $f_k \ge 0$, $0 \le \ell_d \le D_d$ với mọi $k, d$.
  2. Bảo toàn luồng / nhu cầu (`1e-7 Tbps`): $\left\lvert \sum_{k \in \mathcal{K}_d^{\mathrm{avail}}} f_k + \ell_d - D_d \right\rvert \le 10^{-7}\ \mathrm{Tbps}$ với mọi $d$.
  3. Cấm dùng liên kết đã cắt: mọi đường chứa liên kết $\tilde{C}_e(t) = 0$ bị loại khỏi $\mathcal{K}_d^{\mathrm{avail}}$ hoặc có luồng đúng bằng `0`.
  4. Giới hạn dung lượng liên kết (`1e-7 Tbps`) & biên $u$ (`1e-8`): $\sum_d \sum_{k: e \in k} f_k \le \tilde{C}_e(t) \cdot u + 10^{-7}$ với mọi $e$ có $\tilde{C}_e > 0$, và **`0 ≤ u ≤ 1`** (dung sai biên `1e-8`).
  5. Trần SRLG thích nghi (`1e-7 Tbps`): $\le 0{,}65\, D_d$ khi $\lvert G_d^{\mathrm{avail}} \rvert \ge 2$; nâng lên $D_d$ kèm **`srlg_redundancy_lost = true`** khi $\lvert G_d^{\mathrm{avail}} \rvert = 1$; cố định $\ell_d = D_d$ kèm **`disconnected = true`** khi $\lvert G_d^{\mathrm{avail}} \rvert = 0$.
  6. **Kiểm riêng ràng buộc khóa pha:** Kiểm độc lập các ràng buộc khóa giữa 3 pha và **không dùng dung sai hậu kiểm chung để nới $\varepsilon_{\mathrm{lex}} = 10^{-9}$**; xác minh cấu hình dung sai khả thi thực tế (mặc định HiGHS là `1e-7`) ở cả hai backend.
  7. **Ưu tiên tuyệt đối của cắt thủ công (`FR-011`):** Bất kỳ phân đoạn nào trong `manual_cut_segment_ids` đều nhận $\tilde{C}_e = 0$ (`capacity_reason = manual_cut`) **ngay cả khi `auto_cut_policy.enabled == false` hay `apply_inference_discount == false`**.
  8. **Ca đủ dung lượng:** Trong mọi kịch bản đủ dung lượng khả dụng dưới trần SRLG, bắt buộc đạt **`total_dropped_tbps = 0`** và **`MLU ≤ 100%`**.
- **Đối sánh đường ngắn nhất mặc định (`FR-012`, `ARCHITECTURE.md §1.6, §7.2`):** Chạy trên **chính xác cùng `topology_hash`, ma trận nhu cầu, dung lượng hiệu dụng $\tilde{C}_e(t)$, tập đường khả dụng và tập phân đoạn cắt**. Khử đường trùng, so sánh RTT ở độ phân giải cố định `µs`, chia đều tải gán cho các đường hòa ngắn nhất, tăng lưu lượng thực chuyển theo tỷ lệ tải đã gán và cố định tại liên kết vừa chạm trần — **không ưu tiên theo ID, không chuyển sang đường dài hơn, không dồn phần rớt sang đường hòa khác**.

#### Bảng 6 bộ fixture bắt buộc của `V-LP` ([MODEL.md](MODEL.md) §4.4 — *giá trị kỳ vọng, chưa chạy*)

| STT | Fixture kiểm thử (`simulated_assumption`) | Đầu vào kịch bản | Giá trị kỳ vọng bắt buộc *(chưa chạy thực tế)* |
| --- | --- | --- | --- |
| **1** | **Bình thường & đứt đơn đủ tải** | Không cắt hoặc cắt `1` phân đoạn nhưng còn $\ge 2$ hành lang SRLG đủ tải dưới trần `0,65 D_d`. | `total_dropped_tbps = 0`, `mlu_pct ≤ 100%`, `srlg_redundancy_lost = false`, `disconnected = false`, `100%` bảo toàn luồng. |
| **2** | **Kịch bản `82,0 Tbps`** *(mô phỏng ngày `25/08/2026`, `RESEARCH.md §3.6`)* | Tổng cầu `82,0 Tbps` (`32` HK, `28` SG, `16` Tokyo, `6` EU), `VSTN = 8,0 Tbps`; cắt 4 nhánh HK (`SJC2_HK`, `AAE1_HK`, `AAG_S1`, `IA_HK`), còn `40,0 Tbps` trực tiếp HK. | **LP:** `0 Tbps` rớt, **`MLU = 67,4%`**.<br>**Đường ngắn nhất đối chứng:** **`8,0 Tbps` rớt**, **`MLU` thực chuyển `= 100,0%`**, **tải danh nghĩa trước giới hạn `= 120,0%`**. |
| **3** | **Ca A bắt buộc `10 Tbps`** *(còn 1 hành lang `8 Tbps`, `Q-03`)* | $D_d = 10\ \mathrm{Tbps}$; chỉ còn `1` hành lang SRLG khả dụng với $\tilde{C} = 8\ \mathrm{Tbps}$. | Nâng trần SRLG từ `6,5` lên `10 Tbps`, bật **`srlg_redundancy_lost = true`**, chuyển đúng **`8 Tbps`**, rớt đúng **`2 Tbps`**, **`MLU = 100%`**. |
| **4** | **Ca B bắt buộc `10 Tbps`** *(mất kết nối hoàn toàn, `Q-03`)* | $D_d = 10\ \mathrm{Tbps}$; mọi đường qua liên kết $\tilde{C}_e = 0$ ($\mathcal{K}_d^{\mathrm{avail}} = \varnothing$). | Bật **`disconnected = true`**, chuyển **`0 Tbps`**, rớt đúng **`10 Tbps`**, **`MLU = 0%`**, `weighted_rtt_ms = null` (`na_reason = NO_SERVED_FLOW`). |
| **5** | **Ca nghẽn đối chứng `12 & 8` qua `10 Tbps`** (`ARCHITECTURE.md §7.2`) | Hai hướng có đường ngắn nhất gửi `12 Tbps` và `8 Tbps` đi qua cùng một liên kết dung lượng `10 Tbps`. | Tỷ lệ tải gán `12 : 8` ($\delta^* = 0{,}5$): đường ngắn nhất thực chuyển đúng **`6 Tbps` và `4 Tbps`**, rớt đúng **`6 Tbps` và `4 Tbps`**, **`raw_unconstrained_overload_pct = 200%`**, **`mlu_pct = 100%`**. |
| **6** | **Ca biên `zero-demand` & đầu vào sai** | (a) $D_d = 0$; (b) $D_d < 0$, hoặc `manual_cut_segment_ids` chứa ID không có trong `GraphSnapshot`, hoặc `capacity_tbps < 0`, hoặc `srlg_cap_ratio` ngoài `(0, 1]`. | (a) `total_dropped_tbps = 0`, `mlu_pct = 0%`, `weighted_rtt_ms = null` (`NO_SERVED_FLOW`);<br>(b) trả **`status = invalid_input`** kèm lỗi có kiểu tường minh — **cấm tự kẹp về `0` hay thay dung lượng mặc định**. |

- **Các trường bắt buộc xuất trong mọi báo cáo `V-LP`:** `total_dropped_tbps` (`Tbps`), `mlu_pct` (`%` thực chuyển trong `[0, 100]`), `raw_unconstrained_overload_pct` (`%` tải danh nghĩa của đường ngắn nhất), `weighted_rtt_ms` (`ms` hoặc `null` + `NO_SERVED_FLOW`), `status` của solver, sai số / vi phạm lớn nhất của hậu kiểm bảo toàn luồng – năng lực – SRLG, `topology_hash`, và kiểm tra vòng đời giải phóng bộ nhớ `model.dispose()` (`ADR-03`).

### 4.3. Kiểm chứng cổng phát hành, giấy phép, bảo mật và clean clone (`V-RELEASE`)

| Hạng mục kiểm chứng (`V-RELEASE`) | Cách kiểm tra & Fixture âm tính bắt buộc | Tiêu chí đạt |
| --- | --- | --- |
| **Kiểm `ReleaseManifest` từng tệp & `O-01` (`DATA-008`, `DATA-009`)** | Xác thực 100% mục trong `files[]` mang đủ 6 yếu tố kiểm toán (`source_name`, `source_version_or_fetched_at_utc`, `transformations_summary`, `license_id` + `license_terms_ref`, `attribution_text` + `modifications_notice`, `redistribution_approved = true` + `approved_by` + `approved_at_utc`). Fixture âm tính: thiếu bất kỳ trường duyệt nào hoặc `redistribution_approved = false` $\Rightarrow$ `C5` phải chặn đóng gói. | `0` tệp thiếu duyệt hoặc thiếu ghi công lọt vào kênh `public_demo`. |
| **Cấm gộp giấy phép & chặn nguồn hạn chế (`DATA-001`, `DATA-007`, `DATA-008`)** | Quét gói đầu ra của `C5` bằng fixture âm tính chứa: (a) tệp GeoParquet gộp $\ge 2$ nguồn khác giấy phép; (b) bất kỳ dẫn xuất hạn chế chưa vượt `O-01` gồm `GFW 4Wings`, `RIPE RIS`, `RIPE Atlas`, `CAIDA`, `PeeringDB`, `IODA`, **`Cloudflare Radar`** *(dữ liệu API là `CC BY-NC 4.0`, không gắn nhầm `CC BY 4.0`)* và gói sự cố lịch sử; (c) nguồn bị cấm `SRC-BANNED` (Kaggle synthetic / Hugging Face gated); (d) trường `external_flags` trong `public_demo`; (e) dẫn xuất Submarine Cable Map thiếu danh sách sửa đổi (`VSTN`/`ALC`/`VTS`) hoặc mặc định portfolio là phi thương mại. | `C5` chặn 100% ca vi phạm; kênh `public_demo` mặc định chỉ chứa **fixture tổng hợp độc lập**, mô hình FP32 đã duyệt và **trình tải từ nguồn gốc** (khi điều khoản cho phép), đạt trần nén `< 15 MB` (`essential_boot`) và `< 65 MB` (tổng). |
| **Toàn vẹn `SHA-256`, tương thích phiên bản & rollback (`NFR-005`)** | Đối chiếu lại `SHA-256` cho 100% tệp trong thư mục phát hành bất biến dự kiến (`releases/<release_id>/` theo `ARCHITECTURE.md §5.5`) và tệp chỉ mục `current_release.json`; kiểm tra trường `app_min_version`, `app_max_compatible_version` và quy trình trỏ ngược `current_release.json` về `previous_compatible_release_id`. Fixture âm tính: đổi `1 byte` của một tệp tĩnh hoặc nâng `MAJOR` $\Rightarrow$ `C5`/`C6` phải từ chối. | `100%` tệp khớp `SHA-256`; quay lui thành công không cần dựng lại bản cũ. |
| **Quét bí mật (`NFR-007`)** | Quét tự động chuỗi bí mật/token (GFW, Cloudflare Radar Bearer Token, khóa API) trên toàn bộ repo, gói bản dựng tĩnh dự kiến, nhật ký chạy đường ống và lưu lượng mạng trình duyệt; kiểm tra tệp cấu hình mẫu chỉ chứa giá trị giả (`REPLACE_ME_LOCAL_ONLY`). | `0` bí mật thật phát hiện ở mọi vị trí. |
| **Truy cập chỉ đọc & hạn mức nguồn (`DATA-004`, `NFR-008`)** | Rà soát tĩnh mã nguồn và kiểm tra động mạng trình duyệt: không có yêu cầu từ trình duyệt tới PeeringDB/GFW/Radar; RIPE Atlas chỉ dùng *built-in*/*anchoring*; USGS có `eventtype=earthquake` và trần `≤ 20.000`; không có mô-đun đo chủ động trái phép hay giao thức cấu hình mạng (`BGP speaker`, `NETCONF`, `gNMI`, `SSH`). | `100%` kết nối tuân thủ phạm vi chỉ đọc và hạn mức đã khai báo. |
| **Dựng sạch từ clean clone (`NFR-010`, `ADR-06`)** | Sao chép kho mã vào thư mục trống sạch, cài phụ thuộc từ các tệp lockfile đã khóa (dự kiến cho Python và JS/TS), chạy toàn bộ bộ kiểm thử tự động và dựng bản tĩnh — **không yêu cầu dựng máy chủ DVC hay MLflow**. | Hoàn tất không lỗi và không cần thao tác thủ công ngoài hướng dẫn tái lập. |

### 4.4. Kiểm chứng đường dự phòng `WASM` và phục hồi lỗi (`V-FALLBACK`, `NFR-004`, `NFR-005`)

- **Kiểm chứng tương đương số học và quyết định ONNX FP32 (`MODEL.md §6.1`, `ADR-02`):**
  1. Chuẩn bị bộ kịch bản kiểm thử gồm đủ 5 nhóm: (a) đầy đủ kênh hợp lệ, (b) một phần kênh `NA` (`degraded_partial_na`), (c) toàn bộ kênh `NA` (`insufficient_observations`), (d) chế độ chuyển giao `ADR-05` (`zero_delta_r_transfer`), và (e) **các fixture có xác suất $\hat{p}_e$ nằm sát ngưỡng Conformal ($1 - \hat{q}_{0{,}90}$) và sát ngưỡng cắt tự động ($\tau_{\mathrm{cut}}$)**.
  2. Chạy cùng bộ kịch bản trên 3 môi trường: **(1) Python/PyTorch tham chiếu ngoại tuyến**, **(2) ONNX Runtime Web `WebGPU`**, và **(3) ONNX Runtime Web `WASM` dự phòng** (thực hiện trong phiên ép tắt WebGPU, trong khi bản đồ 2D `C9` vẫn chạy trên WebGL2 dùng chung).
  3. **Tiêu chí đạt cho TỪNG backend (`WebGPU` và `WASM`) độc lập so với tham chiếu ngoại tuyến:**
     - **`max_abs_diff ≤ 1e-4`** cho **cả $\hat{\mathbf{x}}$ và $\hat{\mathbf{p}}$** trên mọi phân đoạn khả kiến.
     - **Khớp chính xác 100%** trên: mặt nạ `m_valid`, cờ và mã `na_reason`, mọi trường trạng thái (`status`, `observability_status`, `inference_mode`, `conformal_calibration_status`), tập Conformal $\mathcal{C}_{0{,}90}$, và quyết định cắt theo $\tau_{\mathrm{cut}}$.
     - **Quy tắc bác bỏ nghiêm ngặt:** Nếu sai số số học đạt `≤ 1e-4` nhưng quyết định thành viên tập $\mathcal{C}_{0{,}90}$ hoặc quyết định cắt theo $\tau_{\mathrm{cut}}$ khác tham chiếu ngoại tuyến $\Rightarrow$ kết luận **`KHÔNG ĐẠT` (`FAIL`)**.
- **Đối chiếu nối tiếp sang bộ giải LP (`C7` $\to$ `C8`, quy tắc `D-03` — dung sai đề xuất chưa chạy, xác minh trên fixture phát triển trước khi khóa):** Chạy chuỗi nối `C7` (`WebGPU` và `WASM`) $\to$ `C8` (HiGHS WASM) trên cùng đầu vào với chuỗi ngoại tuyến; **không buộc từng $f_k$ giống nhau khi có nghiệm đồng tối ưu**. Hai backend phải độc lập vượt hậu kiểm, khớp trạng thái, `capacity_reason`, cờ rời rạc (`srlg_redundancy_lost`, `disconnected`), khớp chính xác `weighted_rtt_ms = null` và `NO_SERVED_FLOW` khi không có lưu lượng phục vụ, rồi đối chiếu 3 mục tiêu theo dung sai đề xuất: tổng tải rớt `1e-6 Tbps`, $u^*$ `1e-7` không thứ nguyên, và tổng $\sum_k \tau_k^{\mathrm{RTT}} f_k$ `1e-4 Tbps·ms`.
- **Kiểm thử phục hồi sự cố phần cứng và mạng (`NFR-004`, `NFR-005`):**
  1. **Giả lập `device.lost` (WebGPU tại `C7`):** Xác minh ứng dụng tự tái khởi tạo phiên WebGPU hoặc hạ cấp sang `WASM` nếu lỗi lặp lại, gắn nhãn chế độ dự phòng và không làm đứng giao diện.
  2. **Giả lập `webglcontextlost` / `webglcontextrestored` (bản đồ 2D tại `C9`):** Xác minh bản đồ phục hồi và giữ nguyên mốc phát lại `5 phút` lẫn danh sách phân đoạn đang cắt thủ công.
  3. **Giả lập ngắt mạng / gói hỏng `SHA-256` / gói lệch `MAJOR` (`C6`):** Xác minh `C6` không ghi đè khe lưu đệm `last_known_good`, tiếp tục chạy Luồng 2 và Luồng 3 bình thường trên bản đã xác minh trước đó và hiển thị rõ thông báo nguồn/gói lỗi (không tự điền dữ liệu giả).

### 4.5. Kiểm chứng hiệu năng máy chuẩn M1, vòng đời bộ nhớ và giao diện (`V-PERF-UX`, `NFR-003`, `DESIGN.md`)

#### Cấu hình máy chuẩn và quy trình đo lặp (`ARCHITECTURE.md §8.2`, `Q-04`)

- **Máy chuẩn bắt buộc:** **`MacBook Pro M1, RAM 8 GB, macOS 27.0, Chrome 157.0.8103.79`**, cắm nguồn điện, bật tăng tốc phần cứng, khung nhìn `1280×800`, màn hình `60 Hz`.
- **Tính đồng nhất workload:** Đo trên **cùng bản dựng tích hợp và workload phát triển** cho nghiệm thu `NFR-003` và benchmark khóa `ADR-01`–`ADR-04`; **tuyệt đối không dùng tập `200 test` khóa để chọn runtime**.
- **Trình tự đo chuẩn hóa:** **`3` lượt tải lạnh** *(xóa sạch cache trình duyệt)* $\rightarrow$ **`5` lượt làm nóng** $\rightarrow$ **`50` thao tác** *(chuyển bước 5 phút Luồng 2 / cắt–khôi phục cáp & giải LP Luồng 3)* $\rightarrow$ **`30 giây` tương tác bản đồ liên tục** *(kéo, thu phóng, chọn phân đoạn trên bản dựng tích hợp cả bản đồ 2D lẫn suy luận)*.

#### Bảng 7 cổng định lượng `NFR-003` (`Q-04`)

| Chỉ số (`NFR-003`) | Cổng đạt bắt buộc | Phương pháp đo chuẩn hóa | Lưu ý bắt buộc |
| --- | --- | --- | --- |
| **1. Thời gian giải LP (tổng 3 pha)** | **`p95 < 15 ms`** | Đo `performance.now()` bên trong `C8` cho **tổng cả ba pha** trên `50` thao tác sau làm nóng. | **Bắt buộc báo tách riêng** thời gian giải LP thuần 3 pha trong `C8` và tổng thời gian khứ hồi Worker (`round-trip`). |
| **2. Độ trễ thao tác $\rightarrow$ hiển thị** | **`p95 ≤ 100 ms`** | Đo từ dấu thời gian sự kiện người dùng (`pointerdown`/`keydown`) đến `requestAnimationFrame` sau khi cập nhật đồng bộ bản đồ 2D + bảng số liệu trên `50` thao tác. | Bao gồm cả xử lý hủy kết quả cũ (`epoch`). |
| **3. Tốc độ khung hình tương tác** | **Trung vị `≥ 55 FPS`** | Ghi khoảng thời gian khung hình (`rAF` delta / Chrome DevTools Performance trace) suốt **`30 giây`** tương tác bản đồ liên tục ở `60 Hz`. | Đo trên **bản dựng tích hợp cả bản đồ 2D lẫn suy luận**. |
| **4. Ngân sách bộ nhớ GPU logic cấp phát** | **`< 150 MB`** *(mỗi buffer **`≤ 256 MB`**)* | Bộ đếm cấp phát buffer của ứng dụng cộng tổng bộ nhớ GPU logic đang sống của **cả bản đồ 2D WebGL2 (`C9`) lẫn suy luận WebGPU (`C7`)**. | **Không gọi là VRAM vật lý rời** trên kiến trúc bộ nhớ hợp nhất Apple M1 (`Q-04`). |
| **5. Bộ nhớ tuyến tính `WASM`** | **`≤ 210 MB`** *(và `0` rò rỉ)* | Cộng đỉnh `WebAssembly.Memory.buffer.byteLength` của các Worker WASM (`C6`, `C8`, và `C7` ở chế độ WASM) qua toàn bộ `50` lượt. | Kiểm tra xu hướng bộ nhớ không tăng đơn điệu qua các lượt (xác nhận vòng đời `model.dispose()` của `NFR-002`, `ADR-03`). |
| **6. Gói tải khởi tạo thiết yếu (nén)** | **`< 15 MB`** | Tổng `transferSize` nén Brotli/ZSTD của nhóm `essential_boot` trong Chrome DevTools Network qua `3` lượt tải lạnh. | Phải khớp với kiểm toán tĩnh tại `C5`. |
| **7. Tổng tài nguyên tải (nén)** | **`< 65 MB`** | Tổng `transferSize` nén Brotli/ZSTD của toàn bộ ứng dụng + gói dữ liệu chuẩn qua `3` lượt tải lạnh. | Phải khớp với kiểm toán tĩnh tại `C5`. |

#### Kiểm tra vận hành tĩnh, khả năng tiếp cận và tuân thủ `DESIGN.md`

- **Vận hành tĩnh & đồng bộ Luồng 2/3 (`NFR-001`, `NFR-002`, `FR-014`):** Phục vụ từ máy chủ tệp tĩnh thuần túy không có backend API; xác minh Luồng 2 (phát lại 5 phút tách 3 lớp) và Luồng 3 (cắt/khôi phục cáp) cập nhật đồng bộ bản đồ 2D với bảng số liệu, luồng chính không bị khóa cứng khi chạy 3 Web Workers (`C6`, `C7`, `C8`), cơ chế hủy `replay_epoch`/`sim_epoch` hoạt động đúng, và `model.dispose()` giải phóng sạch bộ nhớ sau chu kỳ giải lại.
- **Khả năng tiếp cận (`NFR-006`):** Thao tác 100% Luồng 2 và Luồng 3 chỉ bằng bàn phím (bao gồm chọn, cắt và khôi phục phân đoạn ngay trong bảng danh sách), hiển thị rõ `focus-visible`, kiểm tra độ tương phản nền phẳng, kiểm tra dưới **bộ lọc thang xám (grayscale)** bảo đảm phân biệt đủ mọi trạng thái qua kiểu nét + nhãn/biểu tượng, thông báo thay đổi trạng thái bằng `aria-live="polite"`, và kiểm tra chế độ **`prefers-reduced-motion: reduce`** (tắt/giảm chuyển động nhưng giữ đủ 100% thông tin và thao tác).
- **Checklist bắt buộc theo [DESIGN.md](DESIGN.md) (`UX-001`–`UX-007`, `ADR-04`):**
  1. **Bản đồ 2D phẳng:** Dùng MapLibre + Deck.gl chia sẻ ngữ cảnh WebGL2; **không có Three.js, không có địa cầu 3D và không có hệ 100.000 hạt**.
  2. **0 gradient:** Kiểm tra tĩnh CSS/SVG/Canvas/shader xác nhận `0` khai báo `linear-gradient`, `radial-gradient`, `conic-gradient`.
  3. **Bố cục mở & 0 lồng card:** Kiểm tra cây DOM xác nhận các chỉ số (`MLU %`, `Tbps` rớt, `ms`, độ phủ) đặt trong dải chỉ số phẳng/bảng có đường phân cách, không bọc từng chỉ số đơn lẻ trong card và `0` phần tử card lồng trong card.
  4. **Nội dung chữ tối giản:** Đủ đơn vị đo, tử số/mẫu số/CI, cảnh báo giới hạn suy luận & cảnh báo đọc đồng thời `MLU` với `Tbps` rớt; **`0` slogan, `0` lời trích dẫn trang trí, `0` dòng subtext lặp lại tiêu đề/giá trị**.
  5. **Thư viện bắt buộc:** Toàn bộ component điều khiển chuẩn sử dụng **`coss ui`** (không tự viết trùng); mọi chuyển động trạng thái sử dụng **`Animate UI`** (bỏ chuyển động nếu không cần thiết hoặc chỉ để trang trí).

---

## 5. Ma trận nghiệm thu toàn hệ thống

### 5.1. Quy ước trạng thái nghiệm thu

| Trạng thái | Ý nghĩa áp dụng |
| --- | --- |
| **`CHƯA CHẠY`** | Yêu cầu thuộc phạm vi kiểm chứng MVP nhưng chưa có mã nguồn, dữ liệu hoặc bằng chứng thực thi trong kho mã. |
| **`BỊ CHẶN`** | Không thể thực thi hoặc kết luận nghiệm thu do điều kiện tiên quyết chưa hoàn tất, như khóa dữ liệu, phê duyệt pháp lý hoặc thiếu bằng chứng để xác minh chốt chống rò rỉ. Bắt buộc ghi rõ điều kiện gỡ chặn; không công bố điểm nếu chưa xác minh được `V-LEAKAGE`. |
| **`ĐẠT`** | Đã chạy đủ kịch bản kiểm chứng trên bản khóa và lưu đủ bằng chứng xác minh đạt 100% tiêu chí. **Hiện có `0` yêu cầu ở trạng thái `ĐẠT`.** |
| **`KHÔNG ĐẠT`** | Đã chạy kiểm chứng và có bằng chứng xác nhận vi phạm ít nhất một tiêu chí bắt buộc, kể cả vi phạm chống rò rỉ. |
| **`KHÔNG ÁP DỤNG`** | Nằm ngoài đường chạy nghiệm thu MVP hiện tại (thuộc GĐ2 hoặc GĐ3 theo `Q-01`, `Q-02`, `Q-07`); bắt buộc nêu rõ lý do và giai đoạn kích hoạt. |

### 5.2. Ma trận nghiệm thu 54/54 yêu cầu (`REQUIREMENTS.md`)

Chỉ sử dụng đúng 6 nhóm kiểm chứng của [ARCHITECTURE.md](ARCHITECTURE.md) §10.1: **`V-CONTRACT`**, **`V-LEAKAGE`**, **`V-LP`**, **`V-RELEASE`**, **`V-FALLBACK`**, **`V-PERF-UX`**. Mọi tên tệp, nhật ký và báo cáo nêu trong cột **Bằng chứng đầu ra** là đặc tả artifact dự kiến khi triển khai, chưa tồn tại trong kho mã hiện tại.

| ID yêu cầu | Nhóm `V-*` | Pha / Điều kiện chạy | Đầu vào / Fixture | Phương pháp kiểm chứng | Tiêu chí đạt | Bằng chứng đầu ra | Trạng thái hiện tại |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `FR-001` | `V-CONTRACT` | MVP (`Must`) | Danh mục cáp + `VSTN`/`TEA` + CLS + eo biển nghẽn | Kiểm thử đơn vị ngoại tuyến `C2` | $\mathcal{G}_{\mathrm{phys}}$ hợp lệ; $\mathbf{A}^{\mathrm{SRLG}}$ đối xứng $\in [0,1]$, chung eo biển $=1$, ven bờ $\le 25\ \mathrm{km} > 0$ | Báo cáo unit test `C2` + `GraphSnapshot` mẫu | **`CHƯA CHẠY`** |
| `FR-002` | `V-CONTRACT`, `V-LEAKAGE` | MVP (`Must`) | Kênh ngoài RFS, hợp cơ sở rỗng, $b_m < 2L_e/204$, thiếu $b_m$/PoP | Kiểm thử đơn vị `C2` & kiểm chứng `V-LEAKAGE` | Ngoài RFS & trượt SoL $\Rightarrow 0$; hợp rỗng $\Rightarrow J=0$; thiếu $b_m$/PoP $\Rightarrow$ `NA`; có chú giải trọng số | Log unit test `R_t` + xác thực `routing_weight_disclaimer` | **`CHƯA CHẠY`** |
| `FR-003` | `V-CONTRACT` | MVP (`Must`) | Luồng MRT 5 phút có bản tin trùng, reset phiên và mẫu số `= 0` | Kiểm thử đơn vị ngoại tuyến `C1` | Loại trùng & reset phiên; trả $(W_m+P_m)/2 \in [0,1]$; mẫu số `0`/gap trả `NA` | Log unit test `C1` + báo cáo QC (`duplicate_count`, `session_reset_filtered_count`) | **`CHƯA CHẠY`** |
| `FR-004` | `V-CONTRACT`, `V-PERF-UX` | GĐ2 (`Should`, `Q-01` — ngoài MVP) | Luồng RIPE RIS Live có `?client=` & lọc `ris_subscribe` | Kiểm thử tích hợp GĐ2 chạy liên tục $\ge 5$ phút | Đệm `250 ms` ổn định, không `ris_error`, xuất đúng $\tilde{y}_m^{\mathrm{BGP}}$ | Log chạy WebSocket `E1` (GĐ2) | **`KHÔNG ÁP DỤNG`** *(Luồng 1 ngoài MVP theo `Q-01`)* |
| `FR-005` | `V-FALLBACK` | GĐ2 (`Should`, `Đề xuất` — ngoài MVP) | Phiên Luồng 1 giả lập mất mạng / `ris_error` | Tiêm lỗi kết nối trên trình duyệt (GĐ2) | Hiện trạng thái mất kết nối + mốc cuối, gắn `NA`, giữ cửa sổ cuối, chuyển được Luồng 2/3 | Bản ghi E2E tiêm lỗi `E1` (GĐ2) | **`KHÔNG ÁP DỤNG`** *(Luồng 1 ngoài MVP theo `Q-01`)* |
| `FR-006` | `V-CONTRACT` | MVP (`Must`) | 3 ca biên RTT: `< 2` mẫu/$b_m \le 0$; nhiễu BGP thuần; đứt cáp che khuất BGP | Kiểm thử đơn vị ngoại tuyến `C1` | Ca (a) trả `NA` & không dùng $\tau > t$; ca (b) $g_m$ nén thành phần BGP; ca (c) giữ $0{,}55\,\tilde{y}_m^{\mathrm{RTT}} > 0$ | Log unit test `formula-33` & cổng $g_m(t)$ | **`CHƯA CHẠY`** |
| `FR-007` | `V-CONTRACT`, `V-RELEASE` | MVP nội bộ (metadata tùy chọn) / GĐ2 đầy đủ (`Should`, `O-01`) | `SignalWindow` nội bộ có/không `external_flags`; gói xuất `public_demo` | Kiểm thử bất biến suy luận MVP (`V-CONTRACT`) & cổng `C5` (`V-RELEASE`) | MVP: `external_flags` không đổi đầu ra `C7`, thiếu là `null`, bị bỏ hoàn toàn ở `public_demo` *(tích hợp loại trừ IODA/Radar thuộc GĐ2)* | Log so sánh bất biến `C7` + báo cáo quét `public_demo` | **`CHƯA CHẠY`** *(ranh giới MVP; tích hợp đầy đủ GĐ2 là `KHÔNG ÁP DỤNG` chờ `O-01`)* |
| `FR-008` | `V-CONTRACT`, `V-LEAKAGE` | MVP hỗ trợ `uniform_w1` / GĐ2 đầy đủ (`Should`, `O-01`) | (a) Cấu hình MVP `uniform_w1`; (b) vector $h_e(t) \in (0,1)$ hợp lệ | Kiểm thử đơn vị `C4`/`C7` cho cả hai chế độ | (a) Chạy hợp lệ với $w_e = 1$; (b) tính đúng công thức nghịch đảo chuẩn hóa $\epsilon_0 = 0{,}05$ *(nguồn GFW bật ở GĐ2)* | Log unit test trọng số `w(t)` và cấu hình `A2`/`Full` | **`CHƯA CHẠY`** *(chế độ `w_e=1` & benchmark; nguồn GFW GĐ2 là `KHÔNG ÁP DỤNG` chờ `O-01`)* |
| `FR-009` | `V-CONTRACT`, `V-FALLBACK` | MVP (`Must`) | `SignalWindow` có một phần hoặc toàn bộ kênh `NA` | Kiểm thử lõi `C7` + kiểm tra giao diện Lớp 2 | Loại kênh `NA` khỏi phần dư qua `m_valid`, xuất $\hat{p}_e \in [0,1]$, $\mathcal{C}_{0{,}90}$ (khi đã hiệu chuẩn), tỷ lệ hợp lệ, `unobservable_na` & chú giải giới hạn | Log unit test `m_valid` + ảnh chụp/DOM Lớp 2 | **`CHƯA CHẠY`** |
| `FR-010` | `V-CONTRACT`, `V-LEAKAGE`, `V-PERF-UX` | MVP (`Must`) — phụ thuộc khóa danh mục `ML-011` | Gói $\ge 3$ sự cố lịch sử độc lập bước 5 phút + fixture phát lại độc lập | Kiểm thử cô lập thông cáo (`V-CONTRACT`), chống rò rỉ (`V-LEAKAGE`), E2E giao diện 3 lớp (`V-PERF-UX`) | Phát lại 5 phút đúng tô-pô đóng băng cho $\ge 3$ sự cố lịch sử xác minh, tách bạch 3 lớp đo / suy luận / thông cáo | Bản ghi E2E Luồng 2 + kiểm chứng `ML-011` | **`BỊ CHẶN`** *(khung phát lại trên fixture: `CHƯA CHẠY`; nghiệm thu $\ge 3$ ca lịch sử thật bị chặn bởi `ML-011`, công bố công khai bị chặn bởi `O-01`)* |
| `FR-011` | `V-LP`, `V-PERF-UX` | MVP (`Must`) | Kịch bản Luồng 3 với `auto_cut_policy.enabled = false` và cắt thủ công $1..k$ phân đoạn | Kiểm thử lõi `C8` (`V-LP`) + E2E nút cắt/khôi phục (`V-PERF-UX`) | Cắt thủ công đặt $\tilde{C}_e = 0$ ngay lập tức bất kể $\tau_{\mathrm{cut}}$, giải lại LP và khôi phục nguyên trạng | Log unit test ưu tiên `manual_cut` + bản ghi E2E Luồng 3 | **`CHƯA CHẠY`** |
| `FR-012` | `V-LP`, `V-PERF-UX` | MVP (`Must`) | 6 bộ fixture LP (Mục 4.2), gồm `82,0 Tbps`, hai ca `10 Tbps`, `12 & 8 / 10` | Hậu kiểm độc lập 3 pha LP + đối sánh đường ngắn nhất (`V-LP`) + kiểm tra bảng so sánh UI | Đạt 100% ràng buộc, khớp 6 fixture, hiển thị đồng thời `MLU (%)`, rớt (`Tbps`), `raw_unconstrained_overload_pct` và `ms` | Báo cáo hồi quy `V-LP` + ảnh chụp/DOM bảng đối sánh | **`CHƯA CHẠY`** |
| `FR-013` | `V-LP`, `V-PERF-UX` | GĐ2 (`Could`, `Q-01`, `O-03` — ngoài MVP) | Tô-pô quy hoạch 2030 (`ALC`, `VTS`, `MViSTA`, đất liền thứ 2) | Kiểm thử bật/tắt tuyến quy hoạch & tính chuẩn 1+2, IRI (GĐ2) | Cập nhật đúng SRLG, LP 3 pha, chuẩn 1+2 và IRI | Báo cáo kiểm thử `E2` (GĐ2) | **`KHÔNG ÁP DỤNG`** *(Luồng 4 ngoài MVP theo `Q-01` và chờ `O-03`)* |
| `FR-014` | `V-PERF-UX` | MVP (`Must`) | Luồng 2 (đổi bước 5 phút) và Luồng 3 (cắt/khôi phục cáp) trên trình duyệt | Kiểm thử E2E đồng bộ bản đồ 2D MapLibre + Deck.gl và bảng số liệu | 100% trạng thái/màu/nhãn trên bản đồ khớp số liệu bảng; hủy đúng kết quả `epoch` cũ | Bản ghi E2E + log kiểm tra `epoch` | **`CHƯA CHẠY`** |
| `DATA-001` | `V-RELEASE` | MVP (`Must`) | Cấu hình thu thập `C1` và bảng nguồn đầu vào | Kiểm toán truy vết nguồn & quét danh sách nguồn bị cấm `SRC-BANNED` | 100% dữ liệu gốc truy vết nguồn mở sơ cấp; `0` nguồn synthetic cộng đồng/gated | Báo cáo kiểm toán nguồn `V-RELEASE` | **`CHƯA CHẠY`** |
| `DATA-002` | `V-CONTRACT`, `V-PERF-UX` | MVP (`Must`) | Mọi tệp hợp đồng, báo cáo đánh giá và màn hình Luồng 2/3 | Xác thực lược đồ `data_provenance_class` + rà soát nhãn UI | Gắn đúng 1 trong 4 nhãn bản chất dữ liệu trên 100% gói, báo cáo và màn hình | Log xác thực lược đồ + checklist rà soát giao diện | **`CHƯA CHẠY`** |
| `DATA-003` | `V-CONTRACT` | MVP (`Must`) | Hình học cáp sơ đồ hóa, tuyến `VSTN`/`TEA` và kênh có nhãn rDNS | Kiểm thử đơn vị tiền xử lý không gian & cổng $\Psi_{\mathrm{SoL}}$ | Trích xuất theo vùng đệm hành lang; bổ sung `VSTN`/`TEA`; không loại nhầm kênh đất liền hợp lệ | Log unit test không gian & rDNS trong `C1`/`C2` | **`CHƯA CHẠY`** |
| `DATA-004` | `V-RELEASE` | MVP (`Must`) | Mã nguồn `C1`, bản dựng tĩnh trình duyệt và log mạng trình duyệt | Kiểm tra tĩnh giới hạn tốc độ & bắt gói mạng khi chạy bản tĩnh | `0` truy vấn trình duyệt tới PeeringDB/GFW/Radar; chỉ dùng Atlas *built-in*/*anchoring*; đúng hạn mức từng nguồn | Báo cáo rà soát tĩnh + HAR/Network log trình duyệt | **`CHƯA CHẠY`** |
| `DATA-005` | `V-CONTRACT` | MVP (`Must`) | Tệp MRT chứa Class A/B, bản tin trùng, reset phiên; phản hồi USGS có sự kiện phi động đất | Kiểm thử đơn vị ngoại tuyến `C1` | Loại 100% Class A/B, trùng, reset phiên và sự kiện `eventtype != earthquake`; không truy vấn nào `> 20.000` sự kiện | Log unit test `C1` + bộ đếm lọc QC | **`CHƯA CHẠY`** |
| `DATA-006` | `V-CONTRACT` | MVP (`Must`) | Fixture thiếu $b_m$/PoP, `< 2` mẫu RTT, mẫu số BGP `0`, sai đơn vị/múi giờ | Kiểm thử xác thực lược đồ và cờ `NA` trên `C1`/`C2`/`C5`/`C6` | Chặn 100% bản ghi sai lược đồ/đơn vị/`UTC`; gắn `null` + `na_reason` đúng tập đóng, không thay bằng `0` | Báo cáo chạy bộ fixture âm tính `V-CONTRACT` | **`CHƯA CHẠY`** |
| `DATA-007` | `V-RELEASE` | MVP (`Must`) | Thư mục phát hành dự kiến và bảng khai báo giấy phép | Kiểm tra tự động tách nguồn/giấy phép tại `C5` | `0` tệp GeoParquet gộp đa giấy phép; Radar API không gắn `CC BY 4.0`; `SRC-SCM` đủ ghi công + danh sách sửa đổi + `CC BY-NC-SA 4.0`/`TeleGeography 3.0` | Báo cáo kiểm toán giấy phép `V-RELEASE` | **`CHƯA CHẠY`** |
| `DATA-008` | `V-RELEASE` | MVP (`Must`) | Bộ đóng gói `public_demo` có/không tệp thiếu duyệt hoặc nguồn hạn chế | Kiểm thử cổng chặn `C5` với các ca âm tính | Chặn đóng gói 100% tệp thiếu duyệt hoặc thuộc nhóm hạn chế (`4Wings`, `RIPE`, `CAIDA`, `PeeringDB`, `IODA`, `Radar`, lịch sử); mặc định chỉ có fixture độc lập + trình tải nguồn gốc | Log chạy cổng chặn `V-RELEASE` + `release_manifest.json` | **`CHƯA CHẠY`** |
| `DATA-009` | `V-RELEASE` | Cổng pháp lý `O-01` (`Should`, `Cần xác nhận`) | Đề xuất mở công khai tệp dẫn xuất từ nguồn hạn chế | Kiểm tra trích dẫn điều khoản tái phân phối & chữ ký người phụ trách trong `manifest` | Chỉ chuyển sang `public_demo` khi có căn cứ điều khoản hợp lệ và chữ ký duyệt của người phụ trách | Hồ sơ phê duyệt `O-01` + mục `files[]` đã ký | **`BỊ CHẶN`** *(chưa cần cho mặc định MVP; bị chặn mở công khai bởi cổng pháp lý `O-01`)* |
| `ML-001` | `V-LEAKAGE` | MVP (`Must`) | Chuỗi lịch sử + danh mục $(t_0, t_{\mathrm{repair}})$ và bộ bán tổng hợp `C3` | Kiểm thử tự động Chốt 1 (ca dương + ca âm Mục 2.3) | Đúng 3 khung `UTC`, `0` mẫu âm trong `±72 giờ`, `0` trùng `incident_group_id`, giữ nguyên `200 cal + 200 test` | Nhật ký `V-LEAKAGE` Chốt 1 (`violations = 0`) + `split_manifest` | **`CHƯA CHẠY`** |
| `ML-002` | `V-LEAKAGE` | GĐ2 (`Should`, `E3` — ngoài đường chạy LP-GISTA MVP) | Tập phân đoạn 4 lưu vực và lược đồ $\mathbf{z}_e$ | Kiểm thử LOBO 4 lượt & kiểm lược đồ $\mathbf{z}_e$ | `0` giao cắt `segment_id` trong từng lượt LOBO; `0` cột kinh/vĩ độ thô trong $\mathbf{z}_e$ | Nhật ký `V-LEAKAGE` Chốt 2 (GĐ2) | **`KHÔNG ÁP DỤNG`** *(MVP không huấn luyện bộ mã hóa tiên nghiệm `E3`; logic chia LOBO có thể kiểm thử đơn vị sớm)* |
| `ML-003` | `V-LEAKAGE` | MVP kiểm logic (`Must`) / GĐ2 bật nguồn GFW (`O-01`) | Fixture AIS/SAR chứa tàu trong cửa sổ trễ và tàu sửa cáp sau $t_0$ | Kiểm thử đơn vị tự động Chốt 3 (ca dương + ca âm Mục 2.3) | `100%` AIS $\le \min(t,t_0)-96\text{h}$, `100%` SAR $\le \min(t,t_0)-120\text{h}$, `0` tàu sửa cáp còn lại | Nhật ký `V-LEAKAGE` Chốt 3 (`violations = 0`) | **`CHƯA CHẠY`** *(kiểm logic MVP; luồng GFW thật thuộc GĐ2 chờ `O-01`)* |
| `ML-004` | `V-LEAKAGE` | MVP (`Must`) | Hai mốc hồi quy `2023-02` và `2026-08-25` + bản chụp có/sai cutoff | Kiểm thử tự động Chốt 4 (Mục 2.3) | `2023-02`: SMW-3 bật, ADC/SJC2/VSTN `= 0`; `2026-08-25`: ngược lại; 100% `source_cutoffs_utc < effective_at_utc` | Nhật ký `V-LEAKAGE` Chốt 4 (`violations = 0`) | **`CHƯA CHẠY`** |
| `ML-005` | `V-LEAKAGE` | MVP (`Must`) | Môi trường `train_and_calibrate` đã tháo tệp `D_test` khỏi đĩa | Kiểm thử cô lập tiến trình + kiểm toán `run_provenance.json` | Huấn luyện/đóng băng $b_m$/hiệu chuẩn hoàn tất không cần `D_test`; mở `D_test` đúng `1` lần chỉ đọc; không dùng `D_test` chọn runtime | Log chạy không có `D_test` + `locked_test_access_count` | **`CHƯA CHẠY`** |
| `ML-006` | `V-LEAKAGE` | MVP (`Must`) | `val_tau` (`400`) + 3 bộ fixture biên (hòa recall; mọi ngưỡng `≥ 5%`; thiếu nhãn đứt hoàn toàn) | Kiểm thử đơn vị bộ chọn $\tau_{\mathrm{cut}}$ độc quyền trên `val_tau` | Quét `0,50–0,99` bước `0,01`, cắt nhầm `< 5%`, hòa chọn cao hơn; tắt ở 2 ca biên; không chạm `cal`/`test` | Log 3 ca unit test $\tau_{\mathrm{cut}}$ + `auto_cut_policy` | **`CHƯA CHẠY`** |
| `ML-007` | `V-LEAKAGE` | MVP (`Must`) | Cấu hình huấn luyện MVP không có dữ liệu trầm tích duyệt + mã nguồn `C1`/`C4` | Kiểm tra cấu hình + kiểm tra tĩnh + ca âm tính bật cờ | `mohr_coulomb_enabled = false`; `0` ánh xạ GEBCO $\to (c_e,\varphi_e)$; `0` gán $S_e=1$ từ thiếu báo cáo sự cố | Báo cáo kiểm tra khóa `ML-007` (`violations = 0`) | **`CHƯA CHẠY`** |
| `ML-008` | `V-LEAKAGE` | GĐ2 (`Could`, `O-02` — ngoài MVP) | Bộ dữ liệu cắt trực tiếp $\ge 3$ mức `kPa`, `radian`, $\sigma'_{n,e}$ đã duyệt `O-02` | Kiểm thử đơn vị công thức `formula-6` & đơn vị đo (GĐ2) | Chỉ tính tổng trên $S_e(t)=1$ có căn cứ địa kỹ thuật, đúng đơn vị `kPa` và `radian` | Hồ sơ duyệt `O-02` + log kiểm thử `E3` (GĐ2) | **`KHÔNG ÁP DỤNG`** *(hoãn sang GĐ2 theo `Q-02`/`ML-007`, bị chặn bởi `O-02`)* |
| `ML-009` | `V-LEAKAGE` | MVP (`Must`) | Ma trận $\mathbf{R}_t$ thưa ở cả chế độ `in_profile` và `zero_delta_r_transfer` | Kiểm thử bất biến hỗ trợ dương & chiếu đơn hình `C4`/`C7` | `100%` vị trí $R_{me}=0$ giữ $\tilde{R}_{me}=0$; $\tilde{R}_{me} \ge 0$; tổng hàng $\mathcal{S}_m \neq \varnothing$ bằng `1`; đầu ra $\in [0,1]^E$ | Báo cáo bất biến `ML-009` (`non_physical_edge_count = 0`) | **`CHƯA CHẠY`** |
| `ML-010` | `V-LEAKAGE` | MVP (`Must`) | Bộ kiểm chuẩn `C3` khóa trước (`1.600/400/200/200`, `24/48`) | Đánh giá 1 lần trên `200 test` khóa trước cho `B1`/`B2`/`A1`/`A2`/`Full` (+ bản lượng tử hóa nếu có) | `0` vi phạm `V-LEAKAGE`; đạt đủ 6 cổng `Q-04` (`Hit@1 ≥ 90%`, `Hit@3 ≥ 95%`, `PR-AUC > 0,85`, `Distractor FAR < 5%`, phủ `≥ 88,5%`, tập TB `≤ 3`) + `ΔPR-AUC < 0,005` nếu lượng tử hóa; không dùng `Accuracy` | Báo cáo đối sánh 5 cấu hình + CI 95% + `run_provenance.json` | **`CHƯA CHẠY`** |
| `ML-011` | `V-LEAKAGE` | MVP (`Must`) — **chờ gỡ chặn khóa danh mục** | $\ge 3$ sự cố lịch sử độc lập khung `test` (`2025-07-01..2026-11-15`) | Kiểm tra 6 điều kiện khóa + chạy `ADR-05` (`ΔR_θ=0`, `uniform_w1`, $b_m \le$ `2024-06-30`) + `V-LEAKAGE` | $\ge 3$ sự cố độc lập, `100%` truy vết nguồn/`UTC`/`snapshot`/`NA`, `0` rò rỉ, không gán phủ `90%`, không áp ngưỡng bán tổng hợp | Báo cáo truy vết lịch sử + nhật ký `V-LEAKAGE` + báo cáo `system`/`segment` | **`BỊ CHẶN`** *(xem điều kiện gỡ chặn tại Mục 3.5 và Mục 5.3)* |
| `ML-012` | `V-LP` | MVP (`Must`) | 6 bộ fixture LP (Mục 4.2) | Chạy bộ kiểm thử tự động `C8` kèm hậu kiểm độc lập | `100%` fixture bảo toàn luồng & $0 \le u \le 1$; `0` rớt khi đủ tải; khớp chính xác `82,0 Tbps`, hai ca `10 Tbps`, `12 & 8 / 10`, `zero-demand`, `invalid_input` | Báo cáo hồi quy `V-LP` kèm `topology_hash` & sai số hậu kiểm | **`CHƯA CHẠY`** |
| `ML-013` | `V-LEAKAGE`, `V-RELEASE` | MVP (`Must`) | `run_provenance.json`, lockfile, `git_commit`, cấu hình và `seed` từng tập | Chạy lại đúng lệnh tái lập trên cùng commit & lockfile | Tái tạo khớp chỉ số đầu ra trong sai số dấu phẩy động tiêu chuẩn | Hai bộ `run_provenance.json` & bảng đối chiếu kết quả chạy lại | **`CHƯA CHẠY`** |
| `ML-014` | `V-LEAKAGE`, `V-RELEASE` | GĐ3 (`Could`, `Q-07` — ngoài MVP) | 3 chuỗi tuần giả lập (`< 100` cửa sổ/`< 80%`; 1 tuần JSD `0,12`; 2 tuần liên tiếp `≥ 0,10` hoặc RFS mới) | Kiểm thử đường ống giám sát trôi dạt `E4` (GĐ3) | Ca (a) báo thiếu dữ liệu; ca (b) cảnh báo theo dõi; ca (c) sinh ứng viên PR chờ duyệt, không tự xuất bản | Báo cáo kiểm thử `E4` & PR mẫu (GĐ3) | **`KHÔNG ÁP DỤNG`** *(thuộc GĐ3 theo `Q-01` và `Q-07`)* |
| `NFR-001` | `V-RELEASE`, `V-PERF-UX` | MVP (`Must`) | Bản dựng tĩnh phục vụ trên máy chủ tệp tĩnh thuần túy + fixture độc lập | Kiểm thử E2E Luồng 2 & Luồng 3 không có backend API | Chạy đầy đủ phát lại 5 phút, suy luận và LP 3 pha không lỗi và không gọi API máy chủ ứng dụng | Bản ghi E2E + Network trace chỉ tải tệp tĩnh | **`CHƯA CHẠY`** |
| `NFR-002` | `V-PERF-UX` | MVP (`Must`) | 50 lượt thao tác cắt cáp liên tiếp sau 5 lượt làm nóng trên M1 | Theo dõi luồng chính UI + `WebAssembly.Memory` + `model.dispose()` | 3 Web Workers tách biệt, luồng chính không khóa cứng, `0` rò rỉ bộ nhớ WASM qua 50 lượt | Performance trace + biểu đồ bộ nhớ WASM 50 lượt | **`CHƯA CHẠY`** |
| `NFR-003` | `V-PERF-UX` | MVP (`Must`) | Bản dựng tích hợp + workload phát triển trên máy chuẩn M1 (`Q-04`) | Giao thức `3` tải lạnh + `5` làm nóng + `50` thao tác + `30s` tương tác bản đồ | Đạt đồng thời 7 cổng: LP 3 pha `p95 < 15 ms`, thao tác→hiển thị `p95 ≤ 100 ms`, `≥ 55 FPS` trung vị, GPU logic `< 150 MB` (buffer `≤ 256 MB`), `WASM ≤ 210 MB`, tải thiết yếu `< 15 MB`, tổng `< 65 MB` | Báo cáo benchmark M1 + DevTools trace + log bộ đếm buffer | **`CHƯA CHẠY`** |
| `NFR-004` | `V-FALLBACK` | MVP (`Must`) | 5 nhóm kịch bản (gồm sát ngưỡng $1-\hat{q}_{0{,}90}$ và $\tau_{\mathrm{cut}}$) + sự kiện `device.lost` / `webglcontextlost` | So khớp ONNX FP32 `WebGPU` & `WASM` (ép tắt WebGPU) với ngoại tuyến + tiêm lỗi mất ngữ cảnh | `max_abs_diff ≤ 1e-4` cho $(\hat{\mathbf{x}}, \hat{\mathbf{p}})$, khớp chính xác `NA`/trạng thái/$\mathcal{C}_{0{,}90}$/$\tau_{\mathrm{cut}}$ (sai quyết định là `FAIL`), tự phục hồi ngữ cảnh | Báo cáo đối chiếu `V-FALLBACK` + bản ghi phục hồi ngữ cảnh | **`CHƯA CHẠY`** |
| `NFR-005` | `V-FALLBACK` | MVP–GĐ2 (`Should`) | Trình duyệt đã lưu đệm bản hợp lệ + giả lập ngắt mạng / sai `SHA-256` / lệch `MAJOR` | Tiêm lỗi mạng và gói hỏng vào `C6` | Giữ nguyên `last_known_good`, Luồng 2 & 3 chạy bình thường, hiển thị rõ nguồn/gói lỗi, không điền dữ liệu giả | Bản ghi E2E tiêm lỗi ngoại tuyến & gói hỏng | **`CHƯA CHẠY`** |
| `NFR-006` | `V-PERF-UX` | MVP (`Must`) | Trình duyệt bật `prefers-reduced-motion: reduce`, chỉ bàn phím và bộ lọc thang xám | Kiểm thử tiếp cận thủ công + tự động trên Luồng 2 & Luồng 3 | 100% thao tác được bằng bàn phím với `focus-visible`, phân biệt đủ trạng thái ở thang xám, tắt/giảm chuyển động không mất thông tin | Checklist kiểm thử tiếp cận + ảnh chụp thang xám | **`CHƯA CHẠY`** |
| `NFR-007` | `V-RELEASE` | MVP (`Must`) | Toàn bộ kho mã, bản dựng tĩnh, log chạy đường ống và lưu lượng mạng trình duyệt | Quét bí mật tự động + kiểm tra cấu hình mẫu | `0` khóa API / Bearer Token thật ngoài secret store ngoại tuyến; cấu hình mẫu chỉ chứa giá trị giả | Báo cáo quét bí mật `V-RELEASE` | **`CHƯA CHẠY`** |
| `NFR-008` | `V-RELEASE` | MVP (`Must`) | Toàn bộ mã nguồn mạng của mặt phẳng ngoại tuyến và trình duyệt | Rà soát tĩnh & kiểm tra truy vấn mạng | Chỉ tồn tại truy vấn đọc hợp lệ tới nguồn đã khai báo; `0` mô-đun đo chủ động trái phép hoặc cấu hình thiết bị mạng | Báo cáo kiểm toán chỉ đọc `V-RELEASE` | **`CHƯA CHẠY`** |
| `NFR-009` | `V-CONTRACT`, `V-LP`, `V-LEAKAGE` | MVP (`Must`) | Lõi thuật toán `C1`–`C8` tách khỏi UI và I/O | Kiểm kiểu tĩnh, lint, kiểm hậu tố đơn vị và chạy toàn bộ test lõi ngoại tuyến | 100% kiểm kiểu/lint/unit test lõi đạt trong môi trường không trình duyệt, không mạng | Log kiểm kiểu tĩnh, lint và bộ kiểm thử đơn vị ngoại tuyến | **`CHƯA CHẠY`** |
| `NFR-010` | `V-RELEASE` | MVP (`Must`) | Thư mục trống sao chép sạch kho mã (*clean clone*) | Cài phụ thuộc từ lockfile, chạy toàn bộ kiểm thử và dựng bản tĩnh theo `ADR-06` | Mọi bước hoàn tất không lỗi, không cần DVC/MLflow hay can thiệp ngoài tài liệu | Nhật ký chạy clean clone + `SHA-256` bản dựng | **`CHƯA CHẠY`** |
| `UX-001` | `V-PERF-UX` | MVP (`Must`) | Toàn bộ tệp CSS/Tailwind/SVG/Canvas/shader và màn hình kết xuất | Quét tĩnh gradient + kiểm tra độ tương phản trên nền phẳng | `0` khai báo `linear-gradient`/`radial-gradient`/`conic-gradient`; độ tương phản đủ đọc | Kết quả quét tĩnh + báo cáo tương phản | **`CHƯA CHẠY`** |
| `UX-002` | `V-PERF-UX` | MVP (`Must`) | Màn hình Luồng 2 & Luồng 3 hiển thị dưới bộ lọc thang xám | Rà soát nhóm màu rời rạc, kiểu nét và nhãn/biểu tượng | Phân biệt đủ bình thường, $\mathcal{C}_{0{,}90}$, cắt cáp, quá tải, `NA` và mất dự phòng SRLG không chỉ bằng màu | Ảnh chụp thang xám + bảng ánh xạ token trạng thái | **`CHƯA CHẠY`** |
| `UX-003` | `V-PERF-UX` | MVP (`Must`) | Cây DOM toàn bộ màn hình Luồng 2 & Luồng 3 | Kiểm tra cấu trúc DOM bố cục mở và thẻ card | Dải chỉ số dùng bố cục mở/bảng có đường phân cách, không bọc từng chỉ số đơn lẻ trong card, `0` lồng card | Kết quả kiểm tra DOM + ảnh chụp bố cục | **`CHƯA CHẠY`** |
| `UX-004` | `V-PERF-UX` | MVP (`Must`) | Toàn bộ chuỗi nhãn, bảng số liệu và thông báo giao diện | Rà soát đơn vị, độ bất định, cảnh báo diễn giải và subtext thừa | 100% chỉ số đủ đơn vị & cảnh báo ngữ cảnh (`MLU` + `Tbps` rớt, giới hạn suy luận); `0` subtext lặp lại | Checklist rà soát nội dung văn bản UI | **`CHƯA CHẠY`** |
| `UX-005` | `V-PERF-UX` | MVP (`Must`) | Toàn bộ chuỗi văn bản trên ứng dụng | Rà soát văn bản trước bàn giao | `0` slogan, `0` lời trích dẫn trang trí, `0` câu dẫn không phục vụ thao tác/diễn giải | Checklist rà soát văn bản UI | **`CHƯA CHẠY`** |
| `UX-006` | `V-PERF-UX` | MVP (`Must`) | Mã nguồn mô-đun giao diện (thư mục UI dự kiến theo `ARCHITECTURE.md §9.2`) | Rà soát danh mục component điều khiển so với `coss ui` | 100% component điều khiển chuẩn kế thừa/tổ hợp từ `coss ui`, không tự viết trùng | Bảng ánh xạ component UI tới `coss ui` | **`CHƯA CHẠY`** |
| `UX-007` | `V-PERF-UX` | MVP (`Must`) | Mã nguồn hiệu ứng + chế độ `prefers-reduced-motion: reduce` | Rà soát `Animate UI`, loại bỏ hiệu ứng trang trí/3D/hạt, kiểm giảm chuyển động | 100% chuyển động trạng thái từ `Animate UI`, `0` animation trang trí/hạt 3D, dùng đủ chức năng khi tắt chuyển động | Bảng ánh xạ hiệu ứng + bản ghi kiểm `reduced-motion` | **`CHƯA CHẠY`** |

### 5.3. Tổng hợp trạng thái, điều kiện gỡ chặn và giải trình mục không áp dụng

- **Tổng hợp 54 yêu cầu tại thời điểm lập giao thức:**
  - **`ĐẠT`:** `0 / 54`
  - **`KHÔNG ĐẠT`:** `0 / 54`
  - **`CHƯA CHẠY`:** `45 / 54` *(bao gồm `FR-007`, `FR-008`, `ML-003` đã tính theo phần phạm vi MVP bắt buộc; phần mở rộng GĐ2 của các mục này ghi rõ tách biệt bên dưới)*
  - **`BỊ CHẶN`:** `3 / 54` (`ML-011`, `FR-010`, `DATA-009`)
  - **`KHÔNG ÁP DỤNG` (cho đường chạy nghiệm thu MVP):** `6 / 54` (`FR-004`, `FR-005`, `FR-013`, `ML-002`, `ML-008`, `ML-014`)
- **Điều kiện gỡ chặn cho các mục `BỊ CHẶN`:**
  1. **`ML-011`** *(và phần nghiệm thu phát lại $\ge 3$ sự cố lịch sử thực tế của **`FR-010`**)*: Hoàn tất 6 điều kiện khóa danh mục tại Mục 3.5 trên dữ liệu sơ cấp thực tế ($\ge 3$ sự cố độc lập trong khung `test` `2025-07-01..2026-11-15` có đủ BGP + RIPE Atlas liên quan, `100%` nguồn sơ cấp, mốc `UTC` đúng độ chính xác, `GraphSnapshot` đóng băng trước cửa sổ chấm, cấp nhãn `system`/`segment` kèm bảng kê `NA`/`MISSING_BM`, và `0` vi phạm `V-LEAKAGE`).
  2. **`DATA-009`** *(và việc đưa gói sự cố lịch sử thật của `FR-010` lên kênh `public_demo`)*: Hoàn tất cổng pháp lý **`O-01`** — xác minh văn bản điều khoản tái phân phối đúng định dạng đầu ra cho từng tệp thuộc nguồn hạn chế (`GFW 4Wings`, `RIPE RIS/Atlas`, `CAIDA`, `PeeringDB`, `IODA`, `Cloudflare Radar`, danh mục sự cố lịch sử) và có chữ ký phê duyệt của người phụ trách trong `ReleaseManifest`.
- **Giải trình cho toàn bộ 6 mục `KHÔNG ÁP DỤNG` (và 3 nhánh GĐ2 kèm theo) ở đường chạy MVP:**
  - `FR-004`, `FR-005`: Thuộc Luồng 1 giám sát BGP trực tiếp — bị loại khỏi MVP theo `Q-01`, chuyển sang GĐ2 (`E1`).
  - `FR-013`: Thuộc Luồng 4 phân tích quy hoạch hạ tầng 2030 — bị loại khỏi MVP theo `Q-01`, chuyển sang GĐ2 (`E2`) và cổng xác nhận người dùng `O-03`.
  - `ML-002`: Áp dụng cho huấn luyện bộ mã hóa tiên nghiệm rủi ro địa vật lý/hàng hải `E3` ở GĐ2 (MVP không huấn luyện Hazard Encoder; chỉ hỗ trợ kiểm thử đơn vị sớm bộ chia LOBO và lược đồ $\mathbf{z}_e$).
  - `ML-008`: Số hạng phạt vật lý Mohr–Coulomb bị khóa tắt trong MVP theo `Q-02` và `ML-007`; chỉ mở ở GĐ2 sau khi vượt cổng địa kỹ thuật **`O-02`**.
  - `ML-014`: Giám sát trôi dạt JSD hàng tuần và sinh PR cập nhật thuộc GĐ3 (`E4`) theo `Q-01` và `Q-07`.
  - *Nhánh GĐ2 của `FR-007`, `FR-008`, `ML-003`:* Tích hợp đầy đủ bộ đệm IODA/Radar và đường ống định kỳ GFW 4Wings chỉ kích hoạt ở GĐ2 sau khi vượt `O-01`; đường chạy MVP chỉ kiểm ranh giới metadata `external_flags` (`FR-007`), chế độ `uniform_w1` (`FR-008`) và logic đơn vị mặt nạ trễ `96h/120h` + lọc tàu sửa cáp (`ML-003`).

---

## 6. Mẫu hồ sơ tái lập `run_provenance.json` và Quy trình tái lập Clean Clone (`ADR-06`)

### 6.1. Mẫu lược đồ `run_provenance.json` ([MODEL.md](MODEL.md) §6.2)

> **Lưu ý:** JSON dưới đây là **mẫu lược đồ bắt buộc cho triển khai sau**, sử dụng giá trị giữ chỗ dạng `<...>` — **không phải tệp artifact hay kết quả đã tồn tại trong kho mã**.

```json
{
  "schema_version": "1.0.0",
  "run_id": "<uuid-hoac-ma-luot-chay-duy-nhat>",
  "run_type": "<benchmark_gen | train_and_calibrate | evaluate_locked_test | historical_eval | lp_regression | release_pack>",
  "baseline_label": "new_experimental_baseline",
  "executed_at_utc": "<YYYY-MM-DDTHH:MM:SSZ>",
  "git_commit": "<sha-commit-day-du>",
  "git_working_tree_clean": true,
  "dependency_lockfiles": [
    {
      "path": "<duong-dan-lockfile-python-du-kien>",
      "sha256": "<sha256-64-ky-tu-hex>"
    },
    {
      "path": "<duong-dan-lockfile-js-du-kien>",
      "sha256": "<sha256-64-ky-tu-hex>"
    }
  ],
  "runtime_environment": {
    "os": "<ten-va-phien-ban-os>",
    "cpu_gpu": "<mo-ta-cpu-va-gpu>",
    "python_version": "<x.y.z>",
    "pytorch_version": "<x.y.z>",
    "onnx_opset_and_runtime": "<opset-va-phien-ban-ort>",
    "highs_version": "<phien-ban-highs>",
    "node_and_browser_version": "<node-va-chrome-neu-co>"
  },
  "cli_command": "<cau-lenh-chay-day-du-duoc-ghi-nhan-khi-thuc-thi>",
  "config": {
    "config_path": "<duong-dan-tep-cau-hinh-khoa>",
    "config_sha256": "<sha256-64-ky-tu-hex>",
    "prior_mode": "<synthetic_benchmark_prior | uniform_w1>",
    "mohr_coulomb_enabled": false,
    "loss_function": "weighted_bce_logits",
    "train_fit_pos_weight_ratio": {
      "n_neg_labeled_obs": "<so-nguyen-dem-chi-tren-train_fit>",
      "n_pos_labeled_obs": "<so-nguyen-duong-dem-chi-tren-train_fit>",
      "pos_weight": "<n_neg / n_pos>"
    },
    "conformal_alpha": 0.1,
    "tau_cut_grid": {
      "min": 0.5,
      "max": 0.99,
      "step": 0.01,
      "max_false_cut_scenario_rate": 0.05
    },
    "scoring_conventions_locked": {
      "coverage_ci_method": "wilson_two_sided_95",
      "coverage_gate_applies_to": "point_estimate_ge_0.885",
      "ece_num_bins": 10,
      "ece_binning_rule": "uniform_0_1_left_closed_right_open_last_includes_1",
      "rank_tie_breaking": "unrounded_pessimistic_nonfault_before_fault",
      "fault_gate_denominator": "all_fault_events_with_all_observable_breakdown",
      "f1_threshold_policy": "locked_conformal_1_minus_q_hat_0.90"
    }
  },
  "random_seeds": {
    "split_generation": {
      "train_fit": "<int-seed>",
      "val_tau": "<int-seed>",
      "cal": "<int-seed>",
      "test": "<int-seed>"
    },
    "training_libraries": {
      "python": "<int-seed>",
      "numpy": "<int-seed>",
      "pytorch": "<int-seed>"
    }
  },
  "split_manifest": {
    "graph_profile_id": "<ma-ho-so-do-thi-24seg-48ch>",
    "segments_count": 24,
    "channels_count": 48,
    "incident_group_disjoint": true,
    "counts_by_split": {
      "train_fit": {
        "total": 1600,
        "single_cut": 880,
        "srlg_multi_cut": 480,
        "distractor": 240,
        "single_cut_bgp_masked_actual": "<so-thuc-te-khi-p=0.25>"
      },
      "val_tau": {
        "total": 400,
        "single_cut": 220,
        "srlg_multi_cut": 120,
        "distractor": 60,
        "single_cut_bgp_masked_actual": "<so-thuc-te-khi-p=0.25>"
      },
      "cal": {
        "total": 200,
        "single_cut": 110,
        "srlg_multi_cut": 60,
        "distractor": 30,
        "single_cut_bgp_masked_actual": "<so-thuc-te-khi-p=0.25>"
      },
      "test": {
        "total": 200,
        "single_cut": 110,
        "srlg_multi_cut": 60,
        "distractor": 30,
        "single_cut_bgp_masked_actual": "<so-thuc-te-khi-p=0.25>"
      }
    }
  },
  "source_cutoffs_utc": {
    "<source_name>": {
      "source_version": "<phien-ban-hoac-doi>",
      "published_at_utc": "<YYYY-MM-DDTHH:MM:SSZ>",
      "retrieved_at_utc": "<YYYY-MM-DDTHH:MM:SSZ>"
    }
  },
  "locked_thresholds": {
    "calibration_quantile_q_by_model": {
      "B1": "<float>",
      "B2": "<float>",
      "A1": "<float>",
      "A2": "<float>",
      "Full": "<float>"
    },
    "auto_cut_policy": {
      "enabled": "<bool>",
      "tau_cut": "<float-hoac-null>",
      "disabled_reason": "<string-hoac-null>"
    }
  },
  "anti_leakage_gate_results": {
    "overall_status": "<PASS | BLOCKED | FAIL>",
    "total_violations": 0,
    "gate_1_time_embargo_and_group_disjoint": "<PASS | BLOCKED | FAIL>",
    "gate_2_lobo_and_no_raw_coords": "<NOT_APPLICABLE_MVP | PASS | BLOCKED | FAIL>",
    "gate_3_ais_96h_sar_120h_repair_vessels": "<PASS | BLOCKED | FAIL>",
    "gate_4_rfs_and_snapshot_cutoffs_2023_02_2026_08_25": "<PASS | BLOCKED | FAIL>",
    "gate_5_split_isolation_and_d_test_detached": "<PASS | BLOCKED | FAIL>",
    "support_mask_and_simplex_invariant_ml009": "<PASS | BLOCKED | FAIL>",
    "mohr_coulomb_disabled_lock_ml007": "<PASS | BLOCKED | FAIL>"
  },
  "locked_test_access_count": "<0-trong-train_and_calibrate | 1-trong-evaluate_locked_test>",
  "input_artifacts": [
    {
      "path": "<duong-dan-dau-vao>",
      "sha256": "<sha256-64-ky-tu-hex>",
      "data_provenance_class": "<observed_measurement | stylized_derived | synthetic_calibrated_baseline | simulated_assumption>",
      "retrieved_at_utc": "<YYYY-MM-DDTHH:MM:SSZ>"
    }
  ],
  "output_artifacts": [
    {
      "path": "<duong-dan-dau-ra-du-kien>",
      "sha256": "<sha256-64-ky-tu-hex>",
      "artifact_role": "<onnx_fp32 | eval_report | release_manifest | lp_report>"
    }
  ],
  "unresolved_limitations": [
    "Suy luan khong tu xac nhan dut cap vat ly; bo giai LP khong dieu khien mang san xuat.",
    "<ghi-ro-gioi-han-con-lai-cua-luot-chay-nhu-ML-011-hoac-MISSING_BM>"
  ]
}
```

### 6.2. Quy trình tái lập từ bản sao kho mã sạch (*Clean Clone*, `ADR-06`)

Theo `ADR-06`, MVP **không bắt buộc cài hay vận hành máy chủ DVC hoặc MLflow**. Khi kho mã đã có triển khai đầy đủ, một kỹ sư tái lập (`U2`) phải có thể xác minh toàn bộ giao thức theo đúng 8 bước tuần tự dưới đây bằng các lệnh sẽ được công bố kèm mã nguồn (hiện chưa có kịch bản thực thi trong repo):

1. **Sao chép sạch và khóa trạng thái mã nguồn:** Sao chép repo vào một thư mục trống sạch tại đúng `git_commit` ghi trong `run_provenance.json`; xác nhận cây làm việc sạch (`git_working_tree_clean = true`) và mã băm `SHA-256` của các lockfile phụ thuộc trùng khớp.
2. **Cài đặt phụ thuộc tất định từ lockfile:** Cài đặt môi trường Python và JavaScript/TypeScript hoàn toàn từ lockfile đã khóa (không tự nâng cấp phiên bản gói).
3. **Chạy kiểm định hợp đồng, bộ giải LP và đơn vị chống rò rỉ ngoại tuyến:** Chạy toàn bộ nhóm `V-CONTRACT`, `V-LP` và các kiểm thử đơn vị của `V-LEAKAGE` (gồm Chốt 1–5, hồi quy `2023-02`/`2026-08-25`, 3 ca biên $\tau_{\mathrm{cut}}$, `ML-007`, `ML-009`) trong môi trường không có trình duyệt và không có mạng (`NFR-009`).
4. **Sinh bộ kiểm chuẩn `C3` và đối chiếu `SHA-256`:** Chạy bộ sinh bán tổng hợp với tệp cấu hình và `random_seeds` đã khóa cho 4 tập `1.600 / 400 / 200 / 200`; đối chiếu khớp `split_manifest`, số ca che khuất BGP thực tế và `SHA-256` đầu ra.
5. **Huấn luyện và hiệu chuẩn khi tháo rời `D_test`:** Di chuyển hoặc gỡ gắn tệp `200 test` khỏi môi trường tiến trình `train_and_calibrate`; khớp `B1`, `B2`, `A1`, `A2`, `Full` và $b_m$ trên `train_fit`, chọn $\tau_{\mathrm{cut}}$ trên `val_tau`, ước lượng $\hat{q}_{0{,}90}$ riêng từng cấu hình trên `200 cal`, xuất ONNX FP32 và ghi `run_provenance.json` (`locked_test_access_count = 0`, `overall_status = PASS`).
6. **Mở khóa đánh giá `D_test` đúng một lần ở chế độ chỉ đọc:** Chạy `evaluate_locked_test` bằng bộ tham số đã khóa (`locked_test_access_count = 1`); xác minh đạt đủ 6 cổng `Q-04` (`ML-010`), báo cáo đầy đủ CI 95%, mẫu số kép và danh mục `NA`/lỗi. *(Khi `ML-011` đã được gỡ chặn, chạy bước hồi quy lịch sử riêng theo `ADR-05` trên dữ liệu tải từ trình tải nguồn gốc cục bộ ở kênh `internal_local`.)*
7. **Đóng gói, kiểm tra cổng phát hành và dựng bản tĩnh:** Chạy `C5` tạo thư mục phát hành bất biến dự kiến (quy ước `releases/<release_id>/` theo `ARCHITECTURE.md §5.5`), xác thực `V-RELEASE` (tách giấy phép, duyệt `ReleaseManifest`, `SHA-256`, quét bí mật, kiểm chỉ đọc, trần nén `< 15 MB` / `< 65 MB`) và dựng ứng dụng trình duyệt tĩnh.
8. **Xác minh trình duyệt (`V-FALLBACK` & `V-PERF-UX`):** Trên máy chuẩn M1 (`Q-04`), chạy kiểm chứng tương đương ONNX FP32 `WebGPU` & `WASM` (ép tắt WebGPU) cùng kiểm thử `device.lost`/`webglcontextlost`/`last_known_good`; chạy giao thức đo `3` tải lạnh + `5` làm nóng + `50` thao tác + `30 giây` tương tác bản đồ bằng workload phát triển (không dùng `200 test`), đối chiếu 7 cổng `NFR-003`, khả năng tiếp cận `NFR-006` và checklist `DESIGN.md`.
