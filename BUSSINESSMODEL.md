<!-- GIẢI THÍCH -->

1. Sheet: 問合管理表/CONTACT 																									
Hiện thị tất cả khách hàng mới có nhờ bên mình báo giá. Khách có gửi yêu cầu báo giá từ các kênh liên lạc đến thì Director sẽ update vào sheet này																									
Bắt đầu Lead																									
(A): no.		lead_no	s/d của vtiger	Lead																					
日付 (B): Ngày khách hỏi		customerDate	datetime	Customer Date	Lead																				
C: tháng của nó (auto tính ra)		customerMonth	integer	Customer Month	Lead																				
"会社名 (D): Tên cty
（不明orフリーの場合個人名）"		company	s/d của vtiger		Lead																				
累計 (E)		cumulative	integer	Cumulative	Lead (thêm vào)																				
担当者名 (F): Tên KH, tên người liên lạc với mình			firstname + lastname 		s/d của vtiger	Lead																			
(G): phone		mobile	s/d của vtiger	Mobile Phone		Lead																			
(H): email		email	s/d của vtiger	Primary Email		Lead																			
内容 (I): nội dung liên lạc là gì			contentContact	varchar(255)	Content Contact	Lead																			
ランク (J): phân loại KH (TrangLK ko hiểu, FCJ tạo)				customerType	integer	Customer Type	Lead																		
MTG (K): Ngày meeting, ví dụ: Khách yêu cầu meeting với nó				meetingDate	datetime	Meeting Date	CUSTOM DATETIME	Lead																	
対応状況 (L): Tình trạng đối ứng		responseStatus	varchar(255)	Response Status			PICKLIST / VARCHAR	Lead																	
(M): Ngày mà ông FCJ gửi mail trả lời cho khách (ông ấy đối ứng đầu tiên)					FCJResponseDate	datetime	FCJ Response Date	Lead																	
(N): TrangLK gửi email confirm cho khách nếu chưa clear					FCVResponseDate	datetime	FCV Response Date	Lead																	
Note: Cập nhật thêm thông tin của Lead status					note	varchar(255)	Note	Lead (chưa thêm vào)																	
Lead Reminder Sent: default false		leadReminderSent	boolean	Lead Reminder Sent	Lead																				
Lead Status	leadStatus	s/d của vtiger (picklist)	Lead Status	Lead	kham khảo sheet Training Team																				
																									
Bắt đầu Quote và Deal (Opportunity)																									
(O): Ngày gửi email báo giá		quoteSentDate	datetime	Quote Sent Date	Lead → Deal và Quote																				
(P): Sau 2 ngày kể từ ngày gửi mail báo giá gửi mail remind lần 1				firstReminderDate		datetime	First Reminder Date	Lead → Deal																	
(Q): Ngày nhắn cho ông bên Nhật để ổng call khách hàng (rule sau 1 ngày remind lần 1)						JPCallRequestDate	datetime	JP Call Request Date		Lead → Deal															
"(R): Ngày mà ông Nhật gọi điện cho KH (ko biết chính xác ngày ổng gọi) nên khi ổng call thì ổng tự điền cột này.
Nếu ổng ko call dc thì ổng không điền, tức là mất liên lạc với KH"							JPCallDate	datetime	JP Call Date	Lead → Deal															
(S): Status. ví dụ thất bại với KH		leadstatus (bỏ)	Deal Stage (Sales Stage)		s/d của vtiger	Lead → Deal																			
(T): Chú thích thất bại là KH nhờ cty khác làm rồi			failureReason	varchar(255)	Failure Reason	Lead → Deal																			
(U): 	FCJResponseCountDate		integer																						
(V)	quoteSentCountDate		integer																						
(W): remind lần 2 (rule remind lần 2 sau 2 tuần kể từ ngày nhắn cho ông Nhật gọi điện)					secondReminderDate	datetime	Second Reminder Date	Lead → Deal																	
(X): Thêm 2 tuần nữa, tức là 1 tháng kể từ ngày nhắn ông Nhật gọi điện thì TrangLK bắt đầu gửi khách campaign FREE							thirdReminderDate	datetime	Third Reminder Date	Lead → Deal															
(Y): status: kết quả thành công hay thất bại			saleOrderStatus	varchar(255)	Sale Order Status																				
																									
																									
2. Sheet: 受注案件 / LIST ORDER (hay là Task cho đơn ngoài)																									
(DR làm việc với KH, d/s này tính theo task, mỗi task có thể là new order, hoặc change request của new order này). Tính năng hệ thống cần tạo task																									
H: xuất hoá đơn => F: ngày khách trả tiền => E: giao data cho khách																									
NO (A): đơn mới về nhập số thứ tự		saleOrder_no	s/d của vtiger	Sales Order																					
問合せ: Inquiry																									
"  + 日付 (B): Date: Ngày Khách Booking
   (Khách chưa gửi design data chính thức,
    hỏi booking trước rồi gửi design data sau)"		customerDate	datetime	Customer Date	Sales Order																				
