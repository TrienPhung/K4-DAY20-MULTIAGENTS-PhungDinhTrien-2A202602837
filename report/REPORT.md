# Báo cáo Lab: Self evolving Agentic

> Sao chép tệp này thành `report/REPORT.md` (đã làm ở Phần 0) và điền dần qua các Phần của lab. Xóa các dòng hướng dẫn dạng trích dẫn (bắt đầu bằng `>`). Văn phong kỹ thuật, ngắn gọn, mọi nhận định đi kèm số liệu hoặc bằng chứng. Trong buổi học: điền mục 1 đến 7 (bản nháp). Sau buổi học: hoàn thiện mục 8 đến 10.

## 1. Thông tin nhóm và cấu hình

| Họ tên | Mã sinh viên | Phần đóng góp |
|---|---|---|
| Phùng Đình Triển | 2A202602837 | Toàn bộ bài lab (100%) |

- Nhà cung cấp và mô hình (`LAB_MODEL`, không ghi khóa API), nhiệt độ (`LAB_TEMPERATURE`), `recursion_limit`: `openai:gpt-6-luna`, `LAB_TEMPERATURE=1`, `recursion_limit=60`
- Phiên bản Deep Agents (`pip show deepagents`), hệ điều hành, chạy trực tiếp hay trong Docker: `deepagents 0.7.21`, Windows 11 (Python 3.11.9), chạy trực tiếp
- Số lần chạy tác vụ đã dùng / ngân sách:
- Commit của tag `freeze`:

## 2. Giả thuyết (commit TRƯỚC tag `freeze`, Phần 4.0)

- H1 (subagents so với baseline): Trên tác vụ đánh giá, subagents sẽ đạt điểm tương đương baseline nhưng tiêu tốn lượng token gấp 2.5 đến 3.5 lần do overhead giao tiếp và phân rã ngữ cảnh, đồng thời không cải thiện được các quy ước ngầm của tổ chức (nhóm E) vì các subagent chỉ nhận thông tin từ prompt của tác tử chính mà không thừa kế tri thức đặc thù.
- H2 (skills-auto so với baseline): Trên tác vụ đánh giá, skills-auto sẽ đạt điểm cao hơn baseline ở các tác vụ có tính kế thừa quy ước (như định dạng tiền tệ, cấu trúc metadata, quy tắc changelog) nhờ tri thức thủ tục được nạp dần qua progressive disclosure, nhưng hiệu quả sẽ suy giảm đối với các quy ước hoàn toàn mới xuất hiện ở tập đánh giá.
- H3 (tác vụ học so với tác vụ đánh giá): Điểm số của skills-auto trên tác vụ học sẽ cao hơn rõ rệt so với tác vụ đánh giá (hiện tượng quá khớp quy ước / overfitting ở context layer theo nghiên cứu SkillEvolBench), do skill tự sinh khái quát hóa từ các lỗi cụ thể của tác vụ học.

## 3. Làm quen Deep Agents (Phần 0.3)

1. Tác tử mặc định có 9 công cụ: nhóm công cụ tệp (`ls`, `read_file`, `write_file`, `edit_file`, `delete`, `glob`, `grep`), nhóm shell (`execute`), và nhóm subagent (`task`). Công cụ cho phép chạy lệnh shell trên hệ điều hành là `execute`.
2. Mô tả công cụ `task` nói về subagent `general-purpose`: Đây là subagent đa dụng dùng để nghiên cứu các câu hỏi phức tạp, tìm kiếm tệp/nội dung và thực thi các tác vụ nhiều bước; có quyền truy cập đầy đủ tất cả công cụ như tác tử chính. Subagent này mặc định là không trạng thái (stateless), nó chỉ nhìn thấy duy nhất nội dung prompt mà tác tử chính gửi trong lời gọi và trả về một báo cáo cuối cùng, không nhìn thấy lịch sử hội thoại trước đó của tác tử chính.
3. Trích dẫn câu hướng dẫn hành vi:
   - Từ mô tả công cụ `task`: *"Launch multiple agents concurrently when their tasks are independent, using a single message with multiple tool calls."*
   - Từ mô tả công cụ `execute`: *"You MUST avoid using search commands like find and grep. Instead use the grep, glob tools to search. Use read_file rather than cat/head/tail."*

