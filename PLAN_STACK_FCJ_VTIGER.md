# Kế hoạch kỹ thuật FCJ trên Vtiger 8.4 — Plan stack & phân tích

Tài liệu này bám **source core** trong workspace, **vtlib/Vtiger/NOTE.md** (bản đồ doc chính thức), và nội dung nghiệp vụ trong **BUSSINESSMODEL.md**. Dùng làm “single source of truth” kỹ thuật trước khi làm từng task.

---

## 1. Nguyên tắc bắt buộc (tiêu chí dự án)

| # | Tiêu chí | Cách đáp ứng trên Vtiger |
|---|----------|---------------------------|
| 1 | **Clean code** | Logic nghiệp vụ tách khỏi core: `custom/modules/`, event handlers, workflow; tránh copy-paste block lớn trong file core. |
| 2 | **Tối ưu workflow + DB có sẵn** | Ưu tiên **Workflow** (UI + `com_vtiger_workflow`), **Scheduled** qua cron `com_vtiger_workflow.service`; cập nhật record qua task “Update fields” / Email / Task; chỉ custom task PHP khi UI không đủ. |
| 3 | **Scale-up** | Field trên bảng `*scf`, index khi filter scheduled WF nhiều; điều kiện WF gọn, tránh full scan; tách workflow theo module (Lead vs Potentials vs Quotes). |
| 4 | **Chuẩn chỉnh, security** | Không bypass sharing: ưu tiên **Server APIs / WS** khi code tự động; kiểm tra profile; validate input trong handler custom; không `die()` trong production path (core WF có `die` khi lỗi — custom task nên bắt exception và log). |
| 5 | **Không phá core để dễ upgrade** | Không sửa `modules/Potentials/...` trực tiếp nếu tránh được; dùng **Module Manager fields**, **custom module** package ZIP, **EventHandler** đăng ký trong manifest extension, hoặc **overload** qua `custom/` nếu Vtiger hỗ trợ path đó. Can thiệp core chỉ khi bắt buộc và ghi rõ vào tài liệu merge upgrade. |

---

## 2. “Source map” — flow build module & thành phần nhỏ nhất cần nắm

### 2.1 Lớp kiến trúc (góc nhìn dev)

1. **Module entity (VD: Potentials, Leads, Quotes)**  
   - PHP: `modules/<ModuleName>/` — `Module.php`, `models/`, `views/`, `actions/`.  
   - DB: bảng base (`vtiger_potential`) + **custom field** (`vtiger_potentialscf`) + `vtiger_crmentity` (chung).  
2. **vtlib (đóng gói / bootstrap)**  
   - `Vtiger_Module::save()`, `initTables()`, `addBlock()`, `addField()` — metadata ghi vào `vtiger_field`, cột vào `*scf` hoặc bảng module.  
   - CLI: `php -f vtlib/tools/console.php` (create/import/update module).  
3. **Workflow**  
   - Module hệ thống: `modules/com_vtiger_workflow/` — `VTWorkflowManager`, `VTEventHandler`, `WorkFlowScheduler.php`, `VTTaskQueue`.  
   - **Immediate**: gắn save record (ON_FIRST_SAVE, ON_EVERY_SAVE, …).  
   - **Scheduled**: cron queue record eligible → chạy task; phụ thuộc **cron chạy đúng giờ**.  
4. **Field metadata (3 lớp — không nhầm)**  
   - `columntype` → DB thật.  
   - `uitype` → UI/hành vi (picklist 15/16/33, reference 10, user 53, currency 71, …).  
   - `typeofdata` → validation (V~M, D~O, …).  

Chi tiết và link S1–S12: **vtlib/Vtiger/NOTE.md**.

### 2.2 Case “workflow dùng module/file không tồn tại”

- **Triệu chứng**: Task kiểu “Create Entity” / email template / field update trỏ tới module bị tắt, bị gỡ, hoặc field đã xóa → lỗi runtime hoặc silent skip.  
- **Phòng tránh**:  
  - Checklist trước deploy: mọi field trong điều kiện WF phải tồn tại trong `vtiger_field` cho đúng `tabid`.  
  - Module phụ (SMS, Live Chat, Project) — **chỉ bật workflow** sau khi module đã cài và có quyền.  
  - Custom task PHP: `require_once` path rõ ràng; không giả định file có sẵn nếu extension chưa import.  

---

## 3. Ánh xạ nghiệp vụ FCJ → module core (không đọc Excel; theo BUSSINESSMODEL)

