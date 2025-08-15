<html lang="en">
<head>
<meta charset="utf-8"/>
<meta name="viewport" content="width=device-width,initial-scale=1"/>
<title>JACKRIP – Ripper & Atlas</title>
<style>
  :root{--bg:#0f1115;--fg:#eef0f3;--mut:#9aa4ad;--card:#161b22;--br:#263142;--hl:#7ab7ff}
  *{box-sizing:border-box;font-family:system-ui,Segoe UI,Roboto,Arial}
  html{font-size:clamp(15px,0.95vw+11px,18px)}
  body{margin:0;background:var(--bg);color:var(--fg)}
  header{display:flex;align-items:center;gap:12px;padding:14px 18px;border-bottom:1px solid var(--br);
         position:sticky;top:0;background:#0f1115e6;backdrop-filter:saturate(120%) blur(6px)}
  h1{margin:0 12px 0 0;font-size:1rem}
  .tabs{margin-left:auto;display:flex;gap:10px}
  .tabBtn{border:1px solid var(--br);background:var(--card);color:var(--fg);padding:10px 14px;border-radius:12px;cursor:pointer}
  .tabBtn[aria-pressed="true"]{outline:2px solid var(--hl)}
  main{display:grid;grid-template-columns:270px 1fr;gap:18px;max-width:1600px;margin:0 auto;padding:16px}
  aside{display:flex;flex-direction:column;gap:12px}
  .card{background:var(--card);border:1px solid var(--br);border-radius:14px;padding:16px}
  .title{font-weight:700;margin-bottom:10px}
  .row{display:flex;gap:10px;flex-wrap:wrap;align-items:center}
  label{font-size:.9rem;color:var(--mut)}
  input[type="number"],input[type="text"],button{background:#10151c;border:1px solid var(--br);color:var(--fg);border-radius:10px;padding:10px;font-size:.95rem}
  button{cursor:pointer}
  .drop{border:1px dashed #2b3544;border-radius:12px;padding:18px;text-align:center;background:#10151c}
  .thumbs{display:grid;grid-template-columns:repeat(auto-fill,minmax(96px,1fr));gap:10px;margin-top:10px}
  .thumb{background:#0f1319;border:1px solid var(--br);border-radius:10px;padding:8px}
  .thumb canvas{width:100%;height:auto;border-radius:6px;background:#000}
  .hint{font-size:.85rem;color:var(--mut)}
  .wrap{position:relative;border:1px solid var(--br);border-radius:12px;overflow:auto}
  .checker{
    background:
      linear-gradient(45deg,#0b0f14 25%,transparent 25% 75%,#0b0f14 75%),
      linear-gradient(45deg,#0b0f14 25%,transparent 25% 75%,#0b0f14 75%) 8px 8px/16px 16px,#12161d;
  }
  #ripWrap{height:60vh;min-height:420px}
  #atlasWrap{height:60vh;min-height:420px}
  canvas{image-rendering:pixelated}
</style>
</head>
<body>
<header>
  <h1>JACKRIP</h1>
  <div class="tabs">
    <button class="tabBtn" data-tab="ripper" aria-pressed="true">Ripper</button>
    <button class="tabBtn" data-tab="atlas"  aria-pressed="false">Atlas</button>
  </div>
</header>

<main>
  <aside>
    <div class="card">
      <div class="title">Atlas Settings</div>
      <div class="row">
        <label>Max W <input id="maxW" type="number" value="4096" min="64" step="64"></label>
        <label>Max H <input id="maxH" type="number" value="4096" min="64" step="64"></label>
        <label>Padding <input id="pad" type="number" value="2" min="0" max="64"></label>
        <label><input id="pot" type="checkbox" checked> Power-of-Two</label>
      </div>
      <div class="row" style="margin-top:6px">
        <button id="packBtn">Auto-Pack</button>
        <button id="editLayoutBtn">Edit Layout</button>
        <label style="margin-left:auto"><input id="snapChk" type="checkbox" checked> Snap</label>
      </div>
      <div class="row" style="margin-top:6px">
        <label>Atlas W <input id="atlasW" type="number" value="1024" min="16" step="16"></label>
        <label>Atlas H <input id="atlasH" type="number" value="1024" min="16" step="16"></label>
        <button id="resizeAtlasBtn">Resize Canvas</button>
      </div>
      <div class="hint" id="atlasInfo" style="margin-top:6px"></div>
    </div>
    <div class="card">
      <div class="title">Export</div>
      <div class="row"><button id="downloadAtlasPng">Download PNG</button><button id="downloadAtlasJson">Download JSON</button></div>
      <div class="hint">PNG background is transparent.</div>
    </div>
  </aside>

  <section>
    <!-- RIPPER -->
    <div id="tab_ripper">
      <div class="card">
        <div class="title">Source</div>
        <div class="row">
          <input id="ripperFile" type="file" accept="image/*">
          <div class="toolbarTiny">
            <button id="zoomOutBtn">–</button>
            <button id="zoomInBtn">+</button>
            <button id="zoom100Btn">100%</button>
            <button id="zoomFitBtn">Fit</button>
          </div>
        </div>
        <div class="wrap checker" id="ripWrap" style="margin-top:8px">
          <div id="stage" style="position:relative;transform-origin:0 0">
            <canvas id="srcCanvas"></canvas>
            <canvas id="ovlCanvas" style="position:absolute;left:0;top:0;"></canvas>
          </div>
        </div>
        <div class="hint" style="margin-top:8px">
          Tools:
          <button data-tool="quad">Quad</button>
          <button data-tool="lasso">Lasso</button>
          <label style="margin-left:12px"><input id="curvedChk" type="checkbox"> Curved edges (experimental)</label><br>
          Quad: 4 clicks, 5th click restarts. Drag corners; drag inside to move all.  
          Lasso does not correct perspective. After release, click-drag inside to move.  
          Pan with <b>MMB</b> (or hold Space). Wheel zooms to cursor. Hold <b>Shift</b> while dragging to axis-lock.
        </div>
        <div class="row" style="margin-top:8px"><button id="resetBtn">Reset Tool</button></div>
      </div>

      <div class="card" style="margin-top:16px">
        <div class="title">Extract</div>
        <div class="row">
          <label><input id="autoSize" type="checkbox" checked> Auto-size from quad</label>
          <label>Out W <input id="outW" type="number" value="512" min="1"></label>
          <label>Out H <input id="outH" type="number" value="512" min="1"></label>
          <label>Quality <input id="quality" type="number" value="1" min="1" max="2"></label>
          <button id="extractBtn">Extract (+ Add to Atlas)</button>
        </div>
        <div class="wrap checker" style="margin-top:8px; min-height:260px">
          <canvas id="outCanvas"></canvas>
        </div>
        <div class="row" style="margin-top:8px">
          <input id="ripName" type="text" value="ripped.png" style="min-width:240px">
          <button id="saveRipBtn">Download PNG</button>
        </div>
      </div>
    </div>

    <!-- ATLAS -->
    <div id="tab_atlas" style="display:none">
      <div class="grid">
        <div class="card">
          <div class="title">Images</div>
          <div class="drop" id="dropZone">
            <input id="fileInput" type="file" accept="image/*" multiple hidden>
            <div>Drag images here or <button id="browseBtn" type="button">Browse…</button></div>
          </div>
          <div class="thumbs" id="thumbs"></div>
        </div>

        <div class="card">
          <div class="title">Atlas</div>
          <div class="wrap checker" id="atlasWrap" style="margin-top:8px">
            <canvas id="atlasCanvas"></canvas>
          </div>
          <div class="row" style="margin-top:8px">
            <span id="packStatus" class="hint" style="margin-left:auto"></span>
          </div>
        </div>
      </div>
    </div>
  </section>
</main>

<script>
(()=>{"use strict";
/* ---------- helpers ---------- */
const $=s=>document.querySelector(s), $$=s=>Array.from(document.querySelectorAll(s));
const clamp=(v,a,b)=>Math.max(a,Math.min(b,v));
const dist=(a,b)=>Math.hypot(a.x-b.x,a.y-b.y);
const nextPOT=v=>{let p=1;while(p<v)p<<=1;return p;}
const imgFromBlob=b=>new Promise((res,rej)=>{const u=URL.createObjectURL(b);const i=new Image();i.onload=()=>{URL.revokeObjectURL(u);res(i)};i.onerror=rej;i.src=u;});
const cFromImg=i=>{const c=document.createElement('canvas');c.width=i.naturalWidth||i.width;c.height=i.naturalHeight||i.height;c.getContext('2d').drawImage(i,0,0);return c;}
const dl=(blob,name)=>{const a=document.createElement('a');a.href=URL.createObjectURL(blob);a.download=name;document.body.appendChild(a);a.click();setTimeout(()=>{URL.revokeObjectURL(a.href);a.remove();},100);}
const split=(n)=>{const m=(n||'sprite.png').match(/^(.*?)(\.[^.]*)?$/);return {base:m[1]||'sprite',ext:m[2]||'.png'};}
const uname=(desired,set)=>{const {base,ext}=split(desired);let num=1,fin=desired||'sprite.png';while(set.has(fin))fin=`${base}_${num++}${ext}`;return fin;}
function pointInPoly(pt,poly){let c=false;for(let i=0,j=poly.length-1;i<poly.length;j=i++){const pi=poly[i],pj=poly[j];if(((pi.y>pt.y)!=(pj.y>pt.y))&&(pt.x<(pj.x-pi.x)*(pt.y-pi.y)/(pj.y-pi.y)+pi.x))c=!c;}return c;}

/* ---------- tabs ---------- */
const tabs={ripper:$("#tab_ripper"), atlas:$("#tab_atlas")};
$$(".tabBtn").forEach(btn=>btn.addEventListener("click",()=>{$$(".tabBtn").forEach(x=>x.setAttribute("aria-pressed","false"));btn.setAttribute("aria-pressed","true");const t=btn.dataset.tab;Object.keys(tabs).forEach(k=>tabs[k].style.display=(k===t)?"block":"none");}));

/* ================= ATLAS ================= */
const state={items:[],placements:[],size:{w:1024,h:1024}};
const atlas=$("#atlasCanvas"), aG=atlas.getContext("2d",{alpha:true});
const atlasInfo=$("#atlasInfo");
function syncAtlasCss(){atlas.style.width=state.size.w+"px";atlas.style.height=state.size.h+"px";atlasInfo.textContent=`Atlas: ${state.size.w}×${state.size.h}px`;}
function drawAtlas(){
  atlas.width=state.size.w; atlas.height=state.size.h; syncAtlasCss();
  aG.clearRect(0,0,atlas.width,atlas.height); aG.imageSmoothingEnabled=false;
  for(const p of state.placements){const it=state.items.find(i=>i.name===p.name); if(it) aG.drawImage(it.canvas,p.x,p.y);}
  if(sel) { aG.save(); aG.strokeStyle="#7ab7ff"; aG.lineWidth=1.5; aG.strokeRect(sel.x+.5,sel.y+.5,sel.w,sel.h); aG.restore(); }
}
function thumbs(){
  const host=$("#thumbs"); host.innerHTML="";
  state.items.forEach((it,idx)=>{
    const d=document.createElement("div"); d.className="thumb";
    const c=document.createElement("canvas"); c.width=it.w; c.height=it.h; c.getContext("2d").drawImage(it.canvas,0,0);
    const label=document.createElement("div"); label.className="hint"; label.textContent=`${it.name} (${it.w}×${it.h})`;
    const rm=document.createElement("button"); rm.textContent="Remove"; rm.onclick=()=>{state.items.splice(idx,1); state.placements=state.placements.filter(p=>p.name!==it.name); thumbs(); drawAtlas();};
    d.append(c,label,rm); host.append(d);
  });
}
async function addFiles(list){
  for(const f of list){if(!f.type||!f.type.startsWith("image/")) continue; const img=await imgFromBlob(f); const c=cFromImg(img);
    const name=uname(f.name,new Set(state.items.map(i=>i.name))); state.items.push({name,canvas:c,w:c.width,h:c.height});}
  thumbs();
}
function pack(blocks,maxW,maxH,pad){
  blocks.sort((a,b)=>Math.max(b.w,b.h)-Math.max(a.w,a.h));
  let root={x:0,y:0,w:blocks[0]?blocks[0].w:0,h:blocks[0]?blocks[0].h:0,used:false};
  const find=(r,w,h)=>r?(r.used?(find(r.right,w,h)||find(r.down,w,h)):(w<=r.w&&h<=r.h?r:null)):null;
  const split=(n,w,h)=>{n.used=true;n.down={x:n.x,y:n.y+h+pad,w:n.w,h:n.h-h-pad,used:false};n.right={x:n.x+w+pad,y:n.y,w:n.w-w-pad,h:h,used:false};return n;}
  const growR=(w,h)=>{const nr={x:0,y:0,used:true,w:root.w+w+pad,h:root.h};nr.down=root;nr.right={x:root.w+pad,y:0,w:w,h:root.h,used:false};root=nr;const nd=find(root,w,h);return nd?split(nd,w,h):null;}
  const growD=(w,h)=>{const nr={x:0,y:0,used:true,w:root.w,h:root.h+h+pad};nr.right=root;nr.down={x:0,y:root.h+pad,w:root.w,h:h,used:false};root=nr;const nd=find(root,w,h);return nd?split(nd,w,h):null;}
  blocks.forEach((b,i)=>{if(i===0){root.w=Math.min(Math.max(b.w,64),maxW);root.h=Math.min(Math.max(b.h,64),maxH);}let n=find(root,b.w,b.h);
    if(n) b.fit=split(n,b.w,b.h); else{const canR=root.w+b.w+pad<=maxW,canD=root.h+b.h+pad<=maxH,gr=canR&&(root.h>=root.w),gd=canD&&(root.w>=root.h);
      if(gr) n=growR(b.w,b.h); else if(gd) n=growD(b.w,b.h); else if(canR) n=growR(b.w,b.h); else if(canD) n=growD(b.w,b.h); else b.fit=null;}});
  let maxX=0,maxY=0; blocks.forEach(b=>{if(b.fit){maxX=Math.max(maxX,b.fit.x+b.w);maxY=Math.max(maxY,b.fit.y+b.h);}});
  return {blocks,w:maxX,h:maxY};
}
$("#packBtn").addEventListener("click",()=>{
  if(!state.items.length){$("#packStatus").textContent="No images.";return;}
  const pad=+$("#pad").value|0, maxW=+$("#maxW").value|0, maxH=+$("#maxH").value|0;
  const res=pack(state.items.map(i=>({name:i.name,w:i.w,h:i.h})),maxW,maxH,pad);
  let w=Math.max(1,res.w), h=Math.max(1,res.h); if($("#pot").checked){w=nextPOT(w);h=nextPOT(h);}
  w=Math.min(w,maxW); h=Math.min(h,maxH); state.size={w,h}; $("#atlasW").value=w; $("#atlasH").value=h;
  state.placements=res.blocks.filter(b=>b.fit).map(b=>({name:b.name,x:b.fit.x,y:b.fit.y,w:b.w,h:b.h}));
  const failed=res.blocks.filter(b=>!b.fit).length;
  $("#packStatus").textContent=failed?`${failed} didn’t fit`:`Packed ${state.placements.length}/${state.items.length}`;
  drawAtlas();
});
$("#resizeAtlasBtn").addEventListener("click",()=>{
  let w=+$("#atlasW").value|0, h=+$("#atlasH").value|0;
  if($("#pot").checked){w=nextPOT(w);h=nextPOT(h);$("#atlasW").value=w;$("#atlasH").value=h;}
  state.size={w:Math.max(1,w),h:Math.max(1,h)}; drawAtlas();
});
/* manual layout */
let sel=null, dragOff={x:0,y:0}, editing=false;
$("#editLayoutBtn").addEventListener("click",e=>{editing=!editing; e.target.style.outline=editing?"2px solid var(--hl)":"none";});
atlas.addEventListener("mousedown",e=>{
  if(!editing) return; const r=atlas.getBoundingClientRect();
  const x=(e.clientX-r.left)*(atlas.width/r.width), y=(e.clientY-r.top)*(atlas.height/r.height);
  sel=null; for(let i=state.placements.length-1;i>=0;i--){const p=state.placements[i]; if(x>=p.x&&x<=p.x+p.w&&y>=p.y&&y<=p.y+p.h){sel=p; dragOff.x=x-p.x; dragOff.y=y-p.y; break;}}
  drawAtlas();
});
window.addEventListener("mousemove",e=>{
  if(!editing||!sel||e.buttons!==1) return; const r=atlas.getBoundingClientRect();
  let x=(e.clientX-r.left)*(atlas.width/r.width), y=(e.clientY-r.top)*(atlas.height/r.height);
  let nx=x-dragOff.x, ny=y-dragOff.y; if($("#snapChk").checked){nx=Math.round(nx); ny=Math.round(ny);}
  nx=clamp(nx,0,atlas.width-sel.w); ny=clamp(ny,0,atlas.height-sel.h); sel.x=nx; sel.y=ny; drawAtlas();
});
/* import/export */
$("#browseBtn").addEventListener("click",()=>$("#fileInput").click());
$("#fileInput").addEventListener("change",e=>addFiles(e.target.files).then(drawAtlas));
const drop=$("#dropZone"); ["dragover","dragenter"].forEach(ev=>drop.addEventListener(ev,e=>{e.preventDefault(); drop.style.borderColor="#3a4a63";}));
drop.addEventListener("dragleave",()=>{drop.style.borderColor="#2b3544";});
drop.addEventListener("drop",e=>{e.preventDefault(); drop.style.borderColor="#2b3544"; addFiles(e.dataTransfer.files).then(drawAtlas);});
$("#downloadAtlasPng").addEventListener("click",()=>atlas.toBlob(b=>dl(b,"atlas.png"),"image/png"));
$("#downloadAtlasJson").addEventListener("click",()=>{
  const frames={}; state.placements.forEach(p=>frames[p.name]={x:p.x,y:p.y,w:p.w,h:p.h});
  const meta={app:"JACKRIP",size:state.size,count:state.placements.length};
  dl(new Blob([JSON.stringify({meta,frames},null,2)],{type:"application/json"}),"atlas.json");
});

/* ================= RIPPER ================= */
const ripWrap=$("#ripWrap"), stage=$("#stage");
const src=$("#srcCanvas"), sG=src.getContext("2d",{alpha:true});
const ovl=$("#ovlCanvas"), oG=ovl.getContext("2d");
const out=$("#outCanvas"), outG=out.getContext("2d");
let srcImg=null;
/* view */
const view={scale:1,tx:10,ty:10,space:false,panStart:{x:0,y:0},panAt:{x:0,y:0}};
const applyView=()=>{stage.style.transform=`translate(${view.tx}px,${view.ty}px) scale(${view.scale})`;};
function fit(){ if(!srcImg)return; const r=ripWrap.getBoundingClientRect(); const sx=r.width/src.width, sy=r.height/src.height; view.scale=Math.min(sx,sy,1); view.tx=10; view.ty=10; applyView();}
$("#zoomInBtn").addEventListener("click",()=>zoom(ripWrap.clientWidth/2,ripWrap.clientHeight/2,1.25));
$("#zoomOutBtn").addEventListener("click",()=>zoom(ripWrap.clientWidth/2,ripWrap.clientHeight/2,1/1.25));
$("#zoom100Btn").addEventListener("click",()=>{view.scale=1;view.tx=10;view.ty=10;applyView();});
$("#zoomFitBtn").addEventListener("click",fit);
function zoom(cx,cy,f){if(!srcImg)return; const minS=.1,maxS=8,s0=view.scale,s1=clamp(s0*f,minS,maxS),dx=cx-view.tx,dy=cy-view.ty; view.tx=cx-(dx*(s1/s0)); view.ty=cy-(dy*(s1/s0)); view.scale=s1; applyView();}
ripWrap.addEventListener("wheel",e=>{if(!srcImg)return; e.preventDefault(); const r=ripWrap.getBoundingClientRect(); zoom(e.clientX-r.left,e.clientY-r.top,Math.exp(-e.deltaY/300));},{passive:false});
window.addEventListener("keydown",e=>{if(e.code==="Space") view.space=true;});
window.addEventListener("keyup",e=>{if(e.code==="Space") view.space=false;});
ripWrap.addEventListener("mousedown",e=>{
  if(!srcImg) return;
  if(e.button===1 || view.space){
    view.panStart={x:e.clientX,y:e.clientY}; view.panAt={x:view.tx,y:view.ty}; e.preventDefault();
    const move=(ev)=>{view.tx=view.panAt.x+(ev.clientX-view.panStart.x); view.ty=view.panAt.y+(ev.clientY-view.panStart.y); applyView();};
    const up=()=>{window.removeEventListener("mousemove",move);window.removeEventListener("mouseup",up);};
    window.addEventListener("mousemove",move); window.addEventListener("mouseup",up);
  }
});
/* load image */
$("#ripperFile").addEventListener("change",e=>{const f=e.target.files?.[0]; if(f) setSrc(f);});
["dragenter","dragover"].forEach(ev=>ripWrap.addEventListener(ev,e=>{e.preventDefault();}));
ripWrap.addEventListener("drop",e=>{e.preventDefault(); const f=e.dataTransfer.files?.[0]; if(f && f.type.startsWith("image/")) setSrc(f);});
async function setSrc(file){
  const img=await imgFromBlob(file); srcImg=cFromImg(img);
  src.width=srcImg.width; src.height=srcImg.height; ovl.width=src.width; ovl.height=src.height;
  sG.clearRect(0,0,src.width,src.height); sG.drawImage(srcImg,0,0);
  fit(); drawOverlay();
}
/* coords */
const cpos=e=>{const r=src.getBoundingClientRect(); return {x:(e.clientX-r.left)*(src.width/r.width), y:(e.clientY-r.top)*(src.height/r.height)};}
/* bilinear */
function bilinear(data,sw,sh,x,y){
  if(x<0||y<0||x>sw-1||y>sh-1) return [0,0,0,0];
  const x0=Math.floor(x),y0=Math.floor(y),x1=Math.min(sw-1,x0+1),y1=Math.min(sh-1,y0+1),fx=x-x0,fy=y-y0;
  const idx=(yy,xx)=>((yy*sw+xx)<<2),d=data.data,L=(a,b,t)=>a+(b-a)*t;
  const i00=idx(y0,x0),i10=idx(y0,x1),i01=idx(y1,x0),i11=idx(y1,x1);
  const c00=[d[i00],d[i00+1],d[i00+2],d[i00+3]],c10=[d[i10],d[i10+1],d[i10+2],d[i10+3]],
        c01=[d[i01],d[i01+1],d[i01+2],d[i01+3]],c11=[d[i11],d[i11+1],d[i11+2],d[i11+3]];
  const top=c00.map((v,k)=>L(v,c10[k],fx)), bot=c01.map((v,k)=>L(v,c11[k],fx));
  return top.map((v,k)=>L(v,bot[k],fy));
}

/* ---------- Homography (unit square -> quad) ---------- */
function solve8x8(A,b){
  for(let i=0;i<8;i++){
    let max=i; for(let r=i+1;r<8;r++) if(Math.abs(A[r][i])>Math.abs(A[max][i])) max=r;
    if(max!==i){ [A[i],A[max]]=[A[max],A[i]]; [b[i],b[max]]=[b[max],b[i]]; }
    const piv=A[i][i]||1e-12;
    for(let j=i;j<8;j++) A[i][j]/=piv; b[i]/=piv;
    for(let r=0;r<8;r++){ if(r===i) continue; const f=A[r][i]; if(!f) continue;
      for(let c=i;c<8;c++) A[r][c]-=f*A[i][c]; b[r]-=f*b[i];
    }
  }
  return b;
}
function homographyFromQuad(TL,TR,BR,BL){
  const pts=[[0,0,TL.x,TL.y],[1,0,TR.x,TR.y],[1,1,BR.x,BR.y],[0,1,BL.x,BL.y]];
  const A=[], b=[];
  for(const [u,v,x,y] of pts){
    A.push([u,v,1, 0,0,0, -x*u, -x*v]); b.push(x);
    A.push([0,0,0, u,v,1, -y*u, -y*v]); b.push(y);
  }
  const h=solve8x8(A,b);
  return [ [h[0],h[1],h[2]],
           [h[3],h[4],h[5]],
           [h[6],h[7],1   ] ];
}
function applyH(H,u,v){
  const X=H[0][0]*u+H[0][1]*v+H[0][2];
  const Y=H[1][0]*u+H[1][1]*v+H[1][2];
  const W=H[2][0]*u+H[2][1]*v+1;
  return {x:X/W, y:Y/W};
}

/* ---------- QUAD tool (with axis-lock + snapping) ---------- */
const quadTool={
  placing:true, points:[], corners:[{x:60,y:60},{x:260,y:60},{x:260,y:260},{x:60,y:260}],
  dragCorner:-1, dragAll:false, lastP:null, r:7,
  dragAxisLock:null, cornerStart:{x:0,y:0},

  start(){this.placing=true; this.points=[]; this.dragCorner=-1; this.dragAll=false; this.lastP=null; this.dragAxisLock=null; drawOverlay();},
  reset(){this.start();},
  order(pts){const cx=pts.reduce((s,p)=>s+p.x,0)/4, cy=pts.reduce((s,p)=>s+p.y,0)/4;
    const srt=pts.slice().sort((a,b)=>Math.atan2(a.y-cy,a.x-cx)-Math.atan2(b.y-cy,b.x-cx));
    let idx=0,best=1e9; for(let i=0;i<4;i++){const v=srt[i].x+srt[i].y; if(v<best){best=v; idx=i;}} return [srt[idx],srt[(idx+1)%4],srt[(idx+2)%4],srt[(idx+3)%4]];
  },
  hitCorner(p){for(let i=0;i<4;i++) if(dist(p,this.corners[i])<=this.r+3) return i; return -1;},
  mousedown(p){
    if(this.placing){
      if(this.points.length<4){ this.points.push(p); if(this.points.length===4){ this.corners=this.order(this.points); this.placing=false; toolAPI.maybeAutosize(); } }
      else{ this.start(); this.points=[p]; }
      drawOverlay(); return;
    }
    const ci=this.hitCorner(p);
    if(ci>=0){ this.dragCorner=ci; this.cornerStart={x:this.corners[ci].x,y:this.corners[ci].y}; this.dragAxisLock=null; return; }
    if(pointInPoly(p,this.corners)){ this.dragAll=true; this.lastP=p; this.dragAxisLock=null; }
  },
  mousemove(p,buttons,shift){
    if(this.placing || !buttons) return;
    const snapOn=$("#snapChk").checked;
    const GRID=8, TOL=4; // grid size & snap tolerance (px)

    const snapToOtherCorners=(pt,idx)=>{
      for(let i=0;i<4;i++){ if(i===idx) continue;
        const c=this.corners[i];
        if(Math.abs(pt.x-c.x)<=TOL) pt.x=c.x;
        if(Math.abs(pt.y-c.y)<=TOL) pt.y=c.y;
      }
      return pt;
    };
    const snapCoordToGrid=v=>{
      const gv=Math.round(v/GRID)*GRID;
      return (Math.abs(gv-v)<=TOL)?gv:v;
    };

    if(this.dragCorner>=0){
      let newP={x:p.x,y:p.y};

      // axis lock (decide by initial direction while Shift is held)
      if(shift){
        if(this.dragAxisLock==null){
          const dx=Math.abs(newP.x-this.cornerStart.x), dy=Math.abs(newP.y-this.cornerStart.y);
          this.dragAxisLock = (dx>dy) ? "x" : "y";
        }
        if(this.dragAxisLock==="x") newP.y=this.cornerStart.y;
        else if(this.dragAxisLock==="y") newP.x=this.cornerStart.x;
      }else{
        this.dragAxisLock=null;
      }

      if(snapOn){
        newP.x=snapCoordToGrid(newP.x);
        newP.y=snapCoordToGrid(newP.y);
        newP=snapToOtherCorners(newP,this.dragCorner);
      }

      // clamp inside image bounds
      newP.x=clamp(newP.x,0,src.width);
      newP.y=clamp(newP.y,0,src.height);

      this.corners[this.dragCorner]=newP;
      toolAPI.maybeAutosize();
      drawOverlay();

    } else if(this.dragAll && this.lastP){
      let dx=p.x-this.lastP.x, dy=p.y-this.lastP.y;

      // axis lock for whole-quad move
      if(shift){
        if(this.dragAxisLock==null){
          this.dragAxisLock=(Math.abs(dx)>Math.abs(dy))?"x":"y";
        }
        if(this.dragAxisLock==="x") dy=0; else if(this.dragAxisLock==="y") dx=0;
      }else{
        this.dragAxisLock=null;
      }

      // compute tentative moved corners
      let moved=this.corners.map(c=>({x:c.x+dx,y:c.y+dy}));

      // grid snapping for whole move (apply a shared small adjustment so the quad stays rigid)
      if(snapOn){
        // find best small adjustment toward nearest grid for any corner (independently on axes)
        let bestAdjX=0, bestAdjY=0, bestAx=Infinity, bestAy=Infinity;
        for(const m of moved){
          const gx=Math.round(m.x/GRID)*GRID, gy=Math.round(m.y/GRID)*GRID;
          const ax=Math.abs(gx-m.x), ay=Math.abs(gy-m.y);
          if(ax<=TOL && ax<bestAx){ bestAx=ax; bestAdjX=gx-m.x; }
          if(ay<=TOL && ay<bestAy){ bestAy=ay; bestAdjY=gy-m.y; }
        }
        // apply adjustments uniformly
        if(bestAx!==Infinity) moved=moved.map(pt=>({x:pt.x+bestAdjX,y:pt.y}));
        if(bestAy!==Infinity) moved=moved.map(pt=>({x:pt.x,y:pt.y+bestAdjY}));
      }

      // clamp within image bounds
      const minX=Math.min(...moved.map(p=>p.x)), minY=Math.min(...moved.map(p=>p.y));
      const maxX=Math.max(...moved.map(p=>p.x)), maxY=Math.max(...moved.map(p=>p.y));
      const offX = (minX<0)?-minX : (maxX>src.width)?(src.width-maxX):0;
      const offY = (minY<0)?-minY : (maxY>src.height)?(src.height-maxY):0;
      if(offX||offY) moved=moved.map(pt=>({x:pt.x+offX,y:pt.y+offY}));

      // commit
      this.corners=moved;
      this.lastP={x:p.x,y:p.y};
      drawOverlay();
    }
  },
  mouseup(){ this.dragCorner=-1; this.dragAll=false; this.lastP=null; this.dragAxisLock=null; },
  draw(){
    oG.clearRect(0,0,ovl.width,ovl.height);
    const pts=this.placing?this.points:this.corners; if(!pts.length) return;
    oG.lineWidth=2; oG.strokeStyle="rgba(122,183,255,.9)"; oG.fillStyle="rgba(122,183,255,.08)";
    if(!this.placing && pts.length===4){
      if($("#curvedChk").checked){
        const C=this.corners, M=[0,1,2,3].map(i=>({x:(C[i].x+C[(i+1)%4].x)/2,y:(C[i].y+C[(i+1)%4].y)/2}));
        oG.beginPath();
        oG.moveTo(C[0].x,C[0].y);
        oG.quadraticCurveTo(M[0].x,M[0].y,C[1].x,C[1].y);
        oG.quadraticCurveTo(M[1].x,M[1].y,C[2].x,C[2].y);
        oG.quadraticCurveTo(M[2].x,M[2].y,C[3].x,C[3].y);
        oG.quadraticCurveTo(M[3].x,M[3].y,C[0].x,C[0].y);
        oG.closePath(); oG.stroke(); oG.fill();
      }else{
        oG.beginPath();
        oG.moveTo(pts[0].x,pts[0].y); for(let i=1;i<4;i++) oG.lineTo(pts[i].x,pts[i].y); oG.closePath(); oG.stroke(); oG.fill();
      }
      // Corners
      oG.fillStyle="#7ab7ff"; oG.strokeStyle="#000";
      for(let i=0;i<4;i++){ const c=this.corners[i]; oG.beginPath(); oG.arc(c.x,c.y,this.r,0,Math.PI*2); oG.fill(); oG.stroke(); }
    }else{
      oG.beginPath(); oG.moveTo(pts[0].x,pts[0].y); for(let i=1;i<pts.length;i++) oG.lineTo(pts[i].x,pts[i].y); oG.stroke();
    }
  },
  autosize(){ const C=this.corners, L=(a,b)=>Math.hypot(a.x-b.x,a.y-b.y);
    return {w:Math.max(1,Math.round((L(C[0],C[1])+L(C[3],C[2]))/2)),
            h:Math.max(1,Math.round((L(C[0],C[3])+L(C[1],C[2]))/2))}; },
  extract(w,h,q){
    if(!srcImg || this.placing) return false;
    out.width=w; out.height=h; outG.clearRect(0,0,w,h);
    if($("#curvedChk").checked){
      const TL=this.corners[0],TR=this.corners[1],BR=this.corners[2],BL=this.corners[3];
      const srcData=sG.getImageData(0,0,src.width,src.height), od=outG.createImageData(w,h);
      let p=0; for(let y=0;y<h;y++){ const v=h>1?y/(h-1):0; for(let x=0;x<w;x++){ const u=w>1?x/(w-1):0;
        const sx=(1-u)*(1-v)*TL.x + u*(1-v)*TR.x + (1-u)*v*BL.x + u*v*BR.x;
        const sy=(1-u)*(1-v)*TL.y + u*(1-v)*TR.y + (1-u)*v*BL.y + u*v*BR.y;
        const c=bilinear(srcData,src.width,src.height,sx,sy);
        od.data[p++]=c[0]; od.data[p++]=c[1]; od.data[p++]=c[2]; od.data[p++]=c[3];
      }} outG.putImageData(od,0,0); return true;
    }else{
      const H=homographyFromQuad(this.corners[0],this.corners[1],this.corners[2],this.corners[3]);
      const srcData=sG.getImageData(0,0,src.width,src.height), od=outG.createImageData(w,h);
      let p=0; for(let y=0;y<h;y++){ const v=h>1?y/(h-1):0; for(let x=0;x<w;x++){ const u=w>1?x/(w-1):0;
        const s=applyH(H,u,v); const c=bilinear(srcData,src.width,src.height,s.x,s.y);
        od.data[p++]=c[0]; od.data[p++]=c[1]; od.data[p++]=c[2]; od.data[p++]=c[3];
      }} outG.putImageData(od,0,0); return true;
    }
  }
};

/* ---------- LASSO tool (movable after release) ---------- */
const lassoTool={
  path:[],drawing:false,dragAll:false,lastP:null,
  start(){this.path=[];this.drawing=false;this.dragAll=false;this.lastP=null;drawOverlay();}, reset(){this.start();},
  mousedown(p){
    if(!this.drawing && this.path.length>=3){
      if(pointInPoly(p,this.path)){ this.dragAll=true; this.lastP=p; return; }
    }
    this.path=[p]; this.drawing=true; drawOverlay();
  },
  mousemove(p,buttons){
    if(this.drawing && buttons){ this.path.push(p); drawOverlay(); }
    else if(this.dragAll && buttons && this.lastP){
      const dx=p.x-this.lastP.x, dy=p.y-this.lastP.y; this.lastP=p;
      this.path=this.path.map(pt=>({x:pt.x+dx,y:pt.y+dy})); drawOverlay();
    }
  },
  mouseup(){ this.drawing=false; this.dragAll=false; this.lastP=null; },
  draw(){ oG.clearRect(0,0,ovl.width,ovl.height); if(!this.path.length) return;
    oG.strokeStyle="rgba(122,183,255,.9)"; oG.lineWidth=2; oG.beginPath();
    oG.moveTo(this.path[0].x,this.path[0].y); for(let i=1;i<this.path.length;i++) oG.lineTo(this.path[i].x,this.path[i].y); oG.closePath(); oG.stroke(); },
  extract(){ if(!srcImg||this.path.length<3) return false; const xs=this.path.map(p=>p.x), ys=this.path.map(p=>p.y);
    const minx=Math.floor(Math.min(...xs)), maxx=Math.ceil(Math.max(...xs)), miny=Math.floor(Math.min(...ys)), maxy=Math.ceil(Math.max(...ys));
    const w=Math.max(1,maxx-minx), h=Math.max(1,maxy-miny); out.width=w; out.height=h; outG.clearRect(0,0,w,h);
    outG.save(); outG.beginPath(); outG.moveTo(this.path[0].x-minx,this.path[0].y-miny); for(let i=1;i<this.path.length;i++) outG.lineTo(this.path[i].x-minx,this.path[i].y-miny);
    outG.closePath(); outG.clip(); outG.drawImage(src,-minx,-miny); outG.restore(); return true; }
};

/* router + UI */
const toolAPI={active:quadTool,
  start(n){this.active=(n==="lasso"?lassoTool:quadTool);this.active.start();},
  reset(){this.active.reset();},
  mousedown:p=>toolAPI.active.mousedown?.(p),
  mousemove:(p,b,shift)=>toolAPI.active.mousemove?.(p,b,shift),
  mouseup:()=>toolAPI.active.mouseup?.(),
  draw:()=>toolAPI.active.draw?.(),
  extract:(w,h,q)=>toolAPI.active.extract?.(w,h,q),
  maybeAutosize(){ if($("#autoSize").checked && this.active===quadTool && !quadTool.placing){ const d=quadTool.autosize(); $("#outW").value=d.w; $("#outH").value=d.h; } }
};
function drawOverlay(){toolAPI.draw();}
$("#resetBtn").addEventListener("click",()=>toolAPI.reset());
ovl.addEventListener("mousedown",e=>{ if(e.button!==0||view.space) return; toolAPI.mousedown(cpos(e)); });
window.addEventListener("mousemove",e=>{ if(e.buttons) toolAPI.mousemove(cpos(e),e.buttons,e.shiftKey); });
window.addEventListener("mouseup",()=>toolAPI.mouseup());
$('[data-tool="quad"]').setAttribute("aria-pressed","true");
$$('[data-tool]').forEach(b=>b.addEventListener("click",()=>{$$('[data-tool]').forEach(x=>x.setAttribute("aria-pressed","false")); b.setAttribute("aria-pressed","true"); toolAPI.start(b.dataset.tool);}));

/* extract + auto-add to atlas */
$("#extractBtn").addEventListener("click",()=>{
  if(!srcImg){alert("Load a source image first.");return;}
  if($("#autoSize").checked && toolAPI.active===quadTool && !quadTool.placing){ const d=quadTool.autosize(); $("#outW").value=d.w; $("#outH").value=d.h; }
  const w=Math.max(1,+$("#outW").value|0), h=Math.max(1,+$("#outH").value|0), q=+$("#quality").value|0;
  const ok=toolAPI.extract(w,h,q); if(!ok){alert("Make a selection first.");return;}
  const desired=($("#ripName").value||"ripped.png").replace(/[^a-z0-9_\-.]/gi,"_");
  const name=uname(desired,new Set(state.items.map(i=>i.name)));
  out.toBlob(async b=>{const img=await imgFromBlob(b); const c=cFromImg(img);
    state.items.push({name,canvas:c,w:c.width,h:c.height});
    state.placements.push({name,x:0,y:0,w:c.width,h:c.height});
    $("#ripName").value=name; thumbs(); drawAtlas();
  },"image/png");
});
$("#saveRipBtn").addEventListener("click",()=>{ if(!out.width){alert("Extract first.");return;} out.toBlob(b=>dl(b,($("#ripName").value||"ripped.png").replace(/[^a-z0-9_\-.]/gi,"_")),"image/png");});

/* boot */
(function init(){
  const ctx=src.getContext('2d'); src.width=640; src.height=360; ovl.width=src.width; ovl.height=src.height;
  ctx.fillStyle="#0b0f14"; ctx.fillRect(0,0,src.width,src.height);
  ctx.fillStyle="#8899aa"; ctx.fillText("Drop or browse an image. Pan with MMB/Space. Wheel to zoom. Hold Shift to axis-lock.", 12, 20);
  const w=+$("#atlasW").value|0, h=+$("#atlasH").value|0; state.size={w,h}; drawAtlas(); applyView();
})();
})();</script>
</body>
</html>
