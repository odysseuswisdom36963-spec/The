
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=0.50, user-scalable=yes, minimum-scale=0.2, maximum-scale=3.0">
  <title>K52 Refined Panels</title>
  <style>
    body { font-family: Arial, sans-serif; margin:0; padding:0; background:#000; color:#fff; }

    .toolbar {
      background:#333; color:#fff; padding:10px;
      display:flex; gap:10px; flex-wrap:wrap;
    }
    .toolbar button {
      flex:1; padding:10px; font-size:16px;
      border:none; border-radius:6px; cursor:pointer;
      background:#444; color:#fff;
    }
    .toolbar button:hover { background:#666; }

    .panel {
      display:none;
      position:fixed; top:0; left:0; right:0;
      height:auto; background:#fff; color:#000;
      z-index:1000; padding:14px 18px;
      box-shadow:0 4px 12px rgba(0,0,0,0.2);
      border-radius:12px;
    }

    #optionsPanel h2 { margin-top:0; margin-bottom:12px; font-size:18px; color:#222; }
    #optionsPanel .options-container {
      display:flex; flex-wrap:wrap; gap:14px 20px; align-items:center;
    }
    #optionsPanel label {
      display:flex; flex-direction:column; font-size:14px; font-weight:600; color:#333;
    }
    #optionsPanel input[type="number"],
    #optionsPanel input[type="text"],
    #optionsPanel select {
      width:80px; padding:6px 8px; border:1px solid #ccc; border-radius:8px;
      font-size:14px; text-align:center;
    }
    #optionsPanel input[type="range"] { width:120px; }

    #speechRateValue { font-size:13px; color:#555; margin-left:6px; }

    .data-buttons { display:flex; gap:10px; flex-wrap:wrap; }
    .data-buttons button {
      width:80px; height:80px; font-size:14px; border:none; border-radius:8px; cursor:pointer;
      background:#333; color:#fff; display:flex; align-items:center; justify-content:center;
    }
    .data-buttons button:hover { background:#555; }

    .popup {
  position:absolute; 
  background:#fff; 
  border:1px solid #ccc;
  border-radius:8px; 
  padding:10px; 
  z-index:100000; /* MUCH higher than overlay */
  color:#000;
  box-shadow:0 4px 10px rgba(0,0,0,0.25);
}

    .spreadsheet-container { overflow-x:auto; padding:20px; }
    table { border-collapse:collapse; width:100%; background:white; color:#000; }
    th,td { border:1px solid #ccc; padding:6px; text-align:center; min-width:120px; }
    #spreadsheet td.highlight {
      background:#ccffcc !important; color:#003300 !important; font-weight:bold; border:2px solid #006600;
    }

    #spreadsheet th:nth-child(even), #spreadsheet td:nth-child(even) {
      background:#fff; color:#000;
    }
    #spreadsheet th:nth-child(odd), #spreadsheet td:nth-child(odd) {
      background:#222; color:#fff;
    }

    /* Reader panel */
    #readerPanel {
      display:none; padding:20px; background:#111; color:#fff;
    }
    #readerInput {
      width:100%; min-height:100px; padding:10px; border-radius:6px;
    }

   #readerOverlay {
  display:none;
  position:fixed;
  top:0;
  left:0;
  width:100%;
  height:100%;
  background-color: #fff; /* white background */
  color: #000;            /* black text */
  color:#000; /* text black for readability */
  z-index:9999;
  overflow-y:auto;
  padding:40px 20px;
  font-family:sans-serif;
  line-height:1.6;
  opacity:0;
  transition: opacity 0.3s ease;
}
  
  
  
  #closeReaderBtn {
  position: fixed;
  top: 20px;
  right: 20px;
  z-index: 10001; /* above overlay */
  padding: 8px 12px;
  font-size: 16px;
  font-weight: bold;
  background: #ff4444; /* bright red */
  color: #fff;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  box-shadow: 0 2px 6px rgba(0,0,0,0.3);
}

#closeReaderBtn:hover {
  background: #ff0000;
}
  
  
  
  /* === Box Highlighter transplant === */
.word { display:inline; white-space:pre-wrap; }
.word.selected { background: rgba(255,235,59,0.65); border-radius:3px; }
.word.saved { background: rgba(46, 204, 113, 0.14); border-radius:3px; }

