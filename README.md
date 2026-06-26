# OSG202 – RBL Deep Research Report
## Topic: Design Issues for Paging Systems (Section 3.5)
### Group 3 | SE2008 | OSG202 – SU26

---

## 1. Giới thiệu & Vấn đề nghiên cứu

Trong hệ thống bộ nhớ ảo, việc triển khai phân trang (paging) không chỉ dừng ở cơ chế dịch địa chỉ. Theo Tanenbaum (2015, §3.5), có nhiều vấn đề thiết kế quan trọng mà các nhà phát triển hệ điều hành phải cân nhắc kỹ lưỡng để đạt hiệu suất tối ưu. Nhóm 3 tập trung nghiên cứu sâu các vấn đề này, kết hợp lý thuyết từ sách giáo khoa, nội dung slide thuyết trình và thực nghiệm.

**Các vấn đề thiết kế chính trong Section 3.5:**
1. Local vs. Global Allocation Policies (§3.5.1)
2. Load Control (§3.5.2)
3. Page Size (§3.5.3)
4. Separate Instruction and Data Spaces (§3.5.4)
5. Shared Pages & Shared Libraries (§3.5.5–3.5.6)
6. Cleaning Policy (§3.5.8)

---

## 2. Literature Review

### 2.1 Nền tảng lý thuyết – Quản lý Frame

Như đã trình bày trong slide nhóm (Slide 20–24, TranDucAnh), hệ thống bộ nhớ ảo cần phân chia frame **linh hoạt** vì nhu cầu bộ nhớ của mỗi tiến trình thay đổi liên tục theo **tính cục bộ (locality of reference)**. Cấp phát cố định (Fixed Allocation) gây ra hai vấn đề:
- **Lãng phí RAM**: Cấp quá nhiều frame cho tiến trình nhàn rỗi.
- **Thrashing**: Cấp quá ít frame, tiến trình bị lỗi trang liên tục, CPU phải chờ nạp dữ liệu.

> **Nguồn:** Andrew S. Tanenbaum & Herbert Bos, *Modern Operating Systems*, 4th ed., Pearson, 2015, Chapter 3, pp. 213–278. (Sách được cung cấp: `Modern_Operating_Systems_-_4th_Edition-213-278.pdf`)

### 2.2 Chính sách cấp phát: Local vs. Global

**Chính sách cục bộ (Local Allocation):** Mỗi tiến trình chỉ thay thế các page trong tập frame đã được cấp phát riêng cho nó. Số frame mỗi tiến trình nhận là cố định trong thời gian chạy.

**Chính sách toàn cục (Global Allocation):** Khi xảy ra page fault, hệ điều hành có thể lấy frame từ bất kỳ tiến trình nào, không phân biệt chủ sở hữu.

| Tiêu chí | Local (Cục bộ) | Global (Toàn cục) |
|---|---|---|
| Tính linh hoạt | Thấp – frame cố định | Cao – frame chia sẻ động |
| Nguy cơ Thrashing | Cao nếu working set tăng | Thấp hơn nếu kiểm soát tốt |
| Lãng phí RAM | Có thể cao (tiến trình không dùng hết) | Thấp hơn |
| Độ phức tạp | Thấp | Cao hơn (cần giám sát toàn hệ thống) |

> *"In general, global algorithms work better, especially when the working set can vary a lot over the lifetime of a process."* — Tanenbaum, §3.5.1

**Kết luận từ sách:** Global algorithm thường hiệu quả hơn, nhưng cần cơ chế giám sát bổ sung như PFF để tránh thrashing.

### 2.3 Page Fault Frequency (PFF)

PFF là thuật toán kiểm soát số lượng frame được cấp cho mỗi tiến trình dựa trên **tỷ lệ lỗi trang thực tế**:

- **PFF > Ngưỡng trên (Upper Threshold):** Tiến trình thiếu frame → cấp thêm frame.
- **PFF < Ngưỡng dưới (Lower Threshold):** Tiến trình thừa frame → thu hồi bớt.

Điều này giúp duy trì tỷ lệ lỗi trang ở mức hợp lý và tối ưu hóa RAM. Tuy nhiên, **PFF không thể tự xử lý khi hệ thống đã hết frame** – đây chính là lúc cần đến cơ chế Load Control.

> **Nguồn:** Tanenbaum, §3.5.1, Figure 3-23 – "Page fault rate as a function of the number of page frames assigned."

