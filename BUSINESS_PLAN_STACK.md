## 1. Mục tiêu & Nguyên tắc phát triển

- **Mục tiêu tổng quát**
  - Chuẩn hoá business model (CONTACT / LIST ORDER / MANAGEMENT) trên vtiger 8.4.
  - Tối đa **tái sử dụng core** (modules, workflows, DB schema) và chỉ custom khi cần.
  - Đảm bảo **khả năng nâng cấp source** về sau (ít đụng core nhất có thể).

- **Nguyên tắc kỹ thuật bắt buộc**
  - **Clean code**: 
    - Tách rõ phần core vs custom (`modules/Custom*`, `layouts/v7/modules/Custom*`, vtlib).
    - Đặt tên field/module/workflow rõ nghĩa, thống nhất với business.
  - **Tối ưu & tận dụng core workflow**:
    - Ưu tiên dùng **Workflow**, **Task**, **Event**, **vtlib handler**, **Server API** trước khi viết logic SQL/custom PHP mới.
    - Chỉ thêm file/module khi core không có sẵn; nếu workflow đang gọi file/module bị thiếu → xử lý bằng:
      - Tạo module/file bổ sung đúng chuẩn vtlib, hoặc
      - Điều chỉnh workflow để trỏ đúng module/file hiện có.
  - **Scale-up & maintainable**:
    - Thiết kế field/quan hệ để support volume lớn (index, picklist hợp lý, hạn chế logic nặng trong trigger đơn record).
    - Business rule nên tập trung trong workflow + custom module rõ ràng (ít “rải” code khắp nơi).
  - **Security & data integrity**:
    - Tuân thủ sharing, profile, permission của vtiger (không bỏ qua checkPermission trừ khi rất cần).
    - Tránh query SQL thủ công nếu đã có API vtiger.
  - **Không phá core**:
    - Ưu tiên **extend**: module mới, view/action override trong module, vtlib handler, event handler.
    - Chỉ chỉnh core khi không còn lựa chọn khác, và phải:
      - Ghi chú rõ ràng trong doc.
      - Isolate bằng hook/if-check để giảm rủi ro khi upgrade.

---

## 2. Tóm tắt business model từ góc nhìn người dùng

### 2.1 CONTACT – Sheet 問合管理表 (Lead / Inquiry)

- **Ý nghĩa**: 
  - Lưu toàn bộ khách hàng mới gửi yêu cầu báo giá qua nhiều kênh.
  - Director nhập/kiểm soát danh sách này.
  - Đây là **điểm bắt đầu Lead** trong vtiger.
- **Thông tin chính (map sang Lead)**:
  - `lead_no` → mã lead.
  - `customerDate` (B) → ngày khách hỏi (datetime).
  - `customerMonth` → tháng từ customerDate (integer).
  - `company` (D) → tên công ty (hoặc tên cá nhân nếu free).
  - `cumulative` (E) → luỹ kế (đếm lead của cùng KH).
  - `firstname + lastname` (F) → tên người liên lạc.
  - `mobile`, `email`.
  - `contentContact` (I) → nội dung liên lạc.
  - `customerType` (J) → phân loại khách.
  - `meetingDate` (K) → ngày meeting.
  - `responseStatus` (L) → tình trạng đối ứng (picklist).
  - `FCJResponseDate`, `FCVResponseDate` (M, N).
  - `note` → ghi chú.
  - `leadReminderSent` (bool).
  - `leadStatus` (Lead Status – picklist).

### 2.2 Bắt đầu Quote & Deal (Opportunity / Deal module)

- **Ý nghĩa**:
  - Giai đoạn từ lúc **gửi báo giá** đến khi theo dõi phản hồi, gọi nhắc, đánh giá kết quả (thành công/thất bại).
  - Business muốn tracking chi tiết trên **Deal (Opportunity)** + liên kết với Quote, không chỉ trên Lead.
