# KỊCH BẢN THUYẾT TRÌNH
## Page Tables: Structure, Design, and Performance Analysis
**Môn: OSG202 — Nhóm 3**
**Tổng thời lượng dự kiến: ~12–15 phút (2–3 phút/người)**

---

## 1. Nguyễn Hoàng Anh Khoa — Leader
### Mở đầu & Đặt vấn đề (2-3 phút)

**Lời dẫn mở đầu:**

> "Xin chào thầy/cô và các bạn. Nhóm 3 chúng em xin trình bày đề tài **'Page Tables: Structure, Design, and Performance Analysis'**. Em là Anh Khoa, leader của nhóm, sẽ giới thiệu tổng quan trước khi các bạn đi sâu vào từng phần."

**Nội dung trình bày:**

- Giới thiệu chủ đề: **Page Table** là cấu trúc dữ liệu trung tâm giúp **MMU (Memory Management Unit)** dịch địa chỉ ảo (virtual address) sang địa chỉ vật lý (physical address) — nền tảng của cơ chế bộ nhớ ảo trong hệ điều hành hiện đại.

- Nêu vấn đề đặt ra:
  - Với hệ thống 32-bit và kích thước trang 4KB, một flat page table có thể cần tới ~1 triệu entry cho mỗi tiến trình.
  - Điều này gây ra **overhead lớn về bộ nhớ** và **độ trễ truy cập** (mỗi lần truy cập bộ nhớ đều phải "đi qua" page table).
  - → Đặt câu hỏi: làm sao thiết kế page table vừa tiết kiệm bộ nhớ, vừa đảm bảo tốc độ truy cập?

- Nêu 3 mục tiêu chính của báo cáo:
  1. Phân tích cấu trúc của **Page Table Entry (PTE)**.
  2. So sánh 3 kỹ thuật quản lý: **TLB, Multilevel Page Table, Inverted Page Table**.
  3. Đánh giá hiệu năng thực tế thông qua **mô phỏng (simulation)**.

- Giới thiệu bố cục bài thuyết trình, dẫn sang phần tiếp theo:

> "Tiếp theo, bạn Đăng Khoa sẽ trình bày phần nền tảng lý thuyết và các cơ chế kỹ thuật cốt lõi của page table."

---

## 2. Nguyễn Đăng Khoa — System Engineer
### Nền tảng lý thuyết & Cơ chế kỹ thuật (2-3 phút)

**Câu mở đầu chuyển tiếp:**

> "Cảm ơn Anh Khoa. Em là Đăng Khoa, phụ trách phần nền tảng lý thuyết và các cơ chế kỹ thuật chính liên quan đến page table."

**Nội dung trình bày:**

- **Lược sử phát triển:**
  - Khái niệm paging xuất hiện từ hệ thống **Atlas** (Đại học Manchester, thập niên 1960) — được xem là hệ thống đầu tiên áp dụng bộ nhớ ảo.
  - Đến năm **1985**, **Intel 80386** là CPU x86 đầu tiên hỗ trợ paging 2 cấp (2-level paging), đặt nền móng cho các thiết kế hiện đại.

- **Cấu trúc Page Table Entry (PTE) — ví dụ hệ 32-bit:**
  - **Page Frame Number (PFN):** xác định khung trang vật lý tương ứng.
  - **Bit Present/Absent:** cho biết trang có đang nằm trong RAM hay đã bị swap ra đĩa.
  - **Protection bits (R/W/X):** quyền đọc/ghi/thực thi trên trang.
  - **Modified bit (Dirty bit):** đánh dấu trang đã bị thay đổi nội dung.
  - **Referenced bit:** hỗ trợ các thuật toán thay thế trang (page replacement).

- **3 kỹ thuật cốt lõi được nhóm phân tích:**
  1. **TLB (Translation Lookaside Buffer):** bộ nhớ đệm phần cứng, lưu các ánh xạ địa chỉ vừa dùng để tránh phải tra page table nhiều lần → tăng tốc đáng kể.
  2. **Multilevel Page Table:** chia page table thành nhiều cấp (ví dụ **Intel PML4, PML5**), cho phép cấp phát "lazy" — chỉ tạo bảng con khi thực sự cần, tiết kiệm bộ nhớ.
  3. **Inverted Page Table:** dùng một bảng toàn cục duy nhất cho toàn hệ thống (thay vì mỗi tiến trình một bảng), được áp dụng trong kiến trúc **IBM PowerPC** và **PA-RISC**.

> "Sau khi đã nắm được lý thuyết, mời bạn Đức Anh trình bày phần phương pháp và thiết lập thí nghiệm mà nhóm đã thực hiện."

---

## 3. Trần Đức Anh — Experimental Engineer
### Phương pháp & Thiết lập thí nghiệm (2-3 phút)

**Câu mở đầu chuyển tiếp:**

> "Cảm ơn Đăng Khoa. Em là Đức Anh, phụ trách phần phương pháp nghiên cứu và thiết lập thí nghiệm của nhóm."

**Nội dung trình bày:**

- **Phương pháp nghiên cứu:** kết hợp giữa
  - Phân tích lý thuyết dựa trên tài liệu tham khảo (**Tanenbaum, Modern Operating Systems, Section 3.5**).
  - Thực nghiệm mô phỏng để kiểm chứng và đo lường hiệu năng thực tế.

- **Công cụ mô phỏng:** nhóm sử dụng **OS Simulator** tại địa chỉ `ddpigeon.github.io/os-simulator` để mô phỏng hành vi truy cập bộ nhớ và page table.

- **3 test case chính được thiết kế:**
  1. **Sequential Access** — truy cập bộ nhớ tuần tự.
  2. **Random Access** — truy cập bộ nhớ ngẫu nhiên.
  3. **Working Set Variation** — thay đổi kích thước tập làm việc (working set).

- **Thông số mô phỏng cụ thể:**
  - Page size: **4KB**
  - RAM: **16 frame**
  - TLB: **8 entry**
  - Thuật toán thay thế: **LRU (Least Recently Used)**
  - Số lượng thao tác: **1000 thao tác/test case**

> "Với thiết lập như trên, mời bạn Đức Danh trình bày kết quả thực nghiệm mà nhóm thu được."

---

## 4. Trần Đức Danh — Data Analyst
### Kết quả thực nghiệm (2-3 phút)

**Câu mở đầu chuyển tiếp:**

> "Cảm ơn Đức Anh. Em là Đức Danh, phụ trách phân tích và trình bày kết quả thực nghiệm."

**Nội dung trình bày:**

- **Bảng kết quả TLB Hit Rate theo từng test case:**
  - TC1 (Sequential Access): **98.7%**
  - TC2 (Random Access): mức trung gian
  - TC3 (Working Set Variation lớn): chỉ còn **42.1%**

- **Nhận xét chính:**
  - Truy cập tuần tự tận dụng tốt **tính cục bộ không gian (spatial locality)** → TLB hit rate rất cao, hiệu năng tốt.
  - Khi truy cập ngẫu nhiên hoặc working set lớn, tính cục bộ bị phá vỡ → hit rate giảm mạnh, hiệu năng hệ thống suy giảm rõ rệt.

- **Điểm nhấn ấn tượng nhất:**
  - **EMAT (Effective Memory Access Time)** tăng gần **45 lần** khi so sánh giữa TC1 và TC3.
  - Đây là minh chứng rõ ràng cho tầm quan trọng của việc **thiết kế truy cập bộ nhớ có tính cục bộ (locality-aware)** trong lập trình và thiết kế hệ thống.

> "Từ những kết quả này, mời bạn Lê Nghĩa trình bày phần thảo luận, hạn chế và kết luận của nhóm."

---

## 5. Trần Lê Nghĩa — Reviewer
### Thảo luận, Hạn chế & Kết luận (2-3 phút)

**Câu mở đầu chuyển tiếp:**

> "Cảm ơn Đức Danh. Em là Lê Nghĩa, phụ trách phần thảo luận, đánh giá hạn chế và kết luận chung của báo cáo."

**Nội dung trình bày:**

- **Trade-off giữa các thiết kế page table:**
  - **Flat page table:** đơn giản, dễ triển khai nhưng **rất tốn bộ nhớ** với không gian địa chỉ lớn.
  - **Multilevel page table:** cân bằng tốt giữa bộ nhớ và hiệu năng, nhưng phát sinh thêm **độ trễ truy cập** do phải đi qua nhiều cấp.
  - **Inverted page table:** tối ưu cho hệ thống có **RAM lớn**, nhưng **quá trình lookup phức tạp hơn**.

- **Xu hướng hiện đại:**
  - **ARM64:** sử dụng 4-level paging.
  - **RISC-V:** các mô hình **Sv39, Sv48, Sv57**.
  - **Huge Pages:** giúp giảm áp lực lên TLB bằng cách dùng trang có kích thước lớn hơn.

- **Hạn chế của nghiên cứu:**
  - Công cụ mô phỏng chưa phản ánh đầy đủ hành vi phần cứng thực tế, ví dụ: **cache hierarchy**, **NUMA (Non-Uniform Memory Access)**...

- **Kết luận & khuyến nghị:**
  - Không có một thiết kế page table nào **tối ưu tuyệt đối** cho mọi trường hợp.
  - Việc lựa chọn thiết kế cần dựa trên **đặc điểm workload cụ thể** của hệ thống.
  - **Hướng nghiên cứu tương lai:** mở rộng thử nghiệm với **RISC-V Sv48**, và tìm hiểu sâu hơn về **KPTI (Kernel Page Table Isolation)**.

**Lời kết:**

> "Trên đây là toàn bộ phần trình bày của nhóm 3 về đề tài Page Tables. Cảm ơn thầy/cô và các bạn đã lắng nghe. Nhóm em xin sẵn sàng nhận câu hỏi và góp ý."

---

## Ghi chú phân bổ thời gian

| STT | Thành viên | Vai trò | Nội dung | Thời lượng |
|---|---|---|---|---|
| 1 | Nguyễn Hoàng Anh Khoa | Leader | Mở đầu & Đặt vấn đề | 2-3 phút |
| 2 | Nguyễn Đăng Khoa | System Engineer | Nền tảng lý thuyết & Cơ chế kỹ thuật | 2-3 phút |
| 3 | Trần Đức Anh | Experimental Engineer | Phương pháp & Thiết lập thí nghiệm | 2-3 phút |
| 4 | Trần Đức Danh | Data Analyst | Kết quả thực nghiệm | 2-3 phút |
| 5 | Trần Lê Nghĩa | Reviewer | Thảo luận, Hạn chế & Kết luận | 2-3 phút |
| | | **Tổng cộng** | | **~12-15 phút** |

> **Lưu ý cho cả nhóm:** mỗi phần chỉ nói sơ lược, không đi sâu vào công thức hay chi tiết kỹ thuật phức tạp. Nên có câu chuyển tiếp mượt giữa các thành viên (đã gợi ý sẵn ở đầu mỗi phần) để phần thuyết trình liền mạch, không bị ngắt quãng.
