# Đặc tả Mô hình, Hiệu chuẩn và Đánh giá SubseaBGP-Tomography

Tài liệu này đặc tả **bài toán suy luận cắt lớp, kiến trúc LP-GISTA 12 lớp, quy trình huấn luyện – hiệu chuẩn – chọn ngưỡng, mô phỏng định tuyến dự phòng LP cùng đối chứng đường ngắn nhất, giao thức đánh giá và quy chuẩn đóng gói – tái lập** cho dự án nghiên cứu – mô phỏng **SubseaBGP-Tomography**. Toàn bộ định nghĩa nguồn dữ liệu, đường ống tiền xử lý, lược đồ đầu vào (`GraphSnapshot`, `SignalWindow`, `IncidentAnnouncement`, `ReleaseManifest`), phân hoạch dữ liệu, 5 chốt chống rò rỉ và quy tắc `NA` được dẫn chiếu trực tiếp từ [DATA.md](DATA.md).

- **Căn cứ phạm vi và nghiệm thu:** [REQUIREMENTS.md](REQUIREMENTS.md) (`Q-01`–`Q-07`, Mục 3, Mục 6, Mục 9).
- **Căn cứ kiến trúc và hợp đồng MVP:** [ARCHITECTURE.md](ARCHITECTURE.md) (§1.6, §4, §5, §7, §9.1, §10).
- **Nguồn đặc tả dữ liệu:** [DATA.md](DATA.md).
- **Cơ sở nghiên cứu tham chiếu:** [RESEARCH.md](RESEARCH.md) (khi có điểm khác biệt đã chuẩn hóa trong `Q-01`–`Q-07`, áp dụng [REQUIREMENTS.md](REQUIREMENTS.md); các điểm chưa xử lý được nêu rõ tại Mục 7.2).
- **Quy tắc chung kho mã:** [AGENTS.md](../AGENTS.md).

> **Quy ước phân biệt và hiện trạng thực tế:**
> - **Bắt buộc (`Req`):** Ràng buộc cứng từ `REQUIREMENTS.md`, `ARCHITECTURE.md` và `AGENTS.md`.
> - **Đề xuất (`Prop`):** Chi tiết huấn luyện, cấu hình hàm mất mát hoặc dung sai kỹ thuật đề xuất để triển khai; phải được khóa trong tệp cấu hình trước khi chạy thực nghiệm.
> - **Kết quả thực tế:** Kiểm tra kho mã cho thấy **chưa có mã huấn luyện, chưa có mô hình ONNX, chưa có bộ giải LP triển khai và chưa có bất kỳ lần chạy thực nghiệm nào**. Mọi chỉ số trong `RESEARCH.md §3.6, §6.2` thuộc nguồn `[1]` (`unknown_url`) chỉ mang tính tham chiếu nguyên mẫu cũ, **không phải kết quả tái lập** (`Q-06`). Toàn bộ phép kiểm chứng trong tài liệu này hiện ở trạng thái **Chưa chạy**.
> - **Khóa phạm vi MVP (`Q-01`):** Đường chạy MVP chỉ gồm suy luận ngoại tuyến/tĩnh cho **Luồng 2 (Phát lại lịch sử 5 phút)** và **Luồng 3 (Cắt cáp giả định & giải LP dự phòng)**. **Không** đưa BGP trực tiếp (Luồng 1), quy hoạch 2030 (Luồng 4), số hạng phạt Mohr–Coulomb (`ML-007`) hay giám sát trôi dạt (Giai đoạn 3, `Q-07`) vào đường chạy MVP; **tuyệt đối không** sinh hay đẩy cấu hình điều khiển lên mạng sản xuất (`NFR-008`).

---

## 1. Bài toán, vùng áp dụng và giao diện mô hình

### 1.1. Hệ thống ước lượng gì và không thể xác nhận gì

- **Hệ thống ước lượng gì (`REQUIREMENTS.md §1.1`):**
  1. Từ chuỗi tín hiệu quan sát đa tầng trên các kênh đo hợp lệ ($\tilde{y}_m^{\mathrm{BGP}}$, $\tilde{y}_m^{\mathrm{RTT}}$, cổng vật lý $g_m$, hội chứng $y_m$), ma trận liên thuộc đóng băng $\mathbf{R}_t$, toán tử cấu trúc rủi ro chung $\mathbf{L}_{\mathrm{SRLG}}$ và trọng số tiên nghiệm $\mathbf{w}(t)$, hệ thống ước lượng **mức suy giảm liên tục** $\hat{x}_e(t) \in [0, 1]$ và **xác suất suy giảm** $\hat{p}_e(t) \in [0, 1]$ cho từng phân đoạn cáp khả kiến $e \in \mathcal{E}_{\mathrm{seg}}$.
  2. Trên cơ sở phân vị hiệu chuẩn độc lập $\hat{q}_{0{,}90}$ (khi đủ điều kiện hiệu chuẩn), hệ thống khoanh vùng **tập phân đoạn truyền dẫn nghi vấn** $\mathcal{C}_{0{,}90}(t)$ định lượng độ bất định trong điều kiện các cáp song song có độ cộng tuyến cao.
  3. Từ kết quả suy luận (tùy chọn) và/hoặc thao tác cắt giả định thủ công, bộ giải quy hoạch tuyến tính tuần tự 3 pha ước lượng phương án phân bổ lưu lượng dự phòng mô phỏng (lưu lượng rớt `Tbps`, `MLU %`, độ trễ gia quyền `ms`, cảnh báo mất dự phòng SRLG) so với định tuyến đường ngắn nhất mặc định.
