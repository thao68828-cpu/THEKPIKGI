function khoiTaoLichSanXuatSOP() {
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var sheet = ss.getActiveSheet();
  
  // 1. Xóa sạch dữ liệu và định dạng cũ để làm mới hoàn toàn
  sheet.clear();
  sheet.clearConditionalFormatRules();
  
  // 2. Định nghĩa cấu trúc tiêu đề chuẩn theo SOP
  var headers = [
    "Nền tảng", "Topic", "Dạng kịch bản", 
    "A1 Research", "Deadline", 
    "A2 + A3 Viết + Broll", "Deadline", 
    "A4 QC", "Deadline", 
    "A5 + A6 Quay + Đẩy source", "Deadline", 
    "A7 Edit", "Deadline", 
    "A8 QC cuối + Air", "Deadline"
  ];
  
  // 3. Chuẩn bị mảng dữ liệu mẫu mô phỏng theo ảnh khách hàng cung cấp
  var database = [
    // HÀNG 1: Tiêu đề chính
    headers,
    
    // KHỐI TIKTOK (Dòng 2 -> 13): Nền trắng/mặc định
    ["Tiktok", "KỊCH BẢN TIKTOK: \"BÁC VƯƠNG CHO TIỀN\" & EM VF3", "New", "Khang", "2026-03-27", "Khang", "2026-03-24", "Khang", "2026-03-30", "Khang + Trang", "2026-04-01", "Freelancer", "2026-03-31", "Khang", "2026-03-31"],
    ["Tiktok", "GIẢI MÃ CÁ \"KHOAI SẢN\" 4 TỶ", "Case Study", "Khang", "2026-03-27", "Khang", "2026-03-24", "Khang", "2026-03-30", "Khang + Trang", "2026-04-01", "Freelancer", "2026-04-01", "Khang", "2026-04-01"],
    ["Tiktok", "HỖ TRỢ KH KH VAY 5 TỶ PHÊ DUYỆT HDBANK", "Case Study", "Khang", "2026-03-24", "Khang", "2026-03-23", "Khang", "2026-03-30", "Khang + Trang", "2026-04-01", "Freelancer", "2026-04-01", "Khang", "2026-04-01"],
    ["Tiktok", "LÀM THẾ NÀO ĐỂ TỐI ƯU HÓA TIỀN GỬI TIẾT KIỆM", "New", "Khang", "2026-03-25", "Khang", "2026-03-25", "Khang", "2026-03-30", "Khang + Trang", "2026-04-01", "Freelancer", "2026-04-02", "Khang", "2026-04-02"],
    ["Tiktok", "CÓ NÊN CHUYỂN HỒ SƠ VAY TẠI THỜI ĐIỂM LÃI VAY NHƯ NÀY KHÔNG?", "New", "Khang", "2026-03-25", "Khang", "2026-03-26", "Khang", "2026-03-30", "Khang + Trang", "2026-04-01", "Freelancer", "2026-04-02", "Khang", "2026-04-04"],
    ["Tiktok", "LÃI SUẤT BĐS ĐANG GIẢM XUỐNG LIỆU CÓ ĐÚNG SỰ THẬT?", "New", "Khang", "2026-03-24", "Khang", "2026-03-27", "Khang", "2026-03-30", "Khang + Trang", "2026-04-01", "Freelancer", "2026-04-02", "Khang", "2026-04-04"],
    ["Tiktok", "Lấy từ tiktok", "Reel", "x", "", "x", "", "x", "", "x", "", "x", "", "x", "2026-03-31"],
    ["Tiktok", "Lấy từ tiktok", "Reel", "x", "", "x", "", "x", "", "x", "", "x", "", "x", "2026-03-31"],
    ["Tiktok", "Lấy từ tiktok", "Reel", "x", "", "x", "", "x", "", "x", "", "x", "", "x", "2026-04-01"],
    ["Tiktok", "Lấy từ tiktok", "Reel", "x", "", "x", "", "x", "", "x", "", "x", "", "x", "2026-04-02"],
    ["Tiktok", "Lấy từ tiktok", "Reel", "x", "", "x", "", "x", "", "x", "", "x", "", "x", "2026-04-03"],
    ["Tiktok", "Lấy từ tiktok", "Reel", "x", "", "x", "", "x", "", "x", "", "x", "", "x", "2026-04-04"],
    
    // KHỐI YOUTUBE (Dòng 14 -> 22): Nền màu hồng đỏ nhạt
    ["Youtube", "HƯỚNG DẪN CÁCH LẤY TIỀN TỪ NGÂN HÀNG", "Kịch bản", "Khang", "2026-03-23", "KHANG", "2026-03-26", "Team + BOD", "2026-03-26", "Khang + Trang", "", "x", "", "x", ""],
    ["Youtube", "LÀM THẾ NÀO ĐỂ TRÁNH LÃI SUẤT THẢ NỔI CAO", "Kịch bản", "Khang", "2026-03-23", "KHANG", "2026-03-26", "Team + BOD", "2026-03-26", "Khang + Trang", "", "x", "", "x", ""],
    ["Youtube", "Lấy từ tiktok", "Reel", "x", "", "x", "", "x", "", "x", "", "x", "", "Huyền", "2026-03-23"],
    ["Youtube", "Lấy từ tiktok", "Reel", "x", "", "x", "", "x", "", "x", "", "x", "", "Huyền", "2026-03-24"],
    ["Youtube", "Lấy từ tiktok", "Reel", "x", "", "x", "", "x", "", "x", "", "x", "", "Huyền", "2026-03-25"],
    ["Youtube", "Lấy từ tiktok", "Reel", "x", "", "x", "", "x", "", "x", "", "x", "", "Huyền", "2026-03-26"],
    ["Youtube", "Lấy từ tiktok", "Reel", "x", "", "x", "", "x", "", "x", "", "x", "", "Huyền", "2026-03-27"],
    ["Youtube", "Lấy từ tiktok", "Reel", "x", "", "x", "", "x", "", "x", "", "x", "", "Huyền", "2026-03-28"],
    ["Youtube", "Lấy từ tiktok", "Reel", "x", "", "x", "", "x", "", "x", "", "x", "", "Huyền", "2026-03-29"],
    
    // HÀNG 23: Lặp lại thanh tiêu đề phân tách Khối Facebook (Màu vàng)
    headers,
    
    // KHỐI FACEBOOK (Dòng 24 -> 34): Nền màu xanh dương nhạt
    ["Facebook", "Cập nhật lãi suất theo tiến độ", "Carousel", "Khang", "2026-03-23", "Huyền", "2026-03-24", "Khang", "2026-03-25", "Khang", "2026-03-25", "x", "2026-03-30", "Khang", ""],
    ["Facebook", "Cập nhật thông tin tuyển dụng", "Essay", "Khang", "2026-03-23", "Tuyết Anh", "2026-03-24", "Khang", "2026-03-25", "Khang", "2026-03-25", "x", "2026-03-31", "Khang", ""],
    ["Facebook", "Thông tin phê duyệt khoản vay", "Carousel", "Khang", "2026-03-23", "Tuyết Anh", "2026-03-25", "Khang", "2026-03-26", "Khang", "2026-03-26", "x", "2026-04-01", "Khang", ""],
    ["Facebook", "Tại sao Vin group lại ra lãi suất 9%", "Essay", "Khang", "2026-03-23", "Tuyết Anh", "2026-03-25", "Khang", "2026-03-26", "Khang", "2026-03-26", "x", "2026-04-02", "Khang", ""],
    ["Facebook", "Lấy từ tiktok", "Reel", "x", "", "x", "", "x", "", "x", "", "x", "", "x", ""],
    ["Facebook", "Lấy từ tiktok", "Reel", "x", "", "x", "", "x", "", "x", "", "x", "", "x", ""],
    ["Facebook", "Lấy từ tiktok", "Reel", "x", "", "x", "", "x", "", "x", "", "x", "", "x", ""],
    ["Facebook", "Lấy từ tiktok", "Reel", "x", "", "x", "", "x", "", "x", "", "x", "", "x", ""],
    ["Facebook", "Lấy từ tiktok", "Reel", "x", "", "x", "", "x", "", "x", "", "x", "", "x", ""],
    ["Facebook", "Lấy từ tiktok", "Reel", "x", "", "x", "", "x", "", "x", "", "x", "", "x", ""],
    ["Facebook", "Lấy từ tiktok", "Reel", "x", "", "x", "", "x", "", "x", "", "x", "", "x", ""]
  ];
  
  // 4. Ghi toàn bộ mảng dữ liệu vào Sheet bắt đầu từ ô A1
  var totalRows = database.length;
  var totalCols = headers.length;
  sheet.getRange(1, 1, totalRows, totalCols).setValues(database);
  
  // 5. Đổ màu nền phân vùng hệ thống (Color Block Styling)
  var yellowColor = "#FFF2CC"; // Màu vàng chanh sáng cho tiêu đề chính
  var pinkColor = "#FCE4D6";   // Màu hồng nhạt cho khối Youtube
  var blueColor = "#DDEBF7";   // Màu xanh lam nhạt cho khối Facebook
  
  // Đổ màu hàng tiêu đề số 1 và hàng tiêu đề lặp lại số 23
  sheet.getRange(1, 1, 1, totalCols).setBackground(yellowColor).setFontWeight("bold").setHorizontalAlignment("center");
  sheet.getRange(23, 1, 1, totalCols).setBackground(yellowColor).setFontWeight("bold").setHorizontalAlignment("center");
  
  // Đổ màu khối nội dung Youtube (Dòng 14 đến 22)
  sheet.getRange(14, 1, 9, totalCols).setBackground(pinkColor);
  
  // Đổ màu khối nội dung Facebook (Dòng 24 đến 34)
  sheet.getRange(24, 1, 11, totalCols).setBackground(blueColor);
  
  // 6. Định dạng Font chữ, kích thước và đường viền (Borders) cho toàn bảng
  var fullRange = sheet.getRange(1, 1, totalRows, totalCols);
  fullRange.setFontFamily("Arial").setFontSize(9);
  fullRange.setBorder(true, true, true, true, true, true, "#D3D3D3", SpreadsheetApp.BorderStyle.SOLID);
  
  // Định dạng các cột chứa ngày Deadline (Cột 5, 7, 9, 11, 13, 15) sang chuẩn Ngày Tháng và căn giữa
  var deadlineColumns = [5, 7, 9, 11, 13, 15];
  deadlineColumns.forEach(function(colIndex) {
    sheet.getRange(2, colIndex, totalRows - 1, 1).setNumberFormat("dd/mm/yyyy").setHorizontalAlignment("center");
  });
  
  // Căn giữa cột Nền tảng (Cột 1) và cột Dạng kịch bản (Cột 3)
  sheet.getRange(2, 1, totalRows - 1, 1).setHorizontalAlignment("center");
  sheet.getRange(2, 3, totalRows - 1, 1).setHorizontalAlignment("center");
  
  // Tự động căn chỉnh độ rộng của tất cả các cột cho vừa vặn chữ
  for (var i = 1; i <= totalCols; i++) {
    sheet.autoResizeColumn(i);
  }
  
  // 7. THIẾT LẬP LUẬT ĐỔ MÀU HIGHLIGHT TỰ ĐỘNG (Conditional Formatting)
  var rules = sheet.getConditionalFormattingRules();
  
  // Luật kiểm soát bài Edit trễ hạn (Cột M - Cột số 13): Nếu nhỏ hơn ngày hôm nay thì tự động TÔ ĐỎ ô deadline
  var lateEditRule = SpreadsheetApp.newConditionalFormattingRule()
      .whenFormulaSatisfied('=AND(ISNUMBER($M1), $M1<TODAY())')
      .setBackground("#F8CECC")
      .setFontColor("#C00000")
      .setBold(true)
      .setRanges([sheet.getRange(2, 13, totalRows - 1, 1)])
      .build();
  rules.push(lateEditRule);
  
  // Áp dụng bộ luật vào Sheet
  sheet.setConditionalFormattingRules(rules);
}
