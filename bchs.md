// ═══════════════════════════════════════════════════════════════
//  HỆ THỐNG BÁO CÁO HIỆU SUẤT NHÂN SỰ TOÀN DIỆN (FULL KỊCH BẢN MỚI)
// ═══════════════════════════════════════════════════════════════

var MEMBERS  = ['Vu Chi Khang', 'Ngo Thu Phuong', 'Le Phuong Thuy', 'Vu Dat', 'Hoang Van E'];
var CHANNELS = ['An Binh VayVon', 'ThuyVayVon', 'VayVonDonGianDat', 'KhangVayHay', 'Phantichtaichinhchuyensau', 'Suthattaichinh'];
var BROLL    = ['Video nền (stock footage)', 'Infographic / Motion', 'Ảnh minh họa', 'Icon / Element đồ họa', 'Âm thanh / Nhạc nền', 'Template chỉnh sửa'];

// BẠN CÓ THỂ TỰ ĐIỀN THÊM CÁC DẠNG KỊCH BẢN VÀO DANH SÁCH NÀY
var KBDANG   = ['Casestudy', 'New', 'Data driven', 'Ảnh Cuộn', 'Other'];

var WEEKS    = 4;
var WCOLORS  = ['#3B82F6','#10B981','#F59E0B','#EF4444'];
var PURPLE   = '#6C63FF';
var DARK     = '#374151';
var ALT      = '#F0F4FF';
var WHITE    = '#FFFFFF';

var DR = { cs: MEMBERS.length, vv: CHANNELS.length, br: BROLL.length };
var BS = { cs: MEMBERS.length + 4, vv: CHANNELS.length + 4, br: BROLL.length + 4 };

function onOpen() {
  SpreadsheetApp.getUi().createMenu('Báo Cáo')
    .addItem('1. Tạo/Cài đặt Sheet (Lần đầu)', 'setupSheets')
    .addSeparator()
    .addItem('2. Mở Dashboard', 'openDashboard')
    .addSeparator()
    .addItem('💾 3. Lưu trữ & Chốt sổ Tháng này', 'archiveMonth')
    .addToUi();
}

function setupSheets() {
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var names = ['Casestudy','Video & View','Kịch Bản','B-Roll', 'Tổng Hợp Tháng'];
  for (var i = 0; i < names.length; i++) {
    var old = ss.getSheetByName(names[i]);
    if (old) { try { ss.deleteSheet(old); } catch(e) {} }
  }
  Utilities.sleep(500);
  createCasestudySheet(); createVideoViewSheet(); createKichBanSheet(); createBRollSheet();
  SpreadsheetApp.flush();
  SpreadsheetApp.getUi().alert('Đã thiết lập xong! Dropdown Kịch Bản đã mở khóa, cho phép bạn tự do thêm Data.');
}

