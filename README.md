# NỘI DUNG THUYẾT TRÌNH — RBL: Page Tables (Structure, Design, and Performance Analysis)
### SE2008 — Group 3

---

## 1. Nguyễn Hoàng Anh Khoa (Leader) — Mở đầu & Tổng quan đề tài
*Thời lượng gợi ý: ~2 phút*

Xin chào thầy/cô và các bạn, mình là Khoa, leader của Group 3. Hôm nay nhóm mình xin trình bày đề tài **"Page Tables: Structure, Design, and Performance Analysis"**, dựa trên Section 3.5 của cuốn *Modern Operating Systems (4th Edition)* — Andrew Tanenbaum.

Trong các hệ điều hành hiện đại, mỗi tiến trình được cấp một không gian địa chỉ ảo riêng, có thể lên tới 2^64 byte trên hệ 64-bit. Nhưng bộ nhớ vật lý thực tế lại rất giới hạn, chỉ vài GB đến vài trăm GB. Cơ chế **paging** ra đời để giải quyết mâu thuẫn này, chia không gian địa chỉ thành các trang có kích thước cố định và ánh xạ chúng vào các khung trang trong bộ nhớ vật lý. Và **Page Table** chính là cấu trúc dữ liệu trung tâm thực hiện việc ánh xạ đó.

Tuy nhiên, cấu trúc Page Table cũng đặt ra nhiều thách thức kỹ thuật:
- **Kích thước bảng quá lớn**: với không gian 32-bit và trang 4KB, một tiến trình có thể cần tới 1 triệu entry.
- **Chi phí truy cập**: mỗi lần truy cập bộ nhớ đều phải tra bảng trang trước, gần như tăng gấp đôi thời gian truy cập.
- **Chiếm dụng bộ nhớ**: bảng trang phẳng (flat page table) chiếm một khối bộ nhớ liên tục dù phần lớn không gian địa chỉ không được dùng đến.

Từ đó, nhóm mình đặt ra 3 mục tiêu nghiên cứu chính:
1. Phân tích chi tiết cấu trúc của một Page Table Entry (PTE) và ý nghĩa của từng bit trạng thái.
2. So sánh các kỹ thuật tối ưu: TLB, Multilevel Page Table, và Inverted Page Table.
3. Đánh giá hiệu năng thực nghiệm thông qua mô phỏng, từ đó đưa ra khuyến nghị thiết kế.

Báo cáo của nhóm sẽ đi theo trình tự: cơ sở lý thuyết, phương pháp nghiên cứu, phân tích các thành phần chính, thiết lập thực nghiệm, kết quả, thảo luận và kết luận. Sau đây mình xin mời bạn [Tên bạn Nguyễn Đăng Khoa] trình bày phần cơ sở lý thuyết và cấu trúc kỹ thuật của Page Table.

---

## 2. Nguyễn Đăng Khoa (System Engineer) — Cơ sở lý thuyết & Cấu trúc kỹ thuật
*Thời lượng gợi ý: ~3–4 phút*

Cảm ơn Khoa. Mình là [Nguyễn Đăng Khoa], phụ trách phần lý thuyết nền tảng và phân tích cấu trúc kỹ thuật của Page Table.

**Về lịch sử**, cơ chế paging lần đầu xuất hiện trên hệ thống Atlas tại Đại học Manchester vào đầu thập niên 1960, với mục tiêu ban đầu là cho phép chương trình lớn chạy được trên bộ nhớ vật lý giới hạn — đây chính là khởi nguồn của khái niệm *virtual memory*. Đến năm 1985, Intel 80386 là CPU x86 đầu tiên hỗ trợ paging phần cứng với cấu trúc *two-level page table*, đặt nền móng cho kiến trúc IA-32 vẫn còn được dùng đến ngày nay.

**Về cấu trúc PTE (Page Table Entry)** trên hệ 32-bit, mỗi entry gồm các trường chính:
- Page frame number (20 bit) — địa chỉ khung trang vật lý tương ứng
- Present/Absent bit — trang đang ở RAM hay đã bị swap ra đĩa
- Protection bits — quyền Read/Write/Execute
- Modified (Dirty) bit — trang đã bị ghi hay chưa
- Referenced bit — phục vụ các thuật toán thay thế trang
- Caching disabled bit — dùng cho các thanh ghi thiết bị ánh xạ bộ nhớ

Địa chỉ vật lý được tính theo công thức: **Physical Address = Frame Number × Page Size + Offset**. Nếu Present bit = 0, một page fault sẽ xảy ra, buộc hệ điều hành phải nạp trang còn thiếu từ đĩa vào RAM.