- **Hệ thống KHÔNG thể xác nhận gì (`REQUIREMENTS.md §1.2`, `RESEARCH.md §7`):**
  Mọi đầu ra $(\hat{\mathbf{x}}, \hat{\mathbf{p}}, \mathcal{C}_{1-\alpha})$ chỉ là **giả thuyết xác suất về vùng/phân đoạn nghi suy giảm**, tuyệt đối không tự xác nhận sự cố đứt cáp vật lý dưới đáy biển hay vị trí lỗi cấp độ mét vì bốn giới hạn gốc:
  1. *Âm tính giả do bảo vệ quang tầng 1:* Khi chuyển mạch bảo vệ DWDM/OTN chuyển sang sợi dự phòng đồng hành lang có độ chênh chiều dài $< 50\ \mathrm{km}$ ($\Delta\mathrm{minRTT} < 0{,}5\ \mathrm{ms}$, nhỏ hơn sai số $9{,}5\%$ của ước lượng `minRTT` từ RIPE Atlas) và không rút BGP, sự cố trở nên vô hình với đo kiểm ngoài mạng.
  2. *Dương tính giả do hạ tầng đất liền nội địa:* Đứt cáp trục đất liền nội địa hoặc lỗi trạm trung chuyển nối ra CLS có thể làm nhảy đồng thời cả BGP lẫn độ trễ cuối – cuối.
  3. *Tính sơ đồ hóa và trọng số tiên nghiệm:* Hình học tuyến cáp công khai là đường vẽ sơ đồ hóa (*stylized routes*), còn $(\Omega, \Phi_{\mathrm{PoP}})$ chỉ là điểm trọng số tiên nghiệm (`FR-002`, [DATA.md §3.2](DATA.md#32-mô-hình-hóa-địa-không-gian-vòng-đời-rfs-bằng-chứng-srlg-và-quan-hệ-vật-lýlogic)).
  4. *Điều kiện xác nhận vật lý:* Xác nhận đứt cáp vật lý đòi hỏi phép đo nội bộ tại CLS (OTDR, PFE), thông cáo vận hành chính thức hoặc khảo sát của tàu sửa cáp.

### 1.2. Vùng áp dụng và tách bạch ba loại đầu ra

| Chế độ vận hành (`inference_mode` / trạng thái) | Vùng áp dụng hợp lệ | Ràng buộc bắt buộc |
| --- | --- | --- |
| **`in_profile` + `calibrated_90`** | Cùng `graph_profile_id` đã huấn luyện và đã hiệu chuẩn độc lập trên tập `cal` tương ứng (ví dụ bộ kiểm chuẩn bán tổng hợp 24 phân đoạn / 48 kênh). | Cho phép dùng $\Delta\mathbf{R}_\theta$ (khóa bởi $\mathbf{M}_{\mathrm{mask}}, \Pi_\Delta$) và báo tập nghi vấn $\mathcal{C}_{0{,}90}(t)$ kèm nhãn mức danh định `90%`. |
| **`zero_delta_r_transfer` (`ADR-05`)** | Chuyển giao sang lưu vực khác, sự cố lịch sử ngoài vùng huấn luyện hoặc khi `graph_profile_id` không khớp. | Khóa cứng $\Delta\mathbf{R}_\theta = \mathbf{0}$, giữ $\mathbf{M}_{\mathrm{mask}}$ và $\Pi_\Delta$; nếu chưa có tập hiệu chuẩn phù hợp thì đặt `conformal_calibration_status = uncalibrated_transfer`, `nominal_coverage_target = null` và **cấm gán bảo chứng độ phủ 90%**. |
| **`degraded_partial_na` / `insufficient_observations`** | Khuyết một phần hoặc toàn bộ kênh đo hợp lệ (`valid_channel_count < total_channel_count`). | Kênh `NA` loại tường minh qua `m_valid`; phân đoạn có `observing_valid_channels == 0` trả `null` và `unobservable_na` (không hiển thị như "an toàn"). |

Hệ thống tách bạch tuyệt đối ba khái niệm đầu ra để không nhập nhằng giữa suy luận xác suất và tác động giả định lên dung lượng:
1. **Điểm suy luận ($\hat{x}_e(t), \hat{p}_e(t)$):** Ước lượng liên tục trong $[0, 1]$ cho từng phân đoạn có ít nhất `1` kênh quan sát hợp lệ.
2. **Tập nghi vấn hiệu chuẩn ($\mathcal{C}_{0{,}90}(t)$):** Tập các phân đoạn có $\hat{p}_e(t) \ge 1 - \hat{q}_{0{,}90}$ (với $\hat{q}_{0{,}90}$ ước lượng độc quyền trên `200 cal`). Dùng để khoanh vùng bất định và (khi bật chiết khấu suy luận) chiết khấu mềm dung lượng $C_e(1 - \hat{x}_e(t))$.
3. **Quyết định cắt giả định ($\tilde{C}_e(t) = 0$):** Gồm (a) **cắt thủ công** của người dùng trong Luồng 3 (`manual_cut_segment_ids`, luôn có hiệu lực ưu tiên tuyệt đối) và (b) **cắt tự động theo xác suất** bằng ngưỡng $\tau_{\mathrm{cut}}$ chọn trên `val_tau` (tùy chọn và tự động tắt khi không thỏa điều kiện `Q-03`).

### 1.3. Giao diện đầu vào và đầu ra theo hợp đồng kiến trúc

- **Đầu vào (dẫn chiếu [DATA.md §3](DATA.md#3-đồ-thị-hạ-tầng-và-hợp-đồng-dữ-liệu)):**
  - `GraphSnapshot` (`1.0.0`): cung cấp `snapshot_id`, `graph_profile_id`, `topology_hash`, `segments[]`, `srlg_laplacian`, `routing_matrix_R[]` (kèm cờ `is_scorable` / `na_reason`) và `candidate_paths[]`.
  - `SignalWindow` (`1.0.0`): cung cấp `window_id`, `snapshot_id`, chuỗi kênh `channels[]` (`syndrome_y`, `is_valid`, `na_reason`) và `prior_mode` / `segment_priors[]` (`weight_w`).
  - **Ràng buộc cách ly bắt buộc (`DATA.md §2.4, §3.3`):** `IncidentAnnouncement` **tuyệt đối không đi vào `C7`**; `external_flags` (nếu có ở `internal_local`) **không được đọc bởi mô hình suy luận MVP**.
  - Toàn bộ khóa liên kết (`snapshot_id`, `window_id`, `segment_id`, `channel_id`, `srlg_group_id`, `demand_id`, `path_id`, `model_version`, `run_id`, `request_id`, `epoch`), chuẩn `ISO-8601 UTC` (`Z`), hậu tố đơn vị và tập đóng `na_reason` tuân thủ nguyên văn [DATA.md §3.1](DATA.md#31-quy-tắc-chung-phiên-bản-định-danh-liên-kết-thời-gian-đơn-vị-và-na).
- **Đầu ra mô hình — Hợp đồng `InferenceResult` (`inference_result`, `1.0.0`, `ARCHITECTURE.md §4.2`):**
  - *Bên tạo:* `C7` (trên trình duyệt) hoặc `C4` (đánh giá ngoại tuyến). *Bên đọc:* `C8` (tùy chọn khi bật chiết khấu), `C9` (hiển thị tại **Lớp 2 — Kết quả suy luận**).
  - *Cấu trúc tối thiểu (`Req`):*
    - Siêu dữ liệu: `schema_version`, `request_id`, `epoch`, `snapshot_id`, `window_id`, `window_end_utc`, `model_version`, `execution_backend` (`webgpu | wasm_cpu | offline_ref`), `inference_mode` (`in_profile | zero_delta_r_transfer`), `prior_mode`, `conformal_calibration_status` (`calibrated_90 | uncalibrated_transfer`), `calibration_quantile_q | null`, `nominal_coverage_target` (`0.90 | null`), `valid_channel_count`, `total_channel_count`, `valid_channel_ratio`.
    - Chính sách cắt tự động: `auto_cut_policy` (`enabled: bool`, `tau_cut: number | null`, `disabled_reason | null`).
    - Trạng thái tổng thể: `status` (`ok | degraded_partial_na | insufficient_observations`).
    - Điểm từng phân đoạn (`segment_scores[]`): `segment_id`, `x_hat` (`[0,1] | null`), `p_hat` (`[0,1] | null`), `in_conformal_set: bool | null`, `observing_valid_channels`, `observability_status` (`observable | unobservable_na`).
    - Tóm tắt: `conformal_set_segment_ids[] | null` và `interpretation_disclaimer` (bắt buộc khẳng định đầu ra xác suất không tự xác nhận đứt cáp vật lý).
- **Đầu vào và đầu ra mô phỏng định tuyến — `SimulationRequest` (`1.0.0`) và `SimulationResult` (`1.0.0`, `ARCHITECTURE.md §4.2`):**
  - *`SimulationRequest` (tạo bởi `C9`/`C4`, đọc bởi `C8`):* `schema_version`, `request_id`, `epoch`, `snapshot_id`, `scenario_id`, `data_provenance_class` (`simulated_assumption`), `demands[]` (`demand_id`, `src_region`, `dst_region`, `demand_tbps`), `manual_cut_segment_ids[]`, `apply_inference_discount: bool`, `inference_ref | null` (`p_hat`, `x_hat`, `conformal_set_segment_ids[]`, `auto_cut_policy`), `srlg_cap_ratio = 0.65`, `compute_shortest_path_baseline: bool`.
  - *`SimulationResult` (tạo bởi `C8`, đọc bởi `C9`/`C4`):* `schema_version`, `request_id`, `epoch`, `snapshot_id`, `scenario_id`, `solver_version`, `solve_time_ms` (tổng cả 3 pha), `status` (`optimal | all_paths_unavailable | invalid_input`), `interpretation_warning` (bắt buộc nhấn mạnh đọc đồng thời `total_dropped_tbps` và `mlu_pct`), `effective_capacities[]` (`capacity_reason` $\in \{\texttt{normal}, \texttt{conformal\_discount}, \texttt{auto\_cut\_tau}, \texttt{manual\_cut}\}$), `lp_solution` (`total_demand_tbps`, `total_served_tbps`, `total_dropped_tbps`, `mlu_pct` $\in [0, 100]$, `weighted_rtt_ms | null` kèm `na_reason = NO_SERVED_FLOW` khi `total_served_tbps == 0`, `demands[]` có `available_srlg_corridor_count`, `applied_srlg_cap_tbps`, `srlg_redundancy_lost`, `disconnected`, `path_flows[]`, `segment_loads[]`) và `shortest_path_baseline` (tách bạch `raw_unconstrained_overload_pct` khỏi `mlu_pct` thực chuyển).

---

## 2. Kiến trúc LP-GISTA 12 lớp và các bất biến toán học

### 2.1. Công thức trải phẳng 12 lớp (`ML-009`, `FR-009`, `RESEARCH.md §3.4`)

Xuất phát từ bài toán tối ưu lồi có cấu trúc đồ thị SRLG:
$$\hat{\mathbf{x}}(t) = \underset{\mathbf{x} \in [0,1]^E}{\arg\min}\ \frac{1}{2}\left\lVert \mathbf{m}_{\mathrm{valid}}(t) \odot \left(\mathbf{R}_t \mathbf{x} - \mathbf{y}_{\mathrm{filled}}(t)\right) \right\rVert_2^2 + \lambda \sum_{e=1}^E w_e(t) x_e + \frac{\gamma}{2}\mathbf{x}^\top \mathbf{L}_{\mathrm{SRLG}} \mathbf{x}$$
mạng **LP-GISTA** trải phẳng chính xác **$K = 12$ lớp** ($k = 0, 1, \ldots, 11$), khởi tạo $\mathbf{x}^{(0)} = \mathbf{0} \in \mathbb{R}^E$:

$$\tilde{\mathbf{R}}_t = \Pi_\Delta\left(\mathbf{R}_t + \mathbf{M}_{\mathrm{mask}} \odot \Delta\mathbf{R}_\theta\right), \qquad \mathbf{M}_{\mathrm{mask}} = \mathbb{I}(\mathbf{R}_t > 0)$$

$$\mathbf{r}^{(k)} = \mathbf{m}_{\mathrm{valid}}(t) \odot \left(\tilde{\mathbf{R}}_t \mathbf{x}^{(k)} - \mathbf{y}_{\mathrm{filled}}(t)\right)$$

$$\mathbf{z}^{(k)} = \mathbf{x}^{(k)} - \eta_k \tilde{\mathbf{R}}_t^\top \mathbf{r}^{(k)} - \eta_k \gamma_k \mathbf{L}_{\mathrm{SRLG}} \mathbf{x}^{(k)}$$

$$\mathbf{x}^{(k+1)} = \Pi_{[0,1]}\left(\max\left(\mathbf{0},\, \mathbf{z}^{(k)} - \eta_k \lambda_k \mathbf{w}(t)\right)\right), \qquad k = 0, \ldots, 11$$

$$\hat{x}_e(t) = x_e^{(12)}, \qquad \hat{p}_e(t) = \sigma\left(a_{\mathrm{cal}}\, x_e^{(12)} - b_{\mathrm{cal}}\right)$$

### 2.2. Ký hiệu, kích thước tensor và miền giá trị

Với $E$ là số phân đoạn truyền dẫn và $M$ là số kênh đo trong `GraphSnapshot` (trên bộ kiểm chuẩn bán tổng hợp MVP: $E = 24$, $M = 48$):

| Ký hiệu | Kích thước tensor | Miền giá trị & Kiểu | Nguồn / Vai trò |
| --- | --- | --- | --- |
| $\mathbf{R}_t$ | $(M, E)$ | $[0, 1]$, `float32` | Ma trận liên thuộc đóng băng từ `GraphSnapshot` (các phần tử `is_scorable = false` gán `0` trong phép nhân ma trận sau khi kênh tương ứng bị che bởi $\mathbf{m}_{\mathrm{valid}}$). |
| $\mathbf{M}_{\mathrm{mask}}$ | $(M, E)$ | $\{0, 1\}$, `float32` | Mặt nạ hỗ trợ dương $\mathbb{I}(\mathbf{R}_t > 0)$; khóa cứng các cạnh bằng `0`. |
| $\Delta\mathbf{R}_\theta$ | $(M, E)$ | $\mathbb{R}$, `float32` | Tham số hiệu chỉnh tỷ lệ chia tải học trên `train_fit` (khóa bằng $\mathbf{0}$ ở chế độ `zero_delta_r_transfer`). |
| $\tilde{\mathbf{R}}_t$ | $(M, E)$ | $[0, 1]$, `float32` | Ma trận liên thuộc sau hiệu chỉnh và chiếu đơn hình hàng $\Pi_\Delta$. |
| $\mathbf{L}_{\mathrm{SRLG}}$ | $(E, E)$ | $\mathbb{R}$, đối xứng nửa xác định dương, `float32` | Toán tử Laplacian chuẩn hóa rủi ro chung từ `GraphSnapshot` ($r = 25\ \mathrm{km}$, eo biển nghẽn). |
| $\mathbf{y}_{\mathrm{filled}}(t)$ | $(M,)$ | $[0, +\infty)$, `float32` | Vector `syndrome_y` từ `SignalWindow`, thay tạm `null` bằng `0` **chỉ để tránh `NaN` số học khi nhân với `m_valid[m] = 0`**. |
| $\mathbf{m}_{\mathrm{valid}}(t)$ | $(M,)$ | $\{0, 1\}$, `float32` | Mặt nạ kênh hợp lệ (`1` khi `channel.is_valid == true` và kênh có hàng $\mathbf{R}_t$ khả chấm, `0` khi `NA`). |
| $\mathbf{w}(t)$ | $(E,)$ | $(0, +\infty)$, `float32` | Trọng số phạt thưa thích nghi từ `SignalWindow.segment_priors[]` (mặc định $\mathbf{1}$ khi `prior_mode = uniform_w1`). |
| $\{\eta_k, \lambda_k, \gamma_k\}_{k=0}^{11}$ | $3 \times (12,)$ | $(0, +\infty)$, `float32` | Hệ số bước nhảy ($\eta_k$), phạt thưa ($\lambda_k$) và làm trơn SRLG ($\gamma_k$) theo từng lớp (ràng buộc dương khi huấn luyện, ví dụ qua `softplus` — `Prop`). |
| $a_{\mathrm{cal}}, b_{\mathrm{cal}}$ | $(1,), (1,)$ | $a_{\mathrm{cal}} > 0$, $b_{\mathrm{cal}} \in \mathbb{R}$, `float32` | Tham số hiệu chỉnh logistic ánh xạ $x_e^{(12)} \mapsto \hat{p}_e(t)$ — **khớp trên `train_fit`, không khớp trên `cal`**. |
| $\mathbf{x}^{(k)}, \mathbf{z}^{(k)}, \hat{\mathbf{p}}(t)$ | $(E,)$ | $\mathbf{x}^{(k)}, \hat{\mathbf{p}} \in [0, 1]^E$, `float32` | Trạng thái trung gian và đầu ra cuối cho phân đoạn khả kiến (`null` nếu `observability_status = unobservable_na`). |

### 2.3. Các bất biến bắt buộc, chuyển giao `ADR-05` và khóa tắt Mohr–Coulomb

1. **Mặt nạ kênh hợp lệ và bảo vệ ngữ nghĩa `NA` (`FR-009`, `ARCHITECTURE.md §7.1`):**
   - Với mọi kênh có `m_valid[m] = 0`, phần dư $r_m^{(k)} = 0$ ở cả 12 lớp nên kênh `NA` đóng góp gradient đúng bằng `0`, không kéo $\hat{\mathbf{x}}(t)$ về `0`.
   - Số kênh hợp lệ quan sát được phân đoạn $e$ là $c_{\mathrm{obs}}(e) = \sum_{m=1}^M m_{\mathrm{valid},m}(t)\cdot \mathbb{I}(R_{me}(t) > 0)$.
   - Nếu `valid_channel_count == 0` (quyết định `D-01`) $\Rightarrow$ trả `status = insufficient_observations`, `conformal_set_segment_ids = []` cho `calibrated_90` và `null` cho `uncalibrated_transfer`, mọi phân đoạn có `x_hat = null`, `p_hat = null`, `in_conformal_set = null`, `observability_status = unobservable_na`, và không được tính là âm tính đúng trong `Distractor FAR` (`null` nghĩa là không có tập Conformal được hiệu chuẩn; `[]` trong trường hợp này không có nghĩa là an toàn).
   - Nếu chỉ một phần kênh `NA` $\Rightarrow$ trả `status = degraded_partial_na`; mọi phân đoạn có $c_{\mathrm{obs}}(e) = 0$ trả `x_hat = null`, `p_hat = null`, `in_conformal_set = null`, `observability_status = unobservable_na` (tuyệt đối không gán `0` hay hiển thị như "an toàn").
2. **Bảo toàn hỗ trợ dương và phép chiếu đơn hình $\Pi_\Delta$ (`ML-009`):**
   - Với từng hàng $m \in \{1, \ldots, M\}$, định nghĩa tập hỗ trợ dương $\mathcal{S}_m = \{e : R_{me}(t) > 0\}$.
   - Phép chiếu đơn hình $\Pi_\Delta$ chiếu từng hàng $m$ của $\mathbf{R}_t + \mathbf{M}_{\mathrm{mask}} \odot \Delta\mathbf{R}_\theta$ lên đơn hình xác suất $\Delta_{\mathcal{S}_m} = \{\mathbf{v} \in \mathbb{R}_+^E : \sum_{e \in \mathcal{S}_m} v_e = 1,\ v_e = 0\ \forall e \notin \mathcal{S}_m\}$ khi $\mathcal{S}_m \neq \varnothing$, và giữ nguyên hàng $\mathbf{0}$ khi $\mathcal{S}_m = \varnothing$.
   - *Bất biến kiểm chứng (`V-LEAKAGE`):* `100%` cặp có $R_{me}(t) = 0$ giữ $\tilde{R}_{me}(t) = 0$ (0 cạnh phi vật lý phát sinh), mọi $\tilde{R}_{me}(t) \ge 0$, tổng từng hàng có $\mathcal{S}_m \neq \varnothing$ bằng $1$ trong sai số dấu phẩy động, và đầu ra lớp 12 nằm trong $[0, 1]^E$.
3. **Chế độ chuyển giao ngoài lưu vực hoặc khi `graph_profile_id` không khớp (`ADR-05`):**
   - Khi áp mô hình sang lưu vực khác, tô-pô lịch sử thay đổi số phân đoạn/kênh, hoặc `snapshot.graph_profile_id != model_artifact.graph_profile_id`, động cơ `C7` tự động chuyển sang **`inference_mode = zero_delta_r_transfer`**:
     - Khóa cứng **$\Delta\mathbf{R}_\theta = \mathbf{0}$** (chỉ tái sử dụng các tham số bất biến kích thước đồ thị $\{\eta_k, \lambda_k, \gamma_k\}_{k=0}^{11}$ và $(a_{\mathrm{cal}}, b_{\mathrm{cal}})$).
     - **Giữ nguyên mặt nạ hỗ trợ dương $\mathbf{M}_{\mathrm{mask}} = \mathbb{I}(\mathbf{R}_t > 0)$ và phép chiếu đơn hình $\Pi_\Delta$** trên ma trận $\mathbf{R}_t$ đóng băng của `GraphSnapshot` đích.
     - Nếu chưa có tập hiệu chuẩn Split Conformal độc lập tương thích với lưu vực/tô-pô đích: bắt buộc đặt **`conformal_calibration_status = uncalibrated_transfer`**, `calibration_quantile_q = null`, `nominal_coverage_target = null`, `conformal_set_segment_ids = null` và **tuyệt đối không gán bảo chứng độ phủ 90%**.
     - **Đường cơ sở $b_m$ trong chế độ chuyển giao lịch sử (`ADR-05`):** Áp dụng quy tắc đóng băng theo `(channel_id, path_regime)` từ cửa sổ sạch `60 phút` gần nhất đủ $\ge 2$ mẫu hợp lệ trong `train_fit` lịch sử ($\le$ `2024-06-30`, loại khoảng sự cố và đệm $\pm 72\text{h}$); thiếu đường cơ sở hợp lệ thì trả `NA` (`MISSING_BM`), không mượn cửa sổ ngay trước ca thuộc `test` để bù (chi tiết tại [DATA.md](DATA.md) §2.3). Phát lại lịch sử MVP chạy mặc định với `prior_mode = uniform_w1` ($w_e(t) = 1$).
   - `ADR-05` được khóa bằng kiểm thử hỗ trợ dương (`ML-009`) và hồi quy lịch sử (`ML-011`) — **không khóa bằng tốc độ M1**.
4. **Khóa tắt số hạng phạt vật lý Mohr–Coulomb trong MVP (`ML-007`, `ML-008`, `Q-02`, `O-02`):**
   - Trong toàn bộ đường chạy MVP, số hạng phạt địa cơ học của đặc tả thay thế `formula-6`:
     $$\mathcal{L}_{\mathrm{prior}} = \mathcal{L}_{\mathrm{Focal}} + \frac{0{,}01}{E}\sum_{e : S_e(t)=1}\left[\hat{\tau}_e(t) - c_e - \sigma'_{n,e}(s_e,\mathrm{PGA}_e)\tan\varphi_e\right]_+$$
     bị **khóa ở trạng thái tắt** (`mohr_coulomb_enabled = false`).
   - Nghiêm cấm suy $(c_e, \varphi_e)$ từ độ sâu GEBCO_2026, nghiêm cấm điền giá trị mặc định và nghiêm cấm lấy việc "không có báo cáo sự cố" làm nhãn ổn định $S_e(t) = 1$.
   - Chỉ được mở lại số hạng này ở **Giai đoạn 2 (`E3`)** sau khi vượt cổng `O-02` (xác minh được dữ liệu thí nghiệm cắt trực tiếp thoát nước theo phân đoạn với $\ge 3$ mức ứng suất hữu hiệu tính bằng `kPa`, góc ma sát $\varphi_e = \arctan b_e$ tính bằng `radian`, căn cứ địa kỹ thuật cho $S_e(t) = 1$, và mô hình $\sigma'_{n,e}$ hợp lệ). Trong MVP, khi không có lớp tiên nghiệm hàng hải duyệt (`FR-008`), LP-GISTA chạy hợp lệ với `prior_mode = uniform_w1` ($w_e(t) = 1$).

---

## 3. Huấn luyện, hiệu chuẩn và quy tắc ngưỡng

### 3.1. Kỷ luật sử dụng từng phân hoạch

Dẫn chiếu quy tắc tạo phân hoạch độc lập theo nhóm sự cố (`incident_group_id`) và giữ nguyên `200 cal + 200 test` tại [DATA.md §4.1](DATA.md#41-phân-biệt-ba-họ-tập-dữ-liệu-data-002-q-05-q-06), mọi bước trong `C4` tuân thủ bảng cô lập sau (`ML-005`, `ML-006`, `ARCHITECTURE.md §1.6, §5.2`):

| Tập | Được dùng để làm gì | Không được dùng để làm gì |
| --- | --- | --- |
| **`train_fit`** *(tách nhóm sự cố bên trong `train`; cấu hình khởi đầu `C3` đã duyệt: `n_train_fit = 1.600`, phân bổ `880/480/240`)* | Khớp toàn bộ tham số học được $\Theta = \{\{\eta_k, \lambda_k, \gamma_k\}_{k=0}^{11}, \Delta\mathbf{R}_\theta, a_{\mathrm{cal}}, b_{\mathrm{cal}}\}$; đếm tỷ số trọng số lớp $N_{\text{âm}}/N_{\text{dương}}$ cho hàm mất mát; đóng băng đường cơ sở $b_m = \mathrm{minRTT}_m^{\mathrm{base}}$ (`ms`) và mọi tham số chuẩn hóa đặc trưng. | Không dùng làm điểm nghiệm thu công bố; không trộn nhóm sự cố với `val_tau`, `cal` hay `test`; không rút mẫu từ `200 cal` hay `200 test`. |
| **`val_tau`** *(tách nhóm sự cố độc lập bên trong `train`; cấu hình khởi đầu `C3` đã duyệt: `n_val_tau = 400`, phân bổ `220/120/60`)* | **Chỉ** dùng để quét lưới chọn ngưỡng cắt cứng tự động $\tau_{\mathrm{cut}} \in [0{,}50; 0{,}99]$ bước `0,01` (đúng `ARCHITECTURE.md §1.6`). | Không dùng cập nhật trọng số $\Theta$, không dùng đóng băng $b_m$ hay chuẩn hóa, không dùng ước lượng phân vị Split Conformal $\hat{q}_{0{,}90}$, **không dùng tối ưu thêm ngưỡng $\tau_{\mathrm{F1}}$** và không dùng báo điểm nghiệm thu cuối. |
| **`cal`** *(`200 cal` giữ nguyên vẹn: `110/60/30`)* | **Dùng độc quyền** để tính điểm bất phù hợp $s_i$ trên các biến cố có lỗi ($\mathcal{F}_i \neq \varnothing$, `170` ca) và ước lượng phân vị Split Conformal $\hat{q}_{0{,}90}$. | Không dùng huấn luyện trọng số $\Theta$ (kể cả $a_{\mathrm{cal}}, b_{\mathrm{cal}}$), không dùng đóng băng $b_m$, không dùng chọn kiến trúc/siêu tham số, không dùng quét $\tau_{\mathrm{cut}}$ và **không dùng tối ưu thêm ngưỡng $\tau_{\mathrm{F1}}$**. |
| **`test`** *(`200 test` khóa cứng: `110/60/30`)* | **Khóa hoàn toàn cho nghiệm thu độc lập một lần duy nhất ở chế độ chỉ đọc** (`evaluate_locked_test`) sau khi đã khóa mô hình, $b_m$, $\tau_{\mathrm{cut}}$, $\hat{q}_{0{,}90}$ và `run_provenance.json`. | Không gắn vào môi trường tiến trình `train_and_calibrate`; không dùng chọn kiến trúc, siêu tham số, ngưỡng cắt hay phân vị; **không dùng chọn runtime trình duyệt** (`ADR-01`–`ADR-04`). |

> **Hàm mất mát huấn luyện LP-GISTA trên `train_fit` (cấu hình đã duyệt triển khai `C4`):** `RESEARCH.md §3.4` đặc tả 12 lớp trải phẳng và bộ tham số $\Theta$ nhưng không ghi tường minh hàm mất mát huấn luyện riêng cho LP-GISTA. Cấu hình đã duyệt sử dụng **Weighted Binary Cross-Entropy trên logits** $z_e = a_{\mathrm{cal}}\, x_e^{(12)} - b_{\mathrm{cal}}$ (trước hàm $\sigma$), **chỉ tính trên các cặp `(biến cố, phân đoạn)` có nhãn và khả kiến ($c_{\mathrm{obs}}(e) \ge 1$)**; gán trọng số dương $w_+ = N_{\text{âm}} / N_{\text{dương}}$ (kiểm tra $N_{\text{dương}} > 0$) và trọng số âm bằng `1`, trong đó $N_{\text{âm}}, N_{\text{dương}}$ được đếm trực tiếp trên tập các phân đoạn có nhãn và khả kiến trong `train_fit`. Kết hợp ràng buộc dương cho $\eta_k, \lambda_k, \gamma_k, a_{\mathrm{cal}}$ và khóa bộ tối ưu, siêu tham số, `seed` từng tập trong `run_provenance.json` trước khi huấn luyện.

### 3.2. Phân biệt Phân vị Split Conformal ($\hat{q}_{0{,}90}$) và Ngưỡng cắt cứng ($\tau_{\mathrm{cut}}$) (`Q-03`, `ML-005`, `ML-006`)

Hai ngưỡng có nguồn ước lượng, công thức và vai trò quyết định hoàn toàn tách biệt:

| Khía cạnh | Phân vị Split Conformal ($\hat{q}_{0{,}90} \Rightarrow 1 - \hat{q}_{0{,}90}$) | Ngưỡng cắt cứng tự động ($\tau_{\mathrm{cut}}$) |
| --- | --- | --- |
| **Tập ước lượng độc quyền** | Tập hiệu chuẩn `200 cal` (lấy trên $n_{\mathrm{fault,cal}}$ biến cố có lỗi thực tế $\mathcal{F}_i \neq \varnothing$). | Tập validation `val_tau` tách rời theo nhóm sự cố nằm hoàn toàn trong `train` (tuyệt đối không chạm `cal` hay `test`). |
| **Công thức / Quy tắc chọn** | Tính điểm bất phù hợp $s_i = 1 - \min_{e \in \mathcal{F}_i} \hat{p}_{i,e}$ cho từng biến cố có lỗi khả kiến trong `cal`; lấy phân vị $\hat{q}_{1-\alpha}$ ứng với bậc $\frac{\lceil (n_{\mathrm{fault,cal}} + 1)(1 - \alpha) \rceil}{n_{\mathrm{fault,cal}}}$ tại mức sai số danh định $\alpha = 0{,}10$. | Quét lưới $\tau \in [0{,}50;\, 0{,}99]$ bước `0,01` trên `val_tau`: chọn ngưỡng cho **recall phát hiện đứt hoàn toàn cao nhất** thỏa mãn **`< 5%` kịch bản có ít nhất một liên kết không đứt bị cắt nhầm**; nếu nhiều ngưỡng cùng đạt recall cao nhất thì **chọn ngưỡng lớn hơn**. |
| **Vai trò vận hành** | Lập tập nghi vấn $\mathcal{C}_{0{,}90}(t) = \{e : \hat{p}_e(t) \ge 1 - \hat{q}_{0{,}90}\}$ với mục tiêu bao phủ biên $\mathbb{P}(\mathcal{F} \subseteq \mathcal{C}_{0{,}90}) \ge 0{,}90$ dưới giả thiết hoán đổi được; dùng cho **chiết khấu mềm** $\tilde{C}_e = C_e(1 - \hat{x}_e)$ khi bật chiết khấu. | Dùng cho **cắt cứng tự động** $\tilde{C}_e(t) = 0$ đối với phân đoạn có $\hat{p}_e(t) \ge \tau_{\mathrm{cut}}$ khi cả `apply_inference_discount` và `auto_cut_policy.enabled` đều bật. |

### 3.3. Điều kiện tắt cắt tự động và ưu tiên tuyệt đối của cắt thủ công (`FR-011`, `ML-006`, `Q-03`)

- **Điều kiện tắt cắt tự động (`auto_cut_policy.enabled = false`):** Bắt buộc tắt cơ chế cắt tự động theo xác suất (`tau_cut = null`, ghi rõ `disabled_reason`) nếu xảy ra bất kỳ điều kiện nào sau đây trên `val_tau`:
  1. Không có hoặc thiếu nhãn phân đoạn **đứt hoàn toàn** để tính recall đứt hoàn toàn.
  2. Mọi giá trị $\tau \in [0{,}50;\, 0{,}99]$ đều gây ra $\ge 5\%$ kịch bản có ít nhất một liên kết không đứt bị cắt nhầm.
- **Ưu tiên tuyệt đối của cắt giả định thủ công (`FR-011`):** Trong Luồng 3, bất kỳ phân đoạn nào nằm trong `manual_cut_segment_ids` đều bị đặt cứng **$\tilde{C}_e(t) = 0$ (`capacity_reason = manual_cut`)**, luôn có hiệu lực **bất kể `auto_cut_policy.enabled == false` hay `apply_inference_discount == false`**. Chức năng khôi phục đưa toàn bộ phân đoạn về dung lượng trước khi cắt thủ công.

---

## 4. Mô phỏng và đối chứng định tuyến dự phòng

### 4.1. Bàn giao suy luận sang dung lượng hiệu dụng $\tilde{C}_e(t)$

Tại `C8`, dung lượng hiệu dụng $\tilde{C}_e(t)$ cho từng phân đoạn $e \in \mathcal{E}_{\mathrm{seg}}$ được xác định theo **đúng 4 bậc ưu tiên loại trừ lẫn nhau** (`ARCHITECTURE.md §7.1`):

1. Nếu $e \in \texttt{manual\_cut\_segment\_ids}$ $\Rightarrow$ $\tilde{C}_e(t) = 0$ (`capacity_reason = manual_cut`).
2. Ngược lại, nếu `apply_inference_discount == true`, `auto_cut_policy.enabled == true`, phân đoạn khả kiến và $\hat{p}_e(t) \ge \tau_{\mathrm{cut}}$ $\Rightarrow$ $\tilde{C}_e(t) = 0$ (`capacity_reason = auto_cut_tau`).
3. Ngược lại, nếu `apply_inference_discount == true`, phân đoạn khả kiến và $e \in \mathcal{C}_{0{,}90}(t)$ $\Rightarrow$ $\tilde{C}_e(t) = C_e \cdot (1 - \hat{x}_e(t))$ (`capacity_reason = conformal_discount`).
4. Ngược lại (bao gồm mọi phân đoạn mang trạng thái `unobservable_na`) $\Rightarrow$ giữ nguyên $\tilde{C}_e(t) = C_e$ (`capacity_reason = normal`).

### 4.2. Bài toán quy hoạch tuyến tính tuần tự 3 pha và ràng buộc SRLG (`FR-012`, `ML-012`, `Q-03`)

Trên tập nhu cầu $\{D_d\}_{d \in \mathcal{D}}$ (`demand_tbps`) và tập đường ứng viên $\mathcal{K}_d$ từ `GraphSnapshot`:

1. **Lọc đường khả dụng và trần SRLG thích nghi:**
   - $\mathcal{K}_d^{\mathrm{avail}}(t) = \{k \in \mathcal{K}_d : \forall e \in k,\ \tilde{C}_e(t) > 0\}$ (loại bỏ mọi đường đi qua liên kết có $\tilde{C}_e(t) = 0$).
   - Tập nhóm SRLG khả dụng độc lập của hướng $d$: $G_d^{\mathrm{avail}}(t) = \{\mathrm{srlg}(k) : k \in \mathcal{K}_d^{\mathrm{avail}}(t)\}$.
   - Trần SRLG thích nghi:
     - $\lvert G_d^{\mathrm{avail}}(t) \rvert \ge 2 \Rightarrow \Theta_d = 0{,}65\, D_d$, `srlg_redundancy_lost = false`, `disconnected = false`.
     - $\lvert G_d^{\mathrm{avail}}(t) \rvert = 1 \Rightarrow \Theta_d = D_d$, **`srlg_redundancy_lost = true`**, `disconnected = false` (bắt buộc phát cảnh báo mất dự phòng SRLG).
     - $\lvert G_d^{\mathrm{avail}}(t) \rvert = 0 \Rightarrow$ cố định $\ell_d = D_d$, $f_k = 0$, **`disconnected = true`**.
2. **Hệ ràng buộc chung của cả ba pha:**
   $$\sum_{k \in \mathcal{K}_d^{\mathrm{avail}}(t)} f_k + \ell_d = D_d, \qquad f_k \ge 0,\quad 0 \le \ell_d \le D_d \qquad (\forall d \in \mathcal{D})$$
   $$\sum_{d \in \mathcal{D}} \sum_{k \in \mathcal{K}_d^{\mathrm{avail}}(t):\, e \in k} f_k \le \tilde{C}_e(t)\cdot u, \qquad 0 \le u \le 1 \qquad (\forall e : \tilde{C}_e(t) > 0)$$
   $$\sum_{k \in \mathcal{K}_d^{\mathrm{avail}}(t):\, \mathrm{srlg}(k) = g} f_k \le \Theta_d \qquad (\forall d \in \mathcal{D},\ \forall g \in G_d^{\mathrm{avail}}(t))$$
3. **Thứ tự giải tuần tự từ điển 3 pha** (với dung sai giữ tính khả thi $\varepsilon_{\mathrm{lex}} = 10^{-9}$):
   - **Pha 1 (Giảm lưu lượng rớt):** $L^* = \min \sum_{d \in \mathcal{D}} \ell_d$.
   - **Pha 2 (Giảm MLU):** cố định $\sum_{d \in \mathcal{D}} \ell_d \le L^* + \varepsilon_{\mathrm{lex}}$, giải $u^* = \min u \in [0, 1]$ (nếu $L^* = \sum_d D_d$, đặt trực tiếp $u^* = 0$).
   - **Pha 3 (Giảm độ trễ gia quyền):** giữ $\sum_{d \in \mathcal{D}} \ell_d \le L^* + \varepsilon_{\mathrm{lex}}$ và $u \le \min(1,\, u^* + \varepsilon_{\mathrm{lex}})$, giải $\min \sum_{d \in \mathcal{D}} \sum_{k \in \mathcal{K}_d^{\mathrm{avail}}(t)} \tau_k^{\mathrm{RTT}} f_k$.
4. **Kiểm chứng độc lập tính khả thi của nghiệm (`subsea-routing-verification`, `V-LP`, nguyên tắc & dung sai đề xuất `D-03`):**
   - **Không coi mã trạng thái trả về từ bộ giải (HiGHS) là đủ bằng chứng khả thi.** Hậu kiểm độc lập bắt buộc kiểm tra trực tiếp trong dung sai số học: không âm ($f_k \ge 0, \ell_d \ge 0$), bảo toàn nhu cầu ($\sum_k f_k + \ell_d = D_d$), đường đi qua liên kết có $\tilde{C}_e = 0$ phải bị loại hoặc có luồng đúng `0`, giới hạn dung lượng liên kết ($\sum f_k \le \tilde{C}_e\cdot u$, $0 \le u \le 1$) và trần SRLG ($\le \Theta_d$), đồng thời kiểm riêng các ràng buộc khóa pha và **không dùng dung sai hậu kiểm chung để nới $\varepsilon_{\mathrm{lex}} = 10^{-9}$**.
   - **Dung sai đề xuất `D-03` *(chưa chạy — cần xác minh cấu hình HiGHS thực tế ở cả hai backend trên fixture phát triển trước khi khóa `T5.3` và phần LP của `T8.2`)*:** hậu kiểm tuyệt đối `1e-7 Tbps` cho phần dư bảo toàn nhu cầu, giới hạn dung lượng, trần SRLG và biên luồng, `1e-8` cho biên của $u$ (HiGHS công bố dung sai khả thi mặc định `1e-7`). Với nghiệm đồng tối ưu giữa hai backend, **không buộc từng $f_k$ giống nhau**: hai backend phải độc lập vượt hậu kiểm trên cùng đầu vào, khớp trạng thái và cờ rời rạc (`srlg_redundancy_lost`, `disconnected`, `capacity_reason`), khớp chính xác `weighted_rtt_ms = null` và `NO_SERVED_FLOW` khi không có lưu lượng phục vụ, rồi đối chiếu theo ba mục tiêu: tổng tải rớt `1e-6 Tbps`, $u^*$ `1e-7` không thứ nguyên, và tổng $\sum_k \tau_k^{\mathrm{RTT}} f_k$ `1e-4 Tbps·ms`.
   - **Không tự sửa đầu vào không hợp lệ:** nếu `demand_tbps < 0`, `manual_cut_segment_ids` chứa ID không có trong `GraphSnapshot`, `capacity_tbps < 0` hoặc `srlg_cap_ratio` ngoài $(0, 1]$ $\Rightarrow$ trả `status = invalid_input` kèm lỗi có kiểu rõ ràng (không tự kẹp về `0` hay thay dung lượng mặc định).

### 4.3. Đối sánh định tuyến đường ngắn nhất và quy tắc báo cáo đồng thời (`ARCHITECTURE.md §1.6, §7.2`)

Mốc đối sánh đường ngắn nhất mặc định chạy trên **chính xác cùng `snapshot_id`/`topology_hash`, cùng tập đường khả dụng $\mathcal{K}_d^{\mathrm{avail}}(t)$, cùng dung lượng hiệu dụng $\tilde{C}_e(t)$ và cùng ma trận nhu cầu $\{D_d\}$**:

1. **Chọn đường ngắn nhất và chia đều khi hòa:** Loại bỏ các đường trùng tập phân đoạn, quy đổi và so sánh $\tau_k^{\mathrm{RTT}}$ ở **độ phân giải cố định `µs` (micro-giây)**; xác định $\mathcal{K}_d^{\min}$ là tập các đường có RTT nhỏ nhất bằng nhau. Nếu $\mathcal{K}_d^{\min} = \varnothing$ thì toàn bộ $D_d$ bị rớt; ngược lại chia đều tải gán $a_k = D_d / \lvert \mathcal{K}_d^{\min} \rvert$ cho mọi $k \in \mathcal{K}_d^{\min}$.
2. **Phân bổ thực chuyển khi nghẽn liên kết:** Khởi tạo $\mathcal{A} = \bigcup_d \mathcal{K}_d^{\min}$, $f_k^{\mathrm{sp}} = 0$, $C_e^{\mathrm{rem}} = \tilde{C}_e(t)$. Lặp tới khi $\mathcal{A} = \varnothing$:
   - Tính bước tăng cực đại chung theo tỷ lệ tải đã gán:
     $$\delta^* = \min\left(\min_{k \in \mathcal{A}} \frac{a_k - f_k^{\mathrm{sp}}}{a_k},\ \min_{e:\, \sum_{k \in \mathcal{A}:\, e \in k} a_k > 0} \frac{C_e^{\mathrm{rem}}}{\sum_{k \in \mathcal{A}:\, e \in k} a_k}\right)$$
   - Cập nhật $f_k^{\mathrm{sp}} \leftarrow f_k^{\mathrm{sp}} + \delta^* a_k$ và $C_e^{\mathrm{rem}} \leftarrow C_e^{\mathrm{rem}} - \delta^* \sum_{k \in \mathcal{A}:\, e \in k} a_k$.
   - Cố định và loại khỏi $\mathcal{A}$ mọi đường đã đạt $f_k^{\mathrm{sp}} = a_k$ hoặc đi qua liên kết vừa chạm $C_e^{\mathrm{rem}} = 0$; tiếp tục vòng kế với các đường còn dư.
3. **Bất biến cấm vi phạm:** Tuyệt đối **không ưu tiên theo ID**, **không chuyển sang đường dài hơn** và **không dồn phần lưu lượng rớt của đường bị nghẽn sang đường hòa khác**.
4. **Quy tắc báo cáo đồng thời bắt buộc (`FR-012`, `Q-03`, `UX-004`):**
   - Mọi bảng/màn hình đối sánh phải hiển thị **đồng thời** cho cả LP và đường ngắn nhất: **lưu lượng rớt (`Tbps`)**, **MLU thực chuyển (`%`, trong $[0, 100\%]$)** và **độ trễ trung bình gia quyền (`ms`, trả `null` kèm `NO_SERVED_FLOW` nếu `total_served_tbps == 0`)**, kèm `topology_hash` và trạng thái khả thi.
   - **Tách riêng tải danh nghĩa khỏi MLU thực chuyển:** Báo riêng `raw_unconstrained_overload_pct` (tính từ $\max_e \sum a_k / \tilde{C}_e$, có thể vượt `100%` như mốc `120,0%` trên kịch bản `82,0 Tbps`) khỏi `mlu_pct` thực chuyển ($\max_e \sum f_k^{\mathrm{sp}} / \tilde{C}_e \in [0, 100\%]$).
   - **Tuyệt đối không dùng MLU thấp (kể cả `0%` khi mất kết nối) để kết luận mạng vẫn phục vụ tốt.**

### 4.4. Bộ fixture kiểm chứng bắt buộc của bộ giải định tuyến (`ML-012`, `V-LP`)

| Ca kiểm thử (`V-LP`) | Đầu vào kịch bản mô phỏng | Đầu ra kỳ vọng bắt buộc |
| --- | --- | --- |
| **1. Không sự cố & sự cố đơn phân đoạn đủ dung lượng** | Mạng đầy đủ hoặc cắt `1` phân đoạn nhưng còn $\ge 2$ hành lang SRLG đủ tải dưới trần `0,65 D_d`. | `total_dropped_tbps = 0`, `mlu_pct <= 100%`, `srlg_redundancy_lost = false`, `100%` bảo toàn luồng. |
| **2. Kịch bản `82,0 Tbps` (mô phỏng ngày `25/08/2026`, `RESEARCH.md §3.6`)** | Tổng cầu `82,0 Tbps` (`32` HK, `28` SG, `16` Tokyo, `6` EU), `VSTN = 8,0 Tbps`; cắt 4 nhánh HK (`SJC2_HK`, `AAE1_HK`, `AAG_S1`, `IA_HK`), còn `40,0 Tbps` trực tiếp HK. | *Fixture kiểm chứng mô phỏng (không phải số đo thực tế):* LP đạt `0 Tbps` rớt, `MLU = 67,4%` (giảm quá tải cân bằng qua SG & VSTN); đường ngắn nhất cho tải danh nghĩa `120,0%` trên hành lang HK, tương ứng `MLU thực chuyển = 100,0%` kèm `8,0 Tbps` rớt. |
| **3. Ca A bắt buộc (`10 Tbps` / còn `1` hành lang `8 Tbps`, `Q-03`)** | $D_d = 10\ \mathrm{Tbps}$; chỉ còn duy nhất `1` hành lang SRLG khả dụng với $\tilde{C} = 8\ \mathrm{Tbps}$. | Tự nâng trần SRLG từ `6,5` lên `10 Tbps`, bật **`srlg_redundancy_lost = true`**, chuyển đúng **`8 Tbps`**, rớt đúng **`2 Tbps`**, **`MLU = 100%`**. |
| **4. Ca B bắt buộc (`10 Tbps` / mất kết nối, `Q-03`)** | $D_d = 10\ \mathrm{Tbps}$; toàn bộ đường đi qua liên kết $\tilde{C}_e = 0$ ($\mathcal{K}_d^{\mathrm{avail}} = \varnothing$). | Bật **`disconnected = true`**, chuyển **`0 Tbps`**, rớt đúng **`10 Tbps`**, **`MLU = 0%`**, `weighted_rtt_ms = null` (`NO_SERVED_FLOW`). |
| **5. Ca nghẽn đối chứng đường ngắn nhất (`12 & 8 / 10 Tbps`, `ARCHITECTURE.md §7.2`)** | Hai hướng có đường ngắn nhất gửi `12 Tbps` và `8 Tbps` qua cùng một liên kết dung lượng `10 Tbps`. | Tỷ lệ tải gán `12 : 8` $\Rightarrow \delta^* = 0{,}5$: đường ngắn nhất thực chuyển đúng **`6 Tbps` và `4 Tbps`**, rớt đúng **`6 Tbps` và `4 Tbps`**, `raw_unconstrained_overload_pct = 200%`, `mlu_pct = 100%`. |
| **6. Ca biên `zero-demand` & `invalid_input`** | (a) $D_d = 0$; (b) $D_d < 0$ hoặc `manual_cut_segment_ids` không hợp lệ hoặc `srlg_cap_ratio` ngoài $(0, 1]$. | (a) `total_dropped_tbps = 0`, `mlu_pct = 0`, `weighted_rtt_ms = null` (`NO_SERVED_FLOW`); (b) trả `status = invalid_input`, không tự sửa dữ liệu. |

---

## 5. Giao thức đánh giá, đối sánh và giới hạn diễn giải

### 5.1. Năm cấu hình đối sánh (`B1`, `B2`, `A1`, `A2`, `Full`) và tính đồng nhất điều kiện

Mọi cấu hình đối chứng, biến thể cắt bỏ và bản lượng tử hóa (nếu phát hành) bắt buộc chạy trên **cùng `GraphSnapshot` đóng băng, cùng phân hoạch khóa trước (`train_fit`/`val_tau`/`200 cal`/`200 test`), cùng `random_seed` và cùng mã chấm điểm** (`AGENTS.md §6`, `subsea-evaluation`). **Loại bỏ hoàn toàn chỉ số `Accuracy` tổng thể** do mất cân bằng lớp nghiêm trọng (`ML-010`, `RESEARCH.md §6.1`).

| Mã cấu hình | Mô tả cấu hình | Vai trò khoa học trong đối sánh |
| --- | --- | --- |
| **`B1`** | Heuristic đồng rút tuyến BGP ($\mathbf{R}_t^\top \tilde{\mathbf{y}}^{\mathrm{BGP}}$) | Mốc cơ sở đơn tầng đo tác động của cộng tuyến hành lang và nhiễu BGP phi vật lý khi thiếu cả giải ngược thưa lẫn cổng RTT. |
| **`B2`** | Cảm biến nén L1 đồng nhất (`60` vòng lặp ISTA cổ điển, chỉ BGP) | Tách riêng đóng góp của giải ngược thưa L1 cổ điển (chưa trải phẳng, chưa học tham số theo tầng, chưa có cổng $\Delta\mathrm{minRTT}$). |
| **`A1`** | Cắt bỏ cổng độ trễ RIPE Atlas $\Delta\mathrm{minRTT}$ khỏi LP-GISTA | Cô lập đóng góp của cổng vật lý chéo tầng trong việc khử `Distractor FAR` và giữ tín hiệu khi BGP bị che khuất bởi bảo vệ quang tầng 1. |
| **`A2`** | Cắt bỏ tiên nghiệm hàng hải – địa vật lý (`prior_mode = uniform_w1`, $w_e(t) = 1$) | Cô lập đóng góp của vector trọng số thích nghi $\mathbf{w}(t)$ trong việc phá vỡ cộng tuyến giữa các cáp song song và thu hẹp tập $\mathcal{C}_{0{,}90}$. |
| **`Full`** | SubseaBGP-Tomography đầy đủ (LP-GISTA 12 lớp + cổng $\Delta\mathrm{minRTT}$ + $\mathbf{L}_{\mathrm{SRLG}}$ + $\mathbf{w}(t)$ + Split Conformal) | Hệ thống mục tiêu trên bộ kiểm chuẩn đường cơ sở mới (`synthetic_benchmark_prior` trong benchmark). |

### 5.2. Tách biệt ba cổng nghiệm thu `Q-04` (`ML-010`, `ML-011`, `ML-012`)

Tuyệt đối **không áp các ngưỡng định lượng của tập bán tổng hợp lên danh mục sự cố lịch sử thực tế** và **không nhận số liệu nguyên mẫu cũ từ `[1]` (`unknown_url`) là kết quả tái lập** (`Q-04`, `Q-06`):

| Cổng nghiệm thu | Phạm vi & Bản chất dữ liệu | Chỉ tiêu bắt buộc (`Q-04`) | Quy tắc báo cáo & Lưu ý chống hiểu nhầm |
| --- | --- | --- | --- |
| **Cổng 1 — Đường cơ sở thực nghiệm mới (`ML-010`)** | Tập `200 test` khóa trước (`110/60/30`) của bộ kiểm chuẩn bán tổng hợp xây mới (`24` phân đoạn, `48` kênh, `synthetic_calibrated_baseline`). | Đạt đồng thời:<br>- `Hit@1 ≥ 90%`<br>- `Hit@3 ≥ 95%`<br>- `PR-AUC > 0,85`<br>- `Distractor FAR < 5%`<br>- **Độ phủ Conformal thực nghiệm `≥ 88,5%`** trên các ca có lỗi (mức danh định `90%`)<br>- **Kích thước tập TB `≤ 3` phân đoạn**<br>- Nếu phát hành bản lượng tử hóa: **$\Delta\mathrm{PR\text{-}AUC}_{\mathrm{quant}} < 0{,}005$** | Báo cáo kèm `MRR`, `ECE` và **`F1` phụ chấm trực tiếp tại ngưỡng Conformal đã khóa $1 - \hat{q}_{0{,}90}$** trên các phân đoạn khả kiến có nhãn (không tối ưu thêm $\tau_{\mathrm{F1}}$, giữ đơn mục đích cho `cal` và `val_tau` — khác với `RESEARCH.md §6.1`). Đối sánh đủ `B1`, `B2`, `A1`, `A2`, `Full`; ghi rõ là **"đường cơ sở thực nghiệm mới"**. Số liệu cũ trong `RESEARCH.md §6.2` (`92,4%`, `98,3%`, `0,879`, `3,6%`, `89,5%`, `2,02`) chỉ chú thích là mốc tham chiếu nguyên mẫu cũ `[1]`, **không gọi là tái lập**. |
| **Cổng 2 — Danh mục sự cố lịch sử thực tế (`ML-011` — *chưa khóa danh sách*)** | Ít nhất **`3` sự cố lịch sử độc lập** thuộc khung `test` (`observed_measurement` + `stylized_derived`; hai ca năm `2024` chỉ dùng minh họa/hồi quy phát triển). | - **`≥ 3`** sự cố độc lập đã xác minh đủ dữ liệu phát lại ([DATA.md §4.1](DATA.md#41-phân-biệt-ba-họ-tập-dữ-liệu-data-002-q-05-q-06))<br>- **`100%`** ca truy vết đủ nguồn công bố, mốc `UTC` đúng độ chính xác, `GraphSnapshot` đóng băng và trạng thái `NA`<br>- **`0` vi phạm** 5 chốt chống rò rỉ (`V-LEAKAGE`)<br>- Tuân thủ `ADR-05` ($\Delta\mathbf{R}_\theta = \mathbf{0}$, `prior_mode = uniform_w1`, quy tắc $b_m$ không mượn cửa sổ trước ca `test`, không gán bảo chứng phủ `90%` nếu chưa hiệu chuẩn) | Tách riêng báo cáo ca có `label_granularity = segment` với ca chỉ có `label_granularity = system`. **Không áp cổng `Hit@1`/`PR-AUC`/`Coverage` của bộ bán tổng hợp lên tập lịch sử cỡ nhỏ hoặc thiếu nhãn phân đoạn.** |
| **Cổng 3 — Bộ giải định tuyến LP (`ML-012`, `V-LP`)** | Toàn bộ bộ fixture mô phỏng (`simulated_assumption`) tại Mục 4.4. | - **`100%`** fixture bảo toàn luồng & $0 \le u \le 1$<br>- **`0 Tbps` rớt và `MLU ≤ 100%`** khi đủ dung lượng dưới trần SRLG<br>- Đạt đủ ca `82,0 Tbps`, hai ca `10 Tbps` và ca nghẽn `12 & 8 / 10 Tbps` | Công bố đồng thời `Tbps` rớt, `MLU (%)` thực chuyển, `raw_unconstrained_overload_pct`, `weighted_rtt_ms` và `topology_hash`. |

### 5.3. Quy tắc chấm `Distractor FAR`, báo cáo mẫu số, khoảng tin cậy và giới hạn diễn giải

1. **Quy tắc chấm `Distractor FAR` đã duyệt (áp chung cho cả 5 cấu hình):**
   - Mỗi biến cố nhiễu phi vật lý ($\mathcal{F}_i = \varnothing$) được tính **đúng một lần**: bị tính là **báo động giả** nếu tồn tại **ít nhất một cửa sổ chấm đã khóa trước** thỏa $\mathcal{C}_{0{,}90}(t) \neq \varnothing$.
   - Dùng đúng một quy tắc chấm cho cả `B1`, `B2`, `A1`, `A2`, `Full`, với phân vị Conformal $\hat{q}_{0{,}90}$ được **hiệu chuẩn riêng cho từng cấu hình** trên cùng `200 cal`.
   - **Xử lý ca `NA`:** Báo tường minh số ca nhiễu chưa xác định do `NA` (`n_dist_NA / 30`) và **tuyệt đối không tính các ca `NA` là âm tính đúng**; tính tỷ lệ `Distractor FAR` trên mẫu số các ca nhiễu **chấm được** ($n_{\mathrm{dist,scorable}} = 30 - n_{\mathrm{dist,NA}}$). Với đủ `30` ca nhiễu chấm được trong `200 test`, `1` báo động giả tương ứng **`3,33%`** (đạt `< 5%`), còn `2` báo động giả tương ứng **`6,67%`** (vượt cổng `< 5%`).
2. **Bắt buộc báo tử số, mẫu số, khoảng tin cậy và quy ước chấm điểm đã khóa (`ML-010`, `Q-04`, `D-02`, `subsea-evaluation`):**
   - Với cấu hình khởi đầu `C3` (`200 test` gồm `110` đứt đơn + `60` đa cáp SRLG = `170` ca có lỗi và `30` ca nhiễu, ghi rõ số thực tế các ca đứt đơn bị che khuất BGP theo xác suất `25%`), độ phủ Conformal thực nghiệm trên các ca có lỗi phải công bố dạng `k / n_fault (p%)` kèm **khoảng tin cậy Wilson hai phía 95%** (cổng `≥ 88,5%` áp dụng cho **ước lượng điểm**, không áp dụng cho cận dưới CI).
   - **Quy ước khóa `D-02` áp dụng đồng nhất cho cả 5 cấu hình:** xếp hạng bằng điểm chưa làm tròn và phá hòa bi quan (trong nhóm đồng điểm, mọi phân đoạn không lỗi đứng trước phân đoạn lỗi); với đa cáp SRLG, `Hit@k` cần ít nhất 1 lỗi thật trong top-$k$, `MRR` dùng hạng của lỗi thật đầu tiên, còn độ phủ Conformal yêu cầu chứa toàn bộ lỗi thật; `ECE` dùng `10` bin đều trên $[0, 1]$, trái đóng phải mở, bin cuối chứa `1` trên các cặp có nhãn và khả kiến; cổng `Q-04` dùng toàn bộ ca có lỗi (`n_fault_total`) cho `Hit@k` và độ phủ đồng thời báo riêng trên các ca có tất cả lỗi thật khả kiến (ca đa cáp còn 1 lỗi khả kiến trong top-$k$ vẫn có thể đạt `Hit@k`, nhưng chỉ 1 lỗi không khả kiến cũng đủ làm ca đó trượt độ phủ); `PR-AUC` giữ mẫu số theo cặp có nhãn và khả kiến, không gán điểm `0` cho `NA`.
   - Công bố tương tự tử số/mẫu số cho `Hit@1`, `Hit@3` (trên $n_{\mathrm{fault,total}}$ và phân rã toàn khả kiến), `F1` tại $1 - \hat{q}_{0{,}90}$ (trên phân đoạn khả kiến có nhãn), `Distractor FAR` (trên $n_{\mathrm{dist,scorable}}$ kèm `n_dist_NA / 30`), và kích thước tập trung bình $\lvert \mathcal{C}_{0{,}90} \rvert$ tách riêng cho tập ca có lỗi và toàn bộ `200 test`.
   - **Không bao giờ tuyên bố đạt bảo chứng độ phủ `90%` trước khi hiệu chuẩn đã chạy và được kiểm chứng**, và nhấn mạnh bảo chứng Conformal chỉ là xác suất biên mẫu hữu hạn dưới giả thiết hoán đổi được (`REQUIREMENTS.md §9.3`).
3. **Báo cáo tường minh trường hợp `NA`, âm tính giả và dương tính giả:**
   - Mọi báo cáo đánh giá phải có bảng kê riêng số cửa sổ/ca bị `degraded_partial_na`, `insufficient_observations`, số phân đoạn `unobservable_na`, cùng danh mục ca bỏ sót (như chuyển mạch bảo vệ quang tầng 1) và báo động giả (như nhiễu cáp trục nội địa).
   - **Nghiêm cấm loại ngầm các ca `NA` hay ca không khả kiến khỏi mẫu số** mà không báo cáo riêng cả mẫu số tổng lẫn mẫu số khả kiến.
4. **Tính trung thực của tuyên bố (`AGENTS.md §6`, `subsea-research-integrity`):**
   - Mọi con số phải gắn nhãn bản chất (`observed_measurement`, `synthetic_calibrated_baseline`, `simulated_assumption`). Một bộ kiểm thử đơn vị hay parser vượt qua không thay thế cho bằng chứng kiểm định khoa học hay kiểm tra hiệu năng trình duyệt M1.

---

## 6. Đóng gói mô hình, tái lập và giám sát trôi dạt (Giai đoạn 3)

### 6.1. Đóng gói ONNX FP32, đối chiếu WebGPU/WASM và cổng lượng tử hóa (`ADR-02`, `NFR-004`, `V-FALLBACK`)

1. **Ưu tiên ONNX FP32 cho MVP (`ADR-02`):**
   - Mô hình LP-GISTA 12 lớp sau huấn luyện trên PyTorch (`train_fit`) được xuất mặc định sang **ONNX FP32** (`quantization = "fp32"` trong `ReleaseManifest.model_artifact`) để tránh rủi ro sai số lượng tử hóa ở đường cơ sở đầu tiên và tương thích nhất quán trên cả hai backend của ONNX Runtime Web (`WebGPU` ưu tiên, `WASM` dự phòng).
   - Tách rõ đồ thị tĩnh cho chế độ `in_profile` (có $\Delta\mathbf{R}_\theta$ khóa bởi $\mathbf{M}_{\mathrm{mask}}$) và chế độ chuyển giao `zero_delta_r_transfer` ($\Delta\mathbf{R}_\theta = \mathbf{0}$, chấp nhận kích thước $(M, E)$ biến thiên theo `GraphSnapshot` đóng băng).
2. **Kiểm chứng tương đương giữa WebGPU, WASM và tham chiếu ngoại tuyến (`V-FALLBACK`, `NFR-004`):**
   - Chạy cùng tập kịch bản kiểm thử (gồm cả kịch bản đầy đủ kênh, một phần `NA`, toàn bộ `NA`, và chuyển giao `ADR-05`) trên ba môi trường: **(a) Python/PyTorch tham chiếu ngoại tuyến**, **(b) ONNX Runtime Web `WebGPU`**, và **(c) ONNX Runtime Web `WASM` dự phòng** (ép tắt WebGPU).
   - **Tiêu chí đạt `V-FALLBACK` cho FP32 (đã duyệt):** Đối chiếu từng backend (`WebGPU`, `WASM`) độc lập với bản tham chiếu ngoại tuyến yêu cầu **`max_abs_diff <= 1e-4` cho cả $\hat{\mathbf{x}}$ và $\hat{\mathbf{p}}$**; mặt nạ `m_valid`, cờ `NA` và mọi trạng thái (`status`, `observability_status`, `inference_mode`, `conformal_calibration_status`) phải **khớp chính xác**; đồng thời kiểm riêng các ca quyết định sát ngưỡng Conformal ($1 - \hat{q}_{0{,}90}$) và $\tau_{\mathrm{cut}}$ cùng nghiệm LP $(\ell_d, u, f_k)$ khi nối sang `C8`. **Nếu sai số số học đạt `<= 1e-4` nhưng quyết định tập $\mathcal{C}_{0{,}90}$ hoặc cắt theo $\tau_{\mathrm{cut}}$ khác tham chiếu ngoại tuyến thì vẫn không đạt (`FAIL`).**
   - Benchmark tốc độ/bộ nhớ trình duyệt trên máy chuẩn M1 (`NFR-003`, `V-PERF-UX`: LP `p95 < 15 ms` cả 3 pha, thao tác $\rightarrow$ hiển thị `p95 ≤ 100 ms`, `≥ 55 FPS` trung vị trong 30 giây, GPU logic `< 150 MB`, `WASM ≤ 210 MB`, tải thiết yếu `< 15 MB`, tổng tải `< 65 MB`) **chỉ dùng workload phát triển, tuyệt đối không dùng `200 test` khóa để chọn runtime**.
3. **Cổng lượng tử hóa (chỉ áp dụng nếu tối ưu sang FP16/INT8 sau FP32):**
   - Chỉ được phép đưa bản `fp16` hoặc `int8_mixed` vào `ReleaseManifest` khi: (a) mức giảm tuyệt đối **`ΔPR-AUC < 0,005`** so với bản FP32 trên cùng tập đánh giá (`ML-010`), (b) đạt đồng thời toàn bộ cổng `Q-04` khác (không tụt độ phủ Conformal dưới `88,5%`, không vượt `Distractor FAR < 5%`), và (c) ghi tường minh `quantization` cùng `delta_pr_auc_vs_fp32` vào `ReleaseManifest.model_artifact` và `run_provenance.json`.

### 6.2. Hồ sơ tái lập `run_provenance.json` (`ADR-06`, `ML-013`, `NFR-010`)

Theo `ADR-06`, MVP **không bắt buộc DVC hay MLflow** (không dựng tracking server hay DVC remote). Mọi lượt sinh benchmark (`C3`), huấn luyện/quét ngưỡng/hiệu chuẩn/xuất ONNX/đánh giá (`C4`) và đóng gói (`C5`) bắt buộc sinh tệp **`run_provenance.json`** bất biến gồm đầy đủ các trường sau:

| Nhóm trường | Nội dung bắt buộc trong `run_provenance.json` |
| --- | --- |
| **Định danh & Bản chất lượt chạy** | `schema_version`, `run_id`, `run_type` (`benchmark_gen`, `train_and_calibrate`, `evaluate_locked_test`, `historical_eval`, `lp_regression`, `release_pack`), `baseline_label` (`"new_experimental_baseline"`), `executed_at_utc`. |
| **Mã nguồn & Phụ thuộc** | `git_commit` (SHA-1/256 đầy đủ), `git_working_tree_clean: bool`, `dependency_lockfiles[]` (`path`, `sha256` cho `uv.lock` / lockfile JS), `runtime_environment` (phiên bản Python/PyTorch/ONNX/HiGHS/OS/CPU/GPU). |
| **Cấu hình, Lệnh & Hạt giống** | `cli_command` (lệnh tái lập đầy đủ), `config_path` + `config_sha256` (gồm `prior_mode`, `mohr_coulomb_enabled = false`, hàm mất mát `weighted_bce_logits` + tỷ số $N_{\text{âm}}/N_{\text{dương}}$ trên `train_fit`, siêu tham số huấn luyện, `alpha = 0.10`, lưới `tau_cut`), `random_seeds` (tách riêng theo từng tập `train_fit`/`val_tau`/`cal`/`test` và từng thư viện Python, NumPy, PyTorch). |
| **Phân hoạch & Chống rò rỉ** | `split_manifest` (`n_train_fit = 1.600`, `n_val_tau = 400`, `n_cal = 200`, `n_test = 200`, phân bổ `55%/30%/15%`, số thực tế nhóm đứt đơn che khuất BGP theo `p = 25%`, quy tắc gom nhóm `incident_group_id`), `source_cutoffs_utc`, `anti_leakage_gate_results` (trạng thái từng chốt trong 5 chốt, bắt buộc `0` vi phạm; nếu chưa xác minh được thì `status = BLOCKED`), `locked_test_access_count` (`0` trong `train_and_calibrate`, `1` trong `evaluate_locked_test`). |
| **Dấu vết đầu vào & Artifact đầu ra** | `input_artifacts[]` (`path`, `sha256`, `data_provenance_class`, `retrieved_at_utc`), `output_artifacts[]` (`path`, `sha256` cho trọng số ONNX FP32/lượng tử hóa, `calibration_quantile_q`, `auto_cut_policy`, báo cáo đánh giá `B1`/`B2`/`A1`/`A2`/`Full`, báo cáo lịch sử, `ReleaseManifest`), cùng `unresolved_limitations[]`. |

Điều kiện khóa `ADR-06`: sao chép kho mã sạch (*clean clone*), cài phụ thuộc từ lockfile và chạy lại đúng `cli_command` phải tái tạo khớp mọi chỉ số đầu ra trong sai số dấu phẩy động tiêu chuẩn (`NFR-010`, `ML-013`).

### 6.3. Giám sát trôi dạt ở Giai đoạn 3 (`ML-014`, `Q-07` — Ngoài đường chạy MVP)

Cơ chế giám sát trôi dạt **không nằm trong đường chạy MVP** mà được đặt rõ ở **Giai đoạn 3** (`E4`, phụ thuộc đường cơ sở đã duyệt của GĐ1 và luồng thu thập định kỳ của GĐ2):

1. **Chỉ số và điều kiện tuần hợp lệ:** Tác vụ ngoại tuyến chạy hàng tuần tính khoảng cách Jensen–Shannon ($\mathrm{JSD}$) so với bản mốc đã duyệt bằng **các khoảng chia (`bin`) cố định từ bản mốc** trên 3 chuỗi: (a) mật độ/xáo trộn BGP theo bộ thu RRC, (b) chuỗi $\mathrm{minRTT}$, và (c) phần dư tái tạo $\left\lVert \mathbf{m}_{\mathrm{valid}} \odot (\mathbf{y} - \mathbf{R}_t \hat{\mathbf{x}}) \right\rVert_2$. Một tuần chỉ được coi là hợp lệ khi đạt **$\ge 100$ cửa sổ hợp lệ** và **$\ge 80\%$ dữ liệu dự kiến/tuần** (dưới ngưỡng này chỉ báo *thiếu dữ liệu đánh giá trôi dạt*, không kích hoạt cảnh báo hay cập nhật).
2. **Ngưỡng cảnh báo và kích hoạt ứng viên cập nhật:**
   - `1` tuần hợp lệ có $\mathrm{JSD} \ge 0{,}10$ trên bất kỳ chuỗi nào $\Rightarrow$ phát cảnh báo theo dõi (**bắt buộc ghi rõ trôi dạt đầu vào không tự chứng minh chất lượng định vị của mô hình giảm**).
   - **`2` tuần hợp lệ liên tiếp** có $\mathrm{JSD} \ge 0{,}10$ **hoặc** có **thay đổi tô-pô đã xác nhận** (như tuyến cáp mới chính thức RFS hoặc ngừng khai thác) $\Rightarrow$ sinh gói ứng viên cập nhật $(\mathbf{R}_t, \hat{q}_{0{,}90})$ mang phiên bản mới.
3. **Cổng phê duyệt thủ công qua Pull Request (`Q-07`):**
   - Mọi gói ứng viên phải vượt qua toàn bộ bộ kiểm thử chống rò rỉ (`V-LEAKAGE`) và kiểm định hiệu chuẩn trên nhãn độc lập.
   - Đưa gói ứng viên cùng báo cáo kiểm định vào **Pull Request bắt buộc có người phụ trách dự án duyệt thủ công** trước khi hợp nhất và phát hành qua `C5` (`ReleaseManifest`). **Nghiêm cấm tự động xuất bản hoặc tự ghi đè bản công khai.**

---

## 7. Truy vết yêu cầu và điểm cần xác nhận

### 7.1. Bảng truy vết yêu cầu mô hình, mô phỏng và đánh giá

| ID yêu cầu | Mục đặc tả trong `MODEL.md` | Phép kiểm chứng | Trạng thái |
| --- | --- | --- | --- |
| `FR-008` | Mục 2.3 (mục 4), Mục 5.1 (`A2`) | `V-CONTRACT`, `V-LEAKAGE` (chạy hợp lệ với `uniform_w1` ở MVP; đúng trọng số $w_e(t)$ khi đủ $h_e(t)$ ở GĐ2) | Hỗ trợ $w_e=1$ ở MVP; đầy đủ ở GĐ2 (`O-01`) — Chưa chạy |
| `FR-009` | Mục 1.1–1.3, Mục 2.1–2.3 | `V-CONTRACT`, `V-FALLBACK` (mặt nạ `m_valid` loại kênh `NA`, xuất $\hat{p}_e$, $\mathcal{C}_{0{,}90}$, `unobservable_na` và chú giải giới hạn) | `Req` — Chưa chạy |
| `FR-011` | Mục 1.2, Mục 3.3, Mục 4.1 | `V-LP` (cắt thủ công đặt cứng $\tilde{C}_e = 0$ kể cả khi tắt $\tau_{\mathrm{cut}}$, giải lại LP và khôi phục) | `Req` — Chưa chạy |
| `FR-012` | Mục 1.3, Mục 4.2–4.4 | `V-LP` (LP 3 pha, SRLG thích nghi, ca `10 Tbps`, đối sánh đường ngắn nhất hòa `µs` & nghẽn `12 & 8 / 10`, tách tải danh nghĩa và MLU thực chuyển) | `Req` — Chưa chạy |
| `ML-001`–`ML-004` | Mục 1.3, Mục 5.2, Mục 6.2 (dẫn chiếu [DATA.md §4.2](DATA.md#42-năm-chốt-chặn-chống-rò-rỉ-dữ-liệu-ml-001ml-006-ml-007-v-leakage)) | `V-LEAKAGE` (Chốt 1–4 xác minh `0` vi phạm trước mọi báo cáo điểm mô hình) | `Req` (`ML-002` GĐ2; `ML-003` test MVP / nguồn GĐ2) — Chưa chạy |
| `ML-005` | Mục 2.2, Mục 3.1, Mục 3.2 | `V-LEAKAGE` (khớp $\Theta$ gồm $a_{\mathrm{cal}}, b_{\mathrm{cal}}$ & đóng băng $b_m$ trên `train_fit`; ước lượng $\hat{q}_{0{,}90}$ trên `200 cal`; tháo `D_test` khi huấn luyện) | `Req` — Chưa chạy |
| `ML-006` | Mục 3.1–3.3 | `V-LEAKAGE` (quét $\tau_{\mathrm{cut}} \in [0{,}50; 0{,}99]$ bước `0,01` trên `val_tau`, hòa chọn cao hơn, tắt khi không đạt / thiếu nhãn) | `Req` — Chưa chạy |
| `ML-007` | Mục 2.3 (mục 4), Mục 6.2 | `V-LEAKAGE` (khóa tắt Mohr–Coulomb trong MVP, cấm suy $(c_e,\varphi_e)$ từ GEBCO hoặc $S_e=1$ từ không báo sự cố) | `Req` — Chưa chạy |
| `ML-008` | Mục 2.3 (mục 4) | `V-LEAKAGE` (chỉ mở `formula-6` ở GĐ2 khi vượt `O-02` với cắt trực tiếp $\ge 3$ mức `kPa`, `radian`, $\sigma'_{n,e}$) | `Cần xác nhận` (`O-02`, hoãn GĐ2) — Chưa chạy |
| `ML-009` | Mục 2.1–2.3 | `V-LEAKAGE` (LP-GISTA 12 lớp giữ hỗ trợ dương $\mathbf{M}_{\mathrm{mask}}$, chiếu đơn hình hàng $\Pi_\Delta$, đầu ra $[0,1]^E$, gồm cả chế độ chuyển giao $\Delta\mathbf{R}_\theta=\mathbf{0}$) | `Req` — Chưa chạy |
| `ML-010` | Mục 5.1–5.3, Mục 6.1 | `V-LEAKAGE` (đường cơ sở mới 24/48 đạt cổng `Q-04` trên `200 test`, đối sánh `B1`/`B2`/`A1`/`A2`/`Full`, tử số/mẫu số/CI, $\Delta\mathrm{PR\text{-}AUC}_{\mathrm{quant}} < 0{,}005$) | `Req` — Chưa chạy |
| `ML-011` | Mục 1.2, Mục 2.3, Mục 5.2 | `V-LEAKAGE` ($\ge 3$ sự cố lịch sử độc lập, `100%` truy vết, `0` rò rỉ, chuyển giao `ADR-05` không gán độ phủ `90%`, không áp ngưỡng bán tổng hợp) | `Req` — Chưa chạy |
| `ML-012` | Mục 4.2–4.4, Mục 5.2 | `V-LP` (`100%` fixture bảo toàn luồng & $0 \le u \le 1$, hậu kiểm độc lập, ca `82,0 Tbps`, hai ca `10 Tbps`, ca `12 & 8 / 10`) | `Req` — Chưa chạy |
| `ML-013` | Mục 6.2 | `V-LEAKAGE`, `V-RELEASE` (`run_provenance.json`, lockfile, SHA-256, lệnh tái lập không cần DVC/MLflow, chạy lại khớp) | `Req` — Chưa chạy |
| `ML-014` | Mục 6.3 | `V-LEAKAGE`, `V-RELEASE` (GĐ3: hàng tuần $\mathrm{JSD} \ge 0{,}10$, $\ge 100$ cửa sổ & $\ge 80\%$ dữ liệu, 2 tuần liên tiếp/đổi tô-pô mở PR có duyệt) | Hoãn GĐ3 (`Could`, `Q-07`) — Chưa chạy |
| `NFR-002`, `NFR-003` | Mục 6.1 | `V-PERF-UX` (Web Workers, `model.dispose()`, 7 cổng máy chuẩn M1 trên workload phát triển, không dùng `test` khóa) | `Req` — Chưa chạy |
| `NFR-004` | Mục 6.1 | `V-FALLBACK` (ONNX FP32 trên `WebGPU` & `WASM` đối chiếu tham chiếu ngoại tuyến, phục hồi `device.lost`) | `Req` — Chưa chạy |
| `NFR-009`, `NFR-010` | Mục 1.3, Mục 4.2, Mục 6.2 | `V-CONTRACT`, `V-LP`, `V-RELEASE` (tách lõi khỏi UI/I/O, tái lập từ *clean clone* theo `ADR-06`) | `Req` — Chưa chạy |

### 7.2. Các cấu hình đã duyệt triển khai và hạng mục vẫn mở

#### A. Cấu hình khởi đầu đã được người phụ trách xác nhận duyệt cho `C3`/`C4`

Cùng với các cấu hình dữ liệu tại [DATA.md §6.2](DATA.md#62-các-cấu-hình-đã-duyệt-triển-khai-và-hạng-mục-vẫn-mở) (`n_train_fit = 1.600`, `n_val_tau = 400`, `200 cal + 200 test`, `prior_mode = uniform_w1` cho lịch sử MVP và quy tắc đóng băng $b_m$ cho `ADR-05`), bốn điểm về mô hình và kiểm định dưới đây đã được duyệt để triển khai:

1. **Chỉ số phụ `F1` tại ngưỡng Conformal đã khóa (Mục 3.1, 5.2):** Báo `F1` tại đúng ngưỡng Conformal $1 - \hat{q}_{0{,}90}$ trên các phân đoạn khả kiến có nhãn; **không tối ưu thêm $\tau_{\mathrm{F1}}$** để giữ `cal` chỉ tính $\hat{q}_{0{,}90}$ và `val_tau` chỉ chọn $\tau_{\mathrm{cut}}$ theo `ARCHITECTURE.md §1.6` *(ghi nhận chuẩn hóa khác với mô tả chọn ngưỡng F1 trên tập hiệu chuẩn tại `RESEARCH.md §6.1`)*.
2. **Định nghĩa sự kiện báo động `Distractor FAR` (Mục 5.3):** Mỗi biến cố nhiễu tính một lần; tính là báo động giả nếu có ít nhất một cửa sổ chấm khóa trước thỏa $\mathcal{C}_{0{,}90}(t) \neq \varnothing$; áp chung cho cả 5 cấu hình với hiệu chuẩn Conformal riêng từng cấu hình; báo riêng số ca `NA` và **không tính các ca `NA` là âm tính đúng** (mẫu số là số ca nhiễu chấm được; với đủ `30` ca, `1` báo động giả là `3,33%`, `2` là `6,67%`).
3. **Hàm mất mát huấn luyện LP-GISTA trên `train_fit` (Mục 3.1):** **Weighted BCE trên logits** ($z_e = a_{\mathrm{cal}}\, x_e^{(12)} - b_{\mathrm{cal}}$), chỉ tính trên các cặp `(biến cố, phân đoạn)` có nhãn và khả kiến; trọng số dương $N_{\text{âm}}/N_{\text{dương}}$ và trọng số âm `1`, đếm trên `train_fit`.
4. **Tiêu chí `V-FALLBACK` cho FP32 (Mục 6.1):** `max_abs_diff <= 1e-4` cho cả $(\hat{\mathbf{x}}, \hat{\mathbf{p}})$ giữa từng backend (`WebGPU`, `WASM`) và tham chiếu ngoại tuyến; khớp chính xác mặt nạ, `NA` và trạng thái; kiểm riêng ca sát ngưỡng Conformal và $\tau_{\mathrm{cut}}$ — **sai số số học đạt nhưng quyết định khác vẫn không đạt (`FAIL`)**.
5. **Biên hợp đồng suy luận khi `valid_channel_count == 0` (`D-01`, Mục 2.3):** Trả `null` cho `uncalibrated_transfer` và `[]` cho `calibrated_90`; cả hai giữ `status = insufficient_observations`, phân đoạn là `unobservable_na`, và không tính là âm tính đúng trong `Distractor FAR` (`null` = không có tập Conformal được hiệu chuẩn; `[]` không mang nghĩa an toàn).
6. **Bốn quy ước chấm `200 test` (`D-02`, Mục 5.3):** Điểm chưa làm tròn + phá hòa bi quan (không lỗi đứng trước lỗi), đa cáp SRLG `Hit@k` ≥ 1 lỗi thật / `MRR` lỗi thật đầu tiên / độ phủ chứa toàn bộ lỗi thật; `ECE` `10` bin đều trên $[0,1]$ trái đóng phải mở, bin cuối chứa `1`; CI Wilson hai phía 95% với cổng `≥ 88,5%` áp cho ước lượng điểm; cổng `Q-04` dùng toàn bộ ca có lỗi kèm báo riêng các ca có tất cả lỗi thật khả kiến, `PR-AUC` không gán `0` cho `NA`.

#### B. Các hạng mục vẫn chưa khóa / giữ nguyên cổng mở rộng

1. **Xác minh dung sai đề xuất cho hậu kiểm & đối chiếu LP (`D-03`, Mục 4.2):** Đã duyệt nguyên tắc không buộc từng $f_k$ giống nhau khi có nghiệm đồng tối ưu và bộ dung sai đề xuất (`1e-7 Tbps` hậu kiểm, `1e-8` cho biên $u$, không nới $\varepsilon_{\mathrm{lex}} = 10^{-9}$; đối chiếu 3 mục tiêu `1e-6 Tbps` / `1e-7` / `1e-4 Tbps·ms`, khớp chính xác `weighted_rtt_ms = null` & `NO_SERVED_FLOW`), nhưng **chưa chạy — cần xác minh cấu hình HiGHS thực tế ở cả hai backend trên fixture phát triển trước khi khóa `T5.3` và phần đối chiếu LP của `T8.2`**.
2. **Chưa khóa danh sách nghiệm thu lịch sử `ML-011` ([DATA.md §4.1, §6.2](DATA.md#62-các-cấu-hình-đã-duyệt-triển-khai-và-hạng-mục-vẫn-mở)):** Chỉ khóa sau khi xác minh đầy đủ dữ liệu phát lại thực tế (BGP, RIPE Atlas, `GraphSnapshot`, mốc `UTC` đúng độ chính xác, cấp nhãn, trạng thái `NA`) cho ít nhất `3` ca độc lập trong khung `test`.
3. **Giữ nguyên các cổng mở rộng và ranh giới đã chốt:** Không mở lại phạm vi MVP hay `Q-01`–`Q-07`; giữ nguyên `O-01` (`DATA-009` duyệt từng tệp, tiếp tục chặn dẫn xuất Cloudflare Radar khỏi `public_demo` theo [DATA.md §6.2](DATA.md#62-các-cấu-hình-đã-duyệt-triển-khai-và-hạng-mục-vẫn-mở)), `O-02` (`ML-008` dữ liệu địa kỹ thuật phân đoạn cho Mohr–Coulomb) và `O-03` (`U3`/`U4`).