// ── 1. CÁC HÀM TẠO GIAO DIỆN SHEET ────────────────────────────────
function createKichBanSheet() {
  var ss = SpreadsheetApp.getActiveSpreadsheet(); var sh = ss.insertSheet('Kịch Bản');
  sh.setColumnWidth(1, 150); sh.setColumnWidth(2, 120); sh.setColumnWidth(3, 120); sh.setColumnWidth(4, 100);
  styleTitle(sh, 1, 4, 'THEO DÕI KỊCH BẢN THEO TUẦN', PURPLE);
  var r = 2;
  
  // TẠO DROPDOWN THÔNG MINH (Đã bật setAllowInvalid để user gõ tự do)
  var rule = SpreadsheetApp.newDataValidation()
    .requireValueInList(KBDANG)
    .setAllowInvalid(true) 
    .build();

  for (var w = 0; w < WEEKS; w++) {
    styleWeekHdr(sh, r, 4, 'TUẦN ' + (w + 1), WCOLORS[w]); r++;
    for (var c = 0; c < CHANNELS.length; c++) {
      // Dòng tên Kênh
      sh.getRange(r, 1).setValue('Kênh').setBackground(DARK).setFontColor(WHITE).setFontWeight('bold').setHorizontalAlignment('center').setVerticalAlignment('middle');
      sh.getRange(r, 2, 1, 3).merge().setValue(CHANNELS[c]).setBackground(DARK).setFontColor(WHITE).setFontWeight('bold').setHorizontalAlignment('center').setVerticalAlignment('middle');
      sh.setRowHeight(r, 35);
      r++;
      
      // Dòng Tiêu đề phụ
      styleSubHdr(sh, r, 4, ['Dạng Kịch Bản', 'KB Hoàn Thành', 'KB Mục Tiêu', '% Đạt']); r++;
      
      // Các dòng Dạng Kịch Bản
      var start = r;
      for (var k = 0; k < KBDANG.length; k++) {
        styleDataRow(sh, r, 4, WHITE);
        
        var cellA = sh.getRange(r, 1);
        cellA.setDataValidation(rule); // Gắn Dropdown
        cellA.setValue(KBDANG[k]).setFontWeight('bold');
        
        sh.getRange(r, 2).setValue(0).setHorizontalAlignment('center'); 
        sh.getRange(r, 3).setValue(0).setHorizontalAlignment('center');
        sh.getRange(r, 4).setFormula('=IFERROR(B'+r+'/C'+r+',0)').setNumberFormat('0%').setHorizontalAlignment('center').setFontColor('#059669').setFontWeight('bold');
        r++;
      }
      
      // Dòng Tổng của Kênh
      var end = r - 1;
      styleTotalRow(sh, r, 4, WCOLORS[w]);
      sh.getRange(r, 2).setFormula('=SUM(B'+start+':B'+end+')');
      sh.getRange(r, 3).setFormula('=SUM(C'+start+':C'+end+')');
      sh.getRange(r, 4).setFormula('=IFERROR(B'+r+'/C'+r+',0)').setNumberFormat('0%');
      r++;
    }
  }
}

function createCasestudySheet() {
  var ss = SpreadsheetApp.getActiveSpreadsheet(); var sh = ss.insertSheet('Casestudy');
  sh.setColumnWidth(1, 220); sh.setColumnWidth(2, 180); sh.setColumnWidth(3, 140); sh.setColumnWidth(4, 280);
  styleTitle(sh, 1, 4, 'THEO DÕI CASESTUDY THEO TUẦN', PURPLE);
  var r = 2;
  for (var w = 0; w < WEEKS; w++) {
    styleWeekHdr(sh, r, 4, 'TUẦN ' + (w + 1), WCOLORS[w]); r++;
    styleSubHdr(sh, r, 4, ['Thành Viên', 'Số Casestudy (Form 4F)', 'Tổng Lũy Kế', 'Note']); r++;
    var start = r;
    for (var m = 0; m < MEMBERS.length; m++) {
      styleDataRow(sh, r, 4, (m % 2 === 0) ? ALT : WHITE);
      sh.getRange(r, 1).setValue(MEMBERS[m]).setFontWeight('bold'); sh.getRange(r, 2).setValue(0); r++;
    }
    styleTotalRow(sh, r, 4, WCOLORS[w]); sh.getRange(r, 2).setFormula('=SUM(B'+start+':B'+(r-1)+')'); r += 2;
  }
}

