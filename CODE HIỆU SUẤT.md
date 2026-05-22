// ═══════════════════════════════════════════════════════════════
//  HỆ THỐNG BÁO CÁO HIỆU SUẤT NHÂN SỰ TOÀN DIỆN
// ═══════════════════════════════════════════════════════════════

// ── CẤU HÌNH DANH SÁCH — Bạn có thể thêm bớt tùy ý tại đây ────────────────
var MEMBERS  = ['Vu Chi Khang', 'Ngo Thu Phuong', 'Le Phuong Thuy', 'Vu Dat', 'Hoang Van E'];
var CHANNELS = [
  'An Binh VayVon', 'ThuyVayVon', 'VayVonDonGianDat', 
  'KhangVayHay', 'Phantichtaichinhchuyensau', 'Suthattaichinh'
];
var BROLL    = [
  'Video nền (stock footage)', 'Infographic / Motion', 'Ảnh minh họa', 
  'Icon / Element đồ họa', 'Âm thanh / Nhạc nền', 'Template chỉnh sửa'
];

var WEEKS    = 4;
var WCOLORS  = ['#2563EB','#059669','#D97706','#DC2626'];
var PURPLE   = '#6C63FF';
var DARK     = '#374151';
var ALT      = '#F0F4FF';
var WHITE    = '#FFFFFF';

// ── CẤU HÌNH DÒNG ĐỘNG (TỰ ĐỘNG GIÃN DÒNG) ────────────────────────────────
var DR = {
  cs: MEMBERS.length, 
  vv: CHANNELS.length, 
  kb: CHANNELS.length, 
  br: BROLL.length
};

var BS = {
  cs: MEMBERS.length + 4, 
  vv: CHANNELS.length + 4, 
  kb: CHANNELS.length + 4, 
  br: BROLL.length + 4
};

// ── MENU ─────────────────────────────────────────────────────────
function onOpen() {
  SpreadsheetApp.getUi()
    .createMenu('Báo Cáo')
    .addItem('1. Tạo/Cài đặt Sheet (Lần đầu)', 'setupSheets')
    .addSeparator()
    .addItem('2. Mở Dashboard', 'openDashboard')
    .addItem('3. Xuất báo cáo tháng', 'exportMonthlyReport')
    .addToUi();
}

// ── SETUP SHEET ──────────────────────────────────────────────────
function setupSheets() {
  var ui = SpreadsheetApp.getUi();
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var names = ['Casestudy','Video & View','Kịch Bản','B-Roll', 'BRoll', 'BROLL', 'Kich Ban'];

  for (var i = 0; i < names.length; i++) {
    var old = ss.getSheetByName(names[i]);
    if (old) { try { ss.deleteSheet(old); } catch(e) {} }
  }
  Utilities.sleep(800);

  createCasestudySheet();
  createVideoViewSheet();
  createKichBanSheet();
  createBRollSheet();
  
  SpreadsheetApp.flush();

  var defs = ['Sheet1','Trang tính1','Trang tính 1'];
  for (var d = 0; d < defs.length; d++) {
    var ds = ss.getSheetByName(defs[d]);
    if (ds && ss.getSheets().length > 1) { try { ss.deleteSheet(ds); } catch(e) {} }
  }

  ui.alert('Hoàn tất! Hệ thống đã tự động tạo sheet.');
}

// ── CÁC HÀM TẠO SHEET ────────────────────────────────────────────

function createCasestudySheet() {
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var sh = ss.insertSheet('Casestudy');
  sh.setColumnWidth(1, 220); sh.setColumnWidth(2, 180);
  styleTitle(sh, 1, 4, 'THEO DÕI CASESTUDY THEO TUẦN', PURPLE);
  var r = 2;
  for (var w = 0; w < WEEKS; w++) {
    styleWeekHdr(sh, r, 4, 'TUẦN ' + (w + 1), WCOLORS[w]); r++;
    styleSubHdr(sh, r, 4, ['Thành Viên', 'Số Casestudy', 'Tổng Lũy Kế', 'Note']); r++;
    var start = r;
    for (var m = 0; m < MEMBERS.length; m++) {
      styleDataRow(sh, r, 4, (m % 2 === 0) ? ALT : WHITE);
      sh.getRange(r, 1).setValue(MEMBERS[m]).setFontWeight('bold');
      sh.getRange(r, 2).setValue(0); r++;
    }
    styleTotalRow(sh, r, 4, WCOLORS[w]);
    sh.getRange(r, 2).setFormula('=SUM(B' + start + ':B' + (r-1) + ')'); r += 2;
  }
}

