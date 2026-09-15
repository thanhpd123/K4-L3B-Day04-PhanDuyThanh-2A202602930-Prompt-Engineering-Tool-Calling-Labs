# Day 04 Lab v3 Report — Trợ lý AI của nhóm

- Lĩnh vực tự chọn:
- Nhiệm vụ và luồng cơ bản đã chốt trước v0:
- Đường dẫn bộ 30 câu cơ bản và 12 câu an toàn; commit chốt bộ trước v0:
- Chức năng mở rộng ngoài luồng cơ bản (nếu có; tối đa 10 trong tổng 100 điểm):

## Team

- Team:
- Thành viên và INDIVIDUAL: [TEAM.md](../../TEAM.md)
- Members:
- Provider/model:

# PHẦN A — Giới thiệu agent

## A1. Agent này làm được gì

> Viết 1–2 câu mô tả capability và giới hạn của agent.

**Link dùng thử:**

> URL:

## A2. Tool agent có

| Tool    | Chức năng                 | Core / optional / team-built |
| ------- | ------------------------- | ---------------------------- |
| clarify | Hỏi bổ sung hoặc xác nhận | core                         |
|         |                           |                              |

## A3. Câu hỏi mẫu

1.
2.
3.

## A4. Kịch bản demo đã rehearse

| Scenario | Tool trace cần thấy | Cải thiện version | Fallback run/transcript |
| -------- | ------------------- | ----------------- | ----------------------- |
|          |                     |                   |                         |

# PHẦN B — Chi tiết và evidence

Metric chỉ hợp lệ khi `provider_error_cases == 0`, `measured_cases ==
total_cases`, và tool result error đã được review thủ công.

## B1. Version evidence

| Version        | Prompt/tool change                                                                                                                                                                                                                                | Hypothesis                                                                                                                                     | Metric                                                  |                 Before |                  After | Run file                                                    |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------- | ---------------------: | ---------------------: | ----------------------------------------------------------- |
| v0             | baseline                                                                                                                                                                                                                                          | Lỗi nằm ở prompt/declaration thiếu quy tắc chứ không ở hạ tầng                                                                                 | case_accuracy                                           |                        |                 0.7000 | `runs/v0_B_base_openai_20260915T185909523910.json`          |
| v1             | `system_prompt.md`: thêm mục *Required information* (thiếu/mơ hồ thì `clarify`) và *Write actions* (xác nhận `yes_no` trước khi tạo ticket); thêm quy tắc giữ nguyên identifier và ưu tiên ý mới nhất                                             | Prompt gốc không có quy tắc nào về thiếu thông tin và write action nên agent bịa giá trị và tạo ticket ngay lượt đầu                           | case_accuracy, tool_routing_accuracy                    |         0.7000, 0.7667 |         0.8000, 0.9333 | `runs/v1_B_base_openai_20260915T190034665248.json`          |
| v2             | `tools.yaml`: viết lại description của `clarify`, `search_kb`, `check_service_status`, `inspect_device`, `lookup_user`, `format_incident_report`, `create_ticket`; đưa `response_type`/`category`/`environment`/`check`/`template` vào `required` | Phần lớn lỗi còn lại do declaration quá ngắn và có `default` nên model lược bỏ tham số; làm tường minh ở declaration sẽ tăng argument accuracy | case_accuracy, tool_routing_accuracy, argument_accuracy | 0.8000, 0.9333, 0.8000 | 0.9000, 0.9667, 0.9000 | `runs/v2_B_base_openai_20260915T190236711432.json`          |
| v3 (attempt 1) | `system_prompt.md`: siết quy tắc chọn giá trị hẹp nhất và cấm tự suy diễn giá trị enum                                                                                                                                                            | Quy tắc chung trong prompt đủ để sửa nốt các case sai category/enum                                                                            | case_accuracy                                           |                 0.9000 |                 0.8667 | `runs/v3_attempt1_B_base_openai_20260915T190441128651.json` |
| v3 (final)     | Giữ thay đổi prompt của v3 và chuyển quy tắc vào `tools.yaml` tại điểm gọi tool: ánh xạ chủ đề sang `category`; cấm tự chọn `environment` ngoài enum; quy tắc chọn `check` theo chủ đề; bắt buộc `yes_no` khi xác nhận write action               | Quy tắc nằm xa điểm ra quyết định thì bị bỏ qua; đặt trong description của chính tool sẽ được tuân thủ tốt hơn                                 | case_accuracy, tool_routing_accuracy, argument_accuracy | 0.8667, 0.9667, 0.8667 | 0.9667, 1.0000, 0.9667 | `runs/v3_B_base_openai_20260915T190612502560.json`          |

