<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>파일 검색기 (폐쇄망용)</title>
<style>
  :root{
    --bg:#F5F7FA;
    --surface:#FFFFFF;
    --ink:#1A2332;
    --muted:#64748B;
    --accent:#0E7490;
    --accent-weak:#E0F2F1;
    --border:#E2E8F0;
    --border-strong:#CBD5E1;
    --hit:#FEF08A;
    --danger:#B91C1C;
    --shadow:0 1px 2px rgba(16,32,48,.06),0 4px 16px rgba(16,32,48,.05);
    --mono:"D2Coding",Consolas,"Courier New",monospace;
    --sans:-apple-system,BlinkMacSystemFont,"Segoe UI","Malgun Gothic","맑은 고딕",sans-serif;
  }
  *{box-sizing:border-box}
  html,body{margin:0;height:100%}
  body{
    font-family:var(--sans);
    background:var(--bg);
    color:var(--ink);
    font-size:14px;
    line-height:1.5;
    display:flex;
    flex-direction:column;
    height:100vh;
    overflow:hidden;
  }

  /* Header */
  header{
    background:var(--ink);
    color:#fff;
    padding:14px 22px;
    display:flex;
    align-items:center;
    gap:14px;
    flex:0 0 auto;
  }
  header .mark{
    width:30px;height:30px;border-radius:7px;
    background:var(--accent);
    display:grid;place-items:center;
    font-weight:700;font-size:15px;
    flex:0 0 auto;
  }
  header h1{font-size:15px;font-weight:600;margin:0;letter-spacing:-.2px}
  header .sub{font-size:12px;color:#94A3B8;margin-left:auto}

  /* Layout */
  .body{flex:1 1 auto;display:flex;min-height:0}
  .panel{
    flex:0 0 320px;
    background:var(--surface);
    border-right:1px solid var(--border);
    padding:20px;
    overflow-y:auto;
  }
  .results{flex:1 1 auto;display:flex;flex-direction:column;min-width:0}

  /* Panel controls */
  .field{margin-bottom:18px}
  .field label{
    display:block;font-size:12px;font-weight:600;
    color:var(--muted);margin-bottom:6px;
    text-transform:uppercase;letter-spacing:.4px;
  }
  input[type=text],select{
    width:100%;padding:9px 11px;
    border:1px solid var(--border-strong);
    border-radius:8px;font-size:14px;font-family:var(--sans);
    background:#fff;color:var(--ink);
    transition:border-color .15s,box-shadow .15s;
  }
  input[type=text]:focus,select:focus{
    outline:none;border-color:var(--accent);
    box-shadow:0 0 0 3px rgba(14,116,144,.12);
  }
  .row2{display:flex;gap:10px}
  .row2>*{flex:1;min-width:0}

  .check{display:flex;align-items:flex-start;gap:9px;cursor:pointer;user-select:none;font-size:13.5px}
  .check input{margin-top:2px;width:16px;height:16px;accent-color:var(--accent);cursor:pointer}
  .check .hint{color:var(--muted);font-size:12px;display:block;margin-top:1px}

  .btn{
    width:100%;padding:11px;border:none;border-radius:8px;
    background:var(--accent);color:#fff;font-weight:600;font-size:14px;
    cursor:pointer;font-family:var(--sans);
    transition:background .15s,opacity .15s;
  }
  .btn:hover{background:#0C6478}
  .btn:disabled{opacity:.5;cursor:not-allowed}
  .btn.secondary{background:#fff;color:var(--ink);border:1px solid var(--border-strong)}
  .btn.secondary:hover{background:#F1F5F9}
  .btn.ghost{background:transparent;color:var(--danger);border:1px solid var(--danger)}
  .btn.ghost:hover{background:#FEF2F2}

  .folder-box{
    border:1px dashed var(--border-strong);border-radius:8px;
    padding:12px;background:#FAFBFC;margin-bottom:10px;
    font-size:13px;word-break:break-all;
  }
  .folder-box.empty{color:var(--muted)}
  .folder-box b{color:var(--accent)}

  hr{border:none;border-top:1px solid var(--border);margin:20px 0}

  /* Results header / status */
  .status{
    flex:0 0 auto;padding:12px 22px;background:var(--surface);
    border-bottom:1px solid var(--border);
    display:flex;align-items:center;gap:16px;font-size:13px;
  }
  .status .count{font-weight:600}
  .status .prog{color:var(--muted);font-variant-numeric:tabular-nums}
  .status .spinner{
    width:15px;height:15px;border:2px solid var(--border);
    border-top-color:var(--accent);border-radius:50%;
    animation:spin .7s linear infinite;flex:0 0 auto;
  }
  @keyframes spin{to{transform:rotate(360deg)}}
  @media (prefers-reduced-motion:reduce){.status .spinner{animation:none}}

  /* Result list */
  .list{flex:1 1 auto;overflow-y:auto;padding:6px 0}
  .item{
    padding:12px 22px;border-bottom:1px solid var(--border);
    display:flex;gap:14px;align-items:flex-start;
    transition:background .1s;
  }
  .item:hover{background:#F8FAFC}
  .ext-tag{
    flex:0 0 auto;width:44px;height:34px;border-radius:6px;
    background:var(--accent-weak);color:var(--accent);
    display:grid;place-items:center;font-size:11px;font-weight:700;
    text-transform:uppercase;margin-top:1px;overflow:hidden;
  }
  .item-main{flex:1 1 auto;min-width:0}
  .item-name{font-weight:600;font-size:14px;color:var(--ink);word-break:break-all}
  .item-name mark{background:var(--hit);color:inherit;border-radius:2px;padding:0 1px}
  .item-path{font-family:var(--mono);font-size:11.5px;color:var(--muted);margin-top:2px;word-break:break-all}
  .item-meta{font-size:12px;color:var(--muted);margin-top:4px;display:flex;gap:14px;flex-wrap:wrap}
  .snippet{
    margin-top:7px;font-family:var(--mono);font-size:11.5px;
    background:#F1F5F9;border-radius:6px;padding:7px 9px;
    color:#334155;white-space:pre-wrap;word-break:break-all;
    border-left:2px solid var(--accent);
  }
  .snippet mark{background:var(--hit);border-radius:2px}
  .copy-btn{
    flex:0 0 auto;background:#fff;border:1px solid var(--border-strong);
    border-radius:6px;padding:5px 9px;font-size:12px;cursor:pointer;
    color:var(--muted);font-family:var(--sans);white-space:nowrap;
  }
  .copy-btn:hover{border-color:var(--accent);color:var(--accent)}

  /* Empty / info states */
  .placeholder{
    flex:1 1 auto;display:grid;place-items:center;text-align:center;
    color:var(--muted);padding:40px;
  }
  .placeholder .big{font-size:15px;color:var(--ink);font-weight:600;margin-bottom:6px}
  .warn{
    margin:22px;padding:16px 18px;border-radius:10px;
    background:#FEF2F2;border:1px solid #FECACA;color:#7F1D1D;
    font-size:13.5px;line-height:1.6;
  }
  .warn b{color:var(--danger)}
</style>
</head>
<body>
<header>
  <div class="mark">검</div>
  <h1>파일 검색기</h1>
  <span class="sub">폐쇄망 · 오프라인 · 설치 불필요</span>
</header>

<div class="body">
  <!-- 설정 패널 -->
  <aside class="panel">
    <div class="field">
      <label>검색 대상 폴더</label>
      <div id="folderBox" class="folder-box empty">선택된 폴더가 없습니다.</div>
      <button id="pickBtn" class="btn secondary">폴더 선택</button>
    </div>

    <hr>

    <div class="field">
      <label>파일명 검색어</label>
      <input type="text" id="nameQuery" placeholder="예: 보고서, 2024, .pdf">
    </div>

    <div class="field">
      <label class="check">
        <input type="checkbox" id="contentToggle">
        <span>파일 내용까지 검색
          <span class="hint">txt, csv, log, md, 소스코드 등 텍스트 파일 대상</span>
        </span>
      </label>
    </div>

    <div class="field" id="contentField" style="display:none">
      <label>내용 검색어</label>
      <input type="text" id="contentQuery" placeholder="파일 안에서 찾을 문구">
    </div>

    <hr>

    <div class="field">
      <label>확장자 필터 (선택)</label>
      <input type="text" id="extFilter" placeholder="예: pdf, docx, hwp (쉼표 구분)">
    </div>

    <div class="field">
      <label>수정일 (이후)</label>
      <input type="text" id="dateFilter" placeholder="예: 2024-01-01" inputmode="numeric">
    </div>

    <div class="field">
      <label>대소문자</label>
      <label class="check">
        <input type="checkbox" id="caseToggle">
        <span>대소문자 구분</span>
      </label>
    </div>

    <button id="searchBtn" class="btn" disabled>검색</button>
    <div style="height:8px"></div>
    <button id="cancelBtn" class="btn ghost" style="display:none">검색 중지</button>
  </aside>

  <!-- 결과 영역 -->
  <section class="results">
    <div class="status">
      <span id="spinner" class="spinner" style="display:none"></span>
      <span class="count" id="countLabel">검색 결과 0건</span>
      <span class="prog" id="progLabel"></span>
    </div>

    <div id="unsupported" class="warn" style="display:none">
      <b>이 브라우저에서는 폴더 접근 기능을 사용할 수 없습니다.</b><br>
      Microsoft Edge 또는 Google Chrome에서 이 파일을 열어 주세요.
      (Internet Explorer, 구버전 Firefox 등은 지원되지 않습니다.)
    </div>

    <div id="list" class="list"></div>

    <div id="placeholder" class="placeholder">
      <div>
        <div class="big">왼쪽에서 폴더를 선택하고 검색어를 입력하세요.</div>
        <div>파일명 또는 파일 내용으로 찾을 수 있습니다.</div>
      </div>
    </div>
  </section>
</div>

<script>
(function(){
  "use strict";

  // ---- 지원 여부 확인 ----
  var supported = typeof window.showDirectoryPicker === "function";
  if(!supported){
    document.getElementById("unsupported").style.display="block";
    document.getElementById("placeholder").style.display="none";
    document.getElementById("pickBtn").disabled=true;
  }

  // ---- 요소 ----
  var dirHandle=null, cancelFlag=false, running=false;
  var el = function(id){return document.getElementById(id);};
  var folderBox=el("folderBox"), pickBtn=el("pickBtn"), searchBtn=el("searchBtn"),
      cancelBtn=el("cancelBtn"), list=el("list"), placeholder=el("placeholder"),
      countLabel=el("countLabel"), progLabel=el("progLabel"), spinner=el("spinner"),
      contentToggle=el("contentToggle"), contentField=el("contentField");

  // 내용 검색 대상 텍스트 확장자
  var TEXT_EXT = ["txt","csv","log","md","json","xml","html","htm","css","js","ts",
    "py","java","c","cpp","h","cs","php","rb","go","rs","sql","yaml","yml","ini",
    "conf","cfg","sh","bat","tsv","srt","vtt"];
  var MAX_CONTENT_BYTES = 8*1024*1024; // 내용 검색 시 8MB 초과 파일은 건너뜀

  contentToggle.addEventListener("change",function(){
    contentField.style.display = contentToggle.checked ? "block" : "none";
  });

  // ---- 폴더 선택 ----
  pickBtn.addEventListener("click", async function(){
    try{
      dirHandle = await window.showDirectoryPicker();
      folderBox.classList.remove("empty");
      folderBox.innerHTML = "선택됨: <b>"+escapeHtml(dirHandle.name)+"</b>";
      searchBtn.disabled=false;
    }catch(e){ /* 사용자가 취소 */ }
  });

  // ---- 검색 실행 ----
  searchBtn.addEventListener("click", runSearch);
  cancelBtn.addEventListener("click", function(){ cancelFlag=true; });

  ["nameQuery","contentQuery","extFilter","dateFilter"].forEach(function(id){
    el(id).addEventListener("keydown",function(e){ if(e.key==="Enter") runSearch(); });
  });

  async function runSearch(){
    if(!dirHandle || running) return;
    var nameQ = el("nameQuery").value.trim();
    var contentOn = contentToggle.checked;
    var contentQ = el("contentQuery").value.trim();
    var caseSensitive = el("caseToggle").checked;

    if(!nameQ && !(contentOn && contentQ)){
      alert("파일명 또는 내용 검색어를 하나 이상 입력하세요.");
      return;
    }

    // 필터 파싱
    var extList = el("extFilter").value.split(",").map(function(s){
      return s.trim().replace(/^\./,"").toLowerCase();
    }).filter(Boolean);

    var afterDate = null;
    var dv = el("dateFilter").value.trim();
    if(dv){ var d=new Date(dv); if(!isNaN(d)) afterDate=d.getTime(); }

    // 상태 초기화
    running=true; cancelFlag=false;
    list.innerHTML=""; placeholder.style.display="none";
    spinner.style.display="block"; cancelBtn.style.display="block";
    searchBtn.disabled=true; pickBtn.disabled=true;
    var found=0, scanned=0;

    var nameNeedle = caseSensitive ? nameQ : nameQ.toLowerCase();
    var contentNeedle = caseSensitive ? contentQ : contentQ.toLowerCase();

    try{
      for await (var entry of walk(dirHandle, dirHandle.name)){
        if(cancelFlag) break;
        scanned++;
        if(scanned % 40 === 0){
          progLabel.textContent = scanned+"개 검사 중...";
          await pause(); // UI 응답성 유지
        }

        var name = entry.name, path = entry.path;
        var ext = (name.split(".").pop()||"").toLowerCase();

        // 확장자 필터
        if(extList.length && extList.indexOf(ext)===-1) continue;

        // 파일명 매칭
        var nameHit = false;
        if(nameQ){
          var hay = caseSensitive ? name : name.toLowerCase();
          nameHit = hay.indexOf(nameNeedle) !== -1;
        }

        // 수정일 / 크기 정보
        var file=null, mtime=null, size=null;
        function needFile(){ return file || (contentOn && contentQ) || afterDate!==null; }

        if(needFile()){
          try{ file = await entry.handle.getFile(); mtime=file.lastModified; size=file.size; }
          catch(e){ continue; }
        }
        if(afterDate!==null && mtime!==null && mtime < afterDate) continue;

        // 내용 매칭
        var snippet=null, contentHit=false;
        if(contentOn && contentQ && TEXT_EXT.indexOf(ext)!==-1 && file){
          if(size<=MAX_CONTENT_BYTES){
            try{
              var text = await file.text();
              var thay = caseSensitive ? text : text.toLowerCase();
              var pos = thay.indexOf(contentNeedle);
              if(pos!==-1){ contentHit=true; snippet=makeSnippet(text,pos,contentQ.length); }
            }catch(e){}
          }
        }

        // 조건: 파일명 검색만 → nameHit / 둘 다 켜짐 → 하나라도 매칭
        var match=false;
        if(nameQ && contentOn && contentQ) match = nameHit || contentHit;
        else if(nameQ) match = nameHit;
        else if(contentOn && contentQ) match = contentHit;

        if(match){
          found++;
          addItem({name:name, path:path, ext:ext, size:size, mtime:mtime,
                   snippet:snippet, nameHit:nameHit, nameQ:nameQ, caseSensitive:caseSensitive});
          countLabel.textContent = "검색 결과 "+found+"건";
        }
      }
    }catch(e){
      alert("검색 중 오류가 발생했습니다: "+e.message);
    }

    // 종료
    running=false;
    spinner.style.display="none"; cancelBtn.style.display="none";
    searchBtn.disabled=false; pickBtn.disabled=false;
    progLabel.textContent = cancelFlag ? "(중지됨) 총 "+scanned+"개 검사" : "총 "+scanned+"개 검사 완료";
    countLabel.textContent = "검색 결과 "+found+"건";
    if(found===0){
      placeholder.style.display="grid";
      placeholder.innerHTML='<div><div class="big">조건에 맞는 파일이 없습니다.</div><div>검색어나 필터를 바꿔서 다시 시도해 보세요.</div></div>';
    }
  }

  // ---- 폴더 재귀 순회 ----
  async function* walk(handle, path){
    for await (var entry of handle.values()){
      if(cancelFlag) return;
      var childPath = path + "/" + entry.name;
      if(entry.kind === "file"){
        yield {name:entry.name, path:childPath, handle:entry};
      }else if(entry.kind === "directory"){
        try{ yield* walk(entry, childPath); }catch(e){}
      }
    }
  }

  // ---- 결과 항목 렌더링 ----
  function addItem(r){
    var div=document.createElement("div");
    div.className="item";

    var nameHtml = r.nameHit ? highlight(r.name, r.nameQ, r.caseSensitive) : escapeHtml(r.name);
    var extLabel = escapeHtml((r.ext||"?").slice(0,4));

    var meta = [];
    if(r.size!=null) meta.push(formatSize(r.size));
    if(r.mtime!=null) meta.push(formatDate(r.mtime));

    div.innerHTML =
      '<div class="ext-tag">'+extLabel+'</div>'+
      '<div class="item-main">'+
        '<div class="item-name">'+nameHtml+'</div>'+
        '<div class="item-path">'+escapeHtml(r.path)+'</div>'+
        '<div class="item-meta">'+meta.map(function(m){return "<span>"+m+"</span>";}).join("")+'</div>'+
        (r.snippet ? '<div class="snippet">'+r.snippet+'</div>' : '')+
      '</div>'+
      '<button class="copy-btn">경로 복사</button>';

    div.querySelector(".copy-btn").addEventListener("click",function(){
      copyText(r.path, this);
    });
    list.appendChild(div);
  }

  // ---- 내용 스니펫 (앞뒤 문맥 + 강조) ----
  function makeSnippet(text, pos, len){
    var start=Math.max(0,pos-40), end=Math.min(text.length,pos+len+60);
    var pre=(start>0?"…":"")+text.slice(start,pos);
    var hit=text.slice(pos,pos+len);
    var post=text.slice(pos+len,end)+(end<text.length?"…":"");
    return escapeHtml(pre)+"<mark>"+escapeHtml(hit)+"</mark>"+escapeHtml(post);
  }

  // ---- 유틸 ----
  function highlight(str, q, cs){
    if(!q) return escapeHtml(str);
    var hay=cs?str:str.toLowerCase(), needle=cs?q:q.toLowerCase();
    var i=hay.indexOf(needle);
    if(i===-1) return escapeHtml(str);
    return escapeHtml(str.slice(0,i))+"<mark>"+escapeHtml(str.slice(i,i+q.length))+"</mark>"+escapeHtml(str.slice(i+q.length));
  }
  function escapeHtml(s){
    return String(s).replace(/[&<>"']/g,function(c){
      return {"&":"&amp;","<":"&lt;",">":"&gt;",'"':"&quot;","'":"&#39;"}[c];
    });
  }
  function formatSize(b){
    if(b<1024) return b+" B";
    if(b<1048576) return (b/1024).toFixed(1)+" KB";
    if(b<1073741824) return (b/1048576).toFixed(1)+" MB";
    return (b/1073741824).toFixed(2)+" GB";
  }
  function formatDate(ms){
    var d=new Date(ms), p=function(n){return (n<10?"0":"")+n;};
    return d.getFullYear()+"-"+p(d.getMonth()+1)+"-"+p(d.getDate())+" "+p(d.getHours())+":"+p(d.getMinutes());
  }
  function copyText(t, btn){
    var done=function(){ var o=btn.textContent; btn.textContent="복사됨"; setTimeout(function(){btn.textContent=o;},1200); };
    if(navigator.clipboard && navigator.clipboard.writeText){
      navigator.clipboard.writeText(t).then(done).catch(function(){ fallbackCopy(t); done(); });
    }else{ fallbackCopy(t); done(); }
  }
  function fallbackCopy(t){
    var ta=document.createElement("textarea"); ta.value=t;
    ta.style.position="fixed"; ta.style.opacity="0"; document.body.appendChild(ta);
    ta.select(); try{document.execCommand("copy");}catch(e){} document.body.removeChild(ta);
  }
  function pause(){ return new Promise(function(r){ setTimeout(r,0); }); }
})();
</script>
</body>
</html>