受注日: Order date																									
"  + 日付 (C): Date: Ngày khách nhờ làm đơn
     chính thức (khách đã đưa design)"		officialDataReceivedDate	datetime	Official Data Received Date	Sales Order	"Khi K/H gửi Data chính thức thì updated vào field này. Khi đó gửi vài thông tin qua email 
cho team manager để tạo đơn hàng để làm ( s/d module Project và project task của Vtiger)"																			
  + 月 (D): Month: Có công thức để tính ra được tháng		DueMonth 	integer	Due Month	Sales Order																				
納品日: Delivery date		customerDeliveryDate	datetime or integer	Customer Delivery Date	"Sales Order
(Project: Internal Delivery Target)"																				
"  + 日付 (E): Date: Ngày giao hàng (ngày giao cả data hoàn chỉnh, chứ không phải ngày đưa link test(server fcv) để KH check). Team DR không quản lý ngày đưa link để test, chỉ quản lý ngày giao hàng thôi. 
      Nếu chưa biết ngày giao hàng chính xác thì DR cho mục tiêu giao hàng trước để khỏi quên và dễ theo dõi tiến độ. Ví dụ cho số 7, tức là trong tháng 7 giao hàng. 
      Cột E này ít khi biết trước ngày giao hàng, khi nào Khách báo thì DR mới biết. Setting cho calender tháng 7, tới tháng 7 notify remind tới DR, giúp cho DR follow được những đơn xảy ra vào tháng đó để kịp thời ứng biến. 
      Chổ này có chức năng chọn date cụ thể hoặc chỉ chọn tháng ước lượng"																									
入金日: Payment date		paymentReceivedDate	datetime or integer	Payment Received Date	Sales Order																				
"  + 日付 (F): Date: Ngày khách trả tiền (ngày DR dự đoán cho đến khi nào DR lên Kintone xuất giấy hoá đơn đòi tiền và gửi email cho khách thì mới update ngày cụ thể, 
      nhưng cũng tương đối không chính xác đôi lúc khách vẫn trả tiền trễ. Như vậy chổ này khi nào khách trả tiền cho FCJ và FCJ tag DR trên Kintone là khách này đã trả tiền thì DR update ngày cụ thể vào đây)"																									
  + 月 (G): Month		paymentMonth 	integer	Payment Month 	Sales Order																				
請求書発行日: Invoice date		invoiceDate	datetime or integer	Invoice Date	Sales Order																				
  + 日付 (H): Date: Ngày xuất hoá đơn đòi tiền ở trên Kintone, DR lên Kintone xuất giấy hoá đơn đòi tiền và gửi email cho khách, nd trong file gửi mail có ngày cụ thể để trả tiền cho FCJ																									
納品OK日: Delivery OK date		deliveryOKDate	datetime or integer	Delivery OK Date                	"Sales Order (copy field từ Project)
Đây là mốc hoàn thành delivery"																				
"  + 日付 (I): Date: Sau khi giao hàng cho khách xong, khách gửi lại cho mình 1 câu: giao hàng OK, data giao hàng không vấn đề, KH chấp nhận data giao hàng. 
      Sau khi KH nói câu này xong thì mới được coi là đơn này giao hàng thành công. Confirm này chia ra làm 2, đối với những đơn trên 100k Yên trở lên trước Thuế thì DR chup hình lại email mà KH thông báo OK 
      gửi lên Kintone báo cáo lại cho FCJ, những đơn trước thuế dưới 100k Yên không cần capche. Đôi khi không quan trọng bởi vì thường KH nhận data xong thì không chịu confirm lại mặc dù đã OK. 
      Chốt lại là đơn trên 100k Yên thì DR mới đòi khách bằng được email xác nhận giao hàng thành công. Còn đơn dưới 100k tới giao hàng xong là cho DONE luôn. Nên setting ko bắt buộc phải điền"																									
