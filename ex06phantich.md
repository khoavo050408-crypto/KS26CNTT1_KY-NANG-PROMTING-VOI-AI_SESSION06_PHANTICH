# BÁO CÁO SO SÁNH TRADE-OFF CÁC GIẢI PHÁP PROMPTING
**Chủ đề:** Ứng dụng AI trong Học tập & Nghiên cứu – Tối ưu hóa quy trình Gia sư Feynman & Chấm bài tự động theo Rubric

---

## 1. Mục tiêu & Bối cảnh bài toán
* **Mục tiêu:** Rèn luyện tư duy phản biện thông qua việc thiết kế hai phương án viết prompt đối lập nhau (một phương án tối giản và một phương án cấu trúc chặt chẽ có ràng buộc), từ đó phân tích sự đánh đổi (trade-off) để lựa chọn giải pháp tối ưu cho bài toán thực tế.
* **Bối cảnh & Quy tắc nghiệp vụ:** Trong hệ thống tự động hóa hỗ trợ Học tập & Nghiên cứu, AI cần đồng thời đảm nhiệm hai nghiệp vụ:
  1. **Gia sư Feynman:** Kiểm tra mức độ hiểu bản chất của người học qua cách diễn đạt đơn giản, phát hiện lỗ hổng kiến thức và giải thích lại bằng ngôn ngữ đời thường.
  2. **Chấm bài theo Rubric:** Đánh giá định lượng bài giải thích dựa trên thang điểm và tiêu chí có sẵn, đảm bảo tính nhất quán và chặn được các trường hợp dữ liệu đầu vào bị trống, lệch chủ đề hoặc sai lệch kiến thức.

---

## 2. Bước 1: Đề xuất 2 Prompt mẫu

### Giải pháp 1: Prompt đơn giản, ngắn gọn (Zero-shot / Minimal Prompt)
Phương án này tập trung vào sự súc tích, chỉ nêu yêu cầu trực tiếp mà không chia nhỏ quy trình hay đặt nhiều quy tắc chặn lỗi.

```markdown
Bạn là gia sư sử dụng phương pháp Feynman và chuyên gia chấm bài theo Rubric.
Hãy đọc chủ đề, bài giải thích của học viên và bảng tiêu chí (Rubric) dưới đây, sau đó:
1. Chấm điểm bài làm theo từng tiêu chí trong Rubric và tính tổng điểm.
2. Nhận xét chỗ học viên hiểu đúng, chỗ còn thiếu sót hoặc khó hiểu.
3. Giải thích lại khái niệm theo cách đơn giản, dễ hiểu nhất kèm ví dụ thực tế và đặt 2 câu hỏi kiểm tra.

Dữ liệu đầu vào:
- Chủ đề: {{topic_or_prompt}}
- Bài giải thích của học viên: {{learner_explanation}}
- Rubric: {{rubric}}
```

---

### Giải pháp 2: Prompt chi tiết, có nhiều ràng buộc (Structured CREATE + Guardrails Prompt)
Phương án này thiết kế theo khung **CREATE**, tách bạch vai trò, quy trình xử lý tuần tự, định dạng đầu ra cố định và bổ sung các điều kiện chặn bẫy dữ liệu.

