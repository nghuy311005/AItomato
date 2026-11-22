# Cấu Trúc Dự Án: TomatoCare AI (Smart Agri-Assistant)

Tài liệu này quy định cấu trúc thư mục chuẩn cho toàn bộ hệ thống gồm 3 phân hệ chính (**Mobile App**, **Backend Server**, **AI Core**), được tối ưu hóa để 3 thành viên làm việc song song trong 6 ngày mà không gây xung đột mã nguồn (Git Conflict).

---

## 1. Cấu trúc tổng thể (Monorepo)

```text
tomato-care-ai/
├── mobile-app/          # Thành viên 3: Ứng dụng di động (React Native / Expo)
├── backend-api/         # Thành viên 2: Server xử lý API, Thời tiết & Cloud Inference (FastAPI)
├── ai-core/             # Thành viên 1: Huấn luyện model, đánh giá & System Prompts
├── docs/                # Tài liệu chung: API Contract, Database Schema, Dữ liệu bệnh học
└── README.md            # Hướng dẫn cài đặt và chạy nhanh toàn bộ dự án
```

---

## 2. Chi tiết Mobile App (`mobile-app/`)
*Phụ trách: Thành viên 3 | Công nghệ: React Native (Expo / CLI), TypeScript, Firebase SDK, React Navigation.*

```text
mobile-app/
├── assets/
│   ├── images/                  # Logo, banner, hình minh họa các loại bệnh
│   └── animations/              # File Lottie JSON (hiệu ứng quét AI, loading)
├── src/
│   ├── api/                     # Cấu hình gọi API (Axios instance, interceptors)
│   │   ├── client.ts            # Base URL (Ngrok / Cloud Server) & Timeout
│   │   ├── diagnosisApi.ts      # Gửi ảnh lên FastAPI nhận kết quả chẩn đoán
│   │   ├── weatherApi.ts        # Lấy dữ liệu cảnh báo thời tiết
│   │   └── chatbotApi.ts        # Gọi Gemini API với System Prompt nông nghiệp
│   ├── config/
│   │   ├── firebase.ts          # Khởi tạo Firebase (Auth, Firestore, Storage)
│   │   └── constants.ts         # Màu sắc chủ đạo, ngưỡng cảnh báo, API Keys
│   ├── data/
│   │   └── diseaseLibrary.json  # Dữ liệu tĩnh: Tên bệnh, triệu chứng, phác đồ thuốc (Offline)
│   ├── navigation/
│   │   ├── AppNavigator.tsx     # Luồng điều hướng chính (Auth vs Main)
│   │   └── BottomTabNavigator.tsx # 5 Tab: Trang chủ, Quét AI, Vườn, Bản đồ, Trợ lý
│   ├── screens/
│   │   ├── auth/
│   │   │   ├── LoginScreen.tsx
│   │   │   └── RegisterScreen.tsx
│   │   ├── home/
│   │   │   ├── HomeScreen.tsx           # Dashboard + Widget cảnh báo thời tiết
│   │   │   └── DiseaseDetailScreen.tsx  # Chi tiết phác đồ điều trị bệnh
│   │   ├── scan/
│   │   │   ├── CameraScanScreen.tsx     # Chụp ảnh / Chọn từ thư viện
│   │   │   └── ScanResultScreen.tsx     # Kết quả AI, % mức độ bệnh, nút lưu vào vườn
│   │   ├── garden/
│   │   │   ├── GardenListScreen.tsx     # Danh sách các luống cây (Plots)
│   │   │   └── PlotDetailScreen.tsx     # Lịch sử quét & biểu đồ tiến triển của luống
│   │   ├── map/
│   │   │   └── DiseaseMapScreen.tsx     # Bản đồ nhiệt dịch tễ (react-native-maps)
│   │   └── assistant/
│   │       └── ChatbotScreen.tsx        # Hỏi đáp chuyên gia AI
│   ├── components/              # Các UI Component tái sử dụng
│   │   ├── WeatherAlertCard.tsx
│   │   ├── DiagnosisCard.tsx
│   │   ├── SeverityBadge.tsx    # Nhãn hiển thị mức độ: Nhẹ / Trung bình / Nặng
│   │   └── LoadingOverlay.tsx
│   ├── context/
│   │   └── AuthContext.tsx      # Quản lý trạng thái đăng nhập toàn cục
│   ├── types/
│   │   └── index.ts             # Định nghĩa TypeScript Interfaces (Diagnosis, Plot, Report...)
│   └── utils/
│       ├── imageCompressor.ts   # Nén ảnh trước khi gửi API (tăng tốc độ phản hồi < 1s)
│       └── formatters.ts        # Định dạng ngày giờ, tỷ lệ phần trăm
├── App.tsx                      # Entry point của ứng dụng
├── package.json
└── tsconfig.json
```