- **Thông tin chính (Lead → Deal & Quote)**:
  - `quoteSentDate` (O) → ngày gửi email báo giá (datetime).
  - `firstReminderDate` (P) → sau 2 ngày kể từ `quoteSentDate`, gửi mail remind lần 1 (datetime).
  - `JPCallRequestDate` (Q) → sau 1 ngày kể từ remind 1, ngày nhắn cho bên Nhật để call KH (datetime).
  - `JPCallDate` (R) → ngày ông Nhật thực sự gọi; nếu không điền là mất liên lạc (datetime).
  - `Deal Stage / Sales Stage` (S) → status trên Deal (thay vì dùng `leadstatus` trực tiếp).
  - `failureReason` (T) → ghi chú lý do thất bại (ví dụ KH nhờ công ty khác).
  - `FCJResponseCountDate` (U) → số lần phản hồi từ FCJ (integer – cần define rule rõ hơn).
  - `quoteSentCountDate` (V) → số lần gửi báo giá (integer).
  - `secondReminderDate` (W) → remind lần 2 (sau 2 tuần kể từ khi nhắn JPCallRequest).
  - `thirdReminderDate` (X) → remind/campaign FREE (sau 1 tháng kể từ JPCallRequest).
  - `saleOrderStatus` (Y) → kết quả cuối cùng: thành công hay thất bại (varchar; có thể map sang Sales Order / Deal outcome).
- **Lưu ý module**:
  - Cần **Deal module** (thay thế / mở rộng `Potentials` – Opportunity core) với **các field trên**.
  - Các workflow Quote/Deal phải kiểm tra đầy đủ file/module; nếu thiếu file trong `modules/Deal/*` thì phải:
    - Bổ sung file view/action/model tương ứng, hoặc
    - Điều chỉnh workflow để không gọi vào file/module không tồn tại.

### 2.3 LIST ORDER – Sheet 受注案件 (Sales Order + Project/Task)

- **Ý nghĩa**:
  - Mỗi dòng là **một task/đơn hàng** (new order hoặc change request).
  - Quản lý full lifecycle: inquiry → order → production → delivery → payment.
- **Thông tin chính (map sang Sales Order + Project)**:
  - `saleOrder_no` → số đơn (Sales Order No).
  - `customerDate` (Booking Date) → ngày khách booking.
  - `officialDataReceivedDate` → ngày nhận data chính thức.
  - `DueMonth` → tháng dự kiến.
  - `customerDeliveryDate` → mục tiêu giao hàng.
  - `paymentReceivedDate`, `paymentMonth`.
  - `invoiceDate`.
  - `deliveryOKDate` → ngày khách confirm OK (copy từ Project).
  - `contactPerson` → user DR phụ trách.
  - `defaultStatus` + `status` của Sales Order.
  - `contactName` (KH).
  - `subject` → tên đơn hàng (Project name).
  - `salesAmount` → giá báo (sau thuế).
  - `actualPaymentAmount` → số tiền thực trả.
  - `estimatedLaborHours`, `actualLaborHours`.
  - `costPerHour`, `salesPricePerHour`.
  - `totalCost`, `discount`, `grossProfit`.
  - `paymentDeliveryOK` (picklist DONE/NOT DONE).
  - `customerType` (mới/cũ).
  - `QCCheckSheet` (URL / Documents).
  - `orderNote`, `orderType` (Outsourcing/FCJ).

### 2.4 MANAGEMENT – Sheet quản lý nội bộ (Leader / Coder)

- **Ý nghĩa**:
  - Dùng cho team nội bộ (Leader, Coder) để assign, estimate, track tiến độ.
  - Tập trung vào: ORDER NUMBER, ORDER NAME, TYPE1, TYPE2, PAGE, COST, STATUS, REAL DELIVERED DATE…
- **Liên kết**:
  - Thực chất là **layer quản trị trên Project/Project Task/Sales Order**:
    - ORDER NUMBER ↔ Sales Order / Project.
    - TYPE ↔ picklist trên Project/Sales Order.
    - REAL DELIVERED DATE ↔ Delivery OK / Project Completed.

### 2.5 Workflow phases (Lead → Deal → Quote → Sales Order)

- **Flow cơ bản**:
  - Lead → (Contact / Organization) → Deal (Opportunity) → Quote → Sales Order (task).
  - Nhiều Sales Order có thể xuất phát từ **cùng một Lead/Deal**.
  - Change request cũng là **Sales Order mới** gắn với cùng Lead/Deal.
