VTIGER 8 CHO DEVELOPER
Tài liệu nguồn chính thức, cách build và mô tả type dữ liệu
Bản tổng hợp ngắn gọn theo góc nhìn dev — ưu tiên nguồn chính thức của Vtiger.
Ngày tổng hợp: 12/03/2026
Tài liệu này gom 3 lớp nguồn bạn nên dùng song song: 1) Community Developer Guide/vtlib để custom module bằng PHP-vtlib, 2) Help Center/Knowledge Base để làm REST API, Module Designer, VTAP trên bản cloud, 3) GitLab source để đối chiếu code thực tế khi docs thiếu hoặc còn cũ. [S1][S2][S3][S8][S9][S10]

1. Mục tiêu tài liệu
Tài liệu này nhằm giúp dev hoặc tech lead có một “source map” rõ ràng khi làm Vtiger 8: đọc tài liệu nào trước, build local ra sao, package module theo cấu trúc nào, và phải hiểu thế nào về ba lớp mô tả field gồm columntype, uitype và typeofdata. [S1][S2][S4][S5][S6]
2. Bản đồ nguồn chính thức
Bảng dưới đây là bộ nguồn tối thiểu mình khuyên giữ cạnh IDE. Cột “Ưu tiên” phản ánh mức hữu dụng cho dev custom/self-host trước, rồi mới tới bản cloud/VTAP. [S1][S2][S3][S8][S9][S10]
Mã nguồn	Tài liệu	Dùng để làm gì	Ưu tiên	Domain
S1	Develop Extensions For Vtiger	Trang index cho toàn bộ developer guide: extensions, vtlib, examples, internals.	Rất cao	community.vtiger.com
S2	Module Field	Đọc khi làm field metadata: table/column/columntype/uitype/typeofdata/picklist/relation.	Rất cao	community.vtiger.com
S3	Package Structure	Chuẩn thư mục ZIP cho entity module, extension module, language/layout/theme pack.	Rất cao	community.vtiger.com
S4	Console Tool	CLI để create/import/update/remove module.	Cao	community.vtiger.com
S5	Entity Module Example	Ví dụ bootstrap module chuẩn bằng vtlib.	Cao	community.vtiger.com
S6	Server APIs	Tầng API server có xét sharing/profile/workflow; nên ưu tiên hơn query DB trực tiếp.	Cao	community.vtiger.com
S7	Installation	Yêu cầu môi trường local/self-host.	Cao	community.vtiger.com
S8	Rest API Manual	REST API hiện tại cho cloud/modern integration; có Describe để lấy metadata field/module.	Cao	help.vtiger.com
S9	What is the Vtiger Module Designer?	Mô tả Module Designer/VTAP trong CRM cloud.	Trung bình	help.vtiger.com
S10	Who can use the Module Designer?	Xác định edition hỗ trợ Module Designer.	Trung bình	help.vtiger.com
S11	vtigercrm GitLab Source	Nguồn code để đối chiếu implementation thật khi docs còn thiếu/cũ.	Rất cao	code.vtiger.com
S12	Module Link	Ví dụ tài liệu còn ghi “REVIEW for Vtiger 8.0”, nhắc rằng phải test trên source/version thực tế.	Tham khảo	community.vtiger.com

