// ═══════════════════════════════════════════════════════════════
//  BAO CAO HIEU SUAT NHAN SU  |  Code.gs
//  1. Paste vao Apps Script → Luu (Ctrl+S)
//  2. Reload Google Sheets
//  3. Menu "Bao Cao" → "Tao sheet bao cao"
//  4. Menu "Bao Cao" → "Mo Dashboard"
// ═══════════════════════════════════════════════════════════════

// ── CAU HINH — sua ten thanh vien / kenh tai day ─────────────────
var MEMBERS  = ['Vu Chi Khang  ','Ngo Thu Phuong ','Le Phuong Thuy ','Vu Dat ','Hoang Van E'];
var CHANNELS = ['An Binh VayVon ','ThuyVayVon ','VayVonDonGianDat ', 'KhangVayHay ','Phantichtaichinhchuyensau','Suthattaichinh'];
var BROLL    = ['Video nen (stock footage)','Infographic / Motion',
                'Anh minh hoa','Icon / Element do hoa',
                'Am thanh / Nhac nen','Template chinh sua'];
var WEEKS    = 4;
var WCOLORS  = ['#2563EB','#059669','#D97706','#DC2626'];
var PURPLE   = '#6C63FF';
var DARK     = '#374151';
var ALT      = '#F0F4FF';
var WHITE    = '#FFFFFF';

// ── MENU ─────────────────────────────────────────────────────────
function onOpen() {
  SpreadsheetApp.getUi()
    .createMenu('Bao Cao')
    .addItem('Tao sheet bao cao (lan dau)', 'setupSheets')
    .addSeparator()
    .addItem('Mo Dashboard',       'openDashboard')
    .addItem('Xuat bao cao thang', 'exportMonthlyReport')
    .addToUi();
}

// ════════════════════════════════════════════════════════════════
//  SETUP — XOA CU + TAO MOI 4 SHEET
// ════════════════════════════════════════════════════════════════
function setupSheets() {
  var ui = SpreadsheetApp.getUi();
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var names = ['Casestudy','Video & View','Kich Ban','B-Roll'];

  // Xoa sheet cu
  for (var i = 0; i < names.length; i++) {
    var old = ss.getSheetByName(names[i]);
    if (old) {
      try { ss.deleteSheet(old); } catch(e) {}
    }
  }
  Utilities.sleep(800);

  // Tao 4 sheet
  createCasestudySheet();
  SpreadsheetApp.flush();
  createVideoViewSheet();
  SpreadsheetApp.flush();
  createKichBanSheet();
  SpreadsheetApp.flush();
  createBRollSheet();
  SpreadsheetApp.flush();

  // Xoa sheet mac dinh trong
  var defs = ['Sheet1','Trang tinh1','Trang tinh 1'];
  for (var d = 0; d < defs.length; d++) {
    var ds = ss.getSheetByName(defs[d]);
    if (ds && ss.getSheets().length > 1) {
      try { ss.deleteSheet(ds); } catch(e) {}
    }
  }

  var csSheet = ss.getSheetByName('Casestudy');
  if (csSheet) ss.setActiveSheet(csSheet);

  ui.alert('Hoan tat! Da tao du 4 sheet:\n- Casestudy\n- Video & View\n- Kich Ban\n- B-Roll\n\nNhap so lieu vao o trang roi mo Dashboard.');
}

// ════════════════════════════════════════════════════════════════
//  STYLE UTILS
// ════════════════════════════════════════════════════════════════
function styleTitle(sh, row, ncols, text, color) {
  var rng = sh.getRange(row, 1, 1, ncols);
  rng.merge();
  rng.setValue(text);
  rng.setBackground(color);
  rng.setFontColor(WHITE);
  rng.setFontWeight('bold');
  rng.setFontSize(13);
  rng.setFontFamily('Arial');
  rng.setHorizontalAlignment('center');
  rng.setVerticalAlignment('middle');
  sh.setRowHeight(row, 40);
}