- **Workflow ví dụ trong file**:
  - PHASE 1 – INQUIRY (Lead Status).
  - PHASE 2 – MEETING & QUALIFICATION.
  - Rất nhiều kịch bản chi tiết: 
    - Not Contacted > 24h → auto tạo Task + gửi email.
    - Attempted to Contact > 48h → follow-up.
    - Contacted / In Progress / Qualified / Lost Lead.

---

## 3. Vấn đề kỹ thuật & rủi ro được nêu

- **1. Thiếu file/module so với workflow**:
  - Có case: workflow đang gọi tới file/module X nhưng **module không tồn tại hoặc thiếu file**, dẫn tới lỗi runtime.
  - Bài học: trước khi thiết kế workflow, phải:
    - Kiểm tra **module + file** có tồn tại trong `modules/<Module>/...`.
    - Kiểm tra **DB schema** trong `vtigercrm.sql` để biết field/module có thật hay không.
- **2. Lệch giữa business model & core module**:
  - Business sử dụng Excel với logic phức tạp (màu sắc, rule thời gian, threshold tiền…).
  - Core vtiger có sẵn modules (Leads, Deals, Quotes, SalesOrder, Project…), nhưng **không tự động khớp** với mọi cột Excel.
- **3. Rủi ro khi chỉnh core**:
  - Dễ phá khả năng upgrade nếu **sửa trực tiếp** file core.
  - Phải ưu tiên **extend/override** thay vì edit core, trừ khi thật sự cần.

---

## 4. Chiến lược kỹ thuật tổng thể

### 4.1 Mapping business ↔ core modules (và CÁCH LÀM thực tế)

- **Lead / CONTACT sheet**:
  - Giữ nguyên module `Leads` của core.
  - Thêm custom fields tương ứng: `customerDate`, `customerMonth`, `cumulative`, `contentContact`, `customerType`, `meetingDate`, `responseStatus`, `FCJResponseDate`, `FCVResponseDate`, `leadReminderSent`, `note`...
  - Dùng **Lead Status**, Task, Event, Documents, SMS theo core.
  - **Cách làm**:
    - B1: Kiểm tra trong `vtigercrm.sql` + `vtiger_field` xem field nào đã tồn tại, field nào chưa.
    - B2: Với field chưa có:
      - Ưu tiên tạo qua **Settings → Module Manager → Leads → Add Custom Field**, cấu hình:
        - `uitype`, `typeofdata`, length… theo bảng thiết kế.
      - Nếu cần script tự động (import nhiều môi trường) → viết script vtlib sử dụng `Vtiger_Field` để add field.
    - B3: Nếu cần xử lý thêm về UI (ẩn/hiện, validation đặc biệt) → override view trong `modules/Leads/views` hoặc dùng JS trong `layouts/v7/modules/Leads/resources`.

- **Deal / Opportunity (Deal module custom)**:
  - Dùng module `Potentials` của core làm nền tảng, hoặc tạo **module Deal riêng** nếu đã tồn tại trong source (thay thế cho Opportunity).
  - Bổ sung các field:
    - `quoteSentDate`, `firstReminderDate`, `JPCallRequestDate`, `JPCallDate`,
      `failureReason`, `FCJResponseCountDate`, `quoteSentCountDate`,
      `secondReminderDate`, `thirdReminderDate`, `saleOrderStatus`.
  - Map **Qualified Lead** → auto tạo Deal (theo workflow convert Lead).
  - Deal Stage / Sales Stage sẽ là **trục chính để đánh giá kết quả** (thay cho việc nhồi thêm status vào Lead).
  - **Cách làm**:
    - B1: Kiểm tra trong source:
      - Có module `Potentials` chuẩn hay đã có module `Deal` riêng (ví dụ `modules/Deal/*`) không.
    - B2: Quyết định:
      - Nếu dùng lại `Potentials`:
        - Thêm các field trên vào module Potentials bằng Module Manager/vtlib.
      - Nếu có/định tạo module `Deal`:
        - Dùng **vtlib console** hoặc script PHP mẫu (Entity Module Example) để tạo module Deal chuẩn (bảng, CRMEntity, vtlib_handler).
        - Thêm các field quote/reminder/JPCall/failureReason trên module Deal.
    - B3: Cấu hình **Deal Stage (Sales Stage)**:
      - Chuẩn hoá các stage (New, Quoted, Follow-up 1st, JP Call Requested, JP Called, Lost, Won…).
    - B4: Workflow convert Lead:
      - Sử dụng workflow on Lead Status = Qualified để:
        - Gọi convert Lead (Contacts + Organization).
        - Tạo Deal (Potentials/Deal) với link từ Lead, map các field cần thiết.