**Về TLB (Translation Lookaside Buffer)** — đây là một cache phần cứng tốc độ cao nằm trong MMU, lưu các ánh xạ virtual-to-physical được dùng gần nhất. Theo Chen & Baer (1995), tỷ lệ hit rate trên 95% là hoàn toàn khả thi với các workload thông thường, nhờ tính *locality* của truy cập bộ nhớ.

**Về Multilevel Page Table** — kiến trúc hai cấp của Intel 386 sau này mở rộng thành 4 cấp (PML4) trên x86-64 để hỗ trợ không gian địa chỉ 48-bit. Đặc điểm cốt lõi là *lazy allocation*: các bảng trang cấp thấp chỉ được cấp phát khi vùng địa chỉ tương ứng thực sự được sử dụng, giúp tiết kiệm đáng kể bộ nhớ.

**Về Inverted Page Table** — thay vì mỗi tiến trình có một bảng trang riêng, hệ thống duy trì một bảng toàn cục duy nhất với số entry cố định bằng số khung trang vật lý. Ưu điểm là kích thước bảng tỉ lệ với RAM vật lý chứ không phải với không gian địa chỉ ảo; nhược điểm là việc tra cứu phức tạp hơn, thường cần cấu trúc hash. IBM PowerPC và HP PA-RISC là các kiến trúc tiêu biểu sử dụng kỹ thuật này.

Tiếp theo, mình xin mời bạn [Trần Đức Anh] trình bày phần phương pháp nghiên cứu và thiết lập thực nghiệm của nhóm.

---

## 3. Trần Đức Anh (Experimental Engineer) — Phương pháp nghiên cứu & Thiết lập thực nghiệm
*Thời lượng gợi ý: ~2–3 phút*

Cảm ơn bạn. Mình là Trần Đức Anh, phụ trách phần phương pháp nghiên cứu và thực nghiệm của nhóm.

Nhóm mình sử dụng kết hợp hai phương pháp: **phân tích lý thuyết** và **thực nghiệm mô phỏng**.

Về phương pháp lý thuyết, nhóm thực hiện *systematic literature analysis*, tập trung vào Section 3.5 của Tanenbaum kết hợp với các tài liệu bổ sung về kiến trúc phần cứng. Quy trình gồm 4 bước: đọc và tổng hợp nội dung lý thuyết, mô hình hóa cấu trúc, so sánh các phương án thiết kế kiến trúc, và liên hệ với các implementation thực tế trong Linux, Windows.

Về phương pháp thực nghiệm, nhóm sử dụng công cụ mô phỏng hệ điều hành trực tuyến **OS Simulator** (ddpigeon.github.io/os-simulator) để kiểm chứng các lý thuyết đã tổng hợp. Nhóm thiết kế 3 kịch bản kiểm thử:
- **Test case 1 — Sequential access**: truy cập trang tuần tự, đo TLB hit rate trong điều kiện locality cao.
- **Test case 2 — Random access**: truy cập ngẫu nhiên, đo hit rate trong điều kiện locality thấp.
- **Test case 3 — Working set variation**: thay đổi kích thước working set để quan sát điểm "gãy" hiệu năng.

Các chỉ số được đo gồm: tổng số page fault, TLB hit rate, Effective Memory Access Time (EMAT), và tổng số lần page table walk.

Về thiết lập cụ thể, môi trường mô phỏng được cấu hình như sau: kích thước trang 4KB, bộ nhớ vật lý 16 khung trang (64KB), không gian địa chỉ ảo 64 trang (256KB), TLB có 8 entry quản lý bằng phần cứng, chính sách thay thế LRU, và mỗi test case chạy 1000 thao tác truy cập.

Với thiết lập này, nhóm đảm bảo có đủ dữ liệu để so sánh hiệu năng giữa các mẫu truy cập khác nhau. Sau đây mình xin mời bạn [Trần Đức Danh] trình bày phần kết quả và phân tích số liệu.

---

## 4. Trần Đức Danh (Data Analyst) — Kết quả & Phân tích số liệu
*Thời lượng gợi ý: ~3 phút*

Cảm ơn Đức Anh. Mình là Trần Đức Danh, phụ trách phân tích số liệu thực nghiệm của nhóm.

Sau khi chạy 4 kịch bản kiểm thử, nhóm thu được kết quả như sau:

| Test Case | TLB Hit Rate | Page Faults | EMAT |
|---|---|---|---|
| TC1 — Sequential Access | 98.7% | 16 | 2.59 ns |
| TC2 — Random Access (small) | 87.3% | 48 | 27.1 ns |
| TC3 — Random Access (large) | 42.1% | 312 | 116.8 ns |
| TC4 — Working set = 8 pages | 96.2% | 22 | 5.52 ns |

