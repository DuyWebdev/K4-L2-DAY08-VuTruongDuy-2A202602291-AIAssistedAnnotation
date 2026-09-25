# Báo cáo Lab Ngày 08: Học chủ động cho bộ phát hiện xe

Họ và tên: Vũ Trường Duy

Công cụ gán nhãn đã dùng: CVAT Docker local

## 1. Dữ liệu và cách chia tập

Tập dữ liệu được chia thành pool chưa gán nhãn và test set theo trình tự thời gian thay vì chia ngẫu nhiên. Cách chia này giúp hạn chế việc các frame rất gần nhau về thời gian xuất hiện đồng thời ở train và test, từ đó giảm nguy cơ rò rỉ thông tin giữa hai tập.

Nếu chia ngẫu nhiên một video hoặc chuỗi frame theo thời gian, các frame gần như giống nhau có thể xuất hiện ở cả train và test. Khi đó kết quả đánh giá có thể cao hơn do mô hình được đánh giá trên những cảnh rất tương tự với dữ liệu đã nhìn thấy trong quá trình huấn luyện.

Theo `rounds_table.md`, test set gồm 20 ảnh với 403 box tham chiếu. Có 14 box rất nhỏ dưới ngưỡng 16 px được bỏ qua khi tính các chỉ số tương ứng. Test set không được chỉnh sửa trong quá trình active learning.

## 2. Mô hình khởi đầu lạnh (cold start)

Ở Round 0, mô hình sử dụng YOLOv8n ở trạng thái cold start với mô hình COCO có khả năng phát hiện các nhóm xe như car, bus và truck. Chưa có ảnh nào từ pool được dùng để fine-tune cho dataset của bài.

Kết quả Round 0:

- AP50: 0.771
- Precision: 0.925
- Recall: 0.489
- F1: 0.640
- Recall xe nhỏ: 0.182
- Recall xe trung bình: 0.547
- Recall xe lớn: 0.561

Kết quả cho thấy precision khá cao nhưng recall thấp hơn đáng kể. Đặc biệt recall của xe nhỏ chỉ đạt 0.182, thấp hơn recall của xe trung bình và xe lớn. Điều này cho thấy các đối tượng nhỏ là một nhóm khó đối với mô hình khởi đầu.

Ảnh `outputs/compare_round0.jpg` được dùng để đối chiếu trực quan giữa dự đoán và nhãn tham chiếu. Tuy nhiên, cần phân biệt lỗi của mô hình với khả năng nhãn tham chiếu chưa hoàn toàn đầy đủ. Vì vậy không nên kết luận mô hình sai chỉ dựa trên một trường hợp nếu chưa kiểm tra nhãn tham chiếu.

## 3. Chiến lược chọn mẫu

Chiến lược active learning sử dụng:

`score = W_U*U + W_A*A + W_D*D`

Trong đó:

- U là mức độ bất định của các dự đoán;
- A là tỷ lệ các box có confidence thấp;
- D biểu diễn khoảng cách thời gian với mẫu đã được gán nhãn.

Các trọng số mặc định của chiến lược là W_U = 0.5, W_A = 0.3 và W_D = 0.2.

`MIN_GAP_S` được sử dụng để hạn chế việc chọn nhiều frame quá gần nhau về thời gian. Điều này giúp lô được chọn có độ đa dạng thời gian tốt hơn thay vì chứa nhiều frame gần như giống nhau.

Round 0 chọn 12 ảnh để đưa sang Round 1. Sau đó 12 ảnh này được đưa vào CVAT local để kiểm tra và chỉnh sửa pre-label.

Việc chọn mẫu dựa trên uncertainty không có nghĩa rằng mọi mẫu có score cao đều chắc chắn giúp mô hình tốt hơn. Score chỉ là tín hiệu để ưu tiên những mẫu mà mô hình có khả năng chưa chắc chắn hoặc thiếu thông tin.

## 4. Các vòng học chủ động

Round 0 là cold start:

- 0 ảnh train
- 0 box train
- AP50 = 0.771

Round 1 sử dụng 12 ảnh được chọn và sau khi review có tổng cộng 222 box.