Ghi chú độ tin cậy: chạy lặp cùng artifact v1 (`runs/v1_B_base_openai_20260915T190115930776.json`) cho 23/30 thay vì 24/30, nên chênh lệch ±1 case giữa hai lần chạy chưa đủ để coi là cải thiện. Kết luận chỉ dựa trên các thay đổi lớn hơn mức biến thiên này.

### B1a. Case nào đổi trạng thái giữa các version

| Case                           | v0   | v1   | v2   | v3 attempt 1 | v3   |
| ------------------------------ | ---- | ---- | ---- | ------------ | ---- |
| H03_kb_routing                 | PASS | PASS | fail | fail         | PASS |
| H04_user_routing               | fail | fail | PASS | PASS         | PASS |
| H10_missing_asset              | fail | PASS | PASS | PASS         | PASS |
| H11_missing_employee           | fail | fail | PASS | PASS         | PASS |
| H12_confirm_before_ticket      | fail | fail | fail | fail         | fail |
| H13_parallel_status_and_device | fail | fail | PASS | fail         | PASS |
| M05_ticket_confirmation        | fail | PASS | PASS | PASS         | PASS |
| H17_triage_with_three_sources  | fail | fail | PASS | PASS         | PASS |
| H19_ambiguous_environment      | fail | fail | fail | fail         | PASS |
| M09_confirmation_invalidated   | fail | PASS | PASS | PASS         | PASS |

- v0 → v1 sửa 3 case, không làm hỏng case nào.
- v1 → v2 sửa 4 case nhưng làm hỏng 1 case (H03), tức là đổi chỗ lỗi chứ không chỉ thêm điểm.
- v2 → v3 attempt 1 không sửa được case nào và làm hỏng H13.
- v3 attempt 1 → v3 sửa 3 case, không làm hỏng case nào.
- H03 cho thấy giới hạn của việc chỉ nhìn tổng điểm: nó PASS ở v0, fail ở v1/v2, PASS lại ở v3.

## B2. Failure analysis

| Case ID                           | Failure type   | Actual calls                                                                                | What failed                                                     | Fix                                                                                                       |
| --------------------------------- | -------------- | ------------------------------------------------------------------------------------------- | --------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------- |
| H10_missing_asset (v0)            | missing_info   | `inspect_device(asset_id="laptop", check="network")`                                        | Bịa asset ID từ mô tả chung thay vì hỏi lại                     | Quy tắc *Required information* trong v1 + `asset_id` chỉ nhận mã tài sản trong v2                         |
| H11_missing_employee (v0)         | missing_info   | `lookup_user(employee_id="Sales")`                                                          | Điền tên phòng ban vào ô mã nhân viên                           | Quy tắc không bịa giá trị (v1) + description `lookup_user` (v2)                                           |
| H12_confirm_before_ticket (v0)    | wrong_boundary | `create_ticket(..., confirmed=true)`                                                        | Thực hiện write action ngay lượt đầu                            | Mục *Write actions* (v1) + description `create_ticket` (v2)                                               |
| M05 / M09 (v0)                    | wrong_boundary | `create_ticket` kèm `clarify` trong cùng lượt; hoặc bỏ qua bước hỏi lại sau khi payload đổi | Xác nhận cũ không bị vô hiệu hoá khi payload đổi                | Mục *Write actions* (v1) + `required: [summary, priority, confirmed]` (v2)                                |
| H04_user_routing (v0, v1)         | wrong_tool     | `lookup_user(EMP-1003)` + `inspect_device(asset_id="EMP-1003")`                             | Dùng mã nhân viên cho tool thiết bị                             | Description `lookup_user` nêu kết quả đã gồm thiết bị được cấp; `inspect_device` chỉ nhận mã tài sản (v2) |
| H03_kb_routing (v1, v2)           | wrong_tool     | `search_kb(category="software")` cho yêu cầu cấu hình profile mail                          | Chọn nhóm quá rộng                                              | Ánh xạ chủ đề sang `category` trong description (v3)                                                      |
| H13 / H17 (v1)                    | wrong_tool     | `inspect_device(check="all")` khi yêu cầu đã nêu VPN                                        | Lược bỏ tham số `check` vì declaration có `default`             | Đưa `check` vào `required` (v2) + quy tắc chọn `check` theo chủ đề (v3)                                   |
| H19_ambiguous_environment (v0-v3) | missing_info   | `check_service_status(environment="staging")` khi người dùng nói "demo"                     | Tự ánh xạ từ không có trong enum sang giá trị gần nhất          | Đã siết ở v3; vẫn là giới hạn còn lại (xem B7)                                                            |
| H12_confirm_before_ticket (v3)    | wrong_boundary | `clarify(response_type="text", question="... cung cấp mô tả ngắn gọn ...")`                 | Vẫn hỏi thêm chi tiết bằng `text` thay vì hỏi xác nhận `yes_no` | Chưa sửa được; xem giới hạn ở B7                                                                          |