function styleWeekHdr(sh, row, ncols, text, color) {
  var rng = sh.getRange(row, 1, 1, ncols);
  rng.merge();
  rng.setValue(text);
  rng.setBackground(color);
  rng.setFontColor(WHITE);
  rng.setFontWeight('bold');
  rng.setFontSize(11);
  rng.setFontFamily('Arial');
  rng.setHorizontalAlignment('center');
  rng.setVerticalAlignment('middle');
  sh.setRowHeight(row, 30);
}

function styleSubHdr(sh, row, ncols, labels) {
  for (var i = 0; i < labels.length; i++) {
    var c = sh.getRange(row, i + 1);
    c.setValue(labels[i]);
    c.setBackground(DARK);
    c.setFontColor(WHITE);
    c.setFontWeight('bold');
    c.setFontSize(10);
    c.setFontFamily('Arial');
    c.setHorizontalAlignment('center');
    c.setVerticalAlignment('middle');
    c.setWrap(true);
  }
  sh.setRowHeight(row, 36);
}

function styleDataRow(sh, row, ncols, bg) {
  var rng = sh.getRange(row, 1, 1, ncols);
  rng.setBackground(bg);
  rng.setFontSize(10);
  rng.setFontFamily('Arial');
  rng.setVerticalAlignment('middle');
  sh.setRowHeight(row, 24);
}

function styleTotalRow(sh, row, ncols, color) {
  var rng = sh.getRange(row, 1, 1, ncols);
  rng.setBackground(color);
  rng.setFontColor(WHITE);
  rng.setFontWeight('bold');
  rng.setFontSize(10);
  rng.setFontFamily('Arial');
  rng.setHorizontalAlignment('center');
  rng.setVerticalAlignment('middle');
  sh.getRange(row, 1).setValue('TONG TUAN');
  sh.setRowHeight(row, 28);
}

// ════════════════════════════════════════════════════════════════
//  SHEET 1: CASESTUDY
// ════════════════════════════════════════════════════════════════
function createCasestudySheet() {
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var sh = ss.insertSheet('Casestudy');

  sh.setColumnWidth(1, 220);
  sh.setColumnWidth(2, 180);
  sh.setColumnWidth(3, 140);
  sh.setColumnWidth(4, 280);

  // Title
  styleTitle(sh, 1, 4, 'THEO DOI CASESTUDY THEO TUAN', PURPLE);

  var r = 2;
  for (var w = 0; w < WEEKS; w++) {
    var wc = WCOLORS[w];
    var wLabel = 'TUAN ' + (w + 1);

    // Week header
    styleWeekHdr(sh, r, 4, wLabel, wc);
    r++;

    // Sub-headers
    styleSubHdr(sh, r, 4, ['Thanh Vien', 'So Casestudy (Form 4F)', 'Tong Luy Ke', 'Note']);
    r++;

    // Data rows
    var dataStart = r;
    for (var m = 0; m < MEMBERS.length; m++) {
      var bg = (m % 2 === 0) ? ALT : WHITE;
      styleDataRow(sh, r, 4, bg);
      sh.getRange(r, 1).setValue(MEMBERS[m]).setFontWeight('bold').setHorizontalAlignment('left').setFontColor('#1E293B');
      sh.getRange(r, 2).setValue(0).setHorizontalAlignment('center').setFontColor('#1E293B');
      sh.getRange(r, 3).setValue('').setHorizontalAlignment('center');
      sh.getRange(r, 4).setValue('').setHorizontalAlignment('left').setFontColor('#374151');
      r++;
    }

    // Total row
    styleTotalRow(sh, r, 4, wc);
    sh.getRange(r, 2).setFormula('=SUM(B' + dataStart + ':B' + (r-1) + ')').setBackground(wc).setFontColor(WHITE).setFontWeight('bold').setHorizontalAlignment('center');
    r++;

    // Gap
    sh.setRowHeight(r, 8);
    r++;
  }
}