---

## 3. Chi tiết Backend Server (`backend-api/`)
*Phụ trách: Thành viên 2 | Công nghệ: Python 3.10+, FastAPI, Uvicorn, Ultralytics/PyTorch.*

```text
backend-api/
├── app/
│   ├── main.py                  # Khởi tạo FastAPI app, cấu hình CORS
│   ├── core/
│   │   └── config.py            # Đọc biến môi trường (.env): OpenWeather Key, Model Path
│   ├── api/
│   │   └── routes/
│   │       ├── diagnosis.py     # POST /api/v1/diagnose (Nhận ảnh -> Trả về JSON bệnh)
│   │       ├── weather.py       # GET /api/v1/weather-alert (Phân tích độ ẩm -> Cảnh báo nấm)
│   │       └── heatmap.py       # GET /api/v1/heatmap (Lấy danh sách tọa độ dịch bệnh)
│   ├── services/
│   │   ├── ai_inference.py      # Load model (best.pt), chạy dự đoán & tính % diện tích bệnh
│   │   └── weather_service.py   # Gọi OpenWeatherMap API & logic suy luận nguy cơ bệnh
│   ├── schemas/
│   │   └── response_models.py   # Pydantic models chuẩn hóa dữ liệu JSON đầu ra
│   └── utils/
│       └── image_helper.py      # Chuyển đổi bytes ảnh sang OpenCV/PIL format
├── models/
│   └── best.pt                  # File trọng số model AI do Thành viên 1 bàn giao
├── scripts/
│   └── seed_heatmap_data.py     # Script tạo 30 điểm dịch bệnh giả lập trên Firebase để demo
├── .env                         # Lưu API Keys (Không push lên Git)
├── requirements.txt             # Thư viện: fastapi, uvicorn, ultralytics, pillow, requests...
└── Dockerfile                   # (Tùy chọn) Đóng gói nếu deploy lên Render/Railway
```

---

## 4. Chi tiết AI Core (`ai-core/`)
*Phụ trách: Thành viên 1 | Công nghệ: Google Colab, PyTorch / YOLOv8 / TensorFlow.*

```text
ai-core/
├── notebooks/
│   ├── 01_data_exploration.ipynb    # Kiểm tra phân bổ lớp, lọc ảnh lỗi
│   ├── 02_train_tomato_model.ipynb  # Notebook huấn luyện trên Google Colab GPU
│   └── 03_model_evaluation.ipynb    # Xuất Confusion Matrix, biểu đồ Loss/mAP cho báo cáo
├── src/
│   ├── severity_calculator.py       # Thuật toán tính tỷ lệ diện tích vùng bệnh (Nhẹ/TB/Nặng)
│   └── test_inference.py            # Script chạy thử nghiệm model trên ảnh thực tế
├── prompts/
│   └── agronomist_system_prompt.txt # Kịch bản chuẩn (System Prompt) cho Chatbot tư vấn
└── README.md                        # Ghi chú thông số train (Epochs, Batch size, Accuracy)
```

---

## 5. Tài liệu đồng bộ chung (`docs/`)
*Cả 3 thành viên cùng thống nhất ngay trong Ngày 1.*

```text
docs/
├── API_CONTRACT.md              # Quy định chính xác cấu trúc Request/Response của từng API
├── FIRESTORE_SCHEMA.md          # Cấu trúc các Collection trên Firebase (users, plots, history, reports)
└── DEMO_SCRIPT.md               # Kịch bản thuyết trình và thứ tự bấm app ngày báo cáo
```

---

## 6. Quy tắc làm việc nhóm (Git Workflow trong 6 ngày)

1. **Nhánh chính (`main`):** Chỉ chứa code chạy ổn định.
2. **Nhánh phát triển theo thành viên:**
   * `feat/ai-model` (Thành viên 1)
   * `feat/backend-api` (Thành viên 2)
   * `feat/mobile-ui` (Thành viên 3)
3. **Quy tắc bàn giao (Handoff):**
   * **Ngày 2:** Thành viên 1 xuất file `best.pt` và gửi cho Thành viên 2 đặt vào thư mục `backend-api/models/`.
   * **Ngày 4:** Thành viên 1 gửi nội dung file `agronomist_system_prompt.txt` cho Thành viên 3 nhúng vào `chatbotApi.ts`.
   * **Ngày 5:** Cả nhóm merge code vào nhánh `main` và chạy kiểm thử tích hợp toàn hệ thống.