function createVideoViewSheet() {
  var ss = SpreadsheetApp.getActiveSpreadsheet(); var sh = ss.insertSheet('Video & View');
  sh.setColumnWidth(1, 180); sh.setColumnWidth(2, 120); sh.setColumnWidth(3, 120); sh.setColumnWidth(4, 100); sh.setColumnWidth(5, 120); sh.setColumnWidth(6, 180);
  styleTitle(sh, 1, 6, 'THEO DÕI VIDEO & VIEW THEO TUẦN', PURPLE);
  var r = 2;
  for (var w = 0; w < WEEKS; w++) {
    styleWeekHdr(sh, r, 6, 'TUẦN ' + (w + 1), WCOLORS[w]); r++;
    styleSubHdr(sh, r, 6, ['Kênh', 'Video Hoàn Thành', 'Video Mục Tiêu', '% Đạt', 'Tổng View', 'Ghi Chú']); r++;
    var start = r;
    for (var c = 0; c < CHANNELS.length; c++) {
      styleDataRow(sh, r, 6, (c % 2 === 0) ? ALT : WHITE);
      sh.getRange(r, 1).setValue(CHANNELS[c]).setFontWeight('bold'); 
      sh.getRange(r, 2).setValue(0); sh.getRange(r, 3).setValue(0);
      sh.getRange(r, 4).setFormula('=IFERROR(B'+r+'/C'+r+',0)').setNumberFormat('0%').setHorizontalAlignment('center').setFontColor('#059669').setFontWeight('bold');
      sh.getRange(r, 5).setValue(0); r++;
    }
    styleTotalRow(sh, r, 6, WCOLORS[w]);
    sh.getRange(r, 2).setFormula('=SUM(B'+start+':B'+(r-1)+')'); sh.getRange(r, 3).setFormula('=SUM(C'+start+':C'+(r-1)+')');
    sh.getRange(r, 4).setFormula('=IFERROR(B'+r+'/C'+r+',0)').setNumberFormat('0%'); sh.getRange(r, 5).setFormula('=SUM(E'+start+':E'+(r-1)+')'); r += 2;
  }
}

function createBRollSheet() {
  var ss = SpreadsheetApp.getActiveSpreadsheet(); var sh = ss.insertSheet('B-Roll');
  sh.setColumnWidth(1, 260); sh.setColumnWidth(2, 170); sh.setColumnWidth(3, 280);
  styleTitle(sh, 1, 3, 'THEO DÕI B-ROLL THEO TUẦN', PURPLE);
  var r = 2;
  for (var w = 0; w < WEEKS; w++) {
    styleWeekHdr(sh, r, 3, 'TUẦN ' + (w + 1), WCOLORS[w]); r++;
    styleSubHdr(sh, r, 3, ['Loại Tài Nguyên','Số Lượng Thu Thập','Ghi Chú / Nguồn']); r++;
    var start = r;
    for (var b = 0; b < BROLL.length; b++) {
      styleDataRow(sh, r, 3, (b % 2 === 0) ? ALT : WHITE);
      sh.getRange(r, 1).setValue(BROLL[b]).setFontWeight('bold'); sh.getRange(r, 2).setValue(0); r++;
    }
    styleTotalRow(sh, r, 3, WCOLORS[w]); sh.getRange(r, 2).setFormula('=SUM(B'+start+':B'+(r-1)+')'); r += 2;
  }
}

// ── UTILS STYLE ──
function styleTitle(sh, r, c, t, bg) { var rg = sh.getRange(r, 1, 1, c); rg.merge().setValue(t).setBackground(bg).setFontColor(WHITE).setFontWeight('bold').setHorizontalAlignment('center').setVerticalAlignment('middle').setFontSize(13); sh.setRowHeight(r, 40); }
function styleWeekHdr(sh, r, c, t, bg) { var rg = sh.getRange(r, 1, 1, c); rg.merge().setValue(t).setBackground(bg).setFontColor(WHITE).setFontWeight('bold').setHorizontalAlignment('center').setVerticalAlignment('middle'); sh.setRowHeight(r, 30); }
function styleSubHdr(sh, r, c, l) { for (var i = 0; i < l.length; i++) { sh.getRange(r, i + 1).setValue(l[i]).setBackground(DARK).setFontColor(WHITE).setFontWeight('bold').setHorizontalAlignment('center').setVerticalAlignment('middle'); } sh.setRowHeight(r, 35); }
function styleDataRow(sh, r, c, bg) { sh.getRange(r, 1, 1, c).setBackground(bg).setVerticalAlignment('middle'); sh.setRowHeight(r, 25); }
function styleTotalRow(sh, r, c, bg) { var rg = sh.getRange(r, 1, 1, c); rg.setBackground(bg).setFontColor(WHITE).setFontWeight('bold').setHorizontalAlignment('center'); sh.getRange(r, 1).setValue('TỔNG TUẦN'); sh.setRowHeight(r, 28); }

