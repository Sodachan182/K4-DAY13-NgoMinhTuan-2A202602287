# BÁO CÁO CÁ NHÂN DAY 13 – POINTPILLARS PRE-LABEL

## 1. Thông tin cá nhân

- Họ và tên: Ngô Minh Tuấn
- MSSV: 2A202602287
- Hình thức thực hiện: Cá nhân

## 2. Phạm vi và nguồn evidence

Báo cáo này chỉ sử dụng output thực tế trong folder kết quả hiện tại, gồm `smoke.json`, các file `summary.csv`, các file `boxes-*.json`, sáu ảnh `side-*.png` và toàn bộ folder `qc-cases`. Báo cáo không sử dụng số liệu ví dụ từ bản mẫu.

Ba lượt chạy cùng dùng frame `demo` và dataset được ghi là `KITTI`. Các kết quả là prediction của mô hình, không phải ground truth. Vì folder không có nhãn chuẩn hay metric đánh giá như precision, recall, IoU hoặc AP, báo cáo không xếp hạng cấu hình nào là tốt nhất.

## 3. Trạng thái smoke test

Theo `smoke.json`, trạng thái tổng là `passed`. Các bước `docker-load`, `run-A`, `run-B`, `run-C` và `qc-cases` đều có trạng thái `passed`.

Môi trường chạy được ghi nhận như sau:

- Image: Linux, kiến trúc `amd64`, source tag `day13-pointpillars:lc-20261001-amd64`.
- Runtime: Linux, kiến trúc `amd64`.
- Giới hạn tài nguyên: 4 CPU và 4 GB RAM.
- Checkpoint: `/opt/PointPillars/pretrained/epoch_160.pth`.
- Repo revision: `0831856d921609312d42c7582c366e5a311bb7b1`.
- `working_tree_dirty = true`, nghĩa là cây mã nguồn tại thời điểm chạy không ở trạng thái sạch. Đây là thông tin cần lưu ý khi tái lập kết quả.

Số box trong `smoke.json` là A = 1, B = 13 và C = 6, khớp với các file JSON và CSV tương ứng.

## 4. Kết quả ba lượt chạy

| Lượt chạy | Delta (m) | Voxel size | Số box | Mean z (m) | Phân bố class |
|---|---:|---:|---:|---:|---|
| A | 0 | 0.16 | 1 | 0.330 | 1 `vehicles` |
| B | 1.73 | 0.16 | 13 | 1.034 | 10 `vehicles`, 1 `two-wheels`, 2 `pedestrian` |
| C | 1.73 | 0.32 | 6 | 1.091 | 6 `pedestrian` |

Các giá trị `mean_z` trong bảng được lấy từ `summary.csv` và được ghi với ba chữ số thập phân. Khi tính trực tiếp từ các box JSON, các giá trị lần lượt xấp xỉ 0.330105, 1.034280 và 1.090763 m, phù hợp với số đã làm tròn trong CSV.

### 4.1. So sánh A và B

A và B có cùng `voxel_size = 0.16`, nhưng `delta` thay đổi từ 0 thành 1.73 m. Output thay đổi từ 1 box lên 13 box. A chỉ có một box `vehicles`, trong khi B có 10 `vehicles`, 1 `two-wheels` và 2 `pedestrian`. `mean_z` tăng từ 0.330 lên 1.034 m.

Evidence cho thấy việc thay đổi `delta` đi kèm với thay đổi lớn của output trong lần chạy này. Tuy nhiên, chưa đủ bằng chứng để khẳng định riêng `delta` là nguyên nhân duy nhất tạo ra mọi khác biệt hoặc B chính xác hơn A, vì không có ground truth và metric đánh giá.

### 4.2. So sánh B và C

B và C có cùng `delta = 1.73 m`, nhưng `voxel_size` thay đổi từ 0.16 lên 0.32. Số box giảm từ 13 xuống 6. B có ba class, còn cả 6 box của C đều là `pedestrian`. `mean_z` thay đổi từ 1.034 lên 1.091 m.

Evidence cho thấy output nhạy với thay đổi voxel size trong lần chạy này. Chưa đủ bằng chứng để kết luận voxel 0.16 hay 0.32 tốt hơn, vì chưa biết box nào đúng, box nào là false positive và đối tượng nào bị bỏ sót.

### 4.3. Vì sao không thể chỉ nhìn số lượng box