"担当者 (J): Contact person:
     Team DR phụ trách
     (list chọn người trong Team DR)"		contactPerson	User/Picklist	Contact Persion	Sales Order																				
ステータス (K): Status		defaultStatus	varchar(255)	Default Status	Sales Order																				
Sử dụng Status của Sales Order		status	s/d của vtiger	Status	Sales Order																				
会社名 (L): Company name: ghi tên KH		contactName	s/d của vtiger	Contact Name	Sales Order																				
"案件名 (M): Project name: tên đơn hàng
 (một KH có thể có nhiều đơn.
 Ví dụ khách GENOVA様 mỗi tháng có 
vài chục tên đơn hàng hay còn gọi là task)"		subject	s/d của vtiger	"Subject 
(change lại name là:
 Project name)"	Sales Order																				
"提案金額 (N): Proposed amount
  Giá đã báo cho khách, nghĩa là giá bán.
  Đây là giá thực tế DR báo cho khách
  (giá sau thuế)."		salesAmount	Currency	Sales Amount	Sales Order																				
入金額 (O): Payment amount		actualPaymentAmount	Currency	Actual Payment Amount	Sales Order																				
"  + Show giá tiền mà khách đã trả tiền. DR nhập vào cột F là ngày khách trả tiền và cột O này tiền thực tế mà khách đã trả, lý do tách ra là có khách trả tiền bị thiếu hoặc dư so với giá mà mình đã báo cho khách,
     có thể lúc chuyển khoản tính thêm phí chuyển khoản vào nên bị xê dịch ó với cột N. Nếu giá khác nhau so với N thì báo ngay FCJ để đi confirm với Khách để khách kiểm tra lại. 
     Giúp cho DR double check lại xem có đúng với giá báo khách hay chưa"																									
Từ Côt P trở đi mục đích kiểm tra công số của đơn hàng:																									
見積工数 (P): Estimated labor hours		estimatedLaborHours	Decimal	Estimated labor hours	Sales Order (copy field từ Project)																				
"  + Công số bán mà DR báo giá cho khách. công số cột Q coder báo + 10h check và fix QC + 10h rủi ro vì đơn khó hay vấn đề khác đề phòng trường hợp coder cần thêm công số trong quá trình làm nên có dư ra để bù đắp vào. 
     Nên công số bán cho khách dôi lên nhiều so với công số thực tế mà coder báo."																									
実工数 (Q): Actual labor hours		actualLaborHours	Decimal	Actual labor hours	Sales Order (copy field từ Project)																				
  + Công số thực tế của đơn. Cho phép nhập, nó chỉ mang tính chất án chừng thôi (bởi vì coder chưa làm đơn). Ví dụ coder báo 20h + 3h QC test + 3h coder fix lại.																									
原価単価 (R): Cost price		costPerHour	Currency	Cost Per Hour	Sales Order (copy field từ Project)																				
  + Đơn vị giá vốn (chi phí gốc 1000 yên/h). 1 tiếng FCJ mất chi phí làm 1000 Yên																									
販売単価 (S): Sales price		salesPricePerHour	Currency	Sales Price Per Hour	Sales Order																				
  + Đơn vị giá bán cho khách. 1 tiếng bán cho khách 1500 Yên, lãi được 500 Yên																									
原価合計 (T): Total cost		totalCost	Currency/Formula	Total Cost	Sales Order (copy field từ Project)																				
  + Tổng giá tiền vốn = giá đơn vị vốn x công số vốn (RxQ)																									