.sel-box {
  position: absolute; z-index: 1200; border: 2px solid rgba(0,116,217,0.95);
  box-shadow: 0 6px 18px rgba(0,0,0,0.20);
  background: rgba(0,116,217,0.12); touch-action: none;
  display: flex; align-items: stretch; justify-content: stretch;
}
.sel-handle {
  position: absolute; width:12px; height:12px; background:#fff; border:2px solid rgba(0,116,217,0.95);
  border-radius:2px; box-sizing:border-box;
}
.hdl-tl { left:-8px; top:-8px; cursor:nwse-resize; }
.hdl-tr { right:-8px; top:-8px; cursor:nesw-resize; }
.hdl-bl { left:-8px; bottom:-8px; cursor:nesw-resize; }
.hdl-br { right:-8px; bottom:-8px; cursor:nwse-resize; }

.sel-toolbar {
  position: absolute; z-index:1300; background:#0f1724; color:#fff; padding:8px; border-radius:8px;
  font-size:13px; box-shadow: 0 8px 20px rgba(0,0,0,0.45); display:flex; gap:8px; align-items:center;
}
.sel-toolbar label { color:#dfe7f6; font-size:12px; display:flex; gap:6px; align-items:center; }
  
  </style>
</head>
<body>
  <div class="toolbar">
    <button id="playBtn">▶️ Play</button>
    <button id="openOptions">⚙️ Options</button>
    <button id="openData">📂 Data</button>
      <button id="openReader">  text viewer </button>
    
    <button id="stopBtn">⏹ Stop</button>
    <button id="backBtn">⏮ Back</button>
  </div>

  <!-- Options Panel -->
  <div class="panel" id="optionsPanel">
    <h2>⚙️ Options</h2>
    <div class="options-container">
      <label>Repeat Rows <input type="number" id="repeatRows" min="1" value="6"></label>
      <label>Start Row <input type="number" id="startRow" min="1" value="1"></label>
      <label>End Row <input type="number" id="endRow" min="1" value="100"></label>
      <label>Start Col <input type="number" id="startCol" min="1" value="1"></label>
      <label>End Col <input type="number" id="endCol" min="1" value="2"></label>
      <label>Speech Speed
        <div style="display:flex; align-items:center; gap:6px;">
          <input type="range" id="speechRate" min="0.5" max="3.0" value="1.0" step="0.1">
          <span id="speechRateValue">1.0x</span>
        </div>
      </label>
    </div>
  </div>

  <!-- Data Panel -->
  <div class="panel" id="dataPanel">
    <div class="data-buttons">
      <button id="uploadTableBtn">📂 Upload Table</button>
      <input type="file" id="uploadTableInput" accept=".txt,.json" style="display:none;">
      <button id="saveTableBtn">💾 Save Table</button>
      <button id="uploadColumnBtn">📥 Upload Column</button>
      <button id="extractColumnBtn">📤 Extract Column</button>
    </div>
  </div>

  <!-- Reader Panel -->
  <div id="readerPanel">
    <h2>Reader Input</h2>
    <textarea id="readerInput" placeholder="Paste text here..."></textarea>
  </div>

  <!-- Full-screen overlay -->
  <!-- Highlighter Overlay -->
  <!-- Ultra-compact paste/load input -->
<div id="miniPasteContainer" style="position:fixed; bottom:15px; right:15px; z-index:2000; display:flex; gap:4px; align-items:center;">
  <input id="miniPasteInput" type="text" placeholder="Paste text..." style="width:120px; padding:4px 6px; font-size:13px; border-radius:4px; border:none; background:#222; color:#fff;">
  <button id="miniLoadBtn" style="padding:4px 6px; font-size:13px; border:none; border-radius:4px; background:#4caf50; color:#fff; cursor:pointer;">Load</button>
</div>
<div id="readerOverlay" aria-hidden="true" style="display:none; opacity:0;">
  <div id="readerShell" style="position:absolute; left:50%; top:6%; transform:translateX(-50%);
       width:min(1100px,92%); background:#fff; border-radius:10px; padding:14px; 
       box-shadow:0 18px 40px rgba(0,0,0,0.35); max-height:82vh; overflow:hidden;">
    <div style="display:flex; justify-content:space-between; align-items:center; gap:12px; margin-bottom:8px;">
      <strong>Reader</strong>
      <button id="closeReaderBtn">✖ Close</button>
    </div>
    <div id="readerDisplay" tabindex="0" style="position:relative; overflow:auto; height:56vh; padding:18px; 
         border-radius:8px; border:1px solid #e6eef8; background:#fff; font-size:16px; line-height:1.65; color:#111;"></div>
    <small style="color:#555">Tip: drag a blue box over words, then “Add” to send them into the table.</small>
  </div>
</div>
  
  
  
  <div class="spreadsheet-container">
    <table id="spreadsheet">
      <thead>
        <tr>
          <th></th>
          <th>A<br><select id="voiceCol1"></select></th>
          <th>B<br><select id="voiceCol2"></select></th>
          <th>C<br><select id="voiceCol3"></select></th>
          <th>D<br><select id="voiceCol4"></select></th>
          <th>E<br><select id="voiceCol5"></select></th>
          <th>F<br><select id="voiceCol6"></select></th>
        </tr>
      </thead>
      <tbody id="spreadsheetBody"></tbody>
    </table>
  </div>

  <script>
    const tbody=document.getElementById('spreadsheetBody');
    for(let i=0;i<2000;i++){
      const row=document.createElement('tr');
      const th=document.createElement('th');
      const rowBtn=document.createElement('button');
      rowBtn.textContent = i+1;
      Object.assign(rowBtn.style,{cursor:"pointer",background:"transparent",border:"none",padding:"0",margin:"0",color:"inherit",font:"inherit"});
      rowBtn.onclick=(e)=>{
        e.stopPropagation();
        showPopup(rowBtn,`
          <button id="setStartRowBtn">Set as Start Row (${i+1})</button><br>
          <button id="setEndRowBtn">Set as End Row (${i+1})</button>
        `);
        setTimeout(()=>{
          const s=document.getElementById('setStartRowBtn');
          const eBtn=document.getElementById('setEndRowBtn');
          if(s) s.onclick=()=>{ setRow('start',i+1); };
          if(eBtn) eBtn.onclick=()=>{ setRow('end',i+1); };
        },10);
      };
      th.appendChild(rowBtn);
      row.appendChild(th);
      for(let j=0;j<6;j++){const td=document.createElement('td');td.contentEditable=true;row.appendChild(td);}
      tbody.appendChild(row);
    }

    // Universal popup helper
    function showPopup(button, content){
      document.querySelectorAll('.popup').forEach(p=>p.remove());
      const rect=button.getBoundingClientRect();
      const popup=document.createElement('div');
      popup.className='popup';
      popup.style.top=(rect.bottom+window.scrollY+5)+'px';
      popup.style.left=(rect.left+window.scrollX)+'px';
      popup.innerHTML=content;
      document.body.appendChild(popup);
    }

    function setRow(type,rowNum){
      if(type==='start') document.getElementById('startRow').value=rowNum;
      if(type==='end') document.getElementById('endRow').value=rowNum;
      document.querySelectorAll('.popup').forEach(p=>p.remove());
    }

    // Prevent auto-centering
    document.addEventListener('focusin',(e)=>{
      if(e.target && (e.target.tagName==='INPUT'||e.target.tagName==='BUTTON'||e.target.isContentEditable)){
        try{e.target.scrollIntoView({block:'nearest',inline:'nearest'});}catch{}
      }
    });

    // Voice setup
    let voices=[];
    function refreshVoiceLists(){
      for(let i=1;i<=6;i++){
        const sel=document.getElementById('voiceCol'+i); if(!sel) continue;
        const prev=sel.value; sel.innerHTML='';
        voices.forEach(v=>{
          const opt=document.createElement('option');
          opt.value=v.name; opt.textContent=`${v.name} (${v.lang})${v.default?' [default]':''}`;
          sel.appendChild(opt);
        });
        if(voices.find(v=>v.name===prev)) sel.value=prev;
      }
    }
    function ensureVoicesReady(){voices=speechSynthesis.getVoices();
      if(voices.length>0) refreshVoiceLists(); else setTimeout(ensureVoicesReady,200);}
    window.speechSynthesis.onvoiceschanged=ensureVoicesReady; ensureVoicesReady();

    // Panels open/close
    function openPanel(id){document.getElementById(id).style.display='block';document.addEventListener('click',outsideClickListener);}
    function closePanel(id){document.getElementById(id).style.display='none';document.removeEventListener('click',outsideClickListener);}
    function outsideClickListener(e){
      if(!e.target.closest('.panel') && !e.target.closest('.toolbar button') && !e.target.closest('.popup')){
        closePanel('optionsPanel'); closePanel('dataPanel');
        document.querySelectorAll('.popup').forEach(p=>p.remove());
      }
    }
    document.getElementById('openOptions').onclick=(e)=>{e.stopPropagation();openPanel('optionsPanel');};
    document.getElementById('openData').onclick=(e)=>{e.stopPropagation();openPanel('dataPanel');};
    window.addEventListener('keydown',e=>{if(e.key==='Escape'){closePanel('optionsPanel');closePanel('dataPanel');document.querySelectorAll('.popup').forEach(p=>p.remove());}});

    // // Reader speech and toggle (UNCHANGED)
let stopFlag=false; let currentRow=0;
function clearHighlights(){document.querySelectorAll('#spreadsheet td.highlight').forEach(td=>td.classList.remove('highlight'))}

function speakRow(rowIndex){return new Promise(resolve=>{
  const row=tbody.rows[rowIndex]; if(!row) return resolve();
  let col=parseInt(document.getElementById('startCol').value);
  const endCol=parseInt(document.getElementById('endCol').value);
  function speakNext(){
    if(col>endCol) return resolve();
    const cell=row.cells[col]; if(!cell){col++; return speakNext();}
    const text=cell.innerText.trim();
    if(text){
      const u=new SpeechSynthesisUtterance(text);
      u.rate=parseFloat(document.getElementById('speechRate').value);
      const vs=document.getElementById('voiceCol'+col);
      if(vs&&vs.value){const v=voices.find(vo=>vo.name===vs.value); if(v){u.voice=v;u.lang=v.lang;}}
      u.onstart=()=>{clearHighlights(); cell.classList.add('highlight');};
      u.onend=()=>{col++; speakNext();};
      speechSynthesis.speak(u);
    } else {col++; speakNext();}
  }
  speakNext();
});}

async function playRows(){
  stopFlag=false;
  const startRow=parseInt(document.getElementById('startRow').value)-1;
  const endRow=parseInt(document.getElementById('endRow').value)-1;
  const repeat=parseInt(document.getElementById('repeatRows').value);
  for(let r=startRow;r<=endRow;r++){for(let k=0;k<repeat;k++){if(stopFlag)return;currentRow=r;await speakRow(r);}}
}

document.getElementById('playBtn').onclick=playRows;
document.getElementById('stopBtn').onclick=()=>{stopFlag=true;speechSynthesis.cancel();clearHighlights();};
document.getElementById('backBtn').onclick=()=>{if(currentRow>0){currentRow--;speakRow(currentRow);} };
document.getElementById('speechRate').oninput=e=>{
  document.getElementById('speechRateValue').textContent=parseFloat(e.target.value).toFixed(1)+'x';
};
      
      document.getElementById('uploadColumnBtn').onclick=(e)=>{
  e.stopPropagation();
  showPopup(e.target,`
    <textarea id="uploadColumnText" rows="4" placeholder="Paste words, one per line..."></textarea><br>
    <label>Target Column: <select id="uploadColumnSelect">
      <option value="0">Column A</option><option value="1">Column B</option>
      <option value="2">Column C</option><option value="3">Column D</option>
      <option value="4">Column E</option><option value="5">Column F</option>
    </select></label><br>
    <button id="confirmUpload">Upload</button>
  `);
  document.getElementById('confirmUpload').onclick=()=>{
    const text=document.getElementById('uploadColumnText').value;
    const col=parseInt(document.getElementById('uploadColumnSelect').value);
    if(!text) return;
    const lines=text.split(/\r?\n/);
    for(let i=0;i<tbody.rows.length;i++){
      const cell=tbody.rows[i].cells[col+1]; if(cell) cell.textContent='';
    }
    lines.forEach((line,i)=>{
      if(tbody.rows[i]){
        const cell=tbody.rows[i].cells[col+1];
        if(cell) cell.textContent=line.trim();
      }
    });
  };
};
      
      document.getElementById('extractColumnBtn').onclick=(e)=>{
  e.stopPropagation();
  showPopup(e.target,`
    <label>Choose Column: <select id="extractColumnSelect">
      <option value="0">Column A</option><option value="1">Column B</option>
      <option value="2">Column C</option><option value="3">Column D</option>
      <option value="4">Column E</option><option value="5">Column F</option>
    </select></label><br>
    <button id="confirmExtract">Extract</button>
  `);
  document.getElementById('confirmExtract').onclick=()=>{
    const col=parseInt(document.getElementById('extractColumnSelect').value);
    const values=[];
    for(let i=0;i<tbody.rows.length;i++){
      const cell=tbody.rows[i].cells[col+1];
      values.push(cell?cell.textContent:'');
    }
    let lastFilled=values.length-1;
    while(lastFilled>=0 && !values[lastFilled]) lastFilled--;
    const output=values.slice(0,lastFilled+1).join("\n");
    navigator.clipboard.writeText(output).then(()=>{
      alert(`Extracted Column ${String.fromCharCode(65+col)} copied to clipboard.`);
      document.querySelectorAll('.popup').forEach(p=>p.remove());
    });
  };
};
      
      
      function uploadTable(file){
  const reader=new FileReader();
  reader.onload=e=>{
    try{
      const parsed=JSON.parse(e.target.result);
      if(!parsed.rows) return;
      for(let i=0;i<parsed.rows.length;i++){
        const row=tbody.rows[i]; if(!row) continue;
        const cells=row.querySelectorAll('td');
        parsed.rows[i].forEach((val,j)=>{ if(cells[j]) cells[j].textContent=val; });
      }
    }catch(err){ alert('Invalid file'); }
  };
  reader.readAsText(file,'utf-8');
}
document.getElementById('uploadTableBtn').onclick=()=>document.getElementById('uploadTableInput').click();
document.getElementById('uploadTableInput').onchange=e=>{
  if(e.target.files[0]) uploadTable(e.target.files[0]);
  e.target.value='';
};
      
      
      
      
      
      document.getElementById('saveTableBtn').onclick = () => {
  const rows = [];
  const tbody = document.getElementById('spreadsheetBody');
  
  for (let i = 0; i < tbody.rows.length; i++) {
    const rowCells = [];
    for (let j = 1; j < tbody.rows[i].cells.length; j++) { // skip row number th
      rowCells.push(tbody.rows[i].cells[j].textContent.trim());
    }
    rows.push(rowCells);
  }

  const tableData = { rows: rows };
  const blob = new Blob([JSON.stringify(tableData, null, 2)], { type: 'application/json' });
  const url = URL.createObjectURL(blob);

  const a = document.createElement('a');
  a.href = url;

  // Give a unique filename each time
  const timestamp = new Date().toISOString().replace(/[:.]/g, '-');
  a.download = `spreadsheet_${timestamp}.json`;

  a.click();
  URL.revokeObjectURL(url);
};
      
      
      
      
      
      /* === Highlighter transplant === */
const readerDisplay = document.getElementById('readerDisplay');
let selBox=null, selToolbar=null, currentAction=null, resizeDir=null, moveOffset={x:0,y:0}, startX=0,startY=0;

/* Render words into spans */
function renderTextToReader(text){
  readerDisplay.innerHTML='';
  const tokens=text.split(/(\s+)/);
  for(let i=0;i<tokens.length;i++){
    let t=tokens[i]; if(!t) continue;
    if(/\s+/.test(t)){ readerDisplay.appendChild(document.createTextNode(t)); continue; }
    let word=t; if(i+1<tokens.length && /\s+/.test(tokens[i+1])){ word+=tokens[i+1]; i++; }
    const span=document.createElement('span'); span.className='word'; span.textContent=word;
    readerDisplay.appendChild(span);
  }
}

/* Open/close overlay */
document.getElementById('openReader').onclick=()=>{
  const text=document.getElementById('readerInput').value||'';
  renderTextToReader(text);
  document.getElementById('readerOverlay').style.display='block';
  setTimeout(()=>document.getElementById('readerOverlay').style.opacity='1',20);
};
document.getElementById('closeReaderBtn').onclick=()=>{
  document.getElementById('readerOverlay').style.opacity='0';
  setTimeout(()=>document.getElementById('readerOverlay').style.display='none',220);
};

/* Utility */
function rectsOverlap(a,b){return !(a.right<b.left||a.left>b.right||a.bottom<b.top||a.top>b.bottom);}
function clearTempSelection(){document.querySelectorAll('.word.selected').forEach(s=>s.classList.remove('selected'));}

/* Selection box creation */
function createSelBox(x,y){
  if(selBox){ selBox.remove(); selBox=null; hideToolbar(); clearTempSelection(); }
  selBox=document.createElement('div'); selBox.className='sel-box';
  selBox.style.left=x+'px'; selBox.style.top=y+'px'; selBox.style.width='0px'; selBox.style.height='0px';
  ['tl','tr','bl','br'].forEach(pos=>{
    const hd=document.createElement('div'); hd.className='sel-handle hdl-'+pos; hd.dataset.handle=pos;
    hd.onpointerdown=handlePointerDownOnHandle; selBox.appendChild(hd);
  });
  selBox.onpointerdown=(e)=>{if(e.target.classList.contains('sel-handle'))return;
    e.preventDefault(); currentAction='moving'; selBox.setPointerCapture(e.pointerId);
    const r=selBox.getBoundingClientRect(); moveOffset.x=e.clientX-r.left; moveOffset.y=e.clientY-r.top;};
  document.body.appendChild(selBox);
}
function handlePointerDownOnHandle(e){e.preventDefault();resizeDir=e.target.dataset.handle;currentAction='resizing';e.target.setPointerCapture(e.pointerId);}

/* Drawing logic */
readerDisplay.addEventListener('pointerdown',(e)=>{if(e.target.closest('.sel-box')||e.target.closest('.sel-toolbar'))return;
  e.preventDefault();currentAction='drawing';startX=e.clientX;startY=e.clientY;createSelBox(startX,startY);});
document.addEventListener('pointermove',(e)=>{if(!selBox)return;
  if(currentAction==='drawing'){const minX=Math.min(startX,e.clientX),minY=Math.min(startY,e.clientY);
    selBox.style.left=minX+'px'; selBox.style.top=minY+'px';
    selBox.style.width=Math.max(8,Math.abs(e.clientX-startX))+'px';
    selBox.style.height=Math.max(8,Math.abs(e.clientY-startY))+'px';}
  else if(currentAction==='moving'){selBox.style.left=(e.clientX-moveOffset.x)+'px';selBox.style.top=(e.clientY-moveOffset.y)+'px';}
  else if(currentAction==='resizing'){const r=selBox.getBoundingClientRect();let l=r.left,t=r.top,R=r.right,B=r.bottom;
    if(resizeDir==='tl'){l=Math.min(e.clientX,R-12);t=Math.min(e.clientY,B-12);}
    if(resizeDir==='tr'){R=Math.max(e.clientX,l+12);t=Math.min(e.clientY,B-12);}
    if(resizeDir==='bl'){l=Math.min(e.clientX,R-12);B=Math.max(e.clientY,t+12);}
    if(resizeDir==='br'){R=Math.max(e.clientX,l+12);B=Math.max(e.clientY,t+12);}
    selBox.style.left=l+'px';selBox.style.top=t+'px';selBox.style.width=Math.max(8,R-l)+'px';selBox.style.height=Math.max(8,B-t)+'px';}
  requestAnimationFrame(()=>markIntersectingWords());});
document.addEventListener('pointerup',()=>{if(selBox&&currentAction)showToolbarForBox();currentAction=null;resizeDir=null;});

function markIntersectingWords(){clearTempSelection();if(!selBox)return;
  const selRect=selBox.getBoundingClientRect();document.querySelectorAll('#readerDisplay .word').forEach(w=>{
    if(rectsOverlap(w.getBoundingClientRect(),selRect))w.classList.add('selected');});}

/* Toolbar */
function hideToolbar(){if(selToolbar){selToolbar.remove();selToolbar=null;}}
function showToolbarForBox(){hideToolbar();selToolbar=document.createElement('div');selToolbar.className='sel-toolbar';
  selToolbar.innerHTML=`<label>Row:<input id="tbRow" type="number" min="1" value="1" style="width:70px"></label>
  <label>Col:<select id="tbCol"><option value="0">A</option><option value="1">B</option><option value="2">C</option>
  <option value="3">D</option><option value="4">E</option><option value="5">F</option></select></label>
  <button id="tbAdd">Add</button>`;document.body.appendChild(selToolbar);
  const r=selBox.getBoundingClientRect();selToolbar.style.left=(r.right+8)+'px';selToolbar.style.top=r.top+'px';
  document.getElementById('tbAdd').onclick=()=>{const row=parseInt(document.getElementById('tbRow').value,10);
    const col=parseInt(document.getElementById('tbCol').value,10);addSelectionToSpreadsheet(row,col);};}

/* Add selection → DarkColumn5 table */
function getSelectedSpanElements(){return Array.from(document.querySelectorAll('#readerDisplay .word.selected'));}
function addSelectionToSpreadsheet(row,col){
  const spans=getSelectedSpanElements();if(spans.length===0)return alert('No selection.');
  const text=spans.map(s=>s.textContent).join('').trim();if(!text)return;
  while(tbody.rows.length<row){const tr=document.createElement('tr');const th=document.createElement('th');
    th.textContent=tbody.rows.length+1;tr.appendChild(th);for(let j=0;j<6;j++){const td=document.createElement('td');td.contentEditable=true;tr.appendChild(td);}tbody.appendChild(tr);}
  const target=tbody.rows[row-1].cells[col+1];if(!target)return;target.textContent=text;
  spans.forEach(s=>{s.classList.add('saved');s.classList.remove('selected');});if(selBox){selBox.remove();selBox=null;}hideToolbar();
}
      
      
      
      
document.getElementById('loadPasteBtn').addEventListener('click', ()=>{
    const text = document.getElementById('pasteBox').value.trim();
    if(!text) return alert("Paste something first!");
    
    // Copy into main reader input
    const readerInput = document.getElementById('readerInput');
    readerInput.value = text;

    // Render spans in reader overlay
    renderTextToReader(text);

    // Show the overlay
    const overlay = document.getElementById('readerOverlay');
    overlay.style.display = 'block';
    setTimeout(()=>overlay.style.opacity='1',20);

    // Clear the small paste box
    document.getElementById('pasteBox').value = '';
});
      document.getElementById('miniLoadBtn').addEventListener('click', ()=>{
    const text = document.getElementById('miniPasteInput').value.trim();
    if(!text) return alert("Paste something first!");
    
    // Copy into main reader input
    const readerInput = document.getElementById('readerInput');
    readerInput.value = text;

    // Render spans in reader overlay
    renderTextToReader(text);

    // Show overlay
    const overlay = document.getElementById('readerOverlay');
    overlay.style.display = 'block';
    setTimeout(()=>overlay.style.opacity='1',20);

    // Clear mini input
    document.getElementById('miniPasteInput').value = '';
});
      
      
      
      
      
      
      
      
      </script>
      
      
      
      
      
      
      
      
      
      
      
      <style>
/* Overlay background styling (if needed) */
#readerOverlay {
  position: relative;
  z-index: 1;
}

/* Popup styling */
.popup {
  position: absolute;
  background: #1e1e1e;
  color: #f0f0f0;
  padding: 10px 12px;
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(0,0,0,0.3);
  font-family: Arial, sans-serif;
  font-size: 14px;
  z-index: 9999;
  min-width: 150px;
  transition: opacity 0.2s ease-in-out;
}

/* Input and select styling inside popup */
.popup input[type="number"],
.popup select {
  padding: 4px 6px;
  margin-top: 2px;
  margin-bottom: 5px;
  border-radius: 4px;
  border: 1px solid #555;
  background: #2b2b2b;
  color: #f0f0f0;
  font-size: 13px;
}

/* Button styling */
.popup button {
  padding: 6px 10px;
  background: #4caf50;
  border: none;
  border-radius: 5px;
  color: white;
  font-weight: bold;
  cursor: pointer;
  transition: background 0.2s;
}

.popup button:hover {
  background: #45a049;
}
</style>
      
      
      
      
</body>
</html>