Số box chỉ cho biết mô hình đã xuất ra bao nhiêu prediction sau cấu hình và ngưỡng đang dùng. Nhiều box hơn có thể là phát hiện thêm đối tượng thật, nhưng cũng có thể gồm nhiều false positive. Ít box hơn có thể làm kết quả gọn hơn, nhưng cũng có thể bỏ sót đối tượng.

Folder hiện tại không có ground truth, phép đối sánh prediction với nhãn chuẩn, precision, recall, IoU hay AP. Do đó, chỉ từ A = 1, B = 13 và C = 6 không thể suy ra cấu hình nào tốt hơn. Đặc biệt, không gọi B là cấu hình “tốt nhất”; B chỉ là cấu hình tạo nhiều box nhất trong ba lần chạy đã ghi nhận.

## 5. Nhận xét ảnh Side view

Ảnh Side view biểu diễn điểm và box trên mặt phẳng `x–z`. Các ảnh hỗ trợ quan sát độ cao tương đối của box so với đường z = 0 và giúp nhận ra lỗi dịch z hàng loạt hoặc một box lệch rõ rệt.

Tuy nhiên, Side view làm mất thông tin theo trục y. Một số box khác nhau trong không gian 3D có thể chồng lên nhau khi chiếu sang `x–z`. Ảnh này cũng không đủ để xác nhận đầy đủ vị trí 3D, hướng yaw, độ khớp theo mặt phẳng ngang, class đúng hay sai, false positive, false negative hoặc độ chính xác tổng thể. Với các kết luận đó, hiện **chưa đủ bằng chứng**.

## 6. Phân tích các tình huống QC

Theo `qc-cases/manifest.json`, ba case đều được tạo từ prediction của run B. SHA-256 nguồn trong manifest và các case khớp với `prediction_sha256` của run B trong `smoke.json`. Mỗi case có 13 box với phân bố 10 `vehicles`, 1 `two-wheels` và 2 `pedestrian`.

Giá trị dùng trong lần chạy thật:

- `delta = 1.73 m`.
- `z_ground = 0.075 m`.
- Batch offset: `delta + z_ground = 1.73 + 0.075 = 1.805 m`.

### 6.1. `case-correct`

`case-correct.json` là bản sao không sửa của prediction nguồn. Giá trị mean z tính từ JSON xấp xỉ 1.034280 m. Ảnh `side-correct.png` trùng với Side view của run B về số lượng và vị trí box thể hiện trên hình.

Tên “correct” ở đây chỉ mô tả case không bị chèn lỗi z. Manifest ghi rõ đây vẫn là model output, không phải accuracy reference. Vì vậy, chưa đủ bằng chứng để nói các box hoặc class trong case này đều đúng so với cảnh thật.

### 6.2. `case-batch-z`

Trong `case-batch-z.json`, z của toàn bộ 13 box thấp hơn đúng 1.805 m so với `case-correct.json`; các trường x, y, kích thước, yaw, score và label tương ứng được giữ nguyên. Mean z của case này xấp xỉ -0.770720 m. Ảnh `side-batch-z.png` cho thấy hàng loạt box bị hạ xuống dưới vùng điểm và đường z = 0.

Đây là mẫu lỗi batch, phù hợp với mô tả trong manifest: bỏ sót phép biến đổi nghịch. Khi thấy nhiều hoặc toàn bộ box cùng lệch z có hệ thống, tôi phải dừng pipeline trước khi import hay tiếp tục xử lý. Cần kiểm tra lại hệ tọa độ, dấu của phép dịch, `delta`, `z_ground` và bước chuyển đổi nghịch; không sửa tay từng box vì nguyên nhân có tính toàn cục.

### 6.3. `case-one-box-z`

Trong `case-one-box-z.json`, chỉ box đầu tiên có z bị hạ 1.805 m; 12 box còn lại có z giống `case-correct.json`. Mean z của case xấp xỉ 0.895434 m. Ảnh `side-one-box-z.png` thể hiện một box lệch xuống rõ rệt trong khi các box còn lại giữ vị trí như case không chèn lỗi.

Nếu đã kiểm tra và xác nhận lỗi chỉ thuộc một object, có thể sửa hoặc loại riêng object đó rồi QC lại. Không nên áp dụng phép dịch hàng loạt, vì thao tác đó sẽ làm sai 12 box còn lại. Nếu chưa xác định được lỗi là cục bộ hay dấu hiệu của vấn đề hệ thống, phải tạm dừng quyết định import; hiện **chưa đủ bằng chứng** để tự động suy rộng từ một box sang cả batch.