// ── 2. LOGIC LƯU TRỮ VÀ API DASHBOARD ─────────────────────────────
function archiveMonth() {
  var ui = SpreadsheetApp.getUi();
  var response = ui.prompt('Chốt Sổ Tháng', 'Nhập tên/mã tháng muốn lưu (VD: 05/2026):', ui.ButtonSet.OK_CANCEL);
  if (response.getSelectedButton() !== ui.Button.OK) return;
  var monthStr = response.getResponseText().trim();
  if (!monthStr) { ui.alert('Lỗi: Tên tháng không được để trống!'); return; }

  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var data = getAllData();

  var tCs=0, activeMembers = new Set();
  data.casestudy.forEach(function(w){ tCs += w.total; if(w.members) w.members.forEach(m => { if(m.count > 0) activeMembers.add(m.name); }); });
  var numMembers = activeMembers.size; var avgCs = numMembers > 0 ? (tCs / numMembers).toFixed(1) : 0;

  var tVidDone=0, tVidTgt=0, tView=0;
  data.videoview.forEach(function(w){ tVidDone += w.totalDone; tVidTgt += w.totalTarget; tView += w.totalViews; });
  var pctVid = tVidTgt > 0 ? (tVidDone/tVidTgt) : 0;

  var tKbDone=0, tKbTgt=0, channelKb = {};
  data.kichban.forEach(function(w){
    tKbDone += w.totalDone; tKbTgt += w.totalTarget;
    if(w.channels) w.channels.forEach(c => {
      if(!channelKb[c.name]) channelKb[c.name] = {done: 0, target: 0};
      channelKb[c.name].done += c.done; channelKb[c.name].target += c.target;
    });
  });
  var pctKb = tKbTgt > 0 ? (tKbDone/tKbTgt) : 0;
  var channelsReached = 0; for(var k in channelKb) { if(channelKb[k].target > 0 && channelKb[k].done >= channelKb[k].target) channelsReached++; }

  var tBr=0, brTypes = new Set();
  data.broll.forEach(function(w){ tBr += w.total; if(w.items) w.items.forEach(i => { if(i.qty > 0) brTypes.add(i.type); }); });
  var numBrTypes = brTypes.size;

  var sheetName = 'Tổng Hợp Tháng';
  var sh = ss.getSheetByName(sheetName);
  var headers = [ 'Tháng', '| CASESTUDY |', 'Tổng Case', 'NS Tham gia', 'TB Case/Người', '| VIDEO & VIEW |', 'Video Hoàn Thành', 'Video Mục Tiêu', '% Đạt Video', 'Tổng View', '| KỊCH BẢN |', 'KB Hoàn Thành', 'KB Mục Tiêu', '% Đạt KB', 'Kênh Đạt MT', '| B-ROLL |', 'Tổng Files', 'Số loại TN', 'Data_JSON_Hidden' ];

  if (!sh) {
    sh = ss.insertSheet(sheetName, 0);
    sh.appendRow(headers);
    sh.getRange(1, 1, 1, headers.length).setBackground(PURPLE).setFontColor(WHITE).setFontWeight('bold').setHorizontalAlignment('center');
    sh.hideColumns(headers.length); sh.setColumnWidth(1, 100); [2, 6, 11, 16].forEach(col => sh.setColumnWidth(col, 130));
  }

  var vals = sh.getDataRange().getValues();
  var rowIdx = -1;
  for(var i=1; i<vals.length; i++) { if(vals[i][0] == monthStr) { rowIdx = i + 1; break; } }
  
  // Dữ liệu JSON lưu toàn bộ cục data động
  data.meta.month = "Tháng " + monthStr;
  var rowData = [ monthStr, '◼️', tCs, numMembers, avgCs, '◼️', tVidDone, tVidTgt, pctVid, tView, '◼️', tKbDone, tKbTgt, pctKb, channelsReached, '◼️', tBr, numBrTypes, JSON.stringify(data) ];

  if(rowIdx > -1) { sh.getRange(rowIdx, 1, 1, rowData.length).setValues([rowData]); } else { sh.appendRow(rowData); rowIdx = sh.getLastRow(); }
  sh.getRange(rowIdx, 9).setNumberFormat('0.0%'); sh.getRange(rowIdx, 14).setNumberFormat('0.0%'); sh.getRange(rowIdx, 10).setNumberFormat('#,##0');
  sh.getRange(rowIdx, 1, 1, rowData.length-1).setHorizontalAlignment('center');
  [2, 6, 11, 16].forEach(col => { sh.getRange(rowIdx, col).setBackground('#F0F4FF').setFontColor('#8B93A8'); });

  ui.alert('Thành công! Đã chốt sổ và lưu trữ dữ liệu ' + monthStr);
}

