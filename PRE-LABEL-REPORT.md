# Báo cáo thực hành PointPillars — Day 13

Bản tổng hợp formative từ các kết quả có sẵn trong thư mục. Không ghi điểm của người khác. Cần chuyển bản có thông tin nhóm sang thư mục private do LC chỉ định trước khi nộp.

## Nhóm và provenance

- Mã nhóm/phòng: .
- Thành viên và vai trò: Trần Minh Hiếu - 2A202602292
- Trạng thái: `provided-results` theo bằng chứng hiện có; smoke và ba lượt inference đều passed
- Người thực sự chạy: Trần Minh Hiếu
- Image: `day13-pointpillars:lc-20261001-amd64`; image ID `sha256:e03983bd922ec29890bf547db8de408402efd82583680b62e671c20da2fd2c82`.
- Revision: `0831856d921609312d42c7582c366e5a311bb7b1` 
- PCD/frame: `input/demo.pcd`, `frame_id=demo`, 17.238 điểm; KITTI Vision Benchmark Suite / MMDetection3D demo 000008. SHA-256 PCD: `3b5ea3da13e2b19149cab6a8d521c2ca55f2df93f026b5a3f8c273ce70645d60`. PCD giữ x/y, cộng z +1.73 m, bỏ reflectance gốc và dùng RGB placeholder bằng 0. Dùng theo phạm vi học thuật phi thương mại CC BY-NC-SA 3.0; không có dữ liệu Robotaxi/VinFast. Nơi chạy được LC cho phép/fingerprint LC cấp:  giấy phép CC BY-NC-SA 3.0
- Checkpoint: `/opt/PointPillars/pretrained/epoch_160.pth`; SHA-256 `482dfcf63b932cc5ccf012b4bbdad52aa51aa33becf87d0a39d61c39b377b5b1`.
- Cấu hình: front-window của preset KITTI, score threshold 0.3; giữ nguyên checkpoint. `z_ground` được ước lượng từ dữ liệu, giá trị trong output là 0.075 m, không phải ground truth đo mặt đường.
- Kênh thứ tư: PCD không còn reflectance thật. Adapter dùng giá trị hằng; với preset KITTI, đọc reflectance 0 cho class vehicles và 0.7 cho pedestrian/two-wheels. Đây là giả định adapter, không phải intensity đo được.

## Ba lượt inference thật

| Lượt | delta (m) | Pillar XY (m) | Số hộp | mean_z (m) | File JSON/Side/CSV | Quan sát có bằng chứng |
| --- | ---: | ---: | ---: | ---: | --- | --- |
| A | 0 | 0.16 | 1 | 0.330 | `ket-qua-nhom-01/run-A/boxes-demo-delta-0-voxel-0.16.json`; `ket-qua-nhom-01/run-A/side-demo-delta-0-voxel-0.16.png`; `ket-qua-nhom-01/run-A/summary.csv` | Side có một hộp trong vùng gần x≈13 m; smoke ghi 1 hộp. |
| B | 1.73 | 0.16 | 13 | 1.034 | `ket-qua-nhom-01/run-B/boxes-demo-delta-1.73-voxel-0.16.json`; `ket-qua-nhom-01/run-B/side-demo-delta-1.73-voxel-0.16.png`; `ket-qua-nhom-01/run-B/summary.csv` | JSON: 10 vehicles, 1 two-wheels, 2 pedestrian. Side cho thấy nhiều hộp, gồm hộp ở vùng xa và các hộp gần nhau; cần QC chứ không suy ra độ đúng từ số lượng. |
| C | 1.73 | 0.32 | 6 | 1.091 | `ket-qua-nhom-01/run-C/boxes-demo-delta-1.73-voxel-0.32.json`; `ket-qua-nhom-01/run-C/side-demo-delta-1.73-voxel-0.32.png`; `ket-qua-nhom-01/run-C/summary.csv` | JSON có 6 hộp pedestrian. So với B, số hộp giảm 13 xuống 6 và phân bố class khác. |