// ════════════════════════════════════════════════════════════════
//  SHEET 2: VIDEO & VIEW
// ════════════════════════════════════════════════════════════════
function createVideoViewSheet() {
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var sh = ss.insertSheet('Video & View');

  sh.setColumnWidth(1, 200);
  sh.setColumnWidth(2, 130);
  sh.setColumnWidth(3, 160);
  sh.setColumnWidth(4, 250);

  styleTitle(sh, 1, 4, 'THEO DOI VIDEO & VIEW THEO TUAN', PURPLE);

  var r = 2;
  for (var w = 0; w < WEEKS; w++) {
    var wc = WCOLORS[w];

    styleWeekHdr(sh, r, 4, 'TUAN ' + (w + 1), wc);
    r++;

    styleSubHdr(sh, r, 4, ['Kenh', 'So Video', 'Tong View', 'Ghi Chu']);
    r++;

    var dataStart = r;
    for (var ch = 0; ch < CHANNELS.length; ch++) {
      var bg = (ch % 2 === 0) ? ALT : WHITE;
      styleDataRow(sh, r, 4, bg);
      sh.getRange(r, 1).setValue(CHANNELS[ch]).setFontWeight('bold').setHorizontalAlignment('left').setFontColor('#1E293B');
      sh.getRange(r, 2).setValue(0).setHorizontalAlignment('center').setFontColor('#1E293B');
      sh.getRange(r, 3).setValue(0).setHorizontalAlignment('center').setFontColor('#1E293B');
      sh.getRange(r, 4).setValue('').setHorizontalAlignment('left').setFontColor('#374151');
      r++;
    }

    styleTotalRow(sh, r, 4, wc);
    sh.getRange(r, 2).setFormula('=SUM(B'+dataStart+':B'+(r-1)+')').setBackground(wc).setFontColor(WHITE).setFontWeight('bold').setHorizontalAlignment('center');
    sh.getRange(r, 3).setFormula('=SUM(C'+dataStart+':C'+(r-1)+')').setBackground(wc).setFontColor(WHITE).setFontWeight('bold').setHorizontalAlignment('center');
    r++;

    sh.setRowHeight(r, 8);
    r++;
  }
}

// ════════════════════════════════════════════════════════════════
//  SHEET 3: KICH BAN
// ════════════════════════════════════════════════════════════════
function createKichBanSheet() {
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var sh = ss.insertSheet('Kich Ban');

  sh.setColumnWidth(1, 200);
  sh.setColumnWidth(2, 160);
  sh.setColumnWidth(3, 160);
  sh.setColumnWidth(4, 100);
  sh.setColumnWidth(5, 220);

  styleTitle(sh, 1, 5, 'THEO DOI KICH BAN THEO TUAN', PURPLE);

  var r = 2;
  for (var w = 0; w < WEEKS; w++) {
    var wc = WCOLORS[w];

    styleWeekHdr(sh, r, 5, 'TUAN ' + (w + 1), wc);
    r++;

    styleSubHdr(sh, r, 5, ['Kenh','KB Hoan Thanh','KB Muc Tieu','% Dat','Phan Cong']);
    r++;

    var dataStart = r;
    for (var ch = 0; ch < CHANNELS.length; ch++) {
      var bg = (ch % 2 === 0) ? ALT : WHITE;
      styleDataRow(sh, r, 5, bg);
      sh.getRange(r, 1).setValue(CHANNELS[ch]).setFontWeight('bold').setHorizontalAlignment('left').setFontColor('#1E293B');
      sh.getRange(r, 2).setValue(0).setHorizontalAlignment('center').setFontColor('#1E293B');
      sh.getRange(r, 3).setValue(0).setHorizontalAlignment('center').setFontColor('#1E293B');
      sh.getRange(r, 4).setFormula('=IFERROR(B'+r+'/C'+r+',0)').setNumberFormat('0%').setHorizontalAlignment('center').setFontColor('#059669').setFontWeight('bold').setBackground(bg);
      sh.getRange(r, 5).setValue('').setHorizontalAlignment('left').setFontColor('#374151');
      r++;
    }

    styleTotalRow(sh, r, 5, wc);
    sh.getRange(r, 2).setFormula('=SUM(B'+dataStart+':B'+(r-1)+')').setBackground(wc).setFontColor(WHITE).setFontWeight('bold').setHorizontalAlignment('center');
    sh.getRange(r, 3).setFormula('=SUM(C'+dataStart+':C'+(r-1)+')').setBackground(wc).setFontColor(WHITE).setFontWeight('bold').setHorizontalAlignment('center');
    sh.getRange(r, 4).setFormula('=IFERROR(B'+r+'/C'+r+',0)').setNumberFormat('0%').setBackground(wc).setFontColor(WHITE).setFontWeight('bold').setHorizontalAlignment('center');
    r++;

    sh.setRowHeight(r, 8);
    r++;
  }
}