| Thuật ngữ | Module Vtiger | Bảng / custom table | Ghi chú |
|-----------|---------------|---------------------|---------|
| Deal | **Potentials** | `vtiger_potential`, `vtiger_potentialscf` | Rename label Opportunity → Deal; pipeline = **Sales Stage** (`sales_stage`). |
| Quote | **Quotes** | `vtiger_quotes`, … | Stage báo giá = field stage có sẵn của Quotes (kiểm tra bản 8.4). |
| Lead | **Leads** | `vtiger_leaddetails`, `vtiger_leadscf` | |
| Organization | **Accounts** | `vtiger_account` | |
| Contact | **Contacts** | `vtiger_contactdetails` | |
| Task / Event / Documents | **Calendar** / **Documents** | | Event = Activity type trong Calendar. |

**Quan trọng:** Trong tài liệu cũ có ghi `vtiger_dealscf` — **không đúng core**. Trên Potentials phải là **`vtiger_potentialscf`**.

---

## 4. Ma trận field đề xuất (Deal / Potentials + Lead)

Cột: Tên nghiệp vụ → Gợi ý **uitype** / **columntype** / **typeofdata** → Cách tạo → Ghi chú tối ưu.

### 4.1 Potentials (Deal)

| Field (nghiệp vụ) | uitype | columntype | typeofdata | Tạo bằng | Ghi chú |
|-------------------|--------|------------|------------|----------|---------|
| Deal Name | 2 (text) | VARCHAR(…) | V~M | Đổi label field `potentialname` | Không tạo field trùng; chỉ đổi ngôn ngữ. |
| Sales Stage (FCJ pipeline) | 15 picklist | VARCHAR(200) | V~M | Chỉnh picklist `sales_stage` | Thêm giá trị: JP Calling, Follow-up 1/2, … |
| Note / Failure reason | 19 hoặc 21 | TEXT | V~O | Custom field | Text area cho lý do thất bại. |
| Quote Sent Date | 5 | DATE | D~O | Custom | |
| First / Second / Third Reminder Date | 5 | DATE | D~O | Custom | |
| JP Call Request / JP Call Date | 5 | DATE | D~O | Custom | |
| Days Since … | 1 hoặc 7 | INT(11) | I~O | **Workflow/cron update** hoặc read-only formula | Tránh nhập tay; nên cập nhật bởi WF scheduled hoặc handler. |
| Campaign Result | 1 | VARCHAR(255) | V~O | Custom | Có thể chuyển 15 nếu tập giá trị cố định. |
| Order Received Date | 5 | DATE | D~O | Custom | |
| Remind1/2/3 Created, JP Call Requested | 56 checkbox | TINYINT(1) | C~O | Custom | Đúng pattern boolean CRM; **không** cần ALTER tay nếu tạo qua UI/vtlib. |
| Weighted Revenue | 71 (currency) hoặc 1 | DECIMAL(…)/VARCHAR | N~O hoặc V~O | Custom hoặc báo cáo | Core có amount/forecast; tránh trùng — ưu tiên công thức báo cáo. |

### 4.2 Leads (bổ sung cho workflow Phase 1)

| Field | Gợi ý | Ghi chú |
|-------|--------|---------|
| Customer Date (ngày KH gửi yêu cầu) | uitype 5, D~O | Scheduled WF “+1 day”. |
| Lead Reminder Sent | checkbox 56 | Tránh tạo task trùng. |
| FCJ Response Date / FCV Response Date | uitype 5 | Phục vụ đo “ngày đầu phản hồi”. |
| Update Response Status | Text hoặc Picklist | Picklist ổn định hơn cho báo cáo. |
| MTG (có meeting) | 56 hoặc 15 Yes/No | |

**Quy tắc chọn:**  
- Ngày → uitype **5**, `DATE`, `D~O`/`D~M`.  
- Cờ idempotent workflow → **checkbox** (56), `TINYINT(1)`.  
- Liên kết module → **10** + `setRelatedModules()`.  
- Số đếm ngày → ưu tiên **tự động** (WF Update field / custom task nhỏ), không bắt user nhập.

---

## 5. Tái sử dụng module vs module mới

| Nhu cầu | Khuyến nghị |
|---------|-------------|
| Pipeline bán hàng + báo giá + convert Lead | **Giữ Potentials + Quotes**; chỉ thêm field + picklist + label. |
| Pipeline “FCJ Order” sau Closed Won (Production/QC/Delivery) | **Project** (nếu đã dùng) hoặc **custom module** “FCJ Order” liên kết Potentials/Quotes — tránh nhồi hết vào Potentials làm sales_stage quá tải. |
| Slack / marketing list | **Integration** (VTAP không có trên self-host) → **Webhook custom task** hoặc service ngoài gọi REST — đặt trong extension package. |

**Logic sắp xếp lại:**  
1) Lead + Task + Event = **Acquisition**.  
2) Potentials + Quotes = **Presales / Deal**.  
3) SalesOrder + Invoice + (Project) = **Delivery & revenue**.  
Workflow Phase 3–4 trong BUSSINESSMODEL nên gắn đúng **module trigger** (Quote updated vs Potentials scheduled) để điều kiện không chồng chéo.