- **Quote**:
  - Dùng module `Quotes` core, gắn với Deal.

- **Sales Order / LIST ORDER**:
  - Dùng module `SalesOrder` core, bổ sung:
    - Field ngày: booking, officialDataReceived, delivery target, payment, invoice, delivery OK…
    - Field tài chính: salesAmount, actualPaymentAmount, discount, grossProfit…
    - Field công số: estimated/actual hours, cost/sales price per hour.
    - Field trạng thái tổng hợp: paymentDeliveryOK, defaultStatus, orderType, customerType.
  - **Cách làm**:
    - B1: Đọc kỹ sheet LIST ORDER và đối chiếu `vtiger_salesorder` + `vtiger_field` trong `vtigercrm.sql`.
    - B2: Với field còn thiếu, thêm bằng Module Manager/vtlib:
      - Chọn `uitype` phù hợp: Date/DateTime (5/70), Currency (71), Integer (7), Picklist (15)...
    - B3: Nếu cần tính toán tự động (totalCost, grossProfit):
      - Ưu tiên dùng **workflow Update Field** hoặc **UI formula/report**, tránh viết SQL tay trong core.
    - B4: Kết nối với Project:
      - Dùng relation chuẩn (SalesOrder ↔ Project) hoặc thêm relation qua `Vtiger_Module::setRelatedList`.

- **Project / Project Task / Internal management**:
  - Dùng `Project` + `ProjectTask` của core để phục vụ sheet MANAGEMENT và tracking nội bộ.
  - Copy/mirror một số field từ SalesOrder/Project cho thống nhất (estimated/actual hours, cost, deliveryOKDate, QCCheckSheet…).
  - **Cách làm**:
    - B1: Xác định field nào cần xuất hiện trên cả Project/ProjectTask/SalesOrder.
    - B2: Tạo field tương ứng trên Project/ProjectTask (nếu chưa có).
    - B3: Dùng workflow:
      - Khi tạo SalesOrder chính thức → auto tạo Project + các ProjectTask, copy field cần thiết.
      - Khi Project/Task cập nhật (actual hours, delivery OK) → sync ngược về SalesOrder nếu cần.

### 4.2 Workflow & automation (cách cấu hình cụ thể)

- **Ưu tiên dùng Workflow engine của vtiger**:
  - Scheduled workflows cho:
    - Not Contacted > 24h, Attempted to Contact > 48h…
    - Remind sau 2 ngày, 2 tuần, 1 tháng… (first/second/thirdReminderDate).
  - Event-based workflows khi:
    - Lead Status đổi (Contacted, In Progress, Qualified, Lost).
    - Sales Order Payment/Delivery OK.
  - Action:
    - Tạo Task, Event, gửi Email, cập nhật trường, gửi SMS (nếu module SMS).

- **Rule**:
  - Trước khi config workflow, **check đủ field/module**:
    - Field tồn tại trong DB (`vtiger_field` + `vtigercrm.sql`).
    - Module có file `views`/`actions` cần thiết nếu workflow gọi tới.
  - **Cách làm tổng quát cho từng workflow**:
    - B1: Xác định **trigger**:
      - On create / On update (field X) / Scheduled (daily, hourly…).
    - B2: Xác định **điều kiện** theo business (ví dụ: Today ≥ quoteSentDate + 2 days AND firstReminderDate IS EMPTY).
    - B3: Chọn **actions**:
      - Create Task / Event.
      - Update field (ghi lại firstReminderDate, secondReminderDate…).
      - Send Email / SMS.
    - B4: Test trên **sandbox**:
      - Tạo vài record demo và chờ workflow chạy (đặc biệt với scheduled).
    - B5: Kiểm tra log/lỗi nếu có (vtiger logs, cron).

### 4.3 Customization strategy (chiến lược code cụ thể)

- **Field-level**:
  - Thêm field bằng **vtlib / Module Manager** (không sửa DB tay).
  - Đặt `columntype`, `uitype`, `typeofdata` theo docs (tránh mismatch).