// ════════════════════════════════════════════════════════════════
//  SHEET 4: BRoll
// ════════════════════════════════════════════════════════════════
function createBRollSheet() {
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var sh = ss.insertSheet('BRoll');

  sh.setColumnWidth(1, 260);
  sh.setColumnWidth(2, 170);
  sh.setColumnWidth(3, 280);

  styleTitle(sh, 1, 3, 'THEO DOI BRoll THEO TUAN', PURPLE);

  var r = 2;
  for (var w = 0; w < WEEKS; w++) {
    var wc = WCOLORS[w];

    styleWeekHdr(sh, r, 3, 'TUAN ' + (w + 1), wc);
    r++;

    styleSubHdr(sh, r, 3, ['Loai Tai Nguyen','So Luong Thu Thap','Ghi Chu / Nguon']);
    r++;

    var dataStart = r;
    for (var t = 0; t < BRoll.length; t++) {
      var bg = (t % 2 === 0) ? ALT : WHITE;
      styleDataRow(sh, r, 3, bg);
      sh.getRange(r, 1).setValue(BRoll[t]).setFontWeight('bold').setHorizontalAlignment('left').setFontColor('#1E293B');
      sh.getRange(r, 2).setValue(0).setHorizontalAlignment('center').setFontColor('#1E293B');
      sh.getRange(r, 3).setValue('').setHorizontalAlignment('left').setFontColor('#374151');
      r++;
    }

    styleTotalRow(sh, r, 3, wc);
    sh.getRange(r, 2).setFormula('=SUM(B'+dataStart+':B'+(r-1)+')').setBackground(wc).setFontColor(WHITE).setFontWeight('bold').setHorizontalAlignment('center');
    r++;

    sh.setRowHeight(r, 8);
    r++;
  }
}

// ════════════════════════════════════════════════════════════════
//  DOC DU LIEU → DASHBOARD
// ════════════════════════════════════════════════════════════════
function getAllData() {
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var need = ['Casestudy','Video & View','Kich Ban','B-Roll'];
  var miss = [];
  for (var i = 0; i < need.length; i++) {
    if (!ss.getSheetByName(need[i])) miss.push(need[i]);
  }
  if (miss.length > 0) {
    throw new Error('Chua co sheet: ' + miss.join(', ') + '. Vao menu Bao Cao → Tao sheet bao cao.');
  }
  var d = new Date();
  return {
    meta     : { month: 'Thang ' + (d.getMonth()+1) + '/' + d.getFullYear(),
                 updatedAt: d.toLocaleDateString('vi-VN') },
    casestudy: _readCS(ss),
    videoview: _readVV(ss),
    kichban  : _readKB(ss),
    broll    : _readBR(ss)
  };
}

// blockSize per week: 1(wk-hdr)+1(sub-hdr)+N(data)+1(total)+1(gap)
var BS = {cs:9, vv:7, kb:7, br:10};
var DR = {cs:5, vv:3, kb:3, br:6};

function _getVals(ss, name) {
  return ss.getSheetByName(name).getDataRange().getValues();
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
    out.push({week:w+1,label:'Tuan '+(w+1),members:members,
              total:Number(tot[1])||members.reduce(function(a,m){return a+m.count;},0)});
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
    out.push({week:w+1,label:'Tuan '+(w+1),channels:chs,
              totalVideos:Number(tot[1])||chs.reduce(function(a,c){return a+c.videos;},0),
              totalViews :Number(tot[2])||chs.reduce(function(a,c){return a+c.views; },0)});
  }
  return out;
}