---

## 6. Workflow — tổng hợp vấn đề / cách xử lý / hướng làm

### 6.1 Vấn đề kỹ thuật thường gặp

| # | Vấn đề | Cách xử lý |
|---|--------|------------|
| W1 | Scheduled WF không chạy | Bật cron `cron/modules/com_vtiger_workflow/com_vtiger_workflow.service`; kiểm tra `nexttrigger_time`, timezone server. |
| W2 | Điều kiện ngày phức tạp (Today ≥ X + N ngày) | Dùng scheduled WF daily + điều kiện so sánh field date; hoặc delayed task queue (`trigger` days trên task). |
| W3 | Trùng Task/Email | Field cờ (`Lead Reminder Sent`, `Remind1 Created`) + điều kiện WF = false trước khi set true. |
| W4 | Quote vs Deal làm master ngày gửi báo giá | Thống nhất: **Quote Stage = Delivered** cập nhật field trên **Potentials** (cross-module WF hoặc handler sau save Quote). |
| W5 | Stop reminder khi Closed Won/Lost | Mọi WF follow-up thêm điều kiện `sales_stage NOT IN (Closed Won, Closed Lost)` (và tương đương Quote). |
| W6 | JP gọi xong → Negotiation | WF **ON_MODIFY** Potentials khi `jp_calldate` filled hoặc checkbox “JP called”; update `sales_stage`. |

### 6.2 Bản đồ workflow đề xuất (bước triển khai)

1. **WF-L0** — Lead Created → Task Call 24h (ON_FIRST_SAVE hoặc ON_EVERY_SAVE có điều kiện).  
2. **WF-L1** — Scheduled: Not Contacted + đủ field + Customer Date +1d → Task + email + set cờ.  
3. **WF-L2** — Scheduled: Attempted to Contact + modified +2d → Task.  
4. **WF-Q1** — Quote updated: Stage Delivered → set Potentials `quote_sent_date`, stage Proposal/Price Quote, cờ remind.  
5. **WF-P1..P3** — Scheduled trên Potentials: chuỗi Remind 1/2/3 theo spec cột O,P,Q,W,X (đồng bộ với BUSSINESSMODEL).  
6. **WF-CW** — Deal Closed Won → (tuỳ bật module) SalesOrder + Project + Task kickoff.  
7. **WF-CL** — Closed Lost / Quote Rejected → đồng bộ stage + tắt nhánh remind.

Chi tiết kịch bản người dùng giữ nguyên trong **BUSSINESSMODEL.md**; khi code, map từng dòng sang **một WF** với **module + execution type + điều kiện + task list**.

---

## 7. Plan stack — thứ tự task (để bạn giao việc từng phần)

| Thứ tự | Task | Đầu ra | Phụ thuộc |
|--------|------|--------|-----------|
| T0 | Xác nhận edition 8.4, module bật (SMS, Documents, Project?) | Checklist môi trường | — |
| T1 | Đổi label Potentials → Deal (ngôn ngữ); chỉnh picklist `sales_stage` FCJ | Language pack / Settings | T0 |
| T2 | Tạo field custom Lead (Customer Date, flags, response dates) | Module Manager hoặc script vtlib | T0 |
| T3 | Tạo field custom Potentials (dates, flags, failure note) | MM / vtlib | T1 |
| T4 | Kiểm tra liên kết Quote–Potentials cho WF cross-update | 1 WF thử hoặc handler nhỏ | T3 |
| T5 | Triển khai WF Phase 1 (Lead) | Export WF JSON / doc cấu hình | T2 |
| T6 | Triển khai WF Phase 3 (Quote + Potentials) | Cron ổn định | T4, T5 |
| T7 | Closed Won/Lost + stop rules | WF + test record | T6 |
| T8 | (Tuỳ chọn) Slack / Marketing — extension package | Code trong `pkg/` hoặc `custom/` | T7 |

---

## 8. Tài liệu tham chiếu (research)

- Trong repo: **vtlib/Vtiger/NOTE.md** — URL đầy đủ S1–S12 (community.vtiger.com, help.vtiger.com, code.vtiger.com).  
- Đọc theo thứ tự khuyến nghị NOTE: S1 → S3 → S5 → S2 → S4 → S6 (+ S11 khi lệch doc).  

---

## 9. Ghi chú bàn giao

- Mọi thay đổi sau này nên có **một dòng trong bảng task** (mục 7) và cập nhật **ma trận field** (mục 4) nếu đổi uitype/column.  
- Khi user yêu cầu “làm task Tn”, mở song song **BUSSINESSMODEL** (kịch bản) + file này (kỹ thuật).  

*Cập nhật: 2026-03-23*