function getDashboardInit() {
  var ss = SpreadsheetApp.getActiveSpreadsheet(); var sh = ss.getSheetByName('Tổng Hợp Tháng');
  var months = [{id: 'current', label: 'Tháng hiện tại (Đang nhập)'}];
  if (sh) { var vals = sh.getDataRange().getValues(); for(var i=1; i<vals.length; i++){ if(vals[i][0]) months.push({id: String(vals[i][0]), label: 'Lịch sử: Tháng ' + vals[i][0]}); } }
  return { months: months, currentData: getAllData() };
}

function getMonthData(monthId) {
  if(monthId === 'current') return getAllData();
  var ss = SpreadsheetApp.getActiveSpreadsheet(); var sh = ss.getSheetByName('Tổng Hợp Tháng');
  if(!sh) return getAllData();
  var vals = sh.getDataRange().getValues();
  for(var i=1; i<vals.length; i++){ if(String(vals[i][0]) === monthId && vals[i][18]) { return JSON.parse(vals[i][18]); } }
  return getAllData();
}

function getAllData() {
  var ss = SpreadsheetApp.getActiveSpreadsheet(); var d = new Date();
  return { meta: { month: 'Tháng ' + (d.getMonth()+1) + '/' + d.getFullYear(), updatedAt: d.toLocaleDateString('vi-VN') }, casestudy: _readCS(ss), videoview: _readVV(ss), kichban: _readKB(ss), broll: _readBR(ss) };
}

function _getVals(ss, name) { var sh = ss.getSheetByName(name); return sh ? sh.getDataRange().getValues() : []; }

function _readCS(ss) {
  var data = _getVals(ss, 'Casestudy'), out = [];
  var dr_cs = MEMBERS.length, bs_cs = MEMBERS.length + 4;
  for (var w = 0; w < WEEKS; w++) {
    var ds = 1 + w*bs_cs + 2, members = [];
    for (var i = 0; i < dr_cs; i++) { var r = data[ds+i]||[]; if (r[0]) members.push({name:String(r[0]), count:Number(r[1])||0, note:String(r[3]||'')}); }
    var tot = data[ds+dr_cs]||[]; out.push({week:w+1, label:'Tuần '+(w+1), members:members, total:Number(tot[1])||0});
  } return out;
}