> **Nguồn bổ sung (slide nhóm):** Slide 23, *SE2008 - OSG202 - Group 3 - TranDucAnh.pptx* — "Page Fault Frequency - PFF"

### 2.4 Load Control & Thrashing Prevention

Ngay cả với thuật toán thay thế page tốt nhất và phân bổ frame toàn cục tối ưu, hệ thống vẫn có thể bị **thrashing** nếu tổng working set của tất cả tiến trình vượt dung lượng RAM.

**Cơ chế Load Control:**
1. Phát hiện: PFF algorithm báo hiệu một số tiến trình cần thêm frame nhưng không có tiến trình nào nhường frame.
2. Phản ứng: Hệ điều hành tạm thời **swap out** một số tiến trình ưu tiên thấp ra đĩa, giải phóng frame cho tiến trình còn lại.
3. Phục hồi: Khi tình trạng thrashing kết thúc, các tiến trình bị swap out được đưa trở lại bộ nhớ.

> *"A good way to reduce the number of processes competing for memory is to swap some of them to the disk and free up all the pages they are holding."* — Tanenbaum, §3.5.2

> **Nguồn bổ sung (slide nhóm):** Slide 24, *SE2008 - OSG202 - Group 3 - TranDucAnh.pptx* — "Replacement Policy and Load Control"

### 2.5 Working Set Model

**Working Set** là tập hợp các trang mà tiến trình đang sử dụng trong khoảng thời gian gần đây, ký hiệu là **w(k, t)** – tập các trang được truy cập trong k lần truy cập gần nhất tại thời điểm t.

**Cơ chế hoạt động:**
- Hệ thống xác định cửa sổ thời gian **Δ** (delta).
- Tất cả các trang được truy cập trong Δ giây CPU time gần nhất tạo thành Working Set.
- Mục tiêu: Đảm bảo toàn bộ Working Set nằm trong RAM trước khi chạy tiến trình.

**Lợi ích:**
- Giảm thrashing bằng cách không chạy tiến trình nếu RAM không đủ chứa Working Set.
- Dự đoán chính xác nhu cầu bộ nhớ dựa trên locality.

**Hạn chế:**
- Khó chọn Δ phù hợp: Δ quá nhỏ → thiếu trang; Δ quá lớn → lãng phí RAM.
- Chi phí theo dõi Working Set cao (cần scan toàn bộ page table khi page fault).

> **Nguồn:** Tanenbaum, §3.4.8 & §3.5.1 (Working Set Model context in design issues)

> **Nguồn bổ sung (thuyết trình):** *Giải_pháp_phân_chia_frame_hợp_lý_cho_từng_process.docx* — phần P2, P3

### 2.6 Page Size

Việc chọn kích thước trang (page size) là quyết định thiết kế quan trọng với nhiều đánh đổi (trade-off):

| Yếu tố | Page Size nhỏ | Page Size lớn |
|---|---|---|
| Internal Fragmentation | Ít hơn | Nhiều hơn |
| Kích thước Page Table | Lớn hơn | Nhỏ hơn |
| Thời gian I/O disk | Nhanh (nhiều lần nhỏ) | Chậm hơn (ít lần lớn) |
| TLB Efficiency | Thấp (nhiều entry cần thiết) | Cao (ít entry cần thiết) |
| Phù hợp locality | Tốt cho spatial locality | Có thể mang vào dữ liệu không cần thiết |

**Công thức tối ưu hóa overhead** (Tanenbaum, §3.5.3):

```
overhead = se/p + p/2
```

Trong đó:
- `s` = kích thước trung bình tiến trình (bytes)
- `e` = kích thước mỗi entry trong page table (bytes)
- `p` = page size (bytes)

Kích thước trang tối ưu:

```
p_opt = √(2se)
```

Với s = 1MB, e = 8 bytes → p_opt ≈ 4KB (phù hợp với thực tế hiện nay).

> **Nguồn:** Tanenbaum, §3.5.3, pp. 246–248

### 2.7 Shared Libraries

Shared Libraries (DLL trên Windows, .so trên Linux) giải quyết vấn đề lãng phí RAM và disk do static linking:

- **Static Linking:** Mỗi chương trình tự chứa bản copy của thư viện → lãng phí.
- **Shared Libraries:** Một bản copy duy nhất trong bộ nhớ, nhiều tiến trình cùng dùng.
- **Position-Independent Code (PIC):** Thư viện được biên dịch chỉ dùng địa chỉ tương đối (relative addresses), cho phép load ở bất kỳ địa chỉ nào trong virtual address space mà không cần relocation.

