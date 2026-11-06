# Báo cáo cá nhân — Day 01: Lọc tin nhắn rác

**SEED:** 869. Họ tên và mã học viên được ghi trong tên tệp theo quy định của repository.

## 1. Bằng chứng

| Tệp | Tình trạng |
|---|---|
| [day01_lab.ipynb](day01_lab.ipynb) | Notebook đã chạy, có output ML, DL, embedding và 50 case khó. Các thí nghiệm LLM thật chưa có kết quả. |
| [lab1_report.json](lab1_report.json) | Báo cáo gốc được lưu ở thư mục gốc repository và sao chép vào đây; gồm seed, checks, predictions và results. |
| [hand_calc.jpg](hand_calc.jpg) | Ảnh viết tay có các bước tính. |

Notebook hiện vẫn còn một số ô câu trả lời và phần phản ánh chưa điền. Các kết quả bên dưới được tổng hợp từ output đã lưu; mục AI log ghi rõ phạm vi hỗ trợ.

## 2. Kết quả chính

| Mô hình | F1 test | F1 SUB 40 | p95 ms/tin | USD/1.000 tin |
|---|---:|---:|---:|---:|
| BoW + Logistic Regression | 0,865 | 0,824 | 0,114 | 0 |
| BoW + MLP | 0,933 | 0,857 | 2,405 | 0 |
| MiniLM + kNN | 0,904 | 1,000 | 16,956 | 0 |
| MiniLM + Logistic Regression | 0,943 | 0,974 | 7,870 | 0 |
| MiniLM + MLP head | 0,959 | 0,974 | 10,639 | 0 |

Các hàng này được chép từ block 79 của notebook. `USD/1.000 tin = 0` nghĩa là bảng không tính phí API cho các model local; không bao gồm chi phí máy, điện hoặc vận hành. LLM chưa được gọi thật, nên không có hàng LLM để so sánh.

## 3. Dự đoán và thực tế

Các dự đoán dưới đây được ghi trong notebook và có AI hỗ trợ; không xem chúng là bằng chứng độc lập về dự đoán cá nhân. Hai dự đoán là định tính, nên không thể xếp hạng chính xác ba dự đoán lệch nhiều nhất.

1. **F1 của BoW + Logistic Regression:** ô dự đoán ghi khoảng 0,92; kết quả test là 0,865, thấp hơn khoảng 0,055. Precision 0,985 cao hơn recall 0,770: model chặn tương đối chính xác nhưng vẫn bỏ sót một phần spam.
2. **Learning rate 50:** dự đoán bước học lớn có thể làm loss dao động hoặc tăng. Đồ thị lưu trong notebook cho thấy loss có đỉnh lớn lúc đầu rồi giảm, đạt khoảng 0,003 trên train; do đó không thể kết luận learning rate này luôn phân kỳ trong lượt chạy. Loss train thấp cũng chưa bảo đảm test tốt.
3. **Sáu tin ngụy trang:** dự đoán encoder + LogReg có thể đúng nhiều hơn BoW. Thực tế cả hai đều đúng 5/6 và cùng bỏ sót một tin spam giả thông báo phần thưởng tài khoản. Khả năng hiểu ngữ nghĩa của embedding không bảo đảm phân loại đúng mọi ý định.

## 4. Phân tích 50 case khó

| Mô hình | F1 SUB thông thường | F1 50 case khó | Accuracy 50 case khó |
|---|---:|---:|---:|
| BoW + Logistic Regression | 0,824 | 0,333 | 52% |
| BoW + MLP | 0,857 | 0,450 | 56% |
| MiniLM + kNN | 1,000 | 0,714 | 68% |
| MiniLM + Logistic Regression | 0,974 | 0,612 | 62% |
| MiniLM + MLP head | 0,974 | 0,583 | 60% |

BoW + Logistic Regression giảm nhiều nhất khi so với SUB, từ khoảng 0,824 xuống 0,333. Hai tập khác nhau về kích thước và phân phối nên đây là dấu hiệu cần kiểm tra thêm, không phải mức giảm chính xác ngoài đời. Trong bảng theo nhóm, model này chỉ đúng 10% nhóm spam giọng tự nhiên; BoW + MLP đúng 0% nhóm né từ khóa.

Hai ví dụ ở block 89 mà cả năm model cùng sai:

- **Case #14:** lời mời việc làm được diễn đạt như câu chuyện cá nhân, ít giống các mẫu spam rõ ràng; model bỏ sót ý định quảng cáo/lừa đảo.
- **Case #31:** các từ khóa bị chèn dấu chấm giữa các chữ, làm token khác với từ thường gặp trong train; không phải encoder nào cũng khôi phục đúng ý định.

Bộ 50 case được viết để thử điểm yếu, không đại diện trực tiếp cho phân phối spam ngoài thực tế. Không có kết quả LLM để kết luận prompt phòng thủ hiệu quả hay không.

## 5. Thử thách cá nhân

SEED 869 tương ứng **đề #6**: yêu cầu LLM trả JSON có `label` và `reason`, phân tích JSON và so sánh với câu trả lời một từ trên 10 tin SUB. Code trong block 97 tạo prompt JSON, kiểm tra cấu trúc và nhãn, và dự kiến thống kê F1, token ước tính, chi phí, độ trễ và đầu ra không hợp lệ.

Output của block 97 cho biết chưa chạy thử thách bằng API thật vì chưa có key. Vì vậy chưa có số liệu để kết luận JSON tốn thêm bao nhiêu hoặc có đáng dùng hay không. Đề ghi rõ thử thách không dùng AI; code hiện tại có AI hỗ trợ như đã khai trong mục 7.

## 6. Phương án cho sản phẩm thật

Với 10 triệu SMS/ngày, lưu lượng trung bình là khoảng `10.000.000 / 86.400 ≈ 115,74` tin/giây; lưu lượng cao điểm có thể lớn hơn. Trong lượt đo hiện tại, cả năm phương án local đều có p95 nhỏ hơn 50 ms. MiniLM + kNN có F1 SUB 1,000 và p95 khoảng 16,956 ms, nên có thể chọn làm **ứng viên thử nghiệm**. Tuy nhiên, F1 trên 50 case khó chỉ 0,714 và SUB chỉ có 40 tin; cần tập đánh giá lớn hơn và kiểm thử tải trước khi quyết định triển khai.

Một phương án kết hợp là dùng BoW + Logistic Regression làm tầng nhanh, chuyển các tin chưa chắc chắn sang encoder hoặc hàng chờ kiểm tra. Chọn ngưỡng theo validation và đo lại toàn bộ hệ thống, đặc biệt tỷ lệ chặn nhầm tin hợp lệ và p95 khi có lưu lượng cao.

Theo giả định chi phí trong đề (100 token input, 1 token output cho mỗi tin; giá $0,10 và $0,50 trên một triệu token tương ứng), 10 triệu tin/ngày tốn khoảng **$105/ngày**, hay **$3.150 cho 30 ngày**. Đây là ước tính theo giả định đề bài, không phải số đo API hay báo giá hiện hành.

## 7. AI log và điều rút ra từ kết quả

| Nội dung đã nhờ AI | Hỗ trợ đã nhận | Bằng chứng kiểm tra hiện có |
|---|---|---|
| Đọc, phân tích và làm notebook; giải thích các phần đã làm | Codex viết code cho các hàm ML, DL và embedding ở phần 1–3. Đề yêu cầu tự viết code ở các phần này. | Các kiểm tra tương ứng trong output báo đạt; cần tự rà soát và giải thích được code. |
| Hoàn thành phần LLM | Codex viết code gọi API, tạo prompt, đọc nhãn và phân loại theo lô. | Kiểm tra API giả lập báo đạt; chưa có kết quả gọi LLM thật. |
| Giải thích câu hỏi và tính toán | Codex tạo tài liệu giải thích, dự đoán mẫu và đáp số tham khảo. | Output block 94 báo năm đáp số khớp bộ kiểm tra; ảnh tính tay được đính kèm. |
| Thử thách JSON #6 | Codex viết code tham khảo cho prompt, parser và so sánh; đề quy định không dùng AI cho thử thách. | Chưa chạy API thật, chưa có số đo so sánh. |
| Chuẩn bị báo cáo cá nhân | Codex tổng hợp báo cáo từ các output đã lưu trong notebook. | Các số liệu trong bảng đối chiếu với output block 79 và 88. |

Từ kết quả có thể thấy accuracy cao chưa đủ để đánh giá bộ lọc spam: precision và recall thể hiện hai loại sai sót khác nhau. Điểm tốt trên SUB 40 tin cũng chưa đủ bảo đảm model xử lý tốt các case khó hoặc đạt SLA khi vận hành thật. Phần tự rà soát code, dự đoán và trải nghiệm học tập cá nhân cần do học viên xác nhận.