function createVideoViewSheet() {
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var sh = ss.insertSheet('Video & View');
  sh.setColumnWidth(1, 200); sh.setColumnWidth(3, 160);
  styleTitle(sh, 1, 4, 'THEO DÕI VIDEO & VIEW THEO TUẦN', PURPLE);
  var r = 2;
  for (var w = 0; w < WEEKS; w++) {
    styleWeekHdr(sh, r, 4, 'TUẦN ' + (w + 1), WCOLORS[w]); r++;
    styleSubHdr(sh, r, 4, ['Kênh', 'Số Video', 'Tổng View', 'Ghi Chú']); r++;
    var start = r;
    for (var c = 0; c < CHANNELS.length; c++) {
      styleDataRow(sh, r, 4, (c % 2 === 0) ? ALT : WHITE);
      sh.getRange(r, 1).setValue(CHANNELS[c]).setFontWeight('bold');
      sh.getRange(r, 2).setValue(0); sh.getRange(r, 3).setValue(0); r++;
    }
    styleTotalRow(sh, r, 4, WCOLORS[w]);
    sh.getRange(r, 2).setFormula('=SUM(B'+start+':B'+(r-1)+')');
    sh.getRange(r, 3).setFormula('=SUM(C'+start+':C'+(r-1)+')'); r += 2;
  }
}

function createKichBanSheet() {
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var sh = ss.insertSheet('Kịch Bản');
  sh.setColumnWidth(1, 200); sh.setColumnWidth(5, 220);
  styleTitle(sh, 1, 5, 'THEO DÕI KỊCH BẢN THEO TUẦN', PURPLE);
  var r = 2;
  for (var w = 0; w < WEEKS; w++) {
    styleWeekHdr(sh, r, 5, 'TUẦN ' + (w + 1), WCOLORS[w]); r++;
    styleSubHdr(sh, r, 5, ['Kênh','KB Hoàn Thành','KB Mục Tiêu','% Đạt','Phân Công']); r++;
    var start = r;
    for (var c = 0; c < CHANNELS.length; c++) {
      styleDataRow(sh, r, 5, (c % 2 === 0) ? ALT : WHITE);
      sh.getRange(r, 1).setValue(CHANNELS[c]).setFontWeight('bold');
      sh.getRange(r, 2).setValue(0); sh.getRange(r, 3).setValue(0);
      sh.getRange(r, 4).setFormula('=IFERROR(B'+r+'/C'+r+',0)').setNumberFormat('0%'); r++;
    }
    styleTotalRow(sh, r, 5, WCOLORS[w]);
    sh.getRange(r, 2).setFormula('=SUM(B'+start+':B'+(r-1)+')');
    sh.getRange(r, 3).setFormula('=SUM(C'+start+':C'+(r-1)+')');
    sh.getRange(r, 4).setFormula('=IFERROR(B'+r+'/C'+r+',0)').setNumberFormat('0%'); r += 2;
  }
}

function createBRollSheet() {
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var sh = ss.insertSheet('B-Roll');
  sh.setColumnWidth(1, 260); sh.setColumnWidth(2, 170);
  styleTitle(sh, 1, 3, 'THEO DÕI B-ROLL THEO TUẦN', PURPLE);
  var r = 2;
  for (var w = 0; w < WEEKS; w++) {
    styleWeekHdr(sh, r, 3, 'TUẦN ' + (w + 1), WCOLORS[w]); r++;
    styleSubHdr(sh, r, 3, ['Loại Tài Nguyên','Số Lượng','Ghi Chú / Nguồn']); r++;
    var start = r;
    for (var b = 0; b < BROLL.length; b++) {
      styleDataRow(sh, r, 3, (b % 2 === 0) ? ALT : WHITE);
      sh.getRange(r, 1).setValue(BROLL[b]).setFontWeight('bold');
      sh.getRange(r, 2).setValue(0); r++;
    }
    styleTotalRow(sh, r, 3, WCOLORS[w]);
    sh.getRange(r, 2).setFormula('=SUM(B'+start+':B'+(r-1)+')'); r += 2;
  }
}

// ── STYLE UTILS ──────────────────────────────────────────────────
function styleTitle(sh, r, c, t, bg) {
  var rg = sh.getRange(r, 1, 1, c); rg.merge().setValue(t).setBackground(bg).setFontColor(WHITE).setFontWeight('bold').setHorizontalAlignment('center').setVerticalAlignment('middle').setFontSize(13);
  sh.setRowHeight(r, 40);
}
function styleWeekHdr(sh, r, c, t, bg) {
  var rg = sh.getRange(r, 1, 1, c); rg.merge().setValue(t).setBackground(bg).setFontColor(WHITE).setFontWeight('bold').setHorizontalAlignment('center').setVerticalAlignment('middle');
  sh.setRowHeight(r, 30);
}
function styleSubHdr(sh, r, c, l) {
  for (var i = 0; i < l.length; i++) {
    sh.getRange(r, i + 1).setValue(l[i]).setBackground(DARK).setFontColor(WHITE).setFontWeight('bold').setHorizontalAlignment('center').setVerticalAlignment('middle');
  }
  sh.setRowHeight(r, 35);
}
function styleDataRow(sh, r, c, bg) {
  sh.getRange(r, 1, 1, c).setBackground(bg).setVerticalAlignment('middle');
  sh.setRowHeight(r, 25);
}
function styleTotalRow(sh, r, c, bg) {
  var rg = sh.getRange(r, 1, 1, c); rg.setBackground(bg).setFontColor(WHITE).setFontWeight('bold').setHorizontalAlignment('center');
  sh.getRange(r, 1).setValue('TỔNG TUẦN');
  sh.setRowHeight(r, 28);
}

