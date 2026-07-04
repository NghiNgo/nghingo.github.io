# Ứng dụng Web ngữ nghĩa tìm kiếm văn bản phục vụ quản lý đầu tư chuyển đổi số

**Công nghệ cốt lõi**: RDF · RDFS · OWL · SPARQL · SKOS · SWRL 

---

## Mục lục

1. [Giới thiệu đề tài](#1-giới-thiệu-đề-tài)
2. [Bối cảnh và vấn đề thực tiễn](#2-bối-cảnh-và-vấn-đề-thực-tiễn)
3. [Giải pháp và nền tảng công nghệ](#3-giải-pháp-và-nền-tảng-công-nghệ)
4. [Kiến trúc hệ thống](#4-kiến-trúc-hệ-thống)
5. [Cấu trúc ontology](#5-cấu-trúc-ontology)
6. [Ba phương pháp tìm kiếm và thuật toán xếp hạng](#6-ba-phương-pháp-tìm-kiếm-và-thuật-toán-xếp-hạng)
7. [Kết quả thực nghiệm](#7-kết-quả-thực-nghiệm)
8. [Hướng dẫn cài đặt và khởi chạy từng bước](#8-hướng-dẫn-cài-đặt-và-khởi-chạy-từng-bước)
9. [Hướng dẫn sử dụng ứng dụng](#9-hướng-dẫn-sử-dụng-ứng-dụng)
10. [Hạn chế và hướng phát triển](#10-hạn-chế-và-hướng-phát-triển)
11. [Phụ lục: cấu trúc thư mục và xử lý sự cố](#11-phụ-lục-cấu-trúc-thư-mục-và-xử-lý-sự-cố)

---

## 1. Giới thiệu đề tài

Đề tài nghiên cứu ứng dụng **Web ngữ nghĩa (Semantic Web)** để xây dựng một hệ
thống tìm kiếm văn bản pháp lý phục vụ công tác quản lý đầu tư chuyển đổi số.
Khác với công cụ tìm kiếm truyền thống chỉ khớp từ khóa theo chuỗi ký tự, hệ
thống được xây dựng có khả năng **hiểu ngữ nghĩa** của câu hỏi, **mở rộng từ
đồng nghĩa**, và **suy luận quan hệ pháp lý** (văn bản nào thay thế, sửa đổi,
hướng dẫn văn bản nào) để trả về kết quả chính xác và có giải thích rõ ràng.

Sản phẩm bàn giao gồm: một ontology chuyên ngành, một bộ quy trình chuyển đổi
dữ liệu tự động, một bộ luật suy diễn, một phương pháp đánh giá định lượng, và
một ứng dụng Web hoàn chỉnh — toàn bộ được đóng gói thành **bộ công cụ Python 4
bước** trình bày trong tài liệu này.

---

## 2. Bối cảnh và vấn đề thực tiễn

Hoạt động đầu tư chuyển đổi số chịu sự điều chỉnh **đan xen của nhiều ngành
luật**: đầu tư công, ngân sách nhà nước, đấu thầu, quản lý tài sản công, công
nghệ thông tin, an toàn thông tin... Các văn bản liên quan phân tán trên nhiều
cổng thông tin, thường xuyên được **sửa đổi, thay thế**, khiến chủ đầu tư khó
tra cứu đầy đủ và chính xác căn cứ pháp lý khi lập hồ sơ dự án.

Thực tiễn thẩm định hồ sơ dự án/nhiệm vụ chuyển đổi số tại tỉnh Khánh Hòa năm
2023 cho thấy **100% hồ sơ phải chỉnh sửa**, trong đó lỗi phổ biến nhất là
**sai sót về căn cứ pháp lý**. Trong khi đó, các công cụ tìm kiếm hiện có chủ
yếu khớp từ khóa thông thường, không "hiểu" được quan hệ ngữ nghĩa và quan hệ
hiệu lực giữa các văn bản — ví dụ, tìm "Nghị định 102" sẽ không tự động biết
văn bản này đã bị thay thế bởi văn bản nào.

Đây chính là khoảng trống mà đề tài hướng tới giải quyết.

---

## 3. Giải pháp và nền tảng công nghệ

Đề tài áp dụng bộ công nghệ chuẩn của W3C cho Web ngữ nghĩa:

| Công nghệ | Vai trò trong đề tài |
|---|---|
| **RDF** (Resource Description Framework) | Mô hình hóa văn bản pháp lý dưới dạng bộ ba Chủ thể–Vị từ–Đối tượng |
| **RDFS** (RDF Schema) | Định nghĩa lớp, thuộc tính, quan hệ kế thừa |
| **OWL** (Web Ontology Language) | Xây dựng ontology chi tiết: lớp văn bản, cơ quan, chủ đề, tình trạng hiệu lực |
| **SPARQL** | Ngôn ngữ truy vấn dữ liệu ngữ nghĩa (tương tự SQL cho dữ liệu đồ thị) |
| **SKOS** (Simple Knowledge Organization System) | Quản lý hệ thống chủ đề và từ đồng nghĩa tiếng Việt (vd. "CNTT" = "công nghệ thông tin") |
| **SWRL** (Semantic Web Rule Language) | Luật suy diễn tự động tình trạng hiệu lực văn bản |

Ontology của đề tài dùng namespace riêng:
`http://ntu.edu.vn/ontology/digital-investment#`

---

## 4. Kiến trúc hệ thống

```
   Văn bản gốc (PDF/Word, vanban.chinhphu.vn)
                  │
                  ▼
   Bảng Excel trung gian (danh_sach_van_ban_mau.xlsx)
                  │  BƯỚC 1 — buoc1_excel_to_rdf.py
                  ▼
   Ontology RDF/OWL (digital_investment_v2.owl)
                  │  BƯỚC 2 — buoc2_kiem_tra_suy_luan.py (bộ suy luận Pellet)
                  ▼
   Ontology đã kiểm chứng nhất quán + tình trạng hiệu lực suy luận
                  │  BƯỚC 3 — buoc3_danh_gia.py (đánh giá PP1/PP2/PP3)
                  ▼
   Số liệu đánh giá (Precision@5, Recall@5, MRR, nDCG@10)
                  │  BƯỚC 4 — app.py (Flask) + templates/index.html
                  ▼
   Ứng dụng Web tìm kiếm (http://127.0.0.1:5000)
```

Ứng dụng Web (Bước 4) hỗ trợ **hai chế độ lưu trữ**, chuyển đổi chỉ bằng biến
môi trường mà không cần sửa code:

- **LOCAL** (mặc định): đọc trực tiếp file `.owl` bằng thư viện `rdflib`, phù
  hợp demo, không cần cài đặt gì thêm.
- **GRAPHDB**: truy vấn qua SPARQL endpoint của GraphDB (kho dữ liệu đồ thị
  chuyên dụng), phù hợp triển khai chính thức với dữ liệu lớn.

---

## 5. Cấu trúc ontology

Ontology hiện tại (phiên bản 2) có quy mô:

| Thành phần | Số lượng | Ghi chú |
|---|---|---|
| Tổng số triples | 1.479 | |
| Lớp (Classes) | 38 | 8 lớp loại văn bản (Luật, Nghị định, Thông tư, Quyết định, Kế hoạch, Nghị quyết, Văn bản hợp nhất, Văn bản điều hành), 4 lớp tình trạng hiệu lực, cùng các lớp dự án, hồ sơ, nội dung pháp lý |
| Object properties | 20 | Đầy đủ domain/range, có 7 cặp quan hệ nghịch đảo (`thayThe`/`biThayTheBoi`, `suaDoi`/`duocSuaDoiBoi`...) |
| Datatype properties | 24 | `soVanBan`, `ngayBanHanh`, `ngayHieuLuc`, `trichYeu`, `tuKhoa`... |
| Cá thể (Individuals) | 82 | |
| — Văn bản pháp lý | 47 | 45 văn bản từ Excel + 2 kế hoạch của UBND tỉnh Khánh Hòa |
| — Chủ đề (đồng thời là khái niệm SKOS) | 23 | Có `prefLabel`/`altLabel` và quan hệ `broader`/`narrower` |
| — Cơ quan ban hành | 8 | Quốc hội, Chính phủ, Thủ tướng Chính phủ, Bộ Tài chính, Bộ Kế hoạch và Đầu tư, Bộ Thông tin và Truyền thông, Bộ Khoa học và Công nghệ, UBND tỉnh Khánh Hòa |
| Luật SWRL | 3 | R1, R2, R3 |

### Ba luật SWRL

```
R1: VanBan(?a) ^ VanBan(?b) ^ thayThe(?a,?b) ^ ngayHieuLuc(?a,?d)
    ^ swrlb:lessThanOrEqual(?d, hôm_nay)
    -> coTinhTrang(?b, HetHieuLuc)

R2: VanBan(?a) ^ VanBan(?b) ^ suaDoi(?a,?b)
    -> coTinhTrang(?b, HetHieuLucMotPhan)

R3: VanBan(?a) ^ VanBan(?b) ^ huongDan(?a,?b)
    -> lienQuanDen(?a,?b)
```

Điểm đáng chú ý của **luật R1**: chỉ suy ra "hết hiệu lực" khi văn bản thay thế
**đã đến ngày hiệu lực thi hành** (so sánh bằng builtin `swrlb:lessThanOrEqual`),
tránh trường hợp một văn bản mới được ban hành nhưng chưa thi hành lại khiến hệ
thống vội kết luận văn bản cũ đã hết hiệu lực. Cơ chế này đã được minh chứng
thực tế: Luật An ninh mạng 24/2018/QH14 giữ nguyên "còn hiệu lực" cho đến đúng
ngày Luật 116/2025/QH15 (văn bản thay thế) chính thức thi hành.

### Kiểm chứng bằng bộ suy luận Pellet

Ontology được kiểm tra bằng bộ suy luận **Pellet 2.3.1** (không dùng HermiT vì
phiên bản dòng lệnh của HermiT không hỗ trợ kiểu dữ liệu `xsd:date` cần thiết
cho luật R1). Kết quả: ontology **nhất quán (consistent)**, không phát sinh lớp
rỗng (unsatisfiable class).

---

## 6. Ba phương pháp tìm kiếm và thuật toán xếp hạng

Đề tài cài đặt và so sánh ba phương pháp:

| Phương pháp | Mô tả |
|---|---|
| **PP1 — Từ khóa thông thường** | Khớp chuỗi ký tự trên tên và trích yếu văn bản |
| **PP2 — SPARQL có cấu trúc** | Nhận diện ý định câu hỏi (theo cơ quan, tình trạng hiệu lực, quan hệ pháp lý, chủ đề) rồi sinh truy vấn SPARQL tương ứng |
| **PP3 — Ngữ nghĩa đầy đủ** | Kết hợp SPARQL + mở rộng từ đồng nghĩa qua SKOS + quan hệ pháp lý + thuật toán xếp hạng đa tiêu chí |

Công thức xếp hạng kết quả của PP3 (áp dụng cả trong đánh giá lẫn trong ứng
dụng Web):

```
Score = 0.30·KeywordMatch + 0.25·TopicMatch + 0.15·RelationBoost
      + 0.20·HieuLucBoost + 0.10·RecencyBoost
```

Trong đó `TopicMatch` được tính đầy đủ (1.0) khi khớp đúng chủ đề, hoặc một
phần (0.5) khi khớp qua chủ đề mở rộng bằng quan hệ `skos:broader`, cho phép
tìm "CNTT" ra được cả văn bản gắn nhãn "công nghệ thông tin" hay "phần mềm nội
bộ" — điều mà tìm kiếm từ khóa thông thường không làm được.

---

## 7. Kết quả thực nghiệm

Ba phương pháp được so sánh trên tập **24 truy vấn kiểm thử** (bao phủ 6 nhóm:
quản lý đầu tư, tình trạng/cơ quan, lĩnh vực/chủ đề, văn bản địa phương, quan hệ
pháp lý, tổng hợp) chạy trên corpus 47 văn bản:

| Phương pháp | Precision@5 | Recall@5 | MRR | nDCG@10 |
|---|---|---|---|---|
| PP1 — Từ khóa thông thường | 0,283 | 0,567 | 0,600 | 0,568 |
| PP2 — SPARQL có cấu trúc | 0,383 | 0,647 | 0,719 | 0,702 |
| **PP3 — Ngữ nghĩa đầy đủ** | **0,450** | **0,789** | **0,831** | **0,811** |

**PP3 vượt trội rõ rệt** trên cả 4 chỉ số, đặc biệt là Recall@5 và nDCG@10 —
minh chứng giá trị thực tiễn của việc kết hợp Web ngữ nghĩa (mở rộng đồng
nghĩa, suy luận quan hệ pháp lý) so với tìm kiếm từ khóa hoặc SPARQL cứng nhắc
thông thường.

---

## 8. Hướng dẫn cài đặt và khởi chạy từng bước

### Bước 0 — Chuẩn bị môi trường

**Yêu cầu:**
- Python 3.9 trở lên
- Java ≥ 8 (chỉ cần cho Bước 2 — kiểm tra bằng `java -version`)

**Cài thư viện Python** (chạy một lần duy nhất, trong thư mục gốc của bộ công cụ):

```bash
pip install -r requirements.txt
```

Nếu máy chưa có Java, cài OpenJDK:

```bash
# Ubuntu / Debian
sudo apt install default-jre

# macOS (dùng Homebrew)
brew install openjdk
```

---

### Bước 1 — Sinh ontology từ dữ liệu Excel

```bash
python buoc1_excel_to_rdf.py
```

**Việc script này làm:** đọc file `danh_sach_van_ban_mau.xlsx`, chuẩn hóa dữ
liệu, ánh xạ từng dòng thành các bộ ba RDF (cá thể văn bản, cơ quan, chủ đề,
quan hệ pháp lý), rồi ghi ra `digital_investment_v2.owl`.

**Kết quả mong đợi trên màn hình:**

```
======================================================================
BƯỚC 1 - SINH ONTOLOGY TỪ FILE EXCEL
======================================================================
Đọc dữ liệu từ : danh_sach_van_ban_mau.xlsx
Ghi ontology ra: digital_investment_v2.owl

classes: 38
object_properties: 20
datatype_properties: 24
individuals: 82
van_ban: 47
trieu_triples: 1479
...
HOÀN TẤT BƯỚC 1. Tiếp theo chạy: python buoc2_kiem_tra_suy_luan.py
```

Muốn dùng file Excel khác (ví dụ khi cập nhật thêm văn bản mới), chỉ định
đường dẫn:

```bash
python buoc1_excel_to_rdf.py duong_dan/file_moi.xlsx digital_investment_v2.owl
```

---

### Bước 2 — Kiểm tra tính nhất quán và suy luận SWRL

```bash
python buoc2_kiem_tra_suy_luan.py
```

**Việc script này làm:** nạp `digital_investment_v2.owl` vào bộ suy luận
Pellet 2.3.1, kiểm tra ontology có nhất quán không, in ra tình trạng hiệu lực
của từng văn bản sau khi áp dụng luật SWRL, và kiểm tra các quan hệ nghịch đảo
được suy ra tự động.

**Lưu ý:** bước này gọi một chương trình Java chạy ngầm nên có thể mất
**15–60 giây** — đây là hiện tượng bình thường, không phải bị treo.

**Kết quả mong đợi (rút gọn):**

```
[Pellet] Ontology NHẤT QUÁN (consistent) | Lớp rỗng: không có

Tình trạng hiệu lực sau suy luận (SWRL):
  VB_102_2009_N_CP       -> TINHTRANG_HetHieuLuc
  VB_24_2018_QH14        -> TINHTRANG_HetHieuLuc
  VB_73_2019_N_CP        -> TINHTRANG_HetHieuLuc, TINHTRANG_HetHieuLucMotPhan   <== SWRL suy ra
  ...
HOÀN TẤT BƯỚC 2. Tiếp theo chạy: python buoc3_danh_gia.py
```

Kết quả chi tiết được lưu vào `ket_qua/pellet_output.txt`.

---

### Bước 3 — Đánh giá 3 phương pháp tìm kiếm

```bash
python buoc3_danh_gia.py
```

**Việc script này làm:** chạy 24 truy vấn kiểm thử qua cả PP1, PP2, PP3, tính
các chỉ số Precision@5, Recall@5, MRR, nDCG@10 cho từng truy vấn và trung bình
toàn tập.

**Kết quả mong đợi (phần tổng hợp):**

```
TONG HOP (trung binh 24 truy van):
  PP1: P@5=0.283  R@5=0.567  MRR=0.600  nDCG@10=0.568
  PP2: P@5=0.383  R@5=0.647  MRR=0.719  nDCG@10=0.702
  PP3: P@5=0.450  R@5=0.789  MRR=0.831  nDCG@10=0.811

HOÀN TẤT BƯỚC 3. Tiếp theo chạy: python app.py để mở giao diện tìm kiếm
```

Kết quả chi tiết từng truy vấn được lưu vào `ket_qua/ket_qua_danh_gia.txt`.

---

### Bước 4 — Khởi chạy ứng dụng Web tìm kiếm

```bash
python app.py
```

**Kết quả mong đợi trên màn hình:**

```
======================================================================
BƯỚC 4 - ỨNG DỤNG WEB TÌM KIẾM VĂN BẢN (giao diện chính)
======================================================================
[Store mode: LOCAL] 47 văn bản, 1479 triples
Mở trình duyệt và truy cập: http://127.0.0.1:5000
(Nhấn Ctrl+C để dừng ứng dụng)
```

Mở trình duyệt và truy cập **http://127.0.0.1:5000** để sử dụng giao diện tìm
kiếm. Dừng ứng dụng bằng tổ hợp phím `Ctrl + C` trong cửa sổ dòng lệnh.

> **Chạy nhanh không cần qua Bước 1–3**: file `digital_investment_v2.owl` đã có
> sẵn trong gói, vì vậy nếu chỉ cần dùng giao diện tìm kiếm, có thể bỏ qua Bước
> 1–3 và chạy thẳng `python app.py`.

---

## 9. Hướng dẫn sử dụng ứng dụng

Sau khi mở `http://127.0.0.1:5000`, giao diện gồm:

1. **Ô tìm kiếm** — nhập câu hỏi bằng ngôn ngữ tự nhiên, ví dụ: *"chi phí phần
   mềm nội bộ"*, *"văn bản thay thế Nghị định 102"*, *"hạ tầng số của Khánh
   Hòa"*. Có sẵn 5 gợi ý nhanh (chip) tương ứng các kịch bản trong báo cáo đồ
   án (mục 3.12, Hình 3.19–3.23).
2. **Bộ lọc** — theo loại văn bản, cơ quan ban hành, tình trạng hiệu lực.
3. **Bảng kết quả** — mỗi văn bản hiển thị số hiệu, tên, trích yếu, tình trạng
   hiệu lực (có màu), điểm xếp hạng, và khối **"Vì sao hệ thống trả về văn bản
   này?"** giải thích rõ lý do (khớp từ khóa, khớp chủ đề mở rộng SKOS, nằm
   trong chuỗi quan hệ pháp lý, còn hiệu lực...).
4. **Đồ thị quan hệ pháp lý** — bấm "Xem đồ thị quan hệ" tại mỗi kết quả để
   xem trực quan chuỗi thay thế/sửa đổi liên quan đến văn bản đó.

### Các điểm cuối API (cho lập trình viên)

| Phương thức | Đường dẫn | Mô tả |
|---|---|---|
| GET | `/` | Trang giao diện tìm kiếm |
| GET | `/api/search?q=...&loai=&coQuan=&tinhTrang=` | Tìm kiếm, trả JSON kết quả kèm điểm và lý do |
| GET | `/api/graph/<số_văn_bản>` | Đồ thị quan hệ pháp lý của một văn bản |
| GET | `/api/stats` | Thống kê số văn bản, số triples, chế độ lưu trữ |

### Chuyển sang chế độ GraphDB (triển khai chính thức)

```bash
export STORE_MODE=GRAPHDB
export GRAPHDB_BASE=http://localhost:7200
export GRAPHDB_REPO=digital-investment
python app.py
```

Cần tạo trước repository tên `digital-investment` trên GraphDB và import file
`digital_investment_v2.owl` vào đó (xem chi tiết trong `config.py`).

---

## 10. Hạn chế và hướng phát triển

**Hạn chế hiện tại:**
- Quy mô dữ liệu 47 văn bản, dù đã đạt mục tiêu 30–50 văn bản lõi đề ra, vẫn
  còn giới hạn so với số lượng văn bản pháp lý thực tế của lĩnh vực.
- Giao diện hiện phù hợp demo và nghiên cứu; khi triển khai chính thức cần
  chạy trên WSGI server (gunicorn/uWSGI) phía sau nginx thay cho máy chủ phát
  triển của Flask.
- Chưa khảo sát mức độ hài lòng của người dùng thực tế trên diện rộng.

**Hướng phát triển:**
- Mở rộng corpus văn bản và tập truy vấn kiểm thử (hướng tới 30–50 truy vấn).
- Kết hợp xử lý ngôn ngữ tự nhiên (NLP) để hiểu câu hỏi phức tạp hơn.
- Tích hợp mô hình học máy/AI hỗ trợ tự động gán chủ đề và trích xuất quan hệ
  pháp lý từ văn bản gốc, giảm phụ thuộc vào nhập liệu thủ công qua Excel.
- Triển khai chính thức trên GraphDB với dữ liệu quy mô lớn.

---

## 11. Phụ lục: cấu trúc thư mục và xử lý sự cố

### Cấu trúc thư mục

```
ung_dung_web_tim_kiem/
├── README.md                       # Hướng dẫn kỹ thuật (bản rút gọn)
├── HUONG_DAN_DAY_DU.md             # Tài liệu này
├── requirements.txt
├── danh_sach_van_ban_mau.xlsx
├── digital_investment_v2.owl
├── buoc1_excel_to_rdf.py
├── buoc2_kiem_tra_suy_luan.py
├── buoc3_danh_gia.py
├── app.py
├── config.py
├── templates/
│   └── index.html
└── ket_qua/
    ├── digital_investment_v2_<thời gian>.owl
    ├── pellet_output.txt
    └── ket_qua_danh_gia.txt
```

### Xử lý sự cố thường gặp

| Hiện tượng | Nguyên nhân / cách xử lý |
|---|---|
| `ModuleNotFoundError` | Chạy lại `pip install -r requirements.txt` |
| Bước 2 báo lỗi liên quan Java | Cài Java ≥ 8, kiểm tra bằng `java -version` |
| Bước 2 chạy 15–60 giây rồi mới có kết quả | Bình thường, đang chờ Pellet (chương trình Java) xử lý |
| Bước 4 báo "Address already in use" | Cổng 5000 đang bị chiếm; dừng tiến trình cũ hoặc đổi số cổng trong `app.py` |
| Bước 1 báo thiếu cột dữ liệu | Kiểm tra file Excel có đủ 24 cột tiêu đề đúng chính tả (xem README.md mục cấu trúc dữ liệu) |
| Tìm kiếm không ra kết quả | Thử bỏ bớt bộ lọc, hoặc dùng từ khóa ngắn gọn hơn |

---

*Tài liệu này mô tả đầy đủ nội dung khoa học và kỹ thuật của đồ án, dùng kèm
theo báo cáo thuyết minh chính thức. Mọi số liệu (số văn bản, số triples, kết
quả đánh giá) được trích xuất trực tiếp từ lần chạy thực tế của bộ công cụ.*