- **A/B:** Không phải chỉ dịch cùng một hằng số ở output. Dù delta đầu vào tăng 1.73 m, số hộp đổi từ 1 thành 13; tọa độ và class dự đoán cũng khác. Mô hình/voxelization và bước lọc score có thể phản ứng khác khi đầu vào đổi hệ z; không có tính chất bảo đảm equivariance trong kết quả này. `z_ground` ở hai output đều 0.075 m. Đây là quan sát trên một frame, không phải kết luận tổng quát về mô hình.
- **B/C:** Chỉ đổi pillar XY từ 0.16 sang 0.32 m trong cấu hình được ghi; output đổi từ 13 hộp (10 vehicles, 1 two-wheels, 2 pedestrian) sang 6 hộp (đều pedestrian). Đây là bằng chứng cấu hình ảnh hưởng kết quả, nhưng không có ground truth nên không đủ cơ sở nói cấu hình nào tốt hơn.
- ROI của preset KITTI chỉ giữ vùng trước trong model window (x 0–69.12 m, y −39.68–39.68 m, z −3–1 m); đối tượng ngoài vùng này không được đánh giá. Side là phép chiếu x-z: nó giúp thấy lệch độ cao/hộp rơi khỏi mặt phẳng điểm, nhưng không thể hiện đầy đủ y và không đủ để kết luận yaw. Muốn xác nhận miss/yaw cần xem point cloud từ nhiều góc, đặc biệt top view, cùng tọa độ và hộp trong JSON.
- Không JSON nào ở đây đủ cơ sở để import CVAT: A/B/C là prediction trên demo KITTI, không có nhãn chuẩn và không phải frame Robotaxi; ba JSON QC còn được đánh dấu training-only. Cần xác nhận đúng dữ liệu/frame và quyền sử dụng, kiểm transform/frame_id, class, tọa độ, kích thước, z đáy/tâm, yaw qua nhiều view và duyệt từng hộp trước mọi quyết định import.

## Ca QC có kiểm soát — không import CVAT

| Ca | Số hộp lệch z / tổng hộp | Lượng lệch | Class/x/y/yaw có đổi? | Hành động QC | Bằng chứng |
| --- | ---: | --- | --- | --- | --- |
| case-correct | 0/13 | 0 m; bản sao không đổi của prediction B | Không | Không có lỗi z được cài; vẫn không coi là nhãn đúng hoặc import. | `ket-qua-nhom-01/qc-cases/case-correct.json`, `ket-qua-nhom-01/qc-cases/side-correct.png`; manifest trỏ về prediction B. |
| case-batch-z | 13/13 | −1.805 m mỗi hộp (−(delta 1.73 + z_ground 0.075)) | Không | Dừng batch; kiểm tra phép đổi hệ tọa độ/chiều nghịch đảo trước khi tiếp tục. | `ket-qua-nhom-01/qc-cases/case-batch-z.json`, `ket-qua-nhom-01/qc-cases/side-batch-z.png`; toàn bộ hộp Side nằm thấp hơn mặt phẳng điểm. |
| case-one-box-z | 1/13 | −1.805 m ở hộp đầu tiên | Không | Kiểm hộp bị ảnh hưởng qua point cloud và nhiều view; giữ các hộp khác ở trạng thái chờ QC, không tự động chấp nhận cả batch. | `ket-qua-nhom-01/qc-cases/case-one-box-z.json`, `ket-qua-nhom-01/qc-cases/side-one-box-z.png`; chỉ một hộp rơi xuống dưới mặt phẳng điểm. |

Các ca trên do `practice/pipeline-qc-cases.py` tạo có chủ đích từ prediction B (`2ffb4e85d1a8746b1f290f6704204bf57e5521ae3df98ea81473a087a066fcfc`). Đây không phải inference riêng, nhãn chuẩn hay bằng chứng accuracy. Không import bất kỳ `case-*.json` nào vào CVAT.

## Nhận xét cá nhân

Chưa thể ghi nhận xét riêng vì bảng thành viên/vai trò còn trống và không có xác nhận người chạy. Mỗi thành viên tự bổ sung trong bản private: vai trò thực tế; một quan sát A/B/C dẫn file hoặc hộp/vùng; chiều thuận/ngược của phép z; quyết định khi gặp lỗi batch và hành động; điều còn chưa chắc. Chỉ đọc kết quả có sẵn thì ghi rõ `provided-results`, không ghi đã tự chạy inference.

## LC ghi nhận riêng

- Quyền dùng PCD/image và đúng ca: chờ LC xác nhận; dữ liệu trong gói là demo KITTI, không phải dữ liệu Robotaxi.
- Có chạy thật hay chỉ phân tích; cần lượt bổ sung: chờ LC xác nhận. Kết quả smoke ghi Linux/amd64 passed nhưng không ghi người vận hành.
- Output và bản gốc: thư mục có smoke passed, đủ A/B/C JSON/Side/CSV và ba JSON/Side QC; bản gốc/output cần được giữ theo quy trình nhóm. Không đưa prediction demo hoặc ca QC vào CVAT.
- Nhận xét từng thành viên và quyết định dừng pipeline: LC điền sau khi nghe từng thành viên; dữ kiện QC hiện tại ủng hộ dừng batch khi toàn bộ z lệch.
- Chuyển sang chỉnh/QC hay cần bổ sung: chờ LC quyết định sau khi xác nhận provenance, người thực hiện và các mục QC còn thiếu.
