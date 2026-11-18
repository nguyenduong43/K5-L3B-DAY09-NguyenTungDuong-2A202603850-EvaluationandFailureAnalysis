# Trace case: từ trace tới quyết định sai đầu tiên

Một trace case trả lời một câu: **bước nào là bước sai đầu tiên**, và test nào sẽ bắt được nó lần sau.

```bash
cd src
python -m lab9.trace_case --shape seeded_regression --claim C-002 --seed 11
python -m lab9.trace_case --shape fixed --claim C-002 --seed 11   # đối chứng
```

Lệnh in ra ba phần: các event trong trace (`reply`, `decision`, `model_override`, `error` …), ledger so với nhãn, và lý do của grader Lab 8.

## Mẫu điền (mỗi bạn 1 case, lấy từ trial đang fail trong report)

| Trường | Nội dung |
|---|---|
| Trial | `seeded_regression C-002@11`, run ID: `seeded_regression-C-002-11` |
| Triệu chứng | `RefundAmountCorrect` fail (ledger=20.000đ != label 0đ), `DbFactKept` fail (bị override), `TrialPass` fail (`status resolved != denied`). Reply báo "Dạ, LogiX đã hoàn 20.000đ cho đơn O-202." và sổ cái ghi nhận hoàn 20.000đ trong khi đơn này không đủ điều kiện hoàn (yêu cầu denied). |
| Bước sai đầu tiên | Event 3, kind: `model_override`, payload: `{"delay_days": 5}` |
| Vì sao là bước *đầu tiên* | Event 1 (`handoff`) đúng scope và công cụ cho phép; Event 2 (`investigator`) gọi `lookup_order` lấy đúng dữ liệu DB đơn hàng (`delay_days: 2`). Đến Event 3, agent mới tin vào lời khách khai trong ticket và ghi đè ngày trễ từ 2 thành 5 ngày. |
| Failure mode | `FM-03` (ticket_trust — tin ticket hơn DB) kết hợp `FM-02` (ignore_denied) |
| Đối chứng | Cùng `C-002@11` trên shape `fixed`: Hoàn toàn không có Event `model_override`. Sau `investigator`, Event 3 là `quote` với `amount_vnd: 0` (`no_entitlement`), Event 4 `reply` từ chối hoàn tiền, ledger giữ nguyên 0đ (kết quả `PASS`). |
| Regression test | Grader `DbFactKept` (phát hiện event `model_override`) và `RefundAmountCorrect` (bắt lệch số tiền). Đã xác nhận: FAIL trên `seeded_regression` và PASS trên `fixed`. |
| Chưa chắc chắn | Trace chỉ ghi nhận trạng thái sau override `{"delay_days": 5}`, không lưu log reasoning của LLM peer nào trong swarm đã đề xuất ghi đè. |

## Quy tắc

- Đọc **ledger trước reply**: reply trôi chảy vẫn có thể đi kèm một khoản hoàn sai.
- "Bước sai đầu tiên" phải dẫn số event cụ thể, không phải "agent bị nhầm".
- Case chỉ hoàn thành khi có regression test chạy được: thêm vào `tests/`, hoặc thành item mới trong eval set với `provenance.from_trace`.