Theo kết quả đóng gói nhãn Round 1:

- Model đề xuất: 169 box
- Accepted: 157
- Edited: 4
- Deleted: 8
- Added: 61
- Tổng box sau review: 222

Như vậy quá trình review cho thấy pre-label của model vẫn bỏ sót một số đối tượng. Đặc biệt có 61 box được người gán nhãn bổ sung, cho thấy việc human-in-the-loop vẫn cần thiết thay vì sử dụng trực tiếp toàn bộ pre-label.

Sau khi fine-tune Round 1, kết quả trên test set là:

- AP50 = 0.413
- Precision = 1.000
- Recall = 0.047
- F1 = 0.090
- Recall xe nhỏ = 0.000
- Recall xe trung bình = 0.034
- Recall xe lớn = 0.220

AP50 thay đổi từ 0.771 xuống 0.413, tức giảm 0.358 điểm.

Recall cũng giảm mạnh từ 0.489 xuống 0.047. Vì vậy, trong vòng này việc bổ sung 12 ảnh chưa tạo ra cải thiện trên bộ test hiện tại.

Kết quả này cũng cho thấy active learning không đảm bảo rằng mỗi vòng fine-tune đều làm AP50 tăng. Có thể có nhiều nguyên nhân, trong đó một yếu tố cần kiểm tra là chất lượng và độ đại diện của 12 ảnh được chọn cũng như chất lượng nhãn sau review.

Khi đối chiếu `compare_round0.jpg` và `compare_round1.jpg`, cần kiểm tra riêng những trường hợp mà kết quả sau fine-tune tốt hơn hoặc xấu hơn. Không nên chỉ dựa vào AP50 để kết luận nguyên nhân.

Một trường hợp cần chú ý là các xe nhỏ hoặc bị che khuất. Đây là các trường hợp dễ làm recall giảm và cũng phù hợp với việc Round 0 vốn đã có recall xe nhỏ thấp.

Một trường hợp khác cần kiểm tra là những box mà pre-label bị bỏ sót và phải được người gán nhãn bổ sung. Các trường hợp này được phản ánh trong `round1_diff.md` và `REVIEW_LOG.csv`.

## 5. Kết luận và giới hạn

So với cold start, Round 1 chưa cải thiện AP50 trên test set. AP50 giảm từ 0.771 xuống 0.413. Vì vậy, nếu tiếp tục active learning, cần kiểm tra nguyên nhân trước khi đưa thêm dữ liệu vào huấn luyện.

Hai nhóm trường hợp cần ưu tiên kiểm tra ở vòng sau là:

1. Xe nhỏ hoặc ở xa, vì recall xe nhỏ ở Round 0 đã thấp và Round 1 giảm xuống 0.
2. Xe bị che khuất hoặc nằm sát các xe khác, vì các trường hợp này dễ gây bỏ sót hoặc box sai vị trí.

Trước khi train thêm, cần kiểm tra chất lượng nhãn của các frame Round 1, đặc biệt các trường hợp có nhiều box `added`, `edited` hoặc `deleted`. Đồng thời cần kiểm tra sự phân bố của 12 ảnh đã chọn để bảo đảm chúng thực sự bổ sung các tình huống khó thay vì tập trung vào một loại cảnh.

Bộ test chỉ có 20 ảnh và nhãn tham chiếu chưa được kiểm tra thủ công hoàn toàn. Ngoài ra có 14 box rất nhỏ được bỏ qua trong các phép tính liên quan. Vì vậy AP50 phản ánh mức độ khớp với bộ nhãn tham chiếu hiện tại, không phải bằng chứng tuyệt đối về khả năng hoạt động ngoài thực tế.

Nếu AP50 tiếp tục giảm ở vòng sau, cần kiểm tra chất lượng nhãn và độ đại diện của dữ liệu trước khi tăng thêm số lượng ảnh. Active learning nên được xem là một quy trình lặp gồm chọn mẫu, con người kiểm tra nhãn, huấn luyện và đánh giá, thay vì chỉ tăng số lượng dữ liệu.