### 6.4. Phép biến đổi z thuận và nghịch

Có thể hiểu đơn giản rằng phép biến đổi thuận đưa dữ liệu từ hệ nguồn sang hệ dùng cho mô hình bằng một độ dịch z đã quy định. Khi đưa prediction trở lại hệ nguồn, phép biến đổi nghịch phải hoàn lại đúng độ dịch đó với chiều ngược lại.

Trong bộ QC này, lượng dịch kiểm tra là `delta + z_ground = 1.805 m`. Nếu bước nghịch bị bỏ sót, mọi box có thể thấp hơn vị trí đúng trong hệ nguồn đúng 1.805 m, như `case-batch-z`. Nếu chỉ một object bị áp dụng sai, kết quả giống `case-one-box-z`. Công thức tổng quát cụ thể của pipeline ngoài quan hệ được manifest mô tả là **chưa đủ bằng chứng**, vì folder không chứa mã chuyển đổi hoặc tài liệu định nghĩa toàn bộ các hệ tọa độ.

## 7. Khả năng import JSON

Ba JSON trong `qc-cases` đều chứa `training_only: true` và cảnh báo: `TRAINING ONLY: not ground truth, not for CVAT import or quality claims.` Vì vậy, các file này không được import thẳng vào CVAT và không được dùng để tuyên bố chất lượng.

Các JSON prediction trong run A, B và C là cấu trúc nội bộ gồm metadata và danh sách box. Folder hiện tại không cung cấp evidence về schema import đích, mapping class, quy ước tâm hay đáy box, định nghĩa góc yaw, phép đổi hệ tọa độ đầy đủ hoặc bước kiểm tra tương thích với CVAT. Do đó, các JSON này cũng **chưa đủ cơ sở để import thẳng**. Cần có bước chuyển đổi schema và QC được xác nhận trước khi import.

## 8. Nhận xét cá nhân

Trong bài cá nhân này, tôi đã chạy và kiểm tra workflow gồm smoke test, ba cấu hình A/B/C và các tình huống QC; sau đó tôi đối chiếu `smoke.json`, CSV, JSON box và ảnh Side view. Evidence cho thấy tất cả bước trong smoke workflow đều `passed`, đồng thời số box giữa smoke, CSV và JSON khớp nhau.

Quan sát có evidence rõ nhất của tôi là run B và run C dùng cùng `delta = 1.73 m` nhưng khi voxel size đổi từ 0.16 sang 0.32, output đổi từ 13 box thuộc ba class thành 6 box đều là `pedestrian`. Điều này cho thấy output của lần thử này thay đổi đáng kể theo cấu hình voxel, nhưng chưa chứng minh cấu hình nào chính xác hơn.

Tôi hiểu phép đổi z như việc chuyển cùng một vật thể giữa hai thước đo có mốc cao độ khác nhau: chiều thuận dịch dữ liệu sang mốc mà mô hình sử dụng, còn chiều nghịch trả prediction về mốc ban đầu. Hai chiều phải hoàn nhau. Với lần chạy này, offset kiểm tra là 1.805 m; quên bước nghịch có thể làm cả batch lệch cùng một lượng.

Khi gặp lỗi batch như `case-batch-z`, quyết định của tôi là dừng pipeline, không import và không sửa tay từng object. Tôi sẽ kiểm tra phép đổi hệ tọa độ rồi chạy lại và QC lại. Khi chỉ một object lệch như `case-one-box-z`, tôi chỉ sửa hoặc loại object đó sau khi xác nhận lỗi thực sự cục bộ.

Điều tôi chưa chắc chắn là cấu hình nào cho prediction chính xác nhất và các class dự đoán có đúng với cảnh hay không. Folder hiện tại không có ground truth hoặc metric đánh giá, nên **chưa đủ bằng chứng** để trả lời hai vấn đề này.

## 9. Kết luận

Quy trình đã hoàn tất về mặt thực thi kỹ thuật theo trạng thái `passed` trong `smoke.json`. Ba cấu hình tạo ra output khác nhau rõ rệt, nhưng số lượng box không phải bằng chứng về độ chính xác. Side view hữu ích để phát hiện lệch z, đặc biệt là lỗi batch 1.805 m, song không thay thế được đánh giá 3D và đối chiếu ground truth. Trước khi dùng prediction để pre-label trong công cụ đích, cần xác nhận schema, hệ tọa độ và thực hiện QC; với evidence hiện có, chưa thể tuyên bố cấu hình tốt nhất hoặc import thẳng các JSON.
