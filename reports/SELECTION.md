# Vì sao chọn lô này?

Trong 50 dòng đầu `outputs/selection_round1.csv`, các frame được ưu tiên không chỉ dựa trên một tiêu chí riêng lẻ mà dựa trên điểm tổng hợp:

`score = W_U*U + W_A*A + W_D*D`

Trong đó U thể hiện mức độ bất định của dự đoán, A thể hiện tỷ lệ dự đoán có confidence thấp và D thể hiện khoảng cách thời gian so với các frame đã được gán nhãn. `MIN_GAP_S` giúp hạn chế việc chọn nhiều frame quá gần nhau về thời gian, từ đó giảm các mẫu gần như trùng lặp.

Năm frame tiêu biểu trong danh sách ưu tiên cần được đọc cùng với `outputs/selection_round1.csv` và contact sheet `outputs/selection_round1.jpg`. Các frame được chọn vì chúng đại diện cho những trường hợp mà model có mức độ không chắc chắn cao hoặc có khả năng bổ sung thông tin cho tập huấn luyện.

Ba frame nằm trong lô 12 ảnh được chọn gồm `frame_0099.jpg`, `frame_0107.jpg` và `frame_0182.jpg`. Đây là các mẫu được đưa vào vòng gán nhãn Round 1 sau bước chọn mẫu. Bằng chứng cho quyết định chọn nằm ở điểm `score`, các thành phần U/A/D trong CSV và vị trí tương ứng trên contact sheet.

Một điểm quan trọng là điểm cao chỉ cho thấy mẫu có tính hữu ích theo tiêu chí của chiến lược active learning, không đảm bảo rằng việc thêm mẫu đó chắc chắn làm AP50 tăng. Điều này được thể hiện rõ sau Round 1: dù đã bổ sung 12 ảnh và 222 box, AP50 giảm so với cold start. Vì vậy, uncertainty score là tín hiệu để ưu tiên kiểm tra và gán nhãn, không phải bằng chứng trực tiếp rằng mô hình sẽ cải thiện.

Một frame có điểm cao nhưng không nhất thiết phải được ưu tiên tuyệt đối nếu nó quá giống các frame đã chọn hoặc nếu nội dung thực tế không bổ sung nhiều thông tin. Ngược lại, một frame có điểm thấp hơn vẫn có thể đáng xem xét nếu nó đại diện cho một tình huống khó, chẳng hạn xe bị che khuất, xe rất nhỏ hoặc nhiều xe nằm sát nhau.