3. Cách đọc bộ docs theo đúng thứ tự
3.1 Bắt đầu từ trang index developer guide
Trang “Develop Extensions For Vtiger” là điểm vào tốt nhất vì nó liệt kê đầy đủ Extension Types, Development Essentials, vtlib, Examples và Internals. Đây là “bản đồ điều hướng”, không phải nơi chứa mọi chi tiết. [S1]
3.2 Nếu bạn build module PHP-vtlib
Hãy đọc ngay theo thứ tự: Package Structure → Entity Module Example → Module Field → Console Tool → Server APIs. Bộ này đủ để dựng module, thêm field, đóng gói ZIP và import/update bằng CLI. [S2][S3][S4][S5][S6]
3.3 Nếu bạn làm integration hoặc cloud customization
Ngoài bộ community docs, bạn cần thêm Rest API Manual để gọi API hiện đại và dùng Describe để đọc metadata module/field, cùng Module Designer/VTAP nếu đang custom trên bản One/Cloud. [S8][S9][S10]
4. Build và dựng môi trường
4.1 Dựng local/self-host
Trang Installation ghi Vtiger CRM là web application viết bằng PHP; pre-requisites gồm Apache 2.1+, MySQL 5.1+ (InnoDB, local_infile=ON, sql_mode phù hợp), PHP 8.0+ cùng php-imap, php-curl, php-xml; memory_limit tối thiểu 256MB, max_execution_time tối thiểu 60 giây; sau đó unzip vào web root, cấp quyền ghi, và chạy wizard qua index.php. [S7]
4.2 Build skeleton/module bằng CLI
php -f vtlib/tools/console.php
Console Tool cho phép: Create New Module, Create New Layout, Create New Language Pack, Import Module, Update Module, Remove Module; đồng thời có shortcut như --import, --update, --remove. [S4]
4.3 Build bằng bootstrap script vtlib
Entity Module Example minh họa flow chuẩn: kiểm tra module đã tồn tại hay chưa, tạo Vtiger_Module, save(), initTables(), addBlock(), addField(), setEntityIdentifier(), rồi tiếp tục thêm các field còn lại. Đây là ví dụ thực chiến đáng đọc nhất trước khi tự viết module mới. [S5]
$moduleInstance = new Vtiger_Module();
$moduleInstance->name = $MODULENAME;
$moduleInstance->parent = 'Tools';
$moduleInstance->save();
$moduleInstance->initTables();
4.4 Build package ZIP để deploy/import
Package Structure mô tả rõ ZIP của entity module phải có tối thiểu manifest.xml và thư mục modules/<MODULENAME>/; ngoài ra có thể kèm crons, templates, resources, languages, settings... Extension module có cấu trúc tương tự nhưng không nhất thiết có đầy đủ phần entity-specific. [S3]
5. Mô tả type dữ liệu: columntype, uitype, typeofdata
Đây là phần quan trọng nhất khi custom field trong Vtiger. Một field thường có 3 lớp mô tả khác nhau, và bạn không nên nhầm lẫn chúng với nhau. [S2][S5][S8]
Thuộc tính	Ý nghĩa	Ví dụ	Ghi chú dev
columntype	Kiểu cột ở database.	VARCHAR(100), VARCHAR(255), Date	Nếu không set thì Module Field ghi mặc định là VARCHAR(255). [S2]
uitype	Kiểu hiển thị/hành vi ở UI.	1, 2, 5, 10, 15, 19, 53, 71	Nếu không set thì mặc định là 1 theo Module Field. [S2]
typeofdata	Chuỗi validation + required/optional.	V~M, V~O, D~O	Nếu không set thì mặc định là V~O. [S2]
Describe API	Metadata runtime của module/field qua REST.	mandatory, quickcreate, type.name, refersTo, picklistValues	Rất hữu ích để inspect field mà không đoán tay. [S8]
5.1 Defaults và ví dụ chính thức
Trang Module Field đưa ví dụ với name/table/column/columntype/uitype/typeofdata và ghi rõ các default: table = base table của module; column = tên field viết thường; columntype = VARCHAR(255); uitype = 1; typeofdata = V~O. [S2]
$fieldInstance->name = 'PayslipName';
$fieldInstance->table = 'vtiger_payslip';
$fieldInstance->column = 'payslipname';
$fieldInstance->columntype = 'VARCHAR(100)';
$fieldInstance->uitype = 2;
$fieldInstance->typeofdata = 'V~M';
5.2 Cách đọc nhanh typeofdata
Từ ví dụ chính thức có thể hiểu theo thực hành: V~M là chuỗi bắt buộc, V~O là chuỗi không bắt buộc, D~O là ngày không bắt buộc. Entity Module Example cũng dùng Date + D~O cho field ngày. [S2][S5]
5.3 uitype không tự quyết định kiểu DB
Entity Module Example cho thấy field Amount dùng uitype = 71 nhưng columntype vẫn là VARCHAR(255). Điều này nhắc rằng uitype là lớp UI/behavior, còn storage trong DB vẫn do bạn tự thiết kế qua columntype. [S5]
5.4 Các trường hợp thường gặp
• Picklist: Module Field nêu rõ các uitype 15, 16, 33, 55, 111 có thể set giá trị ban đầu bằng setPicklistValues(). [S2]
• Popup relation: uitype = 10, có thể setRelatedModules() để chỉ định module được chọn qua popup. [S2]
• Entity identifier: một field mandatory phải được set làm entity identifier để hiển thị label record. [S2][S5]
6. Dùng REST Describe để kiểm tra metadata runtime
Rest API Manual mô tả endpoint Describe trả về metadata của module gồm quyền thao tác (createable/updateable/deleteable/retrieveable), danh sách fields và từng thuộc tính như mandatory, quickcreate, summaryfield, default, type.name, length, refersTo, picklistValues, isunique, nullable, editable... Khi team cần map field giữa nhiều môi trường, đây là cách đáng tin hơn so với chỉ nhìn DB hoặc UI. [S8]
GET endpoint/describe?elementType=moduleName
7. Những lưu ý thực tế khi làm Vtiger 8
• Community docs vẫn là nguồn nền tảng rất tốt, nhưng nhiều trang hiển thị bản quyền 2022; vì vậy khi gặp chỗ không chắc, nên đối chiếu thêm source code GitLab. [S1][S11]
• Repo GitLab chính thức hiện hiển thị 3,708 commits, 17 branches, 20 tags; đây là nơi nên đọc code thực tế của implementation. [S11]
• Một số trang docs vẫn cảnh báo chưa rà soát xong cho Vtiger 8; ví dụ Module Link có dòng “REVIEW for Vtiger 8.0”. Điều này là tín hiệu rõ rằng bạn phải test trên version đang chạy của mình. [S12]
• Module Designer/VTAP thuộc nhánh cloud/One. FAQ và Knowledge Base hiện mô tả Module Designer là IDE trong CRM để tạo module, customize UI, component và script/style VTAP; đồng thời FAQ riêng cho biết tính năng này có trên One Professional, One Enterprise, One AI và cả Developer Edition để build marketplace extensions. [S9][S10]
8. Lộ trình đọc gợi ý cho dev mới vào dự án
Bước	Đọc gì	Kết quả mong muốn
1	Develop Extensions For Vtiger [S1]	Nắm được bản đồ docs.
2	Package Structure [S3]	Hiểu module ZIP phải có gì.
3	Entity Module Example [S5]	Biết bootstrap một entity module.
4	Module Field [S2]	Hiểu field metadata và validation.
5	Console Tool [S4]	Tạo/import/update module bằng CLI.
6	Server APIs + REST Describe [S6][S8]	Biết cách đọc metadata và thao tác dữ liệu an toàn hơn.
7	GitLab source [S11]	Đối chiếu code thật khi docs chưa đủ.
9. Phụ lục nguồn
Mỗi mã nguồn dưới đây tương ứng với ký hiệu [S1]...[S12] được dùng trong tài liệu. Mình giữ URL đầy đủ để bạn có thể share nội bộ hoặc đưa vào wiki dự án.
S1 — Develop Extensions For Vtiger: https://community.vtiger.com/help/vtigercrm/developers/develop-extensions-for-vtiger.html
S2 — Module Field: https://community.vtiger.com/help/vtigercrm/developers/vtlib/module-field.html
S3 — Package Structure: https://community.vtiger.com/help/vtigercrm/developers/package-structure.html
S4 — Console Tool: https://community.vtiger.com/help/vtigercrm/developers/vtlib/console-tool.html
S5 — An Entity Module Example: https://community.vtiger.com/help/vtigercrm/developers/extensions/examples/entity-module.html
S6 — Server APIs: https://community.vtiger.com/help/vtigercrm/developers/server-apis.html
S7 — Installation: https://community.vtiger.com/help/vtigercrm/administrators/installation.html
S8 — Rest API Manual: https://help.vtiger.com/article/147111249-Rest-API-Manual
S9 — What is the Vtiger Module Designer?: https://help.vtiger.com/faq/149967856-What-is-the-Vtiger-Module-Designer
S10 — Who can use the Module Designer?: https://help.vtiger.com/faq/164168969-Who-can-use-the-Module-Designer
S11 — vtigercrm GitLab Source: https://code.vtiger.com/vtiger/vtigercrm
S12 — Module Link: https://community.vtiger.com/help/vtigercrm/developers/vtlib/module-link.html

Gợi ý dùng tài liệu này: nếu team bạn đang build theo hướng self-host/PHP-vtlib thì bám S1–S7 + S11 trước; nếu đang đi theo bản cloud/One thì thêm S8–S10.