割引 (U): Discount		discount	?	Discount	Sales Order																				
"  + % giảm giá cho khách, hay chạy chương trình giảm giá, ví dụ giảm 30%, tự tính tay: lấy giá tiền bán đúng - 30% rồi sau đó + 10% thuế là ra k/q cột N. 
     Cải tiến chổ này, input vào cột U % giảm giá thì tự tính ra k/q cột N, N hiện tại là giá sau thuế, nếu discount thì lấy giá trước thuế - % giảm giá rồi sau đó + 10% VAT"																									
粗利 (V): Gross profit		grossProfit	Currency/Formula	Gross Profit	Sales Order																				
  + Tính lãi, giá bán cho khách - tổng tiền vốn (N-T)																									
"入金＆納品OK (W): Payment & delivery OK
 Điều kiện để 1 đơn được tính là xong: 
  1. Khách hàng đã trả tiền (Cột F: có ngày cụ thể)
  2. Cột O có giá trị thực tế đúng với cột N
  3. Đơn trên 100k Yên cột I bắt buộc có ngày cụ thể
 Auto cột này chuyển status DONE và filter ẩn luôn đỡ rối"		paymentDeliveryOK	picklist	Payment & Delivery OK	Sales Order																				
顧客タイプ (X): Customer type		customerType	picklist	Customer Type	Sales Order																				
"  + Loại KH. có 2 loại:
     新規客: mới (KH gửi đơn hàng lần đầu)
     既存客: cũ (đơn lần 2 trở đi)"																									
"QCチェックシートURL (Y): QC check sheet URL
 (Đính kèm sheet QC của đơn đó vào đây,
  đa phần ko s/d)"		QCCheckSheet	URL	QC Check Sheet URL	"Sales Order (copy field từ Project)
hoặc Documents attach vào Project"																				
"備考 (Z): Note
  + Chú thích có gì đặt biệt thì ghi vào"		orderNote	varchar(255)	Order Note	Sales Order																				
Order Type		orderType	picklist	Order Type	Sales Order																				
"  + Có 2 loại:
     Outsourcing
     FCJ"																									
																									
*** Lưu ý:																									
 Màu đỏ: tất cả những đơn chưa giao hàng, or giao hàng khách chưa chốt, chưa trả tiền (báo động đỏ). Ví dụ đơn nợ lâu lắm (quá khứ). nhiều thứ ko phải chỉ tiền không																									
 Màu tím: đơn cần đươc trả tiền trong tháng này. Ví dụ trả tiền trong tháng hiện tại. Chỉ là hạn trả tiền. Màu tím tháng này chưa trả tiền qua tháng sau mà chưa trả thì chuyển sang màu đỏ																									
 Màu vàng: đơn có hạn trả tiền từ tháng sau trở đi (tương lai). Chỉ là hạn trả tiền																									
																									
Đơn hàng đầu tiên																									
Leads → Contacts/Organizations → Opportunity (Deal) 1(Quote 1) → Sales Order 1 (chính là task 1 trong sheet List Order)																									
Đơn hàng thứ 2																									
Leads → Contacts/Organizations → Opportunity(Deal) 2(Quote 2) → Sales Order 2 (chính là task 2 trong sheet List Order)																									
Đơn hàng thứ 3 (change request của đơn 3)																									
Leads → Contacts/Organizations → Opportunity 3(Quote 3) → Sales Order 3 (chính là task 3 trong sheet List Order)																									
																									
3. Sheet management																									
																									
NO. (Cột B): Order Number																									
" + Leader và Coder quan tâm order number
   để assign và làm việc chung với nhau"																									
ORDER NAME (Cột C)																									
TYPE1 (Cột D): kiểu picklist																									
"  PC, SP, Responsive, Responsive(FCV-Design),
  Responsive(Mobile First), Blog, Design, Trial, WordPress,
  WordPress+Responsive, LP, Gtag, SSL"																									
TYPE2 (Cột E): kiểu picklist																									
  + NEW: là order mới của FCJ																									
  + 外注: là order mới của outsourcing (Team Web Director)																									
  + Design: là order mới của bộ phận design																									
STANDARD COST (FC) (Cột F)																									
 + Nếu tồn tại Blog custom/Email Form																									
