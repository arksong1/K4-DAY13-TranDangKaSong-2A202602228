# Báo cáo thực hành PointPillars — Day 13

Giữ bản đã điền ngoài Git, trong thư mục nhóm private do LC thu. Đây là kiểm tra formative; không ghi điểm của người khác.

## Nhóm và provenance

- Mã nhóm/phòng: 01
- Thành viên: xem `TEAMMATES.md` (họ tên/MSSV, vai trò từng lượt).
- Trạng thái: `provided-results`
- Người thực sự chạy; ngày/giờ; hệ máy/architecture: Hệ thống kiểm tra `smoke.json` / `linux amd64` (4 CPUs, 4GB RAM)
- Image tag và image ID; phiên bản repo: `day13-pointpillars:lc-20261001-amd64`
- PCD được cấp / frame_id; nơi được phép chạy; fingerprint nếu LC cấp: `frame_id="demo"`, dữ liệu gốc `KITTI`
- Checkpoint: PointPillars KITTI có sẵn trong image; ghi checkpoint ID/hash nếu LC cấp: `epoch_160.pth`
- Phạm vi: front-window; score threshold:
- Giả định kênh thứ tư/intensity và nguồn z_ground: `z_ground = 0.075`

## Ba lượt inference thật

| Lượt | delta | Pillar XY | Số hộp | mean_z | File JSON/Side/CSV | Quan sát có bằng chứng |
| --- | --- | --- | --- | --- | --- | --- |
| A | 0 | 0.16 | 1 | | run-A/boxes-demo-delta-0-voxel-0.16.json | 1 xe tại x=13.15, y=-0.45, z=0.33, score 0.32 |
| B | 1.73 | 0.16 | 13 | | run-B/boxes-demo-delta-1.73-voxel-0.16.json | 10 vehicles, 1 two-wheels, 2 pedestrian rải rác |
| C | 1.73 | 0.32 | 6 | | run-C/boxes-demo-delta-1.73-voxel-0.32.json | Lưới thô, nhận sai toàn bộ 6 hộp thành pedestrian |

- A/B: thay input trước model có khác dịch cùng một hằng số cho output không? Vì sao?
  Khác biệt. Đổi `delta` trước khi chạy inference làm thay đổi phân phối các điểm rơi vào bên trong voxel. Model thấy cấu trúc hình học khác nên tạo dự đoán mới. Dịch hộp sau inference chỉ cộng/trừ hằng số vào hộp có sẵn, không sinh hộp mới.
- B/C: thấy gì khi đổi pillar? Có đủ bằng chứng để nói cấu hình nào tốt hơn không?
  Pillar thô hơn (0.32) làm nhận diện kém đi hẳn (sai class). Cấu hình 0.16 tốt hơn.
- Giới hạn ROI và góc Side ảnh hưởng cách đọc miss/yaw thế nào?
  Ảnh chiếu x-z đè các hộp khác y lên nhau, khó quan sát hình học, dễ sai nếu không mở JSON.
- JSON nào còn chưa đủ cơ sở để import? Cần kiểm gì tiếp?
  Cần kiểm tra kỹ các file lỗi transform hoặc nhận sai class.

## Ca QC có kiểm soát — không import CVAT

| Ca | Số hộp lệch z / tổng hộp | Lượng lệch | Class/x/y/yaw có đổi? | Dừng batch, kiểm từng hộp hay chưa rõ? | Bằng chứng |
| --- | --- | --- | --- | --- | --- |
| case-correct | 0/13 | 0 | Không | | Giữ nguyên phép biến đổi |
| case-batch-z | 13/13 | 1.805m | Không | Dừng batch | Cùng lệch một hằng số, lỗi transform mức pipeline |
| case-one-box-z | 1/13 | 1 hộp z=-0.883 | Không đổi 12 hộp còn lại | Kiểm từng hộp | Chỉ 1 hộp lệch, lỗi cục bộ |

Ghi rõ helper tạo biến đổi có chủ đích từ prediction, không phải kết quả inference riêng hoặc nhãn đúng. (Tổng độ dời là delta + z_ground = 1.73 + 0.075 = 1.805m)

## Nhận xét cá nhân

Mỗi thành viên tự viết một mục: vai trò đã làm; một quan sát A/B/C có dẫn file hoặc hộp/vùng; diễn giải phép z thuận/ngược; một quyết định lỗi batch và hành động; điều chưa chắc. Chỉ đọc kết quả chuẩn bị trước thì ghi rõ chưa tự chạy.

- **Thành viên:** [Trần Đăng Ka Song] - [2A202602228]
- **Vai trò thực hiện:** `provided-results` (chỉ phân tích bộ output cấp sẵn).
- **Quan sát / Hành động:** Xác định được biến số `delta` tác động lên khả năng phát hiện vật, và hiểu được sự phân biệt giữa việc sai transform pipeline (batch-z) và dự đoán sai đối tượng (one-box-z).
- **Diễn giải phép z thuận/ngược:** Điểm nguồn dời lên z_model = z_source - z_ground - delta. Hộp kết quả dời ngược lại z_source = z_model + z_ground + delta.

## LC ghi nhận riêng

- Quyền dùng PCD/image và đúng ca:
- Có chạy thật / chỉ phân tích; còn cần lượt thực hành bổ sung:
- Output đủ, giữ bản gốc, không đưa ca lỗi vào CVAT:
- Nhận xét từng thành viên và quyết định dừng pipeline:
- Đồng ý chuyển sang chỉnh/QC / cần bổ sung; lý do:
