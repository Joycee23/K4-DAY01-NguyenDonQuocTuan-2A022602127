## 1. Phân loại ảnh – prediction cấp ảnh
1.Record hạng 1 mô tả toàn ảnh như thế nào?
→ Là lớp mà model dự đoán phù hợp nhất với toàn bộ ảnh, có rank = 1 và score cao nhất. Với traffic, model dự đoán cab với score 0.510915.
2.Ai định nghĩa class list mà checkpoint có thể dự đoán?
→ Taxonomy/dataset dùng để huấn luyện checkpoint. Ở đây là ImageNet-1K.
3.Vì sao cần giữ cả ID, tên lớp và tên taxonomy?
→ class_id để định danh kỹ thuật, class_name để dễ đọc, taxonomy_name để biết ID/tên lớp thuộc hệ phân loại nào.
4.Nếu ảnh có nhiều chủ thể, guideline cần quy định điều gì?
→ Cần quy định chủ thể nào đại diện cho nhãn ảnh, hoặc có cho phép nhiều nhãn (multi-label) hay không.
5.Vì sao model score không phải ground truth?
→ Vì score chỉ thể hiện mức độ tin tưởng của model vào dự đoán, không chứng minh đó là nhãn đúng. Ground truth phải đến từ annotation/nhãn chuẩn độc lập.

## 2. Phát hiện vật thể – lớp và box cho từng object

1. Một record (class_name, score, bbox_xyxy, bbox_width, bbox_height):
person, score 0.912625, bbox_xyxy = [385.33, 69.24, 498.92, 348.92], rộng 113.58 px, cao 279.68 px.

2. Diễn giải vị trí box bằng lời:
Box nằm ở khu vực giữa lệch phải ảnh, bao quanh người đang đứng; bắt đầu gần phía trên và kéo xuống gần giữa/phía dưới ảnh.

3. So sánh số prediction ở hai threshold:
Với sample kitchen:
Threshold 0.35: 11 predictions.
Threshold 0.50: 6 predictions.
→ Tăng threshold làm giảm 5 prediction, giữ lại các dự đoán có confidence cao hơn. File hiện lưu score_threshold = 0.35.
4. Điều gì thay đổi đối với độ bao phủ và khối lượng reviewer cần xem?
Threshold thấp → bao phủ nhiều object hơn nhưng reviewer phải xem nhiều prediction và false positive hơn.
Threshold cao → ít prediction hơn, giảm tải reviewer nhưng có nguy cơ bỏ sót object thật.
5. Đề xuất một quy tắc box chặt:
Box phải sát nhất có thể với toàn bộ phần nhìn thấy của object, không bao gồm quá nhiều background và không cắt mất phần object đang nhìn thấy.
6. Với object bị che khuất/cắt mép:
Guideline cần quy định rõ có box phần bị che khuất hay chỉ phần nhìn thấy; object bị cắt mép/khó xác định phải có tiêu chí cụ thể và escalate cho reviewer cấp cao khi không chắc chắn.

## 3. Phân đoạn theo từng đối tượng – polygon cho mỗi instance
1. Một record:
instance_id: kitchen-007, class_name: oven, score: 0.489667, có 213 điểm polygon; polygon_xy chứa các tọa độ pixel tạo thành biên mask.
2. Polygon bổ sung chi tiết gì so với box?
Polygon biểu diễn đường viền thực tế của object, nên chính xác hình dạng và diện tích hơn box chữ nhật, đặc biệt với vật thể không vuông hoặc có hình dạng phức tạp.
3. instance_id dùng để làm gì và không phải loại ID nào?
Dùng để phân biệt từng object cụ thể trong ảnh, kể cả khi cùng class_name. Ví dụ nhiều bowl vẫn có instance riêng. Nó không phải class_id và cũng không phải ID của taxonomy/dataset.
4. Đề xuất một quy tắc biên mask:
Mask phải bám sát đường biên nhìn thấy của object, không ăn sang background và không bỏ sót phần object có thể xác định được.
5. Với vùng mờ/tiếp xúc/che khuất:
Guideline cần quy định rõ phần nào được mask khi object bị che hoặc tiếp xúc với object khác. Nếu không xác định chắc chắn ranh giới thì không tự suy đoán, chuyển reviewer/escalation để quyết định.
## 4. Vòng đời và kiểm tra chất lượng

| Tác vụ | Đơn vị/định dạng ground truth | Lỗi hoặc điểm mờ hồ quan sát được | Annotator làm gì? | Reviewer xem gì? |
|---|---|---|---|---|
| **Phân loại ảnh** | 1 hoặc nhiều **class label** cho toàn ảnh | Nhầm class, nhiều chủ thể, ảnh không rõ hoặc không thuộc class | Chọn label theo guideline, đánh dấu trường hợp không chắc | Kiểm tra label có đúng nội dung ảnh và taxonomy không |
| **Phát hiện vật thể** | **Class + bounding box** `xyxy` cho từng object | Box quá rộng/hẹp, bỏ sót object, false positive, object bị che/cắt mép | Vẽ/chỉnh box sát object, chọn class, xử lý theo guideline | Kiểm tra class, vị trí/kích thước box, object bị bỏ sót hoặc box sai |
| **Instance segmentation** | **Class + polygon/mask** cho từng instance | Mask lệch biên, ăn vào background, thiếu vùng object, vật thể chồng lên/che khuất | Vẽ/chỉnh polygon bám biên object, tách từng instance | Kiểm tra biên mask, tính đầy đủ, tách đúng từng instance và các vùng mờ hồ |

## 5. An toàn dữ liệu

1. Một quy tắc bảo vệ dữ liệu: Không chia sẻ, sao chép hoặc sử dụng dữ liệu/ảnh ngoài phạm vi công việc; chỉ truy cập và xử lý dữ liệu cần thiết cho nhiệm vụ.
2. Nếu thấy ảnh hoặc dữ liệu không đúng phạm vi, tôi sẽ dừng và báo cho: Reviewer/Quản lý dự án (Team Lead).
