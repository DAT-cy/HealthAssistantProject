# Health Assistant - AI-Powered Virtual PT

Health Assistant là hệ thống huấn luyện thể chất ảo kết hợp Computer Vision, Machine Learning và LLM:

```text
AI Posture Analysis -> LLM Coach Planning -> Nutrition Planning -> Real-time Form Correction -> Adaptive Progress
```

## Mục tiêu đồ án

Trong 3 tháng, đồ án tập trung xây dựng một MVP có chiều sâu AI, có dữ liệu đo lường và có thể demo trọn vẹn:

- Phân tích tư thế bằng MediaPipe Pose và bộ luật biomechanics.
- Nhận diện phase, đếm rep và chấm điểm kỹ thuật bằng chuỗi landmark trong Java.
- Dùng LLM sinh roadmap, workout và thực đơn dưới dạng JSON có schema.
- Điều chỉnh độ khó bằng AI workflow dựa trên hiệu suất, RPE và chất lượng form.
- Phân tích tiến độ và cảnh báo bất thường bằng Gemini dựa trên lịch sử tập luyện.

## Các thành phần AI

### Computer Vision

- 33 pose landmarks từ MediaPipe Pose.
- Tính góc khớp 2D/3D, confidence và temporal smoothing.
- Phát hiện lệch vai, nghiêng chậu, đầu nhô trước, knee valgus và lưng cong.

### AI workflow với Gemini

- AI Form Coach: giải thích lỗi form từ các metrics do Java tính.
- Adaptive Planner: đề xuất tăng/giảm bài dựa trên completion, RPE, form score.
- Progress Analyst: tổng hợp và giải thích trend từ workout history.
- Agent tool calling: Gemini gọi các tool Java để lấy posture, nutrition và lịch sử.

### LLM Coach

- Nhận user profile, posture report, mục tiêu và lịch sử tập.
- Sinh roadmap 6-12 tuần, daily workout và thực đơn Việt Nam.
- Bắt buộc structured output, JSON schema validation và fallback khi LLM lỗi.
- Không gửi video thô đến LLM; chỉ gửi dữ liệu landmark/feature tối thiểu.

### NLP và Voice

- Speech-to-text cho lệnh bắt đầu/dừng/tạm nghỉ.
- TTS tiếng Việt cho cảnh báo: "Thẳng lưng", "Đẩy gối ra ngoài".

## Kiến trúc AI-first

```text
Camera -> MediaPipe Pose -> Feature Extraction -> ML Exercise/Form Models
                                      |
                                      v
                             Posture Assessment JSON
                                      |
                    User Profile + History + Goal
                                      |
                                      v
                               LLM Coach Engine
                                      |
                   Roadmap + Daily Workout + Nutrition
                                      |
                                      v
                       Adaptive Progress and Feedback
```

## Trạng thái mã nguồn

Đã có nền tảng backend:

| API | Method | Mô tả |
|---|---|---|
| `/api/v1/posture/assess` | POST | Phân tích lệch vai, nghiêng chậu và đầu nhô trước |
| `/api/v1/pose-engine/angle` | POST | Tính góc không gian giữa ba điểm |

Đang được phát triển theo roadmap: ML classifier, LLM planner, adaptive engine, progress prediction, voice feedback và persistence.

## Tech stack đề xuất

| Thành phần | Công nghệ |
|---|---|
| Backend | Spring Boot 3, Java 17 |
| Pose | MediaPipe Pose |
| AI/LLM | Gemini API, structured JSON output, tool calling |
| Database | PostgreSQL; TimescaleDB nếu cần time-series |
| API docs | OpenAPI/Swagger |
| Voice | Whisper/STT và TTS tiếng Việt |

## Chạy dự án

```bash
./gradlew bootRun
./gradlew test
```

Windows:

```powershell
.\gradlew.bat bootRun
```

Backend mặc định chạy tại `http://localhost:8080`; Swagger tại `http://localhost:8080/swagger-ui/index.html`.

## Cấu hình Gemini

Không commit API key vào Git. Cấu hình bằng biến môi trường:

```powershell
$env:GEMINI_API_KEY = "your-gemini-api-key"
.\gradlew.bat bootRun
```

API tạo roadmap:

```text
POST /api/v1/coach/roadmap
```

Nếu chưa có `GEMINI_API_KEY`, backend trả về một roadmap fallback an toàn để frontend vẫn có thể demo luồng nghiệp vụ. Model mặc định là `gemini-1.5-flash`, có thể đổi bằng `GEMINI_MODEL`.

## Cấu trúc tài liệu

- `Business.md`: nghiệp vụ, persona, rule và KPI.
- `AI_ROADMAP.md`: kế hoạch AI Gemini trong 12 tuần, mỗi tuần có tính năng, đầu ra và tiêu chí nghiệm thu.
- `IMPLEMENTATION_CHECKLIST.md`: checklist Java chi tiết cho từng tuần.
- `phases/`: mục tiêu, tiêu chí hoàn thành và demo của từng phase.

## An toàn

Đây là công cụ hỗ trợ tập luyện, không thay thế chẩn đoán hoặc điều trị y khoa. Khi visibility landmark thấp hơn `0.5`, hệ thống phải dừng kết luận tự động và yêu cầu người dùng điều chỉnh camera. Khi người dùng báo đau, hệ thống phải dừng hoặc giảm tải bài tập.