## 4. Đường cơ sở và phân loại lỗi (Phần 2.2)

| Tác vụ | Check thất bại | Nhóm lỗi (A-G) | Bằng chứng (trích ngắn từ `detail` hoặc vết) |
|---|---|---|---|
| `code-learn` | `tests_not_modified` | E. Vi phạm quy ước tổ chức | `the original files in tests/ must not be modified (new test files are allowed)` |
| `code-learn` | `rule_type_hints` | E. Vi phạm quy ước tổ chức | `RULE: every public function in the package has type annotations on all parameters and return` |
| `code-learn` | `rule_regression_tests` | E. Vi phạm quy ước tổ chức | `RULE: add tests/test_regressions.py with one test function per bug you fixed (at least 3)` |
| `code-learn` | `rule_changelog` | E. Vi phạm quy ước tổ chức | `RULE: record each fix in CHANGELOG.md under heading '## Unreleased' as bullet '- fix(fn): desc'` |
| `data-learn` | `rule_money_in_cents` | E. Vi phạm quy ước tổ chức | `RULE: money values in answer.json are integer cents (1606.67 USD is written 160667)` |
| `data-learn` | `rule_meta_block` | E. Vi phạm quy ước tổ chức | `RULE: answer.json has an object meta = {"source": ..., "rows_in": ..., "rows_used": ...}` |
| `data-learn` | `rule_clean_csv` | E. Vi phạm quy ước tổ chức | `RULE: write workspace/clean.csv with header order_id,timestamp_utc,region,amount_cents` |
| `logs-learn` | `rule_service_names` | E. Vi phạm quy ước tổ chức | `RULE: service names in the output are lower-case with '-' replaced by '_' (payment-service -> payment_service)` |
| `logs-learn` | `rule_sorted_errors` | E. Vi phạm quy ước tổ chức | `RULE: errors is sorted by service, then by timestamp_utc, ascending` |
| `logs-learn` | `rule_schema_header` | E. Vi phạm quy ước tổ chức | `RULE: the top-level object has "schema_version": 2 and "generated_by": "log-triage"` |

Nhận xét: 100% check thất bại ở đường cơ sở (10/10 check) thuộc **Nhóm E (Vi phạm quy ước tổ chức)**. Mô hình `gpt-6-luna` rất mạnh về mặt kỹ thuật, đạt 100% các check kỹ thuật (nhóm A-D: thuật toán parse price, lọc low-stock, tính doanh thu Q1, deduplicate, bóc tách exception), nhưng thất bại vì không thể tự biết trước các quy ước "house rules" nội bộ không được nêu rõ trong đề bài. Skill tự sinh hoàn toàn có thể phòng ngừa nhóm lỗi này bằng cách cung cấp danh sách checklist quy ước chuẩn hóa.

## 5. Điều kiện `subagents` (Phần 2.3)

- Các subagent đã định nghĩa:
  - `explorer`: đọc docstrings, README, khảo sát dữ liệu, báo cáo sự thật, chỉ đọc không sửa.
  - `implementer`: thực thi thay đổi cụ thể, chạy test hoặc script để kiểm tra mã nguồn.
  - `reviewer`: rà soát độc lập kết quả đầu ra theo yêu cầu và kiểm tra các trường hợp biên.
- `subagent_calls` ở từng tác vụ và nhận xét:
  - `code-learn`: 3 lượt gọi (tác tử chính giao việc cho subagent khảo sát và implement).
  - `data-learn`: 3 lượt gọi (tác tử chính phân rã việc đọc cấu trúc và xử lý data).
  - `logs-learn`: 0 lượt gọi (tác tử chính tự nhận định task đọc log đơn giản nên tự hoàn thành trực tiếp mà không cần ủy quyền).
- Thông tin giao việc: Tác tử chính truyền chi tiết mô tả yêu cầu trong prompt cho subagent, nhưng subagent không nhìn thấy lịch sử trước đó và cũng không thừa kế tri thức về quy ước ngầm.
- Ảnh hưởng đến token và thời gian: Chi phí token tăng vọt đáng kể: `code-learn` tăng từ 90,328 (baseline) lên 247,181 tokens (~2.7x); `data-learn` tăng từ 39,738 lên 160,493 tokens (~4x). Thời gian thực thi tăng tương ứng (211s vs 70s). Điểm số không cải thiện vì subagents không khắc phục được lỗi nhóm E.