PAGE (Cột G): số page khách cần làm																									
STANDARD COST (Cột H)																									
" + Công số chuẩn sẽ được auto tính toán bằng hàm mặc định từ trước,
    (sẽ thay đổi sau khi Estimator Sủng/Tiến estimate công số).
    Sau khi có k/q thì comtor update lại."																									
ORDER DATE (Cột I): Data Delivery Date																									
 + Ngày khách giao design data																									
DELIVERY DATE (Cột J):																									
 + Ngày giao hàng mong muốn của khách																									
REAL DELIVERED DATE (Cột L)																									
" + Ngày giao hàng thật sự cho khách.
    Sẽ được update khi status (Cột M) 完了 (done)"																									
STATUS (Cột M)																									


<!-- WORKFLOW -->
															
🔵 PHASE 1 — INQUIRY (LEAD STATUS)															
															
KỊCH BẢN 0: Tạo Lead				WORKFLOW 0.1 — Nhắc nếu Not Contacted quá 24h.											
 Bối cảnh				 Type: Scheduled Workflow											
  Lead mới tạo từ Cold Call / Webform				 Run: Daily											
				 Condition:											
 1️⃣ Lead *				  + Lead Status = Not Contacted											
  Nhập đầy đủ: Name, Phone, Email, Source				  + Customer Date IS NOT EMPTY											
  Gán Assigned To: Sales				  + Today ≥ Customer Date + 1 day											
  Lead Status: Not Contacted				  + Lead Reminder Sent = false (tránh tạo task trùng)											
  Note: Mới tạo, chưa xử lý (chưa Call)				 Action (auto):											
  Customer Date: Ngày khách gửi yêu cầu đầu tiên				  1. Create Task:											
  Update field (responseStatus): tình trạng đối ứng				   + Subject: “Follow-up Lead – Not Contacted > 24h”											
				   + Assigned to: Lead Owner (Sales)											
 2️⃣ Task *				   + Task Type: Call											
  Tạo Task: Call (Email) trong vòng 24h				   + Priority: High											
  Task Status: Pending				  2. Update Lead:											
  Task Type: Call				   + Lead Reminder Sent = true											
				  3. Send Email to Lead Owner: [FCJ] Lead chưa xử lý sau 24h											
KỊCH BẢN 1: Sales (W.D) có call K/H nhưng chưa liên hệ được ***															
 Bối cảnh				WORKFLOW 1.1 — Nhắc K/H nếu status: Attempted to Contact quá 48h.											
  Sales gọi K/H chưa bắt máy				 Type: Scheduled Workflow											
				 Run: Daily											
 1️⃣ Lead *				 Condition:											
  Lead Status: Attempted to Contact				  + Lead Status = Attempted to Contact											
  Update (Note): “Gọi ngày 6/2 – K/H chưa bắt máy”				  + Today ≥ Modified Time + 2 days											
  Update field (responseStatus): tình trạng đối ứng				 Action (auto):											
				  1. Create Task:											
 2️⃣ Task *				   + Subject: “Follow-up Call again – Attempted to Contact > 48h”											
  Tạo Task: Call lại K/H trong vòng 2 days				   + Assigned to: Lead Owner (Sales)											
  Task Status: Open				   + Task Type: Call											
  Task Type: Call				   + Priority: High											
  Due date: Sau 2 ngày khi call lần đầu không bắt máy				  2. Send Email to Lead Owner: [FCJ] Lead Call lại K/H sau 48h											
															
 3️⃣ Các module khác															
  Event ❌ (chưa có hẹn)															
  Deal ❌															
  Quote ❌															
															
KỊCH BẢN 2: Sales (W.D) gọi được K/H – khách hỏi sơ bộ, chưa hẹn meeting															
 Bối cảnh															
  Sales gọi được K/H															
  Khách hỏi sơ qua dịch vụ nhưng chưa sẵn sàng có cuộc họp															
															
 1️⃣ Lead *															
  Lead Status: Contacted															
  Update (Note): “Khách hỏi dịch vụ, cần trả lời thêm từ K/H”															
  Update field (FCJ Response Date): "Ngày bên FCJ gửi mail trả lời đầu tiên cho khách"															
  Update field (responseStatus): tình trạng đối ứng															
															
 2️⃣ Task															
  Tạo Task: Follow-up waiting reply Customer															
  Task Status: Open															
  Task Type: Follow-up															
															
 3️⃣ Documents (optional)															
  Upload: Company Profile.pdf															
  Link document với Lead															
  👉 Vì: sau này Manager vào biết đã gửi gì															
															
 4️⃣ SMS Message (optional)															
  SMS: "Em (Sales) vừa gửi profile qua email, anh check giúp em nhé"															
  👉 Vì: tăng tỉ lệ khách mở mail															
															
 ❌ Deal / Event															
  Chưa tạo															
															
