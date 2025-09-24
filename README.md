<html lang="fr">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Pakchoi faut y aller</title>
  <style>
    :root { --bg:#0b1020; --fg:#eaf2ff; --muted:#9fb3d1; --accent:#7aa2ff; --danger:#ff6b6b; --ok:#2ecc71; }
    html, body { height:100%; }
    body { margin:0; font-family: ui-sans-serif, system-ui, -apple-system, Segoe UI, Roboto, Helvetica, Arial;
           background: linear-gradient(180deg, #0b1020 0%, #0e1630 100%); color: var(--fg); }
    header { padding: 14px 18px; border-bottom: 1px solid #1f2a44; display:flex; align-items:center; gap:12px; }
    header h1 { font-size: 18px; margin:0; font-weight: 700; }
    .badge { background:#121a34; border:1px solid #1f2a44; padding:4px 8px; border-radius:999px; color: var(--muted); font-size:12px; }
    .wrap { display:grid; grid-template-columns: 320px 1fr; gap:16px; height: calc(100% - 60px); }
    @media (max-width: 900px){ .wrap{ grid-template-columns: 1fr; height:auto; } }
    aside { border-right:1px solid #1f2a44; padding:16px; overflow:auto; }
    main { padding:12px; }
    .card { background:#0f1732; border:1px solid #1f2a44; border-radius: 14px; padding:12px; }
    .card + .card{ margin-top:12px; }
    .controls { display:flex; flex-wrap: wrap; gap:8px; align-items:center; }
    .controls button, .controls label.btn, select, input[type="number"] {
      background:#121a34; color:var(--fg); border:1px solid #263253; padding:8px 10px; border-radius:10px; cursor:pointer; font-size:14px;
    }
    .controls button:hover, .controls label.btn:hover, select:hover { border-color:#3a4c7a; }
    .controls button:disabled{ opacity:.5; cursor:not-allowed; }
    .btn-danger{ border-color:#5a2430; background:#1b0f14; color:#ffc9c9; }
    .btn-accent{ border-color:#2b4c9a; background:#13204a; }
    .btn-ok{ border-color:#255e3a; background:#0f1f18; }
    #canvasWrap{ position:relative; display:inline-block; }
    canvas { max-width: 100%; height: auto; background:#060a16; border-radius:14px; border:1px solid #1f2a44; }
    .hint { color: var(--muted); font-size: 13px; }
    .list{ display:flex; flex-direction:column; gap:6px; max-height: 260px; overflow:auto; }
    .item{ background:#0b1228; border:1px solid #1f2a44; border-radius:10px; padding:8px; display:flex; justify-content:space-between; align-items:center; gap:8px; }
    .item small{ color:var(--muted); }
    .kbd{ font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; font-size:12px; border:1px solid #263253; background:#0b1228; padding:2px 6px; border-radius:6px; }
    .footer{ color: var(--muted); font-size: 12px; margin-top:6px; }
  </style>
  <!-- Firebase (optional) -->
    <script type="module">
    import { initializeApp } from "https://www.gstatic.com/firebasejs/10.12.0/firebase-app.js";
    import { getAuth, signInAnonymously } from "https://www.gstatic.com/firebasejs/10.12.0/firebase-auth.js";
    import { getStorage, ref, uploadBytes, getDownloadURL } from "https://www.gstatic.com/firebasejs/10.12.0/firebase-storage.js";
    import { getFirestore, collection, addDoc, serverTimestamp } from "https://www.gstatic.com/firebasejs/10.12.0/firebase-firestore.js";
  
    const firebaseConfig = {
      apiKey: "AIzaSyCUkeiU5KLdj1zpwup_GFHXsBwsL6AUUHg",
      authDomain: "pakchoi-cbbff.firebaseapp.com",
      projectId: "pakchoi-cbbff",
      storageBucket: "pakchoi-cbbff.appspot.com",   // <-- corrigé
      messagingSenderId: "153676600433",
      appId: "1:153676600433:web:c96dbc5abb5802e4fd77d0",
      measurementId: "G-1JE7FR025Z"
    };
  
    // 1. Initialiser Firebase
    const appInst = initializeApp(firebaseConfig);
  
    // 2. Auth anonyme
    const auth = getAuth(appInst);
    signInAnonymously(auth);
  
    // 3. Services
    const storage = getStorage(appInst, "gs://pakchoi-cbbff.appspot.com");
    const db = getFirestore(appInst);
  
    // 4. Exposer au reste du script
    window.__FB__ = { auth, storage, db, ref, uploadBytes, getDownloadURL, collection, addDoc, serverTimestamp };
  </script>
</head>
<body>
  <header>
    <h1>Pakchoi faut y aller</h1>
  </header>

  <div class="wrap">
    <aside>
      <div class="card">
        <div class="controls" style="margin-bottom:8px;">
          <input id="fileInput" type="file" accept="image/*" multiple />
          <label class="btn" for="fileInput">📷 Choisir des images</label>
        </div>
        <div class="hint">Glissez-déposez des images ici ou cliquez sur « Choisir des images ».
          Utilisez <span class="kbd">Z</span> pour annuler le rectangle en cours, <span class="kbd">⌫</span> pour supprimer la sélection, <span class="kbd">←/→</span> pour naviguer.</div>
      </div>
      <div class="card">
        <div class="controls" style="margin-bottom:8px;">
          <button id="prevBtn">◀️ Précédente</button>
          <button id="nextBtn">Suivante ▶️</button>
          <select id="classSelect" title="Classe">
            <option value="pak-choi">pak-choi</option>
            <option value="blette">blette</option>
          </select>
        </div>
        <div class="controls" style="margin-bottom:8px;">
          <button id="undoBtn">↩️ Annuler</button>
          <button id="removeBtn" class="btn-danger">🗑️ Supprimer sélection</button>
          <button id="clearBtn" class="btn-danger">🧹 Tout effacer</button>
        </div>
        <div class="controls" style="margin-bottom:8px;">
          <button id="exportJsonBtn" class="btn-accent">📦 Export JSON (COCO-lite)</button>
          <button id="exportYoloBtn" class="btn-ok">🟢 Export YOLO (zip)</button>
          <button id="saveCloudBtn" class="btn-accent">☁️ Enregistrer sur Firebase</button>
        </div>
        <div class="footer" id="status">0 image</div>
      </div>
      <div class="card">
        <strong>Rectangles de l'image courante</strong>
        <div class="list" id="boxesList"></div>
      </div>
    </aside>
    <main>
      <div id="dropZone" class="card" style="text-align:center; padding:8px;">
        <div id="canvasWrap">
          <canvas id="canvas" width="1024" height="768"></canvas>
        </div>
        <div class="hint" style="margin-top:8px;">Cliquez-glissez pour dessiner un rectangle. Maintenez <span class="kbd">Maj</span> pour contraindre au carré.</div>
      </div>
    </main>
  </div>

  <!-- JSZip for YOLO export -->
  <script src="https://cdn.jsdelivr.net/npm/jszip@3.10.1/dist/jszip.min.js"></script>
  <script>
    // --- State ---
    const state = {
      images: [], // { name, img, width, height, boxes: [{x,y,w,h,class}], scale }
      index: 0,
      drawing: false,
      start: null,
      currentRect: null,
      selectedBoxIdx: -1,
    };

    // --- DOM ---
    const canvas = document.getElementById('canvas');
    const ctx = canvas.getContext('2d');
    const fileInput = document.getElementById('fileInput');
    const classSelect = document.getElementById('classSelect');
    const prevBtn = document.getElementById('prevBtn');
    const nextBtn = document.getElementById('nextBtn');
    const undoBtn = document.getElementById('undoBtn');
    const removeBtn = document.getElementById('removeBtn');
    const clearBtn = document.getElementById('clearBtn');
    const exportJsonBtn = document.getElementById('exportJsonBtn');
    const exportYoloBtn = document.getElementById('exportYoloBtn');
    const saveCloudBtn = document.getElementById('saveCloudBtn');
    const boxesList = document.getElementById('boxesList');
    const status = document.getElementById('status');
    const dropZone = document.getElementById('dropZone');

    // --- Helpers ---
    function fitToCanvas(img){
      const maxW = canvas.width, maxH = canvas.height;
      const ratio = Math.min(maxW / img.width, maxH / img.height);
      const w = img.width * ratio, h = img.height * ratio;
      const x = (maxW - w) / 2, y = (maxH - h) / 2;
      return { x, y, w, h, ratio };
    }
    function clamp(v, a, b){ return Math.max(a, Math.min(b, v)); }

    function draw(){
      ctx.clearRect(0,0,canvas.width, canvas.height);
      if(!state.images.length) return;
      const item = state.images[state.index];
      const {x,y,w,h,ratio} = fitToCanvas(item.img);
      item.scale = {x, y, w, h, ratio};
      ctx.drawImage(item.img, x, y, w, h);

      // existing boxes
      item.boxes.forEach((b, i)=>{
        const sx = x + b.x * ratio;
        const sy = y + b.y * ratio;
        const sw = b.w * ratio;
        const sh = b.h * ratio;
        ctx.lineWidth = 2;
        ctx.strokeStyle = i === state.selectedBoxIdx ? '#7aa2ff' : '#2ecc71';
        ctx.strokeRect(sx, sy, sw, sh);
        // label
        ctx.fillStyle = 'rgba(10,14,28,0.8)';
        ctx.fillRect(sx, sy - 18, ctx.measureText(b.class||'').width + 12, 18);
        ctx.fillStyle = '#eaf2ff';
        ctx.font = '12px ui-sans-serif';
        ctx.fillText(b.class || '', sx + 6, sy - 5);
      });

      // current drawing
      if(state.drawing && state.currentRect){
        const c = state.currentRect;
        ctx.setLineDash([6,4]);
        ctx.strokeStyle = '#7aa2ff';
        ctx.lineWidth = 2;
        ctx.strokeRect(c.x, c.y, c.w, c.h);
        ctx.setLineDash([]);
      }
      renderList();
      updateStatus();
    }

    function renderList(){
      boxesList.innerHTML = '';
      if(!state.images.length) return;
      const item = state.images[state.index];
      item.boxes.forEach((b, i)=>{
        const el = document.createElement('div');
        el.className = 'item';
        el.innerHTML = `<div><strong>${b.class}</strong><br><small>${Math.round(b.x)},${Math.round(b.y)} · ${Math.round(b.w)}×${Math.round(b.h)}</small></div>`;
        const right = document.createElement('div');
        const sel = document.createElement('select');
        sel.innerHTML = `<option value="pak-choi">pak-choi</option><option value="blette">blette</option>`;
        sel.value = b.class;
        sel.onchange = () => { b.class = sel.value; draw(); };
        const del = document.createElement('button');
        del.textContent = '✖';
        del.className = 'btn-danger';
        del.onclick = ()=>{ item.boxes.splice(i,1); state.selectedBoxIdx=-1; draw(); };
        right.style.display='flex'; right.style.gap='6px';
        right.appendChild(sel); right.appendChild(del);
        el.onclick = ()=>{ state.selectedBoxIdx = i; draw(); };
        el.appendChild(right);
        boxesList.appendChild(el);
      });
    }

    function updateStatus(){
      const n = state.images.length;
      if(!n){ status.textContent = '0 image'; return; }
      const item = state.images[state.index];
      status.textContent = `${state.index+1}/${n} – ${item.name} • ${item.boxes.length} rect.`;
    }

    function addFiles(files){
      const arr = Array.from(files || []);
      const readers = arr.map(f => new Promise((resolve)=>{
        const img = new Image();
        img.onload = ()=> resolve({ name: f.name, img, width: img.width, height: img.height, boxes: [], scale: null, file: f });
        img.src = URL.createObjectURL(f);
      }));
      Promise.all(readers).then(items => {
        state.images.push(...items);
        if(state.images.length === items.length) state.index = 0;
        draw();
      });
    }

    // --- Events: file / DnD ---
    fileInput.addEventListener('change', e => addFiles(e.target.files));
    ;['dragenter','dragover'].forEach(ev => dropZone.addEventListener(ev, e=>{ e.preventDefault(); dropZone.style.outline='2px dashed #7aa2ff'; }));
    ;['dragleave','drop'].forEach(ev => dropZone.addEventListener(ev, e=>{ e.preventDefault(); dropZone.style.outline='none'; }));
    dropZone.addEventListener('drop', e => addFiles(e.dataTransfer.files));

    // --- Canvas draw ---
    function canvasToImage(p){
      const item = state.images[state.index];
      const {x,y,ratio} = item.scale;
      const ix = (p.x - x) / ratio; // image-space
      const iy = (p.y - y) / ratio;
      return { x: ix, y: iy };
    }
    function getMousePos(evt){
      const rect = canvas.getBoundingClientRect();
      return { x: (evt.clientX - rect.left) * (canvas.width / rect.width), y: (evt.clientY - rect.top) * (canvas.height / rect.height) };
    }

    canvas.addEventListener('mousedown', e => {
      if(!state.images.length) return;
      const p = getMousePos(e);
      state.drawing = true;
      state.start = p;
      state.currentRect = { x:p.x, y:p.y, w:0, h:0 };
    });
    canvas.addEventListener('mousemove', e => {
      if(!state.drawing) return;
      const p = getMousePos(e);
      let w = p.x - state.start.x; let h = p.y - state.start.y;
      if(e.shiftKey){ const s = Math.min(Math.abs(w), Math.abs(h)); w = Math.sign(w)*s; h = Math.sign(h)*s; }
      state.currentRect = { x: state.start.x, y: state.start.y, w, h };
      draw();
    });
    function finishRect(){
      if(!state.drawing) return;
      state.drawing = false;
      const r = state.currentRect; state.currentRect = null;
      if(!r) return;
      const x = Math.min(r.x, r.x + r.w), y = Math.min(r.y, r.y + r.h);
      const w = Math.abs(r.w), h = Math.abs(r.h);
      if(w < 5 || h < 5) { draw(); return; }
      const item = state.images[state.index];
      // Convert to image pixel coords
      const p1 = canvasToImage({x, y});
      const p2 = canvasToImage({x:x+w, y:y+h});
      const bx = clamp(Math.round(p1.x), 0, item.width-1);
      const by = clamp(Math.round(p1.y), 0, item.height-1);
      const bw = clamp(Math.round(p2.x - p1.x), 1, item.width - bx);
      const bh = clamp(Math.round(p2.y - p1.y), 1, item.height - by);
      item.boxes.push({ x: bx, y: by, w: bw, h: bh, class: classSelect.value });
      state.selectedBoxIdx = item.boxes.length - 1;
      draw();
    }
    canvas.addEventListener('mouseup', finishRect);
    canvas.addEventListener('mouseleave', finishRect);

    // --- Controls ---
    prevBtn.onclick = ()=>{ if(!state.images.length) return; state.index = (state.index - 1 + state.images.length) % state.images.length; state.selectedBoxIdx=-1; draw(); };
    nextBtn.onclick = ()=>{ if(!state.images.length) return; state.index = (state.index + 1) % state.images.length; state.selectedBoxIdx=-1; draw(); };
    undoBtn.onclick = ()=>{ if(!state.images.length) return; if(state.drawing){ state.drawing=false; state.currentRect=null; draw(); return; } const item = state.images[state.index]; item.boxes.pop(); state.selectedBoxIdx=-1; draw(); };
    removeBtn.onclick = ()=>{ if(!state.images.length) return; const item = state.images[state.index]; if(state.selectedBoxIdx>-1){ item.boxes.splice(state.selectedBoxIdx,1); state.selectedBoxIdx=-1; draw(); } };
    clearBtn.onclick = ()=>{ if(!state.images.length) return; const item = state.images[state.index]; item.boxes = []; state.selectedBoxIdx=-1; draw(); };

    // Keyboard shortcuts
    window.addEventListener('keydown', e=>{
      if(e.key === 'ArrowRight'){ nextBtn.click(); }
      if(e.key === 'ArrowLeft'){ prevBtn.click(); }
      if(e.key === 'Delete' || e.key === 'Backspace'){ removeBtn.click(); }
      if(e.key.toLowerCase() === 'z'){ undoBtn.click(); }
      if(e.key.toLowerCase() === 'p'){ classSelect.value='pak-choi'; }
      if(e.key.toLowerCase() === 'b'){ classSelect.value='blette'; }
    });

    // --- Export JSON (COCO-lite) ---
    function download(filename, content, mime='application/json'){
      const blob = new Blob([content], {type: mime});
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url; a.download = filename; a.click();
      setTimeout(()=> URL.revokeObjectURL(url), 2000);
    }

    exportJsonBtn.onclick = ()=>{
      const data = {
        info: { tool: 'Annotateur pak-choi/blette', created: new Date().toISOString() },
        images: state.images.map((it, idx)=> ({ id: idx+1, file_name: it.name, width: it.width, height: it.height })),
        categories: [ {id:1, name:'pak-choi'}, {id:2, name:'blette'} ],
        annotations: []
      };
      state.images.forEach((it, idx)=>{
        it.boxes.forEach((b, j)=>{
          const cat = b.class === 'pak-choi' ? 1 : 2;
          data.annotations.push({ id: `${idx+1}-${j+1}`, image_id: idx+1, category_id: cat, bbox: [b.x, b.y, b.w, b.h] });
        });
      });
      download('annotations.json', JSON.stringify(data, null, 2));
    };

    // --- Export YOLO (zip) ---
    exportYoloBtn.onclick = async ()=>{
      if(!state.images.length){ return; }
      const zip = new JSZip();
      const names = new Set();
      state.images.forEach((it)=>{
        const base = it.name.replace(/\.[^.]+$/, '');
        const used = names.has(base) ? `${base}_${Math.random().toString(36).slice(2,6)}` : base;
        names.add(used);
        const lines = it.boxes.map(b => {
          const cls = b.class === 'pak-choi' ? 0 : 1; // YOLO class ids start at 0
          const x = (b.x + b.w/2) / it.width;
          const y = (b.y + b.h/2) / it.height;
          const w = b.w / it.width;
          const h = b.h / it.height;
          return `${cls} ${x.toFixed(6)} ${y.toFixed(6)} ${w.toFixed(6)} ${h.toFixed(6)}`;
        }).join('\n');
        zip.file(`${used}.txt`, lines + (lines?'\n':''));
      });
      zip.file('classes.txt', 'pak-choi\nblette\n');
      const blob = await zip.generateAsync({type:'blob'});
      download('yolo_labels.zip', blob, 'application/zip');
    };

    // Initial draw
    // Décide si le bouton Firebase doit s'afficher
    if (window.__FB__) { saveCloudBtn.style.display = 'inline-block'; }
    draw();

    // --- Save to Firebase (images + annotations) ---
    async function saveAllToFirebase(){
      if (!window.__FB__) { alert('Firebase non configuré'); return; }
      const { storage, db, ref, uploadBytes, getDownloadURL, collection, addDoc, serverTimestamp } = window.__FB__;
      const annCol = collection(db, 'annotations');
      for (const it of state.images){
        try {
          // 1) Upload image
          const path = `uploads/${Date.now()}_${Math.random().toString(36).slice(2,8)}_${it.name}`;
          const storageRef = ref(storage, path);
          await uploadBytes(storageRef, it.file);
          const url = await getDownloadURL(storageRef);
          // 2) Save annotation doc
          const payload = {
            file_name: it.name,
            storage_path: path,
            download_url: url,
            width: it.width,
            height: it.height,
            boxes: it.boxes.map(b=>({x:b.x,y:b.y,w:b.w,h:b.h,class:b.class})),
            created_at: serverTimestamp(),
          };
          await addDoc(annCol, payload);
        } catch (e){ console.error(e); alert('Erreur Firebase pour '+it.name+': '+e.message); }
      }
      alert('Enregistrement terminé sur Firebase ✅');
    }
    saveCloudBtn.onclick = saveAllToFirebase;
  </script>
</body>
</html>