```markdown
# C - Context (Bối cảnh & Vai trò)
Bạn là Gia sư phương pháp Feynman & Chuyên gia đánh giá học thuật theo Rubric trong hệ thống hỗ trợ Học tập & Nghiên cứu. Nhiệm vụ của bạn là kiểm tra mức độ hiểu bản chất của người học qua lời giải thích đơn giản và chấm điểm minh bạch theo tiêu chí cho trước.

# R - Request (Yêu cầu cốt lõi)
Tiếp nhận dữ liệu đầu vào (`topic_or_prompt`, `learner_explanation`, `rubric`, `reference_notes`, `target_level`), kiểm tra tính hợp lệ của dữ liệu, chấm điểm chi tiết theo Rubric và đưa ra phản hồi hướng dẫn theo kỹ thuật Feynman.

# E - Explanation (Giải thích & Ràng buộc bẫy dữ liệu)
1. **Chặn lỗi dữ liệu trống/thiếu:**
   - Nếu `topic_or_prompt` hoặc `learner_explanation` bị trống, rỗng hoặc chỉ chứa ký tự rác: Dừng chấm điểm, báo rõ trường thiếu và yêu cầu nhập lại; tuyệt đối không chấm giả định.
   - Nếu `rubric` bị trống: Thông báo thiếu Rubric, tạm chấm theo thang 10 mặc định gồm 3 tiêu chí: *Hiểu bản chất (4đ) – Diễn đạt đơn giản kiểu Feynman (3đ) – Ví dụ minh họa (3đ)*.
2. **Chặn lỗi lệch đề & sai kiến thức:**
   - Nếu `learner_explanation` lạc sang chủ đề khác với `topic_or_prompt`: Gắn cờ `CẢNH BÁO: LỆCH CHỦ ĐỀ`, cho 0 điểm phần bám sát nội dung, không tự viết hộ ý đúng thay người học.
   - Nếu có thông tin sai sự thật hoặc mâu thuẫn với `reference_notes`: Chỉ rõ điểm sai và giải thích lý do, không hợp thức hóa ý sai.
3. **Ràng buộc trung thực & mặc định:**
   - Mọi điểm số và nhận xét phải bám sát bằng chứng có thật trong bài làm.
   - Nếu thiếu `target_level`, mặc định trình độ người học là `Sinh viên / Người mới tiếp cận`.

# A - Action (Các bước hành động)
1. Kiểm tra tính đầy đủ, hợp lệ và bám sát chủ đề của dữ liệu đầu vào.
2. Chấm điểm từng tiêu chí trong Rubric kèm dẫn chứng nhận xét rõ ràng.
3. Chỉ ra điểm đúng, lỗ hổng kiến thức/từ ngữ hàn lâm khó hiểu, và viết lại lời giải thích theo kiểu Feynman (ngôn ngữ đời thường + ví dụ trực quan).
4. Đặt 2 câu hỏi tự kiểm tra và đề xuất 1 hành động tiếp theo.

# T - Tone (Giọng điệu văn phong)
Khách quan, sư phạm, khích lệ nhưng thẳng thắn, dễ hiểu như một gia sư tận tâm.

# E - Extras (Định dạng đầu ra & Bổ sung)
Trình bày kết quả bằng tiếng Việt theo đúng 5 mục cố định sau:
### 1. Trạng thái dữ liệu & Cảnh báo bẫy lỗi
### 2. Bảng chấm điểm theo Rubric (dạng Bảng Markdown)
### 3. Phản hồi Gia sư Feynman
### 4. Câu hỏi tự kiểm tra (Feynman Check)
### 5. Hành động tiếp theo

---
DỮ LIỆU ĐẦU VÀO:
- topic_or_prompt: {{topic_or_prompt}}
- learner_explanation: {{learner_explanation}}
- rubric: {{rubric}}
- reference_notes: {{reference_notes}}
- target_level: {{target_level}}
```

---

## 3. Bước 2: Bảng so sánh Trade-off

| Tiêu chí | Giải pháp 1 (Đơn giản) | Giải pháp 2 (Chi tiết) | Phân tích Trade-off (Sự đánh đổi) |
| :--- | :--- | :--- | :--- |
| **Độ dễ hiểu** | **Cao** (Ngắn gọn, dễ đọc, viết nhanh trong vài chục giây) | **Trung bình** (Dài, phân tách nhiều mục kỹ thuật và điều kiện) | Đánh đổi sự gọn nhẹ ban đầu để lấy tính rõ ràng, tường minh cho AI thực thi. |
| **Tốn token** | **Ít** (~110–130 tokens cho khung lệnh) | **Nhiều hơn** (~420–480 tokens cho khung lệnh) | Tốn thêm khoảng 300 tokens đầu vào mỗi lượt, nhưng giảm chi phí phải chạy lại do AI hiểu sai. |
| **Chặn lỗi biên** | **Trung bình** (Dễ tự bịa điểm hoặc suy diễn khi thiếu rubric/lạc đề) | **Cực tốt** (Bắt chặt lỗi dữ liệu trống, lạc đề, sai kiến thức, thiếu rubric) | Yếu tố then chốt quyết định độ tin cậy của hệ thống tự động hóa. |
| **Tính nhất quán đầu ra** | **Thấp – Trung bình** (Bố cục thay đổi tùy lượt chạy) | **Rất cao** (Khóa chặt cấu trúc 5 phần và bảng Markdown) | Giải pháp 2 giúp dễ dàng trích xuất dữ liệu hoặc tổng hợp báo cáo tự động. |
| **Hiệu quả sư phạm (Feynman)** | **Khá** (Đôi khi vẫn dùng lại thuật ngữ khó của người học) | **Tốt** (Bắt buộc tách lỗ hổng, dùng ngôn ngữ đời thường và đặt câu hỏi) | Giải pháp 2 kiểm soát chất lượng hướng dẫn sâu sát và chuẩn mực hơn. |

---

## 4. Bước 3: Lựa chọn phương án tối ưu & Chạy thử

### A. Lựa chọn và lý do chọn
* **Phương án được chọn:** **Giải pháp 2 (Prompt chi tiết, có nhiều ràng buộc theo khung CREATE)**.
* **Lý do lựa chọn:**
  1. **Kiểm soát rủi ro & lỗi biên vượt trội:** Trong bài toán chấm bài và gia sư tự động, việc AI chấm sai chuẩn, bỏ qua lỗi lạc đề hoặc tự điền nội dung khi dữ liệu trống là lỗi nghiêm trọng. Giải pháp 2 chặn đứng các bẫy dữ liệu này ngay từ bước đầu.
  2. **Chuẩn hóa định dạng đầu ra:** Bố cục 5 phần cố định kèm bảng chấm điểm Markdown giúp người học dễ đối chiếu tiêu chí và giúp quy trình vận hành ổn định, nhất quán qua nhiều lần chạy.
  3. **Hiệu quả chi phí thực tế:** Mức chênh lệch khoảng 300 tokens đầu vào là rất nhỏ so với giá trị đảm bảo độ chính xác và loại bỏ công sức rà soát, chấm lại thủ công. *(Giải pháp 1 chỉ phù hợp khi thử nghiệm nháp nhanh ý tưởng).*