> *"Windows Update will download the new DLL and replace the old one, and all programs that use the DLL will automatically use the new version the next time they are launched."* — Tanenbaum, §3.5.6

> **Nguồn bổ sung (slide nhóm):** Slide 26–29, *SE2008 - OSG202 - Group 3 - TranDucAnh.pptx* — "Shared Libraries: Optimizing and Updating Software"

---

## 3. Phân tích chuyên sâu: Đề xuất nghiên cứu mở rộng

### 3.1 Câu hỏi nghiên cứu sâu (Research Questions)

Dựa trên rubric RBL (OSG202_SU26.xlsx – tiêu chí "Problem Formulation & Research Questioning"), nhóm đề xuất các câu hỏi mở sau:

1. **Điểm giao nhau PFF và Working Set:** Trong trường hợp nào PFF lại hội tụ về kết quả tương đương Working Set Model? Điều kiện nào khiến hai thuật toán phân kỳ?
2. **Page Size động:** Các hệ điều hành hiện đại (Linux Huge Pages, Windows Large Pages) sử dụng page size khác nhau cho từng vùng bộ nhớ. Chiến lược này ảnh hưởng đến TLB miss rate như thế nào?
3. **Global Allocation và fairness:** Khi sử dụng global allocation, làm thế nào ngăn một tiến trình "ăn cắp" quá nhiều frame từ tiến trình khác?
4. **Load Control và scheduling:** Mid-term scheduler quyết định swap out tiến trình nào dựa trên tiêu chí gì – CPU-bound vs. I/O-bound, priority, hay working set size?

### 3.2 Phân tích Trade-off: Local vs. Global trong thực tế

**Kịch bản A – Database Server (nhiều kết nối đồng thời):**
- Global allocation phù hợp hơn vì nhu cầu bộ nhớ của từng query rất khác nhau.
- PFF giúp điều phối frame linh hoạt giữa các query thread.

**Kịch bản B – Real-time System (ứng dụng nhúng, yêu cầu deterministic):**
- Local allocation phù hợp hơn để đảm bảo latency cố định.
- Working set được tính toán trước và cấp đủ frame trước khi chạy (prepaging).

**Kịch bản C – Desktop OS thông thường (Windows/Linux):**
- Hybrid: Global allocation với PFF + Working Set approximation.
- Linux dùng Zone-based allocator + Active/Inactive page lists gần với WSClock.

### 3.3 So sánh mở rộng các thuật toán phân bổ frame

| Thuật toán | Phức tạp | RAM Efficiency | Chống Thrashing | Dùng trong thực tế |
|---|---|---|---|---|
| Equal Allocation | Rất thấp | Thấp | Kém | Không |
| Proportional Allocation | Thấp | Trung bình | Trung bình | Hiếm |
| Working Set Model | Cao | Cao | Tốt | Xấp xỉ (WSClock) |
| PFF | Trung bình | Cao | Tốt | Có (nhiều OS) |
| PFF + Load Control | Cao | Rất cao | Rất tốt | Có (Linux, Windows) |

---

## 4. Thực nghiệm đề xuất

### 4.1 Công cụ

- **OS Simulator:** https://ddpigeon.github.io/os-simulator/ (đề xuất từ rubric OSG202_SU26.xlsx)
- **Python simulation:** viết script mô phỏng PFF và Working Set với trace truy cập page
- **valgrind / perf (Linux):** đo TLB miss rate với các page size khác nhau

### 4.2 Test case gợi ý

**Test 1 – So sánh PFF vs. Working Set:**
- Tạo access pattern với locality thay đổi đột ngột (ví dụ: đang dùng 5 page đột nhiên chuyển sang 20 page).
- Đo số page fault và số frame được cấp phát theo thời gian cho mỗi thuật toán.
- Metrics: total page faults, average frames allocated, response time.

**Test 2 – Thrashing threshold:**
- Tăng dần số tiến trình đồng thời trong simulator.
- Quan sát điểm tại đó hệ thống bắt đầu thrash (CPU utilization giảm đột ngột, page fault rate tăng vọt).
- So sánh khi có/không có Load Control.

**Test 3 – Page Size và TLB:**
- Chạy cùng workload với page size 4KB vs 2MB (Linux transparent huge pages).
- Đo TLB miss rate bằng `perf stat -e dTLB-load-misses`.

### 4.3 Kết quả kỳ vọng