- **Module-level**:
  - Nếu core module đủ dùng → chỉ thêm field & workflow.
  - Nếu cần logic mới → tạo **custom module** (vd: `FCVManagement`, `FCVReport`) dùng vtlib.

- **Code-level**:
  - Override view/action bằng file trong `modules/<Module>/views` / `actions` (không đè core vtlib).
  - Dùng `vtlib_handler` để đăng ký quan hệ, sequence, widget, hooks khi install/update module.
  - **Cách làm khi cần chỉnh core**:
    - Chỉ can thiệp khi:
      - Không có hook/workflow/override nào đáp ứng được.
    - Khi buộc phải sửa:
      - Ghi chú rõ trong comment và cập nhật vào `BUSINESS_PLAN_STACK.md`.
      - Hạn chế sửa logic chung, chỉ thêm nhánh `if` cho case business đặc biệt.
      - Giữ nguyên signature method để tránh lỗi khi upgrade.

---

## 5. Roadmap triển khai (sẽ tách task cụ thể sau)

### Phase A – Chuẩn hoá Lead (CONTACT) — Cách làm từng bước

1. **Review & thiết kế field Lead**:
   - Map từng cột CONTACT sheet ↔ field trong `Leads`.
   - Tạo/điều chỉnh field thiếu bằng vtlib.
2. **Thiết kế Lead Status & picklist**:
   - Not Contacted, Attempted to Contact, Contacted, In Progress, Qualified, Lost Lead…
3. **Xây workflows cho PHASE 1 (INQUIRY)**:
   - WF 0.1: Not Contacted > 24h.
   - WF 1.1: Attempted to Contact > 48h.
   - Các kịch bản 2, 3, 4 theo file.
4. **Checklist kỹ thuật**:
   - Xác nhận tất cả field mới xuất hiện trong `vtiger_field` đúng moduleid.
   - Test tạo Lead demo đi qua các trạng thái để xem workflow chạy đúng.

### Phase B – Meeting, Qualification & Convert Lead — Cách làm từng bước

1. **Thiết lập Event/Task chuẩn cho Meeting**.
2. **Workflow convert Lead → Contact/Org/Deal**:
   - Điều kiện: Lead Status = Qualified.
   - Đảm bảo tạo đủ Contact, Organization, Deal.
3. **Kiểm tra mapping**:
   - Thông tin từ Lead (customerType, contentContact, meetingDate,…) có cần copy sang Contact/Org/Deal không.
   - Nếu có → thêm vào workflow/convert mapping.

### Phase C – LIST ORDER (Sales Order + Project) — Cách làm từng bước

1. **Thiết kế field trên SalesOrder** theo sheet LIST ORDER.
2. **Liên kết SalesOrder với Project/ProjectTask**:
   - Rule khi nhận Official Data → tạo Project/Tasks.
3. **Workflow cho Payment & Delivery OK**:
   - Auto set DONE khi thỏa điều kiện (payment date, amount, delivery OK).
4. **Test end-to-end**:
   - Từ Deal Won → tạo Quote → Sales Order → Project/Tasks.
   - Check các mốc: Delivery Date, Payment Date, Delivery OK, PaymentDeliveryOK.

### Phase D – MANAGEMENT & Reporting — Cách làm từng bước

1. **Thiết kế màn hình nội bộ** (view/summary) cho Leader/Coder.
2. **Báo cáo công số & lợi nhuận**:
   - Dùng Reports module hoặc custom report module.
3. **Nếu cần UI riêng**:
   - Tạo custom view trong `modules/Project/views` hoặc module custom (VD: `FCVManagement_View`).
   - Dùng JS + template trong `layouts/v7/modules/...` để gom thông tin từ Project/Task/SalesOrder.

---

## 6. Lưu ý khi bắt đầu implement tasks

- Mỗi task dev sau này cần:
  - Chỉ rõ: **đang động vào module nào**, field nào, workflow nào.
  - Xác nhận: **không phá vỡ** behavior chuẩn của module khác.
  - Ghi doc ngắn (comment + update file plan này nếu là thay đổi kiến trúc).

File này là **kế hoạch tổng & khung kiến trúc**; các bước chi tiết sẽ được tách thành task cụ thể khi bạn yêu cầu tiếp theo.

