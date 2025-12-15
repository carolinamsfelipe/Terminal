<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Mini Terminal — Portfolio + Black-Litterman (MVP)</title>
  <style>
    :root{
      --bg:#0b1220; --card:#111a2e; --muted:#9aa4b2; --txt:#e8eef6;
      --accent:#7c5cff; --accent2:#00d4ff; --border:#22304a;
    }
    *{ box-sizing:border-box; font-family:system-ui,-apple-system,Segoe UI,Roboto,Ubuntu,Arial; }
    body{ margin:0; background:var(--bg); color:var(--txt); }
    .wrap{ max-width:1100px; margin:0 auto; padding:18px 14px 40px; }
    header{ display:flex; gap:14px; align-items:center; justify-content:space-between; margin-bottom:14px; }
    .brand{ display:flex; flex-direction:column; gap:2px; }
    .brand h1{ margin:0; font-size:18px; letter-spacing:.2px; }
    .brand .sub{ color:var(--muted); font-size:12px; }
    .tabs{ display:flex; gap:8px; flex-wrap:wrap; }
    .tabbtn{
      background:transparent; border:1px solid var(--border); color:var(--txt);
      padding:8px 10px; border-radius:12px; cursor:pointer; font-size:13px;
    }
    .tabbtn.active{ border-color:var(--accent); box-shadow:0 0 0 2px rgba(124,92,255,.15) inset; }
    .grid{ display:grid; grid-template-columns:1.25fr .75fr; gap:12px; }
    @media (max-width: 900px){ .grid{ grid-template-columns:1fr; } }
    .card{
      background:linear-gradient(180deg, rgba(124,92,255,.08), rgba(0,0,0,0)) , var(--card);
      border:1px solid var(--border); border-radius:16px; padding:12px;
    }
    .card h2{ margin:0 0 10px 0; font-size:14px; color:#d7def0; }
    .muted{ color:var(--muted); font-size:12px; }
    .row{ display:flex; gap:10px; flex-wrap:wrap; align-items:center; }
    .row > *{ flex:1; }
    label{ display:block; font-size:12px; color:var(--muted); margin:8px 0 4px; }
    input, textarea, select{
      width:100%; padding:9px 10px; border-radius:12px; border:1px solid var(--border);
      background:#0c1527; color:var(--txt); outline:none;
    }
    textarea{ min-height:110px; resize:vertical; }
    .btn{
      background:linear-gradient(90deg, var(--accent), var(--accent2));
      border:none; color:#06101c; padding:10px 12px; border-radius:12px; cursor:pointer; font-weight:700;
    }
    .btn.secondary{
      background:transparent; color:var(--txt); border:1px solid var(--border); font-weight:600;
    }
    table{ width:100%; border-collapse:separate; border-spacing:0; overflow:hidden; border-radius:14px; border:1px solid var(--border); }
    th, td{ padding:8px; border-bottom:1px solid var(--border); font-size:12px; text-align:left; }
    th{ background:#0c1527; color:#cbd5e1; position:sticky; top:0; }
    tr:last-child td{ border-bottom:none; }
    .pill{
      display:inline-block; padding:4px 8px; border-radius:999px; font-size:11px;
      background:rgba(124,92,255,.12); border:1px solid rgba(124,92,255,.28);
      color:#dcd6ff;
    }
    .mono{ font-family:ui-monospace,SFMono-Regular,Menlo,Monaco,Consolas,monospace; }
    .hidden{ display:none; }
    .split{ display:grid; grid-template-columns:1fr 1fr; gap:10px; }
    @media (max-width: 700px){ .split{ grid-template-columns:1fr; } }
    .warn{ color:#ffd48a; }
    .ok{ color:#9ff7c4; }
  </style>
</head>
<body>
  <div class="wrap">
    <header>
      <div class="brand">
        <h1>Mini Terminal (Web) — Portfolio + Black-Litterman</h1>
        <div class="sub">MVP para que lo conviertas en tu “terminal” (Argentina-friendly). Todo en 1 archivo.</div>
      </div>
      <div class="tabs">
        <button class="tabbtn active" data-tab="portfolio">Portfolio</button>
        <button class="tabbtn" data-tab="views">Vistas (BL)</button>
        <button class="tabbtn" data-tab="output">Resultados</button>
      </div>
    </header>

    <div class="grid">
      <div class="card">
        <div id="tab-portfolio">
          <h2>1) Inputs de mercado</h2>
          <div class="muted">Formato simple: definí activos, retornos “prior” (π), pesos de mercado (w<sub>mkt</sub>) y covarianza (Σ). Luego aplicás Black-Litterman con vistas.</div>

          <label>Activos (uno por línea)</label>
          <textarea id="assets">AL30
GD30
SPY</textarea>

          <div class="split">
            <div>
              <label>Retornos prior π (en % anual, una línea por activo)</label>
              <textarea id="prior">60
55
10</textarea>
            </div>
            <div>
              <label>Pesos de mercado w_mkt (suma 1, una línea por activo)</label>
              <textarea id="wmkt">0.45
0.35
0.20</textarea>
            </div>
          </div>

          <label>Matriz de covarianza Σ (filas separadas por línea, valores por espacio). (en varianzas anuales)</label>
          <textarea id="cov">0.35 0.22 0.05
0.22 0.30 0.04
0.05 0.04 0.06</textarea>

          <div class="split">
            <div>
              <label>Aversión al riesgo (λ)</label>
              <input id="lambda" type="number" step="0.1" value="2.5"/>
              <div class="muted">Usada para pesos tipo mean-variance: w ∝ Σ⁻¹ μ / λ.</div>
            </div>
            <div>
              <label>τ (tau) — incertidumbre del prior</label>
              <input id="tau" type="number" step="0.01" value="0.05"/>
              <div class="muted">Valores típicos: 0.02–0.10.</div>
            </div>
          </div>

          <div class="row" style="margin-top:10px;">
            <button class="btn" id="btnCompute">Calcular Black-Litterman + Pesos</button>
            <button class="btn secondary" id="btnLoadSample">Cargar ejemplo</button>
          </div>

          <div class="muted" style="margin-top:8px;">
            Nota: este MVP hace un “long-only” simple recortando negativos y renormalizando (para no complicarte con optimización con restricciones todavía).
          </div>
        </div>

        <div id="tab-views" class="hidden">
          <h2>2) Vistas (Black-Litterman)</h2>
          <div class="muted">
            Definí P y Q. Cada vista es una fila en P.  
            Ejemplo: “AL30 supera a GD30 por 5%” => P: [1, -1, 0] y Q: 5
          </div>

          <label>Matriz P (cada línea = una vista, coeficientes separados por espacio)</label>
          <textarea id="P">1 -1 0</textarea>

          <label>Vector Q (en % anual, una línea por vista)</label>
          <textarea id="Q">5</textarea>

          <label>Omega Ω (opcional). Si lo dejás vacío, se usa diagonal proporcional (regla simple).</label>
          <textarea id="Omega" placeholder="Ej: 0.10"></textarea>

          <div class="muted">
            Si no cargás Ω, se estima como diag(P τ Σ Pᵀ) * 1.0 (simple).
          </div>
        </div>

        <div id="tab-output" class="hidden">
          <h2>3) Resultados</h2>
          <div id="status" class="muted">Todavía no calculaste.</div>
          <div style="margin-top:10px;">
            <div class="pill">Posterior μ (BL)</div>
            <div id="muPost" class="mono muted" style="margin-top:8px; white-space:pre;"></div>
          </div>

          <div style="margin-top:12px;">
            <div class="pill">Pesos sugeridos (long-only simple)</div>
            <div id="weights" style="margin-top:8px;"></div>
          </div>

          <div style="margin-top:12px;">
            <div class="pill">Detalle</div>
            <div id="details" class="muted" style="margin-top:8px;"></div>
          </div>
        </div>
      </div>

      <div class="card">
        <h2>Guía rápida</h2>
        <div class="muted" style="line-height:1.5">
          <div><span class="pill">MVP</span> Portfolio + Black-Litterman + pesos.</div>
          <div style="margin-top:10px;">Siguiente paso típico:</div>
          <ul>
            <li>Agregar módulo “Bonos AR”: TIR, duration, paridad.</li>
            <li>Agregar módulo “Opciones”: payoff + Greeks básicos.</li>
            <li>Traer precios (API) y armar gráficos.</li>
          </ul>
          <div class="warn">Tip: para la facu, documentá supuestos (τ, λ, Ω) y casos de uso.</div>
        </div>

        <div style="margin-top:12px;">
          <h2>Checks</h2>
          <div class="muted" id="checks">—</div>
        </div>
      </div>
    </div>
  </div>

<script>
/* ========= Helpers de álgebra lineal (pequeño, sin librerías) ========= */
function parseLines(txt){
  return txt.trim().split(/\r?\n/).map(s=>s.trim()).filter(Boolean);
}
function parseVector(txt){
  return parseLines(txt).map(v=>Number(v.replace(',','.')));
}
function parseMatrix(txt){
  return parseLines(txt).map(line => line.split(/\s+/).map(x=>Number(x.replace(',','.'))));
}
function zeros(r,c){ return Array.from({length:r}, ()=>Array(c).fill(0)); }
function eye(n){
  const I=zeros(n,n); for(let i=0;i<n;i++) I[i][i]=1; return I;
}
function transpose(A){
  const r=A.length,c=A[0].length, T=zeros(c,r);
  for(let i=0;i<r;i++) for(let j=0;j<c;j++) T[j][i]=A[i][j];
  return T;
}
function matMul(A,B){
  const r=A.length, k=A[0].length, c=B[0].length;
  const out=zeros(r,c);
  for(let i=0;i<r;i++){
    for(let j=0;j<c;j++){
      let s=0;
      for(let t=0;t<k;t++) s += A[i][t]*B[t][j];
      out[i][j]=s;
    }
  }
  return out;
}
function matVec(A,v){
  const r=A.length, c=A[0].length;
  const out=new Array(r).fill(0);
  for(let i=0;i<r;i++){
    let s=0; for(let j=0;j<c;j++) s+=A[i][j]*v[j];
    out[i]=s;
  }
  return out;
}
function add(A,B){
  const r=A.length,c=A[0].length, out=zeros(r,c);
  for(let i=0;i<r;i++) for(let j=0;j<c;j++) out[i][j]=A[i][j]+B[i][j];
  return out;
}
function sub(A,B){
  const r=A.length,c=A[0].length, out=zeros(r,c);
  for(let i=0;i<r;i++) for(let j=0;j<c;j++) out[i][j]=A[i][j]-B[i][j];
  return out;
}
function scale(A,s){
  const r=A.length,c=A[0].length, out=zeros(r,c);
  for(let i=0;i<r;i++) for(let j=0;j<c;j++) out[i][j]=A[i][j]*s;
  return out;
}
function vecAdd(a,b){ return a.map((x,i)=>x+b[i]); }
function vecSub(a,b){ return a.map((x,i)=>x-b[i]); }
function vecScale(a,s){ return a.map(x=>x*s); }
function diagFrom(v){
  const n=v.length, D=zeros(n,n);
  for(let i=0;i<n;i++) D[i][i]=v[i];
  return D;
}
function toCol(v){ return v.map(x=>[x]); }
function fromCol(M){ return M.map(r=>r[0]); }

/* Gauss-Jordan inversion for small matrices */
function invert(A){
  const n=A.length;
  const M = A.map(row=>row.slice());
  const I = eye(n);
  for(let i=0;i<n;i++){
    // pivot
    let pivot=i;
    for(let r=i+1;r<n;r++){
      if(Math.abs(M[r][i]) > Math.abs(M[pivot][i])) pivot=r;
    }
    if(Math.abs(M[pivot][i]) < 1e-12) throw new Error("Matriz singular o mal condicionada.");
    [M[i],M[pivot]]=[M[pivot],M[i]];
    [I[i],I[pivot]]=[I[pivot],I[i]];
    // normalize
    const p=M[i][i];
    for(let j=0;j<n;j++){ M[i][j]/=p; I[i][j]/=p; }
    // eliminate
    for(let r=0;r<n;r++){
      if(r===i) continue;
      const f=M[r][i];
      for(let j=0;j<n;j++){ M[r][j]-=f*M[i][j]; I[r][j]-=f*I[i][j]; }
    }
  }
  return I;
}
function normalizeToOne(w){
  const s=w.reduce((a,b)=>a+b,0);
  if(Math.abs(s) < 1e-12) return w;
  return w.map(x=>x/s);
}
function longOnlySimple(w){
  const clipped=w.map(x=>Math.max(0,x));
  const s=clipped.reduce((a,b)=>a+b,0);
  if(s<=1e-12) return normalizeToOne(w.map(_=>1/w.length));
  return clipped.map(x=>x/s);
}

/* ========= Black-Litterman (forma estándar) =========
μ_bl = [ (τΣ)^{-1} + P'Ω^{-1}P ]^{-1} * [ (τΣ)^{-1}π + P'Ω^{-1}Q ]
*/
function blackLitterman(pi, Sigma, P, Q, tau, Omega){
  const n=pi.length;
  const tauSigma = scale(Sigma, tau);
  const invTauSigma = invert(tauSigma);

  const PT = transpose(P);
  const invOmega = invert(Omega);

  const A = add(invTauSigma, matMul(matMul(PT, invOmega), P)); // n x n

  const b1 = matMul(invTauSigma, toCol(pi));                 // n x 1
  const b2 = matMul(matMul(PT, invOmega), toCol(Q));         // n x 1
  const b  = add(b1,b2);                                     // n x 1

  const mu = matMul(invert(A), b);                           // n x 1
  return fromCol(mu);
}

function omegaDefault(P, Sigma, tau){
  // Ω = diag(P τΣ P')  (simple)
  const tauSigma=scale(Sigma,tau);
  const Pt = transpose(P);
  const M = matMul(matMul(P, tauSigma), Pt); // k x k
  const k=M.length;
  const d=new Array(k).fill(0).map((_,i)=>Math.max(1e-9, M[i][i]));
  return diagFrom(d);
}

/* ========= UI ========= */
const tabButtons=[...document.querySelectorAll(".tabbtn")];
const tabs = {
  portfolio: document.getElementById("tab-portfolio"),
  views: document.getElementById("tab-views"),
  output: document.getElementById("tab-output")
};
tabButtons.forEach(b=>{
  b.addEventListener("click", ()=>{
    tabButtons.forEach(x=>x.classList.remove("active"));
    b.classList.add("active");
    Object.values(tabs).forEach(t=>t.classList.add("hidden"));
    tabs[b.dataset.tab].classList.remove("hidden");
  });
});

function setChecks(msg, ok=true){
  document.getElementById("checks").innerHTML = ok
    ? `<span class="ok">✔</span> ${msg}`
    : `<span class="warn">⚠</span> ${msg}`;
}

function renderWeights(assets, w){
  let html = `<table><thead><tr><th>Activo</th><th>Peso</th></tr></thead><tbody>`;
  assets.forEach((a,i)=>{
    html += `<tr><td>${a}</td><td>${(w[i]*100).toFixed(2)}%</td></tr>`;
  });
  html += `</tbody></table>`;
  return html;
}

function fmtVec(assets, v){
  return assets.map((a,i)=>`${a.padEnd(6)}  ${(v[i]).toFixed(4)}`).join("\n");
}

document.getElementById("btnLoadSample").addEventListener("click", ()=>{
  document.getElementById("assets").value = `AL30\nGD30\nSPY`;
  document.getElementById("prior").value  = `60\n55\n10`;
  document.getElementById("wmkt").value   = `0.45\n0.35\n0.20`;
  document.getElementById("cov").value    = `0.35 0.22 0.05\n0.22 0.30 0.04\n0.05 0.04 0.06`;
  document.getElementById("P").value      = `1 -1 0`;
  document.getElementById("Q").value      = `5`;
  document.getElementById("Omega").value  = ``;
  document.getElementById("lambda").value = `2.5`;
  document.getElementById("tau").value    = `0.05`;
  setChecks("Ejemplo cargado. Tocá “Calcular”.", true);
});

document.getElementById("btnCompute").addEventListener("click", ()=>{
  try{
    const assets=parseLines(document.getElementById("assets").value);
    const piPct=parseVector(document.getElementById("prior").value);
    const wmkt=parseVector(document.getElementById("wmkt").value);
    const Sigma=parseMatrix(document.getElementById("cov").value);

    const lambda=Number(document.getElementById("lambda").value);
    const tau=Number(document.getElementById("tau").value);

    const P=parseMatrix(document.getElementById("P").value);
    const Qpct=parseVector(document.getElementById("Q").value);

    const n=assets.length;

    // basic validations
    if(piPct.length!==n || wmkt.length!==n || Sigma.length!==n || Sigma[0].length!==n){
      throw new Error("Dimensiones inválidas: assets/prior/w_mkt/Σ deben coincidir.");
    }
    const wsum = wmkt.reduce((a,b)=>a+b,0);
    if(Math.abs(wsum-1)>1e-6) setChecks(`w_mkt no suma 1 (suma ${wsum.toFixed(4)}). Igual calculo.`, false);
    else setChecks("Inputs OK.", true);

    // convert % to decimals
    const pi=piPct.map(x=>x/100);
    const Q=Qpct.map(x=>x/100);

    // Omega
    let OmegaTxt = document.getElementById("Omega").value.trim();
    let Omega;
    if(!OmegaTxt){
      Omega = omegaDefault(P, Sigma, tau);
    } else {
      // allow: diagonal as vector (one per view) OR full matrix
      const lines=parseLines(OmegaTxt);
      if(lines.length===1 && lines[0].split(/\s+/).length===1){
        Omega = [[Number(lines[0])]];
      } else if(lines.length===Q.length && lines.every(l=>l.split(/\s+/).length===1)){
        Omega = diagFrom(lines.map(v=>Number(v)));
      } else {
        Omega = parseMatrix(OmegaTxt);
      }
    }

    // BL posterior mean
    const muBL = blackLitterman(pi, Sigma, P, Q, tau, Omega);

    // mean-variance weights (unconstrained), then long-only simple
    const invSigma = invert(Sigma);
    const wUn = vecScale(matVec(invSigma, muBL), 1/Math.max(1e-12, lambda));
    const wNorm = normalizeToOne(wUn);
    const wLO = longOnlySimple(wNorm);

    // render output
    document.querySelector('[data-tab="output"]').click();

    document.getElementById("status").innerHTML =
      `<span class="ok">✔</span> Calculado. μ posterior y pesos listos.`;

    document.getElementById("muPost").textContent = fmtVec(assets, muBL);

    document.getElementById("weights").innerHTML = renderWeights(assets, wLO);

    document.getElementById("details").innerHTML =
      `<div>λ=${lambda.toFixed(2)}, τ=${tau.toFixed(3)}</div>
       <div>Pesos “unconstrained” normalizados (antes de long-only): ${wNorm.map(x=>(x*100).toFixed(1)+'%').join(' | ')}</div>
       <div class="muted">Long-only simple: negativos a 0 y renormaliza. (Luego podés reemplazar por optimización con restricciones.)</div>`;
  } catch(e){
    setChecks(e.message, false);
    alert(e.message);
  }
});
</script>
</body>
</html>