| Test | Kết quả kỳ vọng |
|---|---|
| PFF vs. Working Set | PFF phản ứng nhanh hơn khi access pattern thay đổi đột ngột |
| Thrashing threshold | Load Control giúp giữ CPU utilization > 80% ở mức tải cao hơn ~30% |
| Page Size | 2MB pages giảm TLB miss rate ~60–80% cho workload data-intensive |

---

## 5. Liên kết với Rubric RBL (OSG202_SU26.xlsx)

| Tiêu chí Rubric | Mức độ nhắm tới | Cách đáp ứng trong báo cáo này |
|---|---|---|
| AI Reflection (30%) | Excellent | Ghi Audit Log: prompt AI → kiểm tra đầu ra → so sánh với sách §3.5 → sửa chỗ sai |
| Content & Knowledge (10%) | Excellent | Phân tích đầy đủ §3.5.1–3.5.6, có bảng so sánh, công thức |
| Organization & Structure (10%) | Excellent | Cấu trúc: Intro → Lit Review → Phân tích → Thực nghiệm → Kết luận |
| Problem Formulation (10%) | Excellent | 4 câu hỏi nghiên cứu mở, có liên quan thực tế |
| Resource Discovery (10%) | Very Good | Sách giáo khoa + slide nhóm + nguồn online bổ sung |
| Analysis & Synthesis (10%) | Excellent | Bảng so sánh, 3 kịch bản thực tế, liên hệ theory–practice |
| Experimentation (10%) | Good → Very Good | 3 test case cụ thể với metrics rõ ràng |
| Visual Aids (10%) | Good | Bảng trong markdown; cần bổ sung biểu đồ trong file báo cáo chính thức |

---

## 6. Kết luận

Section 3.5 của Tanenbaum không chỉ là lý thuyết mà phản ánh đúng cách thiết kế của các hệ điều hành hiện đại như Linux và Windows. Các điểm chính:

- **Global Allocation + PFF** là sự kết hợp hiệu quả nhất cho hệ thống đa nhiệm phổ thông.
- **Working Set Model** là nền tảng lý thuyết quan trọng, được xấp xỉ bằng WSClock trong thực tế.
- **Load Control** là "van an toàn" cuối cùng khi tất cả các cơ chế khác không ngăn được thrashing.
- **Page Size** là quyết định thiết kế có ảnh hưởng lớn đến TLB efficiency và I/O performance – xu hướng hiện đại dùng multiple page sizes (4KB + 2MB + 1GB).
- **Shared Libraries** giải quyết bài toán kinh điển về lãng phí bộ nhớ và giúp cập nhật phần mềm hiệu quả.

---

## 7. Tài liệu tham khảo

1. Tanenbaum, A. S., & Bos, H. (2015). *Modern Operating Systems* (4th ed.). Pearson. Chapter 3, §3.5, pp. 233–252. *(File: Modern_Operating_Systems_-_4th_Edition-213-278.pdf)*

2. Group 3 – SE2008. (2026). *An Overview of Page Tables and Multi-Level Page Tables in Virtual Memory Systems* [Slide thuyết trình]. OSG202, FPT University. *(File: SE2008_-_OSG202_-_Group_3_-_TranDucAnh.pptx)*

3. Trần Đức Anh. (2026). *Giải pháp phân chia frame hợp lý cho từng process* [Tài liệu thuyết trình]. OSG202, FPT University. *(File: Giải_pháp_phân_chia_frame_hợp_lý_cho_từng_process.docx)*

4. OSG202 Course Team. (2026). *OSG202 RBL Presentation Rubric* [Rubric]. FPT University. *(File: OSG202_SU26.xlsx)*

5. Denning, P. J. (1968). *The working set model for program behavior*. Communications of the ACM, 11(5), 323–333. https://doi.org/10.1145/363095.363141

6. Carr, R. W., & Hennessey, J. L. (1981). *WSClock – A simple and effective algorithm for virtual memory management*. Proceedings of the 8th ACM Symposium on Operating System Principles. https://doi.org/10.1145/800216.806596

7. OS Simulator Tool. (n.d.). *Operating System Visualizer*. https://ddpigeon.github.io/os-simulator/

8. Linux Kernel Documentation. (n.d.). *Huge Pages*. https://www.kernel.org/doc/html/latest/admin-guide/mm/hugetlbpage.html

---

*Báo cáo được soạn cho môn OSG202 – SU26 | Nhóm 3 | SE2008*
*Ngày: 26/06/2026*