function _readVV(ss) {
  var data = _getVals(ss, 'Video & View'), out = [];
  var dr_vv = CHANNELS.length, bs_vv = CHANNELS.length + 4;
  for (var w = 0; w < WEEKS; w++) {
    var ds = 1 + w*bs_vv + 2, chs = [];
    for (var i = 0; i < dr_vv; i++) { var r = data[ds+i]||[]; if (r[0]) chs.push({name:String(r[0]), done:Number(r[1])||0, target:Number(r[2])||0, pct:Math.round((Number(r[3])||0)*100), views:Number(r[4])||0, note:String(r[5]||'')}); }
    var tot = data[ds+dr_vv]||[]; out.push({week:w+1, label:'Tuần '+(w+1), channels:chs, totalDone:Number(tot[1])||0, totalTarget:Number(tot[2])||0, totalPct:Math.round((Number(tot[3])||0)*100), totalViews:Number(tot[4])||0});
  } return out;
}

// Hàm đọc Kịch Bản phân nhóm tự động mọi tên/data user gõ thêm
function _readKB(ss) {
  var data = _getVals(ss, 'Kịch Bản'), out = [];
  if (!data.length) return out;
  
  var curWeekObj = null;
  var curChannelObj = null;
  
  for (var i = 0; i < data.length; i++) {
    var row = data[i];
    var valA = String(row[0]).trim();
    
    if (valA.indexOf('TUẦN ') === 0) {
      if (curWeekObj) out.push(curWeekObj);
      curWeekObj = { week: parseInt(valA.replace('TUẦN ',''))||0, label: valA, channels: [], totalDone: 0, totalTarget: 0, totalPct: 0 };
    }
    else if (valA === 'Kênh' && curWeekObj) {
      curChannelObj = { name: String(row[1]).trim(), items: [], done: 0, target: 0, pct: 0 };
    }
    else if (valA === 'Dạng Kịch Bản') {
      continue; // Bỏ qua Header phụ
    }
    else if (valA === 'TỔNG TUẦN' && curChannelObj && curWeekObj) {
      curChannelObj.done = Number(row[1])||0;
      curChannelObj.target = Number(row[2])||0;
      curChannelObj.pct = curChannelObj.target > 0 ? Math.round((curChannelObj.done / curChannelObj.target) * 100) : 0;
      curWeekObj.channels.push(curChannelObj);
      curWeekObj.totalDone += curChannelObj.done;
      curWeekObj.totalTarget += curChannelObj.target;
      curChannelObj = null;
    }
    else if (curChannelObj) {
      if (valA !== '') { 
        var d = Number(row[1])||0, t = Number(row[2])||0;
        curChannelObj.items.push({ type: valA, done: d, target: t });
      }
    }
  }
  if (curWeekObj) {
    curWeekObj.totalPct = curWeekObj.totalTarget > 0 ? Math.round((curWeekObj.totalDone / curWeekObj.totalTarget) * 100) : 0;
    out.push(curWeekObj);
  }
  return out;
}

function _readBR(ss) {
  var data = _getVals(ss, 'B-Roll'), out = [];
  var dr_br = BROLL.length, bs_br = BROLL.length + 4;
  for (var w = 0; w < WEEKS; w++) {
    var ds = 1 + w*bs_br + 2, items = [];
    for (var i = 0; i < dr_br; i++) { var r = data[ds+i]||[]; if (r[0]) items.push({type:String(r[0]), qty:Number(r[1])||0, note:String(r[2]||'')}); }
    var tot = data[ds+dr_br]||[]; out.push({week:w+1, label:'Tuần '+(w+1), items:items, total:Number(tot[1])||0});
  } return out;
}

function openDashboard() { SpreadsheetApp.getUi().showModalDialog(HtmlService.createHtmlOutputFromFile('Dashboard').setWidth(1150).setHeight(750), 'Hệ Thống Dashboard Nhân Sự'); }
function doGet() { return HtmlService.createHtmlOutputFromFile('Dashboard').setTitle('Dashboard Hiệu Suất Nhân Sự').setXFrameOptionsMode(HtmlService.XFrameOptionsMode.ALLOWALL); }