function _readKB(ss) {
  var data = _getVals(ss, 'Kich Ban'), out = [];
  for (var w = 0; w < WEEKS; w++) {
    var ds = 1 + w*BS.kb + 2, chs = [];
    for (var i = 0; i < DR.kb; i++) {
      var row = data[ds+i]||[];
      if (!row[0]) continue;
      var done=Number(row[1])||0, tgt=Number(row[2])||0;
      chs.push({name:String(row[0]),done:done,target:tgt,
                pct:tgt>0?Math.round(done/tgt*100):0,assignee:String(row[4]||'')});
    }
    var tot=data[ds+DR.kb]||[];
    var td=Number(tot[1])||chs.reduce(function(a,c){return a+c.done;},0);
    var tt=Number(tot[2])||chs.reduce(function(a,c){return a+c.target;},0);
    out.push({week:w+1,label:'Tuan '+(w+1),channels:chs,
              totalDone:td,totalTarget:tt,totalPct:tt>0?Math.round(td/tt*100):0});
  }
  return out;
}

function _readBR(ss) {
  var data = _getVals(ss, 'BROLL'), out = [];
  for (var w = 0; w < WEEKS; w++) {
    var ds = 1 + w*BS.br + 2, items = [];
    for (var i = 0; i < DR.br; i++) {
      var row = data[ds+i]||[];
      if (row[0]) items.push({type:String(row[0]),qty:Number(row[1])||0,note:String(row[2]||'')});
    }
    var tot = data[ds+DR.br]||[];
    out.push({week:w+1,label:'Tuan '+(w+1),items:items,
              total:Number(tot[1])||items.reduce(function(a,i){return a+i.qty;},0)});
  }
  return out;
}

// ════════════════════════════════════════════════════════════════
//  DASHBOARD & EXPORT
// ════════════════════════════════════════════════════════════════
function openDashboard() {
  var html = HtmlService.createHtmlOutputFromFile('Dashboard').setWidth(1100).setHeight(700);
  SpreadsheetApp.getUi().showModalDialog(html, 'Dashboard Hieu Suat Nhan Su');
}

function exportMonthlyReport() {
  var ss   = SpreadsheetApp.getActiveSpreadsheet();
  var data = getAllData();
  var sh   = ss.getSheetByName('Bao Cao Thang');
  if (sh) sh.clear(); else sh = ss.insertSheet('Bao Cao Thang');

  var hdrs = ['Tuan','Casestudy','Video','Tong View','KB Hoan Thanh','KB Muc Tieu','% KB','B-Roll'];
  sh.getRange(1,1).setValue('BAO CAO TONG HOP — ' + data.meta.month)
    .setFontSize(14).setFontWeight('bold').setFontColor(PURPLE);
  sh.getRange(2,1,1,8).setValues([hdrs]);
  styleSubHdr(sh, 2, 8, hdrs);
  sh.getRange(2,1,1,8).setBackground(PURPLE);

  var rows = [];
  for (var w = 0; w < WEEKS; w++) {
    var cs=data.casestudy[w]||{}, vv=data.videoview[w]||{}, kb=data.kichban[w]||{}, br=data.broll[w]||{};
    rows.push(['Tuan '+(w+1), cs.total||0, vv.totalVideos||0, vv.totalViews||0,
               kb.totalDone||0, kb.totalTarget||0, (kb.totalPct||0)/100, br.total||0]);
  }
  var sum = ['TONG THANG',0,0,0,0,0,'',0];
  for (var ri = 0; ri < rows.length; ri++) {
    for (var ci = 1; ci <= 5; ci++) sum[ci] += rows[ri][ci];
    sum[7] += rows[ri][7];
  }
  rows.push(sum);

  sh.getRange(3,1,rows.length,8).setValues(rows);
  sh.getRange(3,7,rows.length-1,1).setNumberFormat('0.0%');
  sh.getRange(rows.length+2,1,1,8).setBackground(DARK).setFontColor(WHITE).setFontWeight('bold');
  for (var ri2 = 0; ri2 < rows.length-1; ri2++) {
    if (ri2 % 2 === 0) sh.getRange(ri2+3,1,1,8).setBackground(ALT);
  }
  sh.autoResizeColumns(1,8);
  ss.setActiveSheet(sh);
  SpreadsheetApp.getUi().alert('Da tao sheet "Bao Cao Thang"!');
}
