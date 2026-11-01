# BLOXORZ SOLVER

Đồ án môn học Nhập môn Trí tuệ Nhân tạo (CSC14003) — Trường Đại học Khoa học Tự nhiên, ĐHQG-HCM.

Ứng dụng mô phỏng trò chơi Bloxorz (15 màn chơi tham chiếu) với giao diện web 3D Three.js chạy offline, tích hợp các thuật toán tìm kiếm không gian trạng thái: BFS, DFS, UCS và A*.

---

## 1. Cài đặt và Khởi chạy

### Yêu cầu hệ thống
- Python 3.10 trở lên.
- Trình duyệt web hỗ trợ WebGL (Chrome, Edge, Firefox, Brave...).

### Khởi chạy

Mở terminal tại thư mục `Source/`:

```sh
# Khởi tạo và kích hoạt môi trường ảo
python -m venv .venv

# Linux / macOS:
source .venv/bin/activate
# Windows PowerShell:
.venv\Scripts\Activate.ps1
# Windows Command Prompt:
.venv\Scripts\activate.bat

# Cài đặt thư viện phụ thuộc
pip install -r requirements.txt

# Chạy ứng dụng
python main.py
```

Trình duyệt sẽ tự động mở tại địa chỉ `http://127.0.0.1:8000`. Game và các thư viện JavaScript chạy hoàn toàn offline sau khi cài xong package Python, không cần Node.js hay bundler.

Tùy chọn dòng lệnh:
```sh
# Đổi cổng nếu cổng 8000 bị bận
python main.py --port 8080

```

---

## 2. Điều khiển trong game

| Phím / Thao tác | Tác dụng |
|---|---|
| ↑ ↓ ← → hoặc W A S D | Lăn khối hộp / di chuyển khối đang chọn khi split |
| Space | Đổi khối điều khiển khi split (tính 1 bước đi) |
| U hoặc Backspace | Lùi lại một bước đi (Undo) |
| R | Chơi lại màn hiện tại (Restart) |
| Esc | Mở danh sách chọn màn (Level Select) |
| ? hoặc nút Help | Xem bảng chú giải các loại ô cờ và cơ chế |
| Kéo chuột trái | Xoay góc nhìn camera 3D quanh sa bàn |
| Cuộn chuột | Phóng to / thu nhỏ góc nhìn |


---

## 3. Thuật toán tìm kiếm

Hệ thống đã cài đặt đầy đủ các thuật toán tìm kiếm trong thư mục `solvers/`:

- **BFS (Breadth-First Search)** (`solvers/bfs.py`): Tìm đường đi có số bước ít nhất, tối ưu với mô hình chi phí unit.
- **DFS (Depth-First Search)** (`solvers/dfs.py`): Tìm kiếm theo chiều sâu.
- **UCS (Uniform Cost Search)** (`solvers/ucs.py`): Mở rộng nút theo chi phí thực tế g(n), đảm bảo tối ưu chi phí với mọi mô hình trọng số.
- **A\* Search** (`solvers/astar.py`): Tìm kiếm theo hàm đánh giá f(n) = g(n) + h(n), kết hợp thông tin chi phí và hàm đánh giá heuristic.

### Hàm Heuristic cho A* (`solvers/heuristics.py`)

- **center (`h_center`)**: Khoảng cách Manhattan từ tâm hình học của khối hộp đến ô đích chia cho 1.5.
- **relaxed (`h_relaxed`)**: Khoảng cách Manhattan nhỏ nhất từ một trong các khối con đến ô đích chia cho 2.0.

Cả hai hàm đều thỏa điều kiện chấp nhận được (admissible), đảm bảo tính tối ưu của nghiệm tìm được bởi A*.

### Mô hình chi phí (Cost models)

- **unit**: Chi phí mỗi bước di chuyển là 1.0.
- **risk**: Chi phí bước đi tăng lên 2.0 khi khối hộp ở trạng thái nằm trên ô kính mỏng (fragile), phản ánh rủi ro vị trí.

---

## 4. Chạy Benchmark và Đánh giá hiệu năng

Hệ thống cung cấp công cụ đo đạc độc lập bằng subprocess và `tracemalloc` để đo thời gian thực thi và lượng bộ nhớ tiêu thụ đỉnh:

```sh
# Chạy benchmark cả 4 thuật toán trên 15 màn, lặp lại 3 lần lấy trung vị:
python bench/run_bench.py --algos bfs dfs ucs astar --repeat 3 --timeout 30 --cost-model risk --out bench/results

# Chạy so sánh UCS và A* trên các màn chỉ định:
python bench/run_bench.py --algos ucs astar:center astar:relaxed --levels 1-5 10-15 --repeat 3 --out bench/results

# Vẽ biểu đồ kết quả:
python bench/plots.py bench/results/results.csv --out bench/results
```

Kết quả xuất ra tại `bench/results/`:
- `results.csv`: Bảng dữ liệu số liệu đo đạc chi tiết.
- `summary.md`: Tóm tắt kết quả theo định dạng bảng Markdown.
- `environment.json`: Thông tin phần cứng và môi trường chạy.
- Các biểu đồ PNG (240 DPI): thời gian tìm kiếm (`search_time_s.png`), bộ nhớ RAM đỉnh (`peak_memory_kb.png`), số node mở rộng (`expanded_nodes.png`) và độ dài đường đi (`solution_length.png`).

---

## 5. Cấu trúc thư mục

```text
Source/
├── main.py              # Điểm khởi chạy chính của ứng dụng
├── requirements.txt     # Danh sách thư viện phụ thuộc
├── pyproject.toml       # Cấu hình công cụ kiểm tra mã nguồn
├── README.md            # Tài liệu hướng dẫn
├── core/                # Bộ luật game và mô hình hóa trạng thái
│   ├── constants.py     # Định nghĩa Action, Orientation, Status
│   ├── costs.py         # Hàm tính chi phí bước đi (unit, risk)
│   ├── level.py         # Đọc và quản lý dữ liệu màn chơi
│   ├── rules.py         # Logic di chuyển, va chạm, công tắc, điều kiện thắng/thua
│   ├── state.py         # Cấu trúc trạng thái State
│   └── tiles.py         # Định nghĩa các loại ô cờ và công tắc
├── solvers/             # Các thuật toán tìm kiếm
│   ├── base.py          # Lớp Problem và SearchResult
│   ├── bfs.py           # Thuật toán BFS
│   ├── dfs.py           # Thuật toán DFS
│   ├── ucs.py           # Thuật toán UCS
│   ├── astar.py         # Thuật toán A*
│   ├── heuristics.py    # Các hàm heuristic (center, relaxed)
│   └── runner.py        # Module thực thi thuật toán trong subprocess riêng
├── api/                 # Backend FastAPI
│   ├── server.py        # Xử lý session game và API giải tự động
│   └── schemas.py       # Pydantic schemas cho API
├── levels/              # Dữ liệu JSON 15 màn chơi chuẩn Bloxorz
├── bench/               # Công cụ benchmark và vẽ biểu đồ
│   ├── run_bench.py     # Script chạy benchmark tự động
│   ├── plots.py         # Script vẽ biểu đồ từ file CSV
│   └── results/         # Thư mục lưu kết quả đo đạc
└── web/                 # Giao diện web Three.js
    ├── index.html       # Giao diện chính
    ├── main.js          # Khởi tạo giao diện
    ├── GameController.js# Quản lý sự kiện và vòng đời game
    ├── css/             # Bảng phong cách giao diện
    ├── scene/           # Module Three.js (dựng hình 3D, animation, camera)
    └── ui/              # Bảng điều khiển và thanh trạng thái
```

---

## 6. Thông tin đồ án

- Môn học: Nhập môn Trí tuệ Nhân tạo (CSC14003)
- Khoa Công nghệ Thông tin — Trường Đại học Khoa học Tự nhiên, ĐHQG-HCM
- Sinh viên thực hiện:
  - Họ và tên: Phạm Trường Quốc Huy
  - MSSV: 25122071

  - Họ và tên: Trương Tiến Dương
  - MSSV: 25122064

  - Họ và tên: Huỳnh Hoàng Nguyên
  - MSSV: 25122081


  - Họ và tên: Nguyễn Long Thiên
  - MSSV: 25122089

  - Họ và tên: Lê Anh Tuấn
  - MSSV: 25122092