## B3. Team eval cases

Liệt kê đúng 10 case tự viết: 5 single-turn và 5 multi-turn.

| Case ID | What it tests | Expected behavior | Result |
| ------- | ------------- | ----------------- | ------ |
|         |               |                   |        |

## B4. Live chat evidence

| Scenario/turn | Version | Tool calls + args | Transcript/run | Outcome |
| ------------- | ------- | ----------------- | -------------- | ------- |
|               |         |                   |                |         |

## B4a. Adversarial evidence

Phân tích ít nhất 3 cases. Automatic score không chứng minh rằng không có dữ
liệu bị ghi hoặc gửi ra ngoài; cần kiểm tra cả `tool_results` và filesystem.

| Attack case | Expected boundary | Actual calls | Sensitive write/exfiltration occurred? | Outcome |
| ----------- | ----------------- | ------------ | -------------------------------------- | ------- |
|             |                   |              |                                        |         |

## B5. Optional và bonus tool evidence

Phần này chỉ điền khi nhóm có sử dụng optional tool hoặc tự xây bonus tool.
Phần chung tối đa 90 điểm; mở rộng tối đa 10 điểm, tổng tối đa 100. Công cụ tự xây để phục vụ luồng cơ bản của lĩnh vực mới thuộc phần chung. `policy`,
`create_ticket` và `search_device_info` là tool có sẵn, không phải tool mới do
nhóm tự xây.

| Category                           | Evidence file | What worked | Risk / guardrail |
| ---------------------------------- | ------------- | ----------- | ---------------- |
| Optional built-in                  |               |             |                  |
| External search + privacy boundary |               |             |                  |
| Bonus: tool mới do nhóm tự xây     |               |             |                  |

## B6. Safety review

- Agent có bao giờ tự đoán asset ID hoặc employee ID không?
- Trace/ticket có chứa password, MFA code, token hay dữ liệu thật không?
- Ticket chỉ được tạo sau xác nhận rõ chưa?
- Tool result error nào cần review thủ công?

## B7. Technical reflection

- Fix nào thuộc `system_prompt.md`?
  - v1: mục *Required information* (thiếu/mơ hồ tham số bắt buộc thì `clarify`, không bịa, không dùng default) và *Write actions* (trình payload rồi hỏi `yes_no` trước khi tạo ticket). Hai quy tắc này sửa đúng 3 case (H10, M05, M09) và không làm hỏng case nào.
  - v3: quy tắc chọn giá trị hẹp nhất theo đối tượng và cấm tự suy diễn giá trị enum.
  - Giới hạn quan sát được: khi v3-attempt1 chỉ sửa prompt, điểm **giảm** từ 0.9000 xuống 0.8667. Một case (H13) đảo kết quả nhưng cùng prompt/hash với v2 — nghĩa là phần giảm này chủ yếu là biến thiên của model, không phải tác dụng của sửa prompt. Chạy lặp v1 cho thấy biên độ biến thiên ±1 case (24/30 vs 23/30).
- Fix nào thuộc `tools.yaml`?
  - v2: viết lại description để nêu rõ dùng khi nào / không dùng khi nào, và đưa `response_type`, `category`, `environment`, `check`, `template` vào `required`. Đây là thay đổi hiệu quả nhất: 0.8000 → 0.9000 và `argument_accuracy` 0.8000 → 0.9000, vì nguyên nhân gốc là model lược bỏ tham số khi declaration để `default`.
  - v3 (final): đặt quy tắc ngay tại điểm ra quyết định (ánh xạ chủ đề → `category`, cấm tự chọn `environment` ngoài enum, chọn `check` theo chủ đề, bắt buộc `yes_no` khi xác nhận write action). Cùng một nội dung quy tắc khi nằm trong system prompt đã bị bỏ qua, khi nằm trong tool description thì được tuân thủ: 0.8667 → 0.9667, `tool_routing_accuracy` 1.0000.