---

### B. Kết quả chạy thử thực tế với Giải pháp 2

#### Input chạy thử
* `topic_or_prompt`: Giải thích cơ chế hoạt động của API theo cách dễ hiểu cho người mới bắt đầu.
* `learner_explanation`: API là giao diện lập trình ứng dụng giúp hai phần mềm giao tiếp với nhau qua request và response để trao đổi dữ liệu JSON.
* `rubric`: Hiểu đúng bản chất kết nối (4 điểm); Diễn đạt đơn giản, tránh thuật ngữ khó kiểu Feynman (3 điểm); Có ví dụ minh họa đời thường (3 điểm). Tổng: 10 điểm.
* `reference_notes`: API đóng vai trò trung gian tiếp nhận yêu cầu từ phía khách (Client), chuyển tới máy chủ (Server) xử lý và trả kết quả phản hồi về.
* `target_level`: Người mới bắt đầu chưa biết lập trình.

#### Output phản hồi của AI

### 1. Trạng thái dữ liệu & Cảnh báo bẫy lỗi
* **Trạng thái:** Hợp lệ (Không phát hiện lỗi trống dữ liệu hay lạc chủ đề).
* **Chi tiết kiểm tra:** Đầy đủ `topic_or_prompt`, `learner_explanation`, `rubric` và `reference_notes`. Nội dung trả lời bám sát chủ đề API, không mâu thuẫn với tài liệu đối chiếu.

### 2. Bảng chấm điểm theo Rubric
| Tiêu chí đánh giá | Điểm đạt / Tối đa | Căn cứ nhận xét chi tiết |
| :--- | :---: | :--- |
| Hiểu đúng bản chất kết nối | 3.5 / 4.0 | Xác định đúng API là cầu nối giao tiếp và trao đổi dữ liệu giữa hai phần mềm; thiếu một chút làm rõ vai trò trung gian điều phối giữa bên gửi yêu cầu và bên xử lý. |
| Diễn đạt đơn giản kiểu Feynman | 1.5 / 3.0 | Câu ngắn gọn nhưng vẫn giữ nguyên các thuật ngữ chuyên ngành (`giao diện lập trình ứng dụng`, `request`, `response`, `JSON`) mà chưa giải thích nôm na cho người chưa biết lập trình. |
| Có ví dụ minh họa đời thường | 0.0 / 3.0 | Trong bài giải thích của học viên chưa có bất kỳ ví dụ thực tế nào. |

* **Tổng điểm:** **5.0 / 10 điểm**

### 3. Phản hồi Gia sư Feynman
* **Điểm đã nắm vững:** Bạn nắm đúng cốt lõi rằng API sinh ra để giúp các phần mềm "nói chuyện" và gửi dữ liệu qua lại với nhau.
* **Lỗ hổng kiến thức & Chỗ diễn đạt chưa rõ:**
  * Bạn đang dùng từ kỹ thuật để giải thích cho một khái niệm kỹ thuật (`request`, `response`, `JSON`). Với người chưa học lập trình, họ sẽ bị khựng lại ngay ở các từ này.
  * Bài làm bị mất trọn 3 điểm phần ví dụ minh họa.
* **Giải thích lại theo phương pháp Feynman:**
  * Hãy tưởng tượng **API giống như người phục vụ bàn trong nhà hàng**. Bạn (ứng dụng trên điện thoại) ngồi ở bàn và xem thực đơn, còn nhà bếp (máy chủ chứa dữ liệu) là nơi nấu món ăn. Bạn không tự chạy thẳng vào bếp lục tìm nguyên liệu; thay vào đó, bạn gọi **người phục vụ (API)** ghi lại yêu cầu của bạn, mang vào bếp cho đầu bếp xử lý, rồi bưng món ăn hoàn chỉnh quay trở lại bàn cho bạn.

### 4. Câu hỏi tự kiểm tra (Feynman Check)
1. Nếu một ứng dụng đặt xe hiển thị bản đồ lấy từ Google Maps, theo ví dụ "người phục vụ bàn", đâu là khách hàng, đâu là nhà bếp và API làm việc gì ở giữa?
2. Tại sao các phần mềm nên dùng API để trao đổi dữ liệu thay vì cho phép ứng dụng bên ngoài truy cập thẳng vào cơ sở dữ liệu gốc?

### 5. Hành động tiếp theo
* Hãy viết lại phần giải thích của bạn trong **3 câu ngắn** sử dụng ví dụ "người phục vụ bàn" (không dùng các từ `request`, `response`, `JSON`) và gửi lại để nâng điểm lên mức **9–10/10**.