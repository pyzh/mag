## Wikidown 的原始碼解析

wikidown 的原始碼主要包含兩個檔案，一個是前端的 wikidown.html ，另一個是後端的 wikiServer.js 。

前端的 wikidown.html 負責建構使用者介面，並透過 jQuery 的 ajax 遠端呼叫載入或儲存您編輯的 markdown 檔案，而後端的 wikiServer.js 則單純負責 markdown 檔案的讀取與寫入動作，並傳回讀取的內容給前端的 wikidown.html。

以下就讓我們來看看 wikidown 前後兩端程式的原始碼吧！

### 前端網頁：wikidown.html

```
<html>
<head>
<meta charset="utf-8" />
  <link rel="icon" href="favicon.ico">
  <link href="css/bootstrap.min.css" rel="stylesheet">
  <link rel="stylesheet" href="css/highlight.default.min.css">
  <link href="main.css" rel="stylesheet">
</head>
<body onload="init()">
    <nav class="navbar navbar-inverse navbar-fixed-top">
      <div class="container">
        <button type="button" class="navbar-toggle collapsed" data-toggle="collapse" data-target="#navbar" aria-expanded="false" aria-controls="navbar">
          <span class="sr-only">Toggle navigation</span>
          <span class="icon-bar"></span>
          <span class="icon-bar"></span>
          <span class="icon-bar"></span>
        </button>
        <div class="navbar-header">
          <a class="navbar-brand" href="#main:home" style="color:#C0C0C0">
		    <span class="glyphicon glyphicon-home"></span> &nbsp;
	      </a>
          <a id="titlelink" class="navbar-brand" href="#wikidown:home" style="color:#C0C0C0">
			<span id="title">Wikidown</span> 
	      </a>
        </div>
        <div id="navbar" class="navbar-collapse collapse">
          <form class="navbar-form navbar-right">
            <div class="form-group">
	      <button class="btn btn-success" type="button" onclick="show()">預覽</button>
              <button class="btn btn-success" type="button" onclick="edit()">編輯</button>
              <input id="filepath" type="text" class="form-control" placeholder="filepath" aria-describedby="basic-addon1" value="main:home">
	      <button class="btn btn-success" type="button" onclick="load()">載入</button>
              <button class="btn btn-success" type="button" onclick="saveFile()">儲存</button>
			</div>
          </form>
        </div>
      </div>
    </nav>

    <div id="showPanel" class="tab-pane panel">
      <div id="htmlBox" class="container"></div>
    </div>
    <center>
      <div id="editPanel" class="tab-pane panel" style="width:90%; height:90%;">
        <br/>
        <textarea id="mdBox" class="form-control" style="width:100%; height:100%"></textarea>
      </div>
    </center>

    <script src="js/jquery.min.js"></script>
    <script src="js/bootstrap.min.js"></script>
    <script src="js/Showdown/Showdown.min.js"></script>
    <script src="js/Showdown/extensions/table.min.js"></script>
    <script src="js/highlight.min.js"></script>
    <script src="config.js"></script>
	<script src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_SVG"></script>
<script>
function path() {
  var tokens = filepath.split(':');
  return { domain:tokens[0], file:tokens[1] };
}

DB = {};

DB.save=function(domain, file, doc) {
  $.ajax({
    type: "POST",
    url: "/db/"+domain+"/"+file,
	timeout:2000,
    data: { obj: doc },
  })
  .done(function(data) {
    alert( "存檔完成!");
  })
  .fail(function() {
    alert( "存檔失敗！" );
  });
}

DB.load=function(domain, file) {
  return $.ajax({
    type: "GET",
    url: "./db/"+domain+"/md/"+file,
	timeout:2000,
    data: {}
  });
}

var filepath, converter;

function init() {
  converter = new Showdown.converter({ extensions: ['table'] });
  if (window.location.hash === '')
    filepath = $('#filepath').val();
  else
    filepath = window.location.hash.substring(1);

  $('#filepath').keypress(function(e) {
  if (e.which == '13') {
     e.preventDefault();
     filepath = $('#filepath').val();
     loadFile(filepath);
   }
  });
  loadFile(filepath);
}

function switchPanel(name) {
  $('.panel').css( "display", "none");
  $('#'+name).css( "display", "block");
}

function mdToHtml(md) {
  md  = "\n"+md+"\n"+config.append+"\n"; 
  md  = md.replace(/(\s)\!\[\[([^\]]*?)\]\]\((.*?)\)/gi, '$1<div class="figure"><img src="db/'+path().domain+'/img/$3"/><p class="caption">$2</p></div>'); // 內部圖片 ![text](file)
  md  = md.replace(/(\s)\[\[([^\]]+?)\]\]\(([^:\)]+):([^:\)]+)\)/gi, '$1<a href="#$3:$4" class="innerLink">$2</a>'); // 內部連結 [text](file.html)
  md  = md.replace(/(\s)\[\[([^\]]+?)\]\]\((.*?)\)/gi, '$1<a href="#'+path().domain+':$3" class="innerLink">$2</a>'); // 內部連結 [text](file.html)
  md  = md.replace(/(\s)\[\[([^\]:]+):([^\]:]+)\]\]/gi, '$1<a href="#$2:$3" class="innerLink">$2:$3</a>'); // 內部連結 [file](file.html)
  md  = md.replace(/(\s)\[\[([^\]:]+?)\]\]/gi, '$1<a href="#'+path().domain+':$2" class="innerLink">$2</a>'); // 內部連結 [file](file.html)
  md  = md.replace(/(\s)\$\$([^$]+?)\$\$/gi, '$1<script type="math/tex">$2</'+'script>');// 數學式 
  return converter.makeHtml(md);
}

var mdNewFile = '# 標題：文件不存在\n\n您可以編輯後存檔！\n## 語法\n* [內部連結](innerLink.html)\n* [外部連結](link)';

function edit() {
  switchPanel('editPanel');
}

function show() {
  var md = $('#mdBox').val();
  var html = mdToHtml(md);
  $('#htmlBox').html(html);
  $('pre code').each(function(i, block) {
    hljs.highlightBlock(block);
  });
  switchPanel('showPanel');
  MathJax.Hub.Queue(["Typeset",MathJax.Hub, "htmlBox"]);
}

function load() {
  filepath = $('#filepath').val();
  loadFile(filepath);
}

function loadFile(filepath) {
  if (filepath === null || filepath === '')
    return;
  $('#filepath').val(filepath);
  var domain = path().domain;
  window.location.hash = '#'+filepath;
  DB.load(path().domain, path().file)
  .done(function(md) {
    $('#mdBox').val(md);
    show();
  })
  .fail(function() {
    $('#mdBox').val(mdNewFile);
    show();
  });
}

function saveFile() {
  var md = $('#mdBox').val();
  DB.save(path().domain, path().file, md);
}

window.onhashchange = function () {
  filepath = window.location.hash.substring(1);
  loadFile(filepath);
}

window.onbeforeunload = function(){}
</script>
</body>
</html>
```