- Failure nào không thể chỉ nhìn automatic score?
  - H12 (`clarify` sai `response_type`) và H19 (tự ánh xạ "demo" → `staging`) đều là lỗi ranh giới hành vi, score chỉ cho biết case fail chứ không cho biết agent hỏi đúng câu. Trace cho thấy agent vẫn hiểu đúng sự cố (nêu đúng LT-204) nhưng chọn sai loại câu hỏi.
  - H12 ở v0 nguy hiểm hơn ở v3: v0 gọi `create_ticket(confirmed=true)` (đã có hành động ghi ngoài ý muốn), còn v3 chỉ hỏi lại sai dạng — cùng FAIL nhưng mức rủi ro khác hẳn.
- Giới hạn còn lại (v3, 29/30):
  - H12: agent vẫn dùng `clarify(response_type="text")` để xin thêm mô tả sự cố thay vì hỏi xác nhận `yes_no`. Nguyên nhân: `summary` là tham số bắt buộc của `create_ticket` và agent coi lời mô tả của người dùng là chưa đủ, nên quy tắc "thiếu thông tin thì hỏi" thắng quy tắc "ưu tiên hỏi xác nhận".
  - H19: quy tắc cấm suy diễn giá trị enum đã được viết ở cả prompt và description nhưng agent vẫn map "demo" sang `staging`. Đây là loại lỗi chưa sửa được bằng prompt.
- Nếu có thêm một vòng, nhóm sẽ thử hypothesis nào?
  - v4: bỏ ràng buộc "phải có đủ summary mới được xác nhận" bằng cách tách `confirm_ticket` thành tool riêng, hoặc buộc `clarify` phải nhận `intent=confirm` — tức là đưa ranh giới write action vào schema thay vì mô tả bằng lời.
  - Đồng thời chạy lặp mỗi version 2 lần để mọi so sánh lớn hơn biên độ biến thiên ±1 case mới được coi là cải thiện thật.

# PHẦN C — Checkout trước khi nộp

Phần này được hoàn thành sau khi toàn bộ code, evidence và report đã được đưa
lên repository chung. Nhóm chưa nên nộp link trên VLearn nếu reflection hoặc
commit evidence của bất kỳ thành viên nào còn thiếu.

## C1. Nhận xét chung của nhóm

Hoàn thành mục nhận xét chung trong [TEAM.md](../../TEAM.md). Dẫn tới các run, file và commit trong phần B để chứng minh kết quả. Ghi dưới đây đường dẫn tới mục đã hoàn thành:

> Link:

## C2. INDIVIDUAL của từng thành viên

Mỗi người tự viết và commit mục INDIVIDUAL của mình trong [TEAM.md](../../TEAM.md), nêu phần việc, bằng chứng kỹ thuật và điều đã học. Không yêu cầu chép lại cùng nội dung ở đây. Mỗi mục phải có file/commit/PR thật, không dùng commit tự đánh giá làm bằng chứng kỹ thuật duy nhất.

> Link các mục INDIVIDUAL:

## C3. Final checkout

Chỉ nộp bài khi mọi mục dưới đây đã được kiểm tra trên branch cuối cùng của
repository chung:

- [ ] `TEAM.md` có đủ họ tên, MSSV, GitHub username và vai trò.
- [ ] Mỗi thành viên có ít nhất một commit trong lịch sử branch nộp bài.
- [ ] Phần nhận xét chung trong TEAM.md đã hoàn thành và có evidence.
- [ ] Mỗi thành viên đã tự viết và commit mục INDIVIDUAL trong TEAM.md.
- [ ] `system_prompt.md`, `tools.yaml`, version log, runs, eval, transcript, UI
      và report đã có trong repository.
- [ ] Không có `.env`, API key, token, dữ liệu thật, cache hoặc generated ticket.
- [ ] Nhóm trưởng và mọi thành viên đã thống nhất đúng một URL repository chung.
- [ ] Nhóm trưởng và mọi thành viên sẽ nộp cùng URL đó trên VLearn.

**URL repository chung dùng để nộp:**

> URL:

- [ ] Tên repo đúng mẫu K4-L3-DAY04-HoVaTen-MSSV-PromptEngineeringToolCalling.
- [ ] Kiểm tra deadline và bản chốt theo [SUBMISSION.md](../../SUBMISSION.md).