Kết quả TC1 cho thấy truy cập tuần tự tận dụng tốt **spatial locality**, đạt hit rate gần 99%. Ngược lại, TC2 và TC3 cho thấy hiệu năng suy giảm rõ rệt khi working set vượt quá dung lượng TLB (8 entry). Đặc biệt, TC4 là bằng chứng thực nghiệm cho lý thuyết *working set* của Denning: chỉ cần working set nằm gọn trong TLB, hiệu năng vẫn được duy trì cao dù mẫu truy cập không hoàn toàn tuần tự.

Điểm đáng chú ý nhất: EMAT tăng gần **45 lần** giữa TC1 và TC3 — từ 2.59ns lên 116.8ns. Điều này nhấn mạnh tầm quan trọng của việc thiết kế phần mềm theo hướng *locality-aware*, tức tối ưu mẫu truy cập bộ nhớ để tận dụng tối đa TLB.

Những số liệu này cũng là cơ sở thực nghiệm để nhóm đưa ra các đánh giá về đánh đổi kỹ thuật (trade-off) ở phần thảo luận tiếp theo, mình xin mời bạn [Trần Lê Nghĩa] trình bày.

---

## 5. Trần Lê Nghĩa (Reviewer) — Thảo luận, Hạn chế & Kết luận
*Thời lượng gợi ý: ~3 phút*

Cảm ơn Đức Danh. Mình là Trần Lê Nghĩa, phụ trách phần thảo luận, đánh giá hạn chế và kết luận của báo cáo.

**Về đánh đổi kỹ thuật**: dữ liệu thực nghiệm củng cố quan điểm của Tanenbaum rằng thiết kế page table luôn là bài toán đánh đổi đa chiều. Flat page table đơn giản, tra cứu nhanh nhưng tốn nhiều bộ nhớ. Multilevel page table tiết kiệm bộ nhớ nhưng tăng độ trễ khi TLB miss. Inverted page table mở rộng tốt với RAM 64-bit lớn nhưng đường tra cứu phức tạp hơn. TLB chính là mảnh ghép quan trọng giúp giảm thiểu chi phí tra cứu nhiều cấp — tuy nhiên trong hệ thống đa lõi, hiện tượng **TLB shootdown** (khi một lõi cập nhật page table, các lõi khác phải đồng bộ và invalidate TLB) có thể gây nghẽn cổ chai do chi phí Inter-Processor Interrupt.

**Về xu hướng kiến trúc hiện đại**: ARM64 (như Apple Silicon dòng M) dùng page table 4 cấp với granularity linh hoạt (4KB, 16KB, 64KB); RISC-V chuẩn hóa các mô hình Sv39/Sv48/Sv57. Bên cạnh đó, công nghệ **Huge Pages** (2MB, 1GB) trên x86-64 giúp giảm đáng kể áp lực lên TLB — rất quan trọng với các hệ thống HPC và cơ sở dữ liệu lớn, được quản lý tự động qua Linux Transparent Huge Pages.

**Về hạn chế của nghiên cứu**: công cụ mô phỏng phần mềm không phản ánh hoàn toàn các yếu tố phần cứng thực tế như cache hierarchy, hardware prefetcher, hay kiến trúc NUMA. Ngoài ra, các kịch bản kiểm thử còn giới hạn, và chi phí TLB shootdown chưa được đo trong môi trường đa luồng thực sự.

**Kết luận**, nhóm rút ra 4 điểm chính:
1. Cấu trúc PTE cùng các control bit là nền tảng cho việc dịch địa chỉ và thay thế trang.
2. TLB là yếu tố quyết định hiệu năng bộ nhớ — duy trì hit rate trên 98% với truy cập tuần tự nhưng giảm còn ~42% với truy cập ngẫu nhiên diện rộng.
3. Multilevel Page Table là phương án cân bằng nhất cho các hệ điều hành phổ biến như Linux, Windows.
4. Inverted Page Table phù hợp cho các hệ thống có RAM vật lý cực lớn.

Nhóm cũng đề xuất: các nhà phát triển phần mềm nên ưu tiên thiết kế truy cập bộ nhớ theo hướng locality-aware; quản trị hệ thống nên tận dụng Huge Pages cho các dịch vụ tốn nhiều bộ nhớ. Hướng nghiên cứu tiếp theo có thể là đo hiệu năng thực tế trên silicon RISC-V Sv48, hoặc đánh giá tác động của các cơ chế giảm thiểu lỗ hổng Meltdown/Spectre như KPTI.

Cảm ơn thầy/cô và các bạn đã lắng nghe phần trình bày của Group 3!

---

**Lưu ý cho cả nhóm**: mỗi phần nên luyện tập để nói trong khoảng thời gian tương ứng, dùng slide minh họa cho các bảng số liệu (Table 1–4) và công thức EMAT để bài thuyết trình sinh động hơn.