// ── ĐỌC DỮ LIỆU ĐÚNG CẤU TRÚC DASHBOARD CŨ ─────────────────────────
function getAllData() {
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var d = new Date();
  return {
    meta     : { month: 'Tháng ' + (d.getMonth()+1) + '/' + d.getFullYear(), updatedAt: d.toLocaleDateString('vi-VN') },
    casestudy: _readCS(ss),
    videoview: _readVV(ss),
    kichban  : _readKB(ss),
    broll    : _readBR(ss)
  };
}

function _getVals(ss, name) {
  var sh = ss.getSheetByName(name);
  return sh ? sh.getDataRange().getValues() : [];
}

function _readCS(ss) {
  var data = _getVals(ss, 'Casestudy'), out = [];
  for (var w = 0; w < WEEKS; w++) {
    var ds = 1 + w*BS.cs + 2, members = [];
    for (var i = 0; i < DR.cs; i++) {
      var row = data[ds+i]||[];
      if (row[0]) members.push({name:String(row[0]),count:Number(row[1])||0,note:String(row[3]||'')});
    }
    var tot = data[ds+DR.cs]||[];
    out.push({week:w+1,label:'Tuần '+(w+1),members:members, total:Number(tot[1])||members.reduce(function(a,m){return a+m.count;},0)});
  }
  return out;
}

function _readVV(ss) {
  var data = _getVals(ss, 'Video & View'), out = [];
  for (var w = 0; w < WEEKS; w++) {
    var ds = 1 + w*BS.vv + 2, chs = [];
    for (var i = 0; i < DR.vv; i++) {
      var row = data[ds+i]||[];
      if (row[0]) chs.push({name:String(row[0]),videos:Number(row[1])||0,views:Number(row[2])||0});
    }
    var tot = data[ds+DR.vv]||[];
    out.push({week:w+1,label:'Tuần '+(w+1),channels:chs,
              totalVideos:Number(tot[1])||chs.reduce(function(a,c){return a+c.videos;},0),
              totalViews :Number(tot[2])||chs.reduce(function(a,c){return a+c.views; },0)});
  }
  return out;
}

function _readKB(ss) {
  var data = _getVals(ss, 'Kịch Bản'), out = [];
  for (var w = 0; w < WEEKS; w++) {
    var ds = 1 + w*BS.kb + 2, chs = [];
    for (var i = 0; i < DR.kb; i++) {
      var row = data[ds+i]||[];
      if (!row[0]) continue;
      var done=Number(row[1])||0, tgt=Number(row[2])||0;
      chs.push({name:String(row[0]),done:done,target:tgt, pct:tgt>0?Math.round(done/tgt*100):0,assignee:String(row[4]||'')});
    }
    var tot=data[ds+DR.kb]||[];
    var td=Number(tot[1])||chs.reduce(function(a,c){return a+c.done;},0);
    var tt=Number(tot[2])||chs.reduce(function(a,c){return a+c.target;},0);
    out.push({week:w+1,label:'Tuần '+(w+1),channels:chs, totalDone:td,totalTarget:tt,totalPct:tt>0?Math.round(td/tt*100):0});
  }
  return out;
}

function _readBR(ss) {
  var data = _getVals(ss, 'B-Roll'), out = [];
  for (var w = 0; w < WEEKS; w++) {
    var ds = 1 + w*BS.br + 2, items = [];
    for (var i = 0; i < DR.br; i++) {
      var row = data[ds+i]||[];
      if (row[0]) items.push({type:String(row[0]),qty:Number(row[1])||0,note:String(row[2]||'')});
    }
    var tot = data[ds+DR.br]||[];
    out.push({week:w+1,label:'Tuần '+(w+1),items:items, total:Number(tot[1])||items.reduce(function(a,i){return a+i.qty;},0)});
  }
  return out;
}

function openDashboard() {
  var html = HtmlService.createHtmlOutputFromFile('Dashboard').setWidth(1150).setHeight(750);
  SpreadsheetApp.getUi().showModalDialog(html, 'Hệ Thống Dashboard Nhân Sự');
}
function doGet() {
  return HtmlService.createHtmlOutputFromFile('Dashboard')
    .setTitle('Dashboard Hiệu Suất Nhân Sự')
    .setXFrameOptionsMode(HtmlService.XFrameOptionsMode.ALLOWALL);
}