🟡 PHASE 2 — MEETING & QUALIFICATION															
															
KỊCH BẢN 3: Chat trực tiếp với K/H để setup buổi meeting (sau khi call được K/H hỏi sơ bộ)															
 Bối cảnh															
  Khách chat trực tiếp															
  Quan tâm rõ dịch vụ															
															
 1️⃣ Live Chat (optional)															
  Live Chat Status: Completed															
  Nội dung: “Cần meeting + báo giá”															
															
 2️⃣ Lead *															
  Lead Status: In Progress															
  Update (Note): “Khách chủ động chat, nêu rõ nhu cầu”															
  Update meetingDate (MTG): "Ngày Khách yêu cầu meeting"															
  Update field (responseStatus): tình trạng đối ứng															
															
 3️⃣ Task *															
  Task: Setup meeting online															
  Status: Open															
															
 4️⃣ Event (Meeting) *															
  Event: Meeting online															
  Event Status: Planned (đã lên kế hoạch)															
  Thời gian: 8/2 10:00															
															
 → 3.1: W.D (TrangLK) gửi email confirm nếu chưa clear (nếu đã rõ yêu cầu từ k/h thì bỏ qua bước 3.1)															
 Bối cảnh															
  Khách trả lời nhưng:															
   + Nội dung chưa rõ															
   + Thiếu file															
   + Chưa rõ yêu cầu															
															
 1️⃣ Lead *															
  Lead Status: In Progress															
  Update field (FCV Response Date): "Ngày W.D (TrangLK) gửi email confirm cho khách nếu chưa clear"															
  Update field (responseStatus): tình trạng đối ứng															
															
 2️⃣ Task *															
  Task: W.D (TrangLK) gửi mail confirm															
  Status: Open															
															
 3️⃣ SMS Message															
  SMS: "Em (W.D) vừa gửi confirm yêu cầu qua email, anh check giúp em nhé"															
  👉 Mục đích: tăng tỉ lệ khách mở mail															
															
KỊCH BẢN 4: Kết quả sau khi meeting															
 → 4.1: Không tiềm năng															
 Bối cảnh															
  Đã meeting xong															
  Khách hàng không làm hoặc báo làm với công ty khác, hoặc không đủ budget															
															
 1️⃣ Event															
  Event Status: Held (đã được tổ chức)															
															
 2️⃣ Lead *															
  Lead Status: Lost Lead (Kết thúc mà không convert Lead)															
  Update (Note): “lý do thất bại là KH nhờ cty khác làm rồi”															
  Update field (responseStatus): tình trạng đối ứng															
															
 → 4.2: Xác định K/H tiềm năng → Convert Lead			WORKFLOW 4.3 — Change Lead Status: Qualified (ưu tiên sau)				WORKFLOW 4.4 — Thực hiện manual convert Lead ( auto check Deal ) 								
 Bối cảnh			 Trigger: Lead Status = Qualified and submit btn Save				 Trigger: Thực hiện convert manual Lead (click btn Convert Lead)								
  Đã meeting xong			 Action (auto convert Lead):				 Condition: Lead Status != Qualified								
  Khách hàng có budget			  1. Create Contact (popup confirm)				 Action (auto):								
			  2. Create Organization (popup confirm)				  1. Lead Status = Qualified								
 1️⃣ Event			  3. Create Deal (popup confirm)				  2. Notifi slack channel								
  Event Status: Held (đã được tổ chức)			  4. Notifi slack channel				  3. Notify to hệ thống								
			  5. Notify to hệ thống												
 2️⃣ Task *															
  Task: Update Meeting Outcome															
  Status: Open															
  