### 後端伺服器：wikiServer.js

```javascript

var c = console;
var fs = require('fs');
var express = require('express');
var path = require('path');
var bodyParser = require("body-parser"); // 參考：http://codeforgeek.com/2014/09/handle-get-post-request-express-4/
var cookieParser = require('cookie-parser')
var session = require('express-session');
var serveIndex = require('serve-index');

var app = express();
var webDir = path.join(__dirname, 'web');
var dbRoot = path.join(webDir, 'db');

app.use(cookieParser());
app.use(session({secret: '@#$TYHaadfa1', resave: false, saveUninitialized: true}));
app.use(bodyParser.urlencoded({ extended: false }));
app.use('/web/', express.static(webDir));
app.use('/web/', serveIndex(webDir, {'icons': true}));

function response(res, code, msg) {
  res.set('Content-Length', ''+msg.length).set('Content-Type', 'text/plain').status(code).send(msg).end();
  c.log("response: code="+code+"\n");
}

app.get("/", function(req, res) {
  res.redirect('/web/wikidown.html');
});

app.post("/db/:db/:name", function(req, res) {
  var db = req.params.db;
  var name = req.params.name;
  var obj = req.body.obj;
  var msg = "db:"+db+" name:"+name+"\n"+obj;
  c.log(msg);
  fs.writeFile(dbRoot+"/"+db+"/md/"+name, obj, function(err) {
    if (err)
      response(res, 500, 'write fail!');
    else
      response(res, 200, 'write success!');
  })
});

var port = process.env.PORT || 3000; // process.env.PORT for Heroku
app.listen(port);

console.log('Server started: http://localhost:'+port);

```

### 結語

以上的 wikidown 系統設計方式，盡可能的將功能放在前端的單一網頁上，而後端則只是用來『儲存或取出資料』而已，筆者覺得這種方法非常的簡潔，不需要像很多 node.js 系統採用了很多 jade, ejs 等樣板所建構的網頁，將功能重點放在後端，然後不停的轉換頁，讓系統變得較為複雜且龐大。

當然、有一得必有一失，將前端功能全放在單一網頁中，就比較難以分工進行，因此在團隊合作上會比較困難。

不過、對於筆者這樣一個人包辦整個系統的『獨立開發者』而言，這種方式似乎是比較方便且敏捷的阿！