## 6. Self-evolving: skill do curator sinh (Phần 3)

- Số lần chạy curator: 1 lần. Sinh ra 3 skill hợp lệ, không có skill nào bị từ chối hay phải xóa.

| Skill | Tổng quát hay riêng cho tác vụ học? | Đúng hay sai (nêu chỗ sai nếu có) | Độ dài, `description` và `skills_read` ở Phần 3.4 |
|---|---|---|---|
| `package-maintenance-checklist` | Tổng quát cho việc bảo trì mã nguồn gói Python | Đúng hoàn toàn, nêu rõ quy tắc regression tests, changelog, type hints | 11 dòng, mô tả rõ tình huống kích hoạt. `skills_read = 1` ở `code-learn` (điểm tăng từ 0.60 lên 0.90) |
| `tabular-data-normalization` | Tổng quát cho chuẩn hóa và làm sạch dữ liệu bảng | Đúng, hướng dẫn về đơn vị cents, trường meta và tệp clean.csv | 13 dòng, súc tích. `skills_read = 1` ở `data-learn` (điểm 0.625) |
| `log-output-contracts` | Tổng quát cho việc trích xuất và chuẩn hóa log | Đúng, hướng dẫn hạ chữ thường tên service, sắp xếp và gắn header schema | 11 dòng, tập trung. `skills_read = 1` ở `logs-learn` (điểm tăng từ 0.67 lên 0.89) |

## 7. Kết quả so sánh (Phần 4.3, 4.4)

> Dán nội dung `report/table.md` và kết quả `python scripts/check_breakdown.py`. Nêu các lần chạy có `error` hoặc `skills_modified = true` (nếu có) và cách xử lý.

```text
(dán bảng ở đây)
```

## 8. Phân tích

> Trả lời từng câu bằng số liệu từ mục 7 và bằng chứng từ vết. Kết quả âm hoặc không có khác biệt vẫn hợp lệ nếu được phân tích tốt.

1. So với `baseline`, điều kiện nào cải thiện điểm tác vụ **học**? Điều kiện nào cải thiện điểm tác vụ **đánh giá**? Có điều kiện nào cải thiện tác vụ học nhưng không cải thiện tác vụ đánh giá? Nếu có, đó là dấu hiệu gì?
2. Tách điểm thành check kỹ thuật và check quy ước (`rule_`). Skill do curator sinh giúp nhóm check nào? Check quy ước **mới** của tác vụ đánh giá có được skill giúp không, và vì sao?
3. Dựa vào vết và `skills_read`, giải thích một check mà skill giúp đạt và một check mà skill không giúp (skill chưa được đọc, đọc nhưng không làm theo, skill thiếu hoặc sai).
4. Chi phí: so sánh số token trung bình giữa các điều kiện. Điều kiện nào có hiệu quả tốt nhất theo điểm trên mỗi token? Đa tác tử có đáng chi phí trong thí nghiệm này không?
5. Có dấu hiệu rò rỉ dữ liệu hoặc quá khớp nào trong skill sinh ra không? Nhóm đã phòng tránh như thế nào?
6. Nhiễu: so sánh điểm tác vụ học của cùng bộ skill ở Phần 3.4 (đã sao lưu) và sau đóng băng. Chênh lệch bao nhiêu? Nó cho biết điều gì về độ tin cậy của các chênh lệch trong bảng ở mục 7?

## 9. Hạn chế và tính hợp lệ

> Nêu ít nhất 3 hạn chế và ảnh hưởng của từng hạn chế đến kết luận (ví dụ: chỉ 3 tác vụ mỗi vai trò, mỗi cấu hình chạy một lần, nhiễu của mô hình, tác vụ do giảng viên thiết kế sẵn quy ước, chỉ một mô hình).

1.
2.
3.

## 10. Kết luận

> Tối đa 5 câu. Chỉ khẳng định điều số liệu hỗ trợ. Nêu một đề xuất cải tiến tiếp theo.

## Phụ lục

- Lệnh đã chạy (theo thứ tự):
- Thử thách mở rộng (nếu có): hướng chọn, kết quả, nhận xét.
- Ghi chú khác:
