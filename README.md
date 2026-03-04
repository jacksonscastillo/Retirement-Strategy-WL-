# Retirement-Strategy-WL
/* ══════════════════════════════════════════════════════════════════
ROBINHOOD BAR REPLAY — NM WHOLE LIFE VOLATILITY BUFFER

CLEAN MODEL:
• Monthly Contribution → 100% into S&P 500 equities
• Annual NM Premium → 100% into Whole Life cash value
• “Market Only” comparison → premium $ redirected into SPX
• No redundant allocation slider
══════════════════════════════════════════════════════════════════ */

const SP500 = [
-0.0501,0.0197,0.0967,-0.0308,-0.0221,0.0239,-0.0163,0.0607,-0.0528,-0.0049,-0.0801,0.0046,
0.0346,-0.0923,-0.0642,0.0768,0.0051,-0.0250,-0.0108,-0.0626,-0.0817,0.0181,0.0752,0.0076,
-0.0156,-0.0208,0.0367,-0.0614,-0.0091,-0.0726,-0.0790,0.0049,-0.1100,0.0864,0.0571,-0.0603,
-0.0274,-0.0170,0.0084,0.0810,0.0509,0.0113,0.0162,0.0179,-0.0119,0.0550,0.0071,0.0508,
0.0173,0.0122,-0.0164,-0.0168,0.0121,0.0180,-0.0343,0.0023,0.0094,0.0140,0.0386,0.0325,
-0.0253,0.0189,-0.0191,-0.0201,0.0300,-0.0001,0.0360,-0.0112,0.0069,-0.0177,0.0352,-0.0010,
0.0255,0.0005,0.0111,0.0122,-0.0309,0.0001,0.0051,0.0213,0.0246,0.0315,0.0165,0.0126,
0.0141,-0.0218,0.0100,0.0433,0.0325,-0.0178,-0.0321,0.0131,0.0358,0.0148,-0.0440,-0.0086,
-0.0612,-0.0348,-0.0060,0.0475,0.0107,-0.0867,-0.0099,0.0122,-0.0908,-0.1694,-0.0741,0.0078,
-0.0857,-0.1099,0.0854,0.0939,0.0531,0.0002,0.0741,0.0336,0.0357,-0.0198,0.0574,0.0178,
-0.0371,0.0285,0.0588,0.0148,-0.0821,-0.0539,0.0689,-0.0474,0.0876,0.0369,-0.0023,0.0653,
0.0226,0.0320,-0.0010,0.0285,-0.0135,-0.0178,-0.0214,-0.0568,-0.0718,0.1077,-0.0051,0.0085,
0.0436,0.0396,0.0313,-0.0075,-0.0627,0.0396,0.0126,0.0199,0.0245,-0.0198,0.0028,0.0071,
0.0504,0.0111,0.0360,0.0181,0.0209,-0.0150,0.0495,-0.0311,0.0297,0.0444,0.0280,0.0236,
-0.0359,0.0432,0.0069,0.0062,0.0209,0.0191,-0.0150,0.0377,-0.0157,0.0232,0.0245,-0.0042,
-0.0306,0.0549,-0.0174,0.0085,0.0105,-0.0211,0.0200,-0.0644,-0.0264,0.0802,0.0005,-0.0175,
-0.0507,-0.0041,0.0660,0.0027,0.0153,0.0009,0.0356,-0.0012,-0.0012,-0.0194,0.0342,0.0182,
0.0179,0.0372,-0.0004,0.0091,0.0109,0.0048,0.0193,0.0005,0.0193,0.0224,0.0281,0.0098,
0.0565,-0.0389,-0.0269,0.0027,0.0216,0.0046,0.0360,0.0305,0.0043,-0.0694,0.0179,-0.0918,
0.0790,0.0297,0.0179,0.0393,-0.0658,0.0689,0.0131,-0.0181,0.0172,0.0204,0.0340,0.0286,
-0.0016,-0.0841,-0.1251,0.1268,0.0451,0.0184,0.0551,0.0701,-0.0381,-0.0277,0.1075,0.0362,
-0.0127,0.0261,0.0424,0.0530,0.0055,0.0222,0.0233,0.0290,-0.0476,0.0665,-0.0083,0.0436,
-0.0570,-0.0323,0.0358,-0.0880,0.0001,-0.0838,0.0912,-0.0424,-0.0946,0.0793,0.0536,-0.0586,
0.0618,-0.0261,0.0351,0.0146,0.0025,0.0626,0.0306,-0.0177,-0.0487,-0.0218,0.0889,0.0454,
0.0159,0.0516,0.0308,-0.0420,0.0480,0.0345,0.0113,0.0225,0.0200,-0.0100,0.0560,-0.0258
];

const MO = [“Jan”,“Feb”,“Mar”,“Apr”,“May”,“Jun”,“Jul”,“Aug”,“Sep”,“Oct”,“Nov”,“Dec”];
const lbl = i => `${MO[i%12]} ${2000+Math.floor(i/12)}`;

const CRISES = [
{ name: “DOT-COM”, start: 3, end: 35 },
{ name: “GFC”, start: 93, end: 114 },
{ name: “COVID”, start: 241, end: 243 },
];

// ─── NM Policy Engine ───
const NM_DIV = 0.0575;
const NM_GUAR = 0.03;
const COI_T = [[25,1.1],[30,1.35],[35,1.85],[40,2.8],[45,4.3],[50,7.1],[55,11.5],[60,18.2],[65,28],[70,42],[75,65],[80,98]];
function getCOI(age, face) {
let r = 1.1;
for (const [a, c] of COI_T) if (age >= a) r = c;
return (face / 1000) * r;
}
const SURR = [0.12,0.25,0.42,0.58,0.70,0.80,0.87,0.92,0.95,0.97,0.98,0.99,1];

function simulate({ initial, monthly, age, premium, payTo }) {
const moPrem = premium / 12;
const face = premium * 18;
const moGuar = Math.pow(1 + NM_GUAR, 1/12) - 1;
const moDiv = Math.pow(1 + NM_DIV, 1/12) - 1;

let pureSpx = initial;
let bSpx = initial;
let cv = 0;
let totPrem = 0, totCOI = 0, totDiv = 0, totDist = 0;
let peakPure = initial, peakBlend = initial;
let cumIn = initial;
const d = [];

for (let i = 0; i < SP500.length; i++) {
const r = SP500[i];
const curAge = age + i / 12;
const yr = Math.floor(i / 12);
const paying = curAge < payTo;
const isCrisis = CRISES.some(c => i >= c.start && i <= c.end);

```
// Total monthly out-of-pocket is the same for both strategies
const totalMonthlyOOP = monthly + (paying ? moPrem : 0);
cumIn += totalMonthlyOOP;

// ── STRATEGY A: ALL money into SPX ──
pureSpx = pureSpx * (1 + r) + totalMonthlyOOP;
peakPure = Math.max(peakPure, pureSpx);

// ── STRATEGY B: Market sleeve + NM WL sleeve ──
bSpx = bSpx * (1 + r) + monthly;

const pr = paying ? moPrem : 0;
totPrem += pr;
const coi = getCOI(Math.floor(curAge), face) / 12;
totCOI += coi;
const exp = pr * Math.max(0.02, 0.08 - yr * 0.005);
const gInt = cv * moGuar;
const dInt = yr >= 1 ? cv * (moDiv - moGuar) : 0;
totDiv += dInt;
cv = Math.max(0, cv + pr - coi - exp + gInt + dInt);

const sv = cv * (yr < SURR.length ? SURR[yr] : 1);
const blendTotal = bSpx + cv;
peakBlend = Math.max(peakBlend, blendTotal);

let dist = 0;
if (isCrisis && r < -0.03 && cv > 5000) {
dist = Math.min(monthly * 0.5, cv * 0.02);
totDist += dist;
}

d.push({
i, label: lbl(i), r,
pure: Math.round(pureSpx),
pureDD: +((pureSpx - peakPure) / peakPure * 100).toFixed(2),
bSpx: Math.round(bSpx),
cv: Math.round(cv),
total: Math.round(blendTotal),
blendDD: +((blendTotal - peakBlend) / peakBlend * 100).toFixed(2),
sv: Math.round(sv),
db: Math.round(Math.max(face, face + totDiv * 0.7)),
totPrem: Math.round(totPrem),
totCOI: Math.round(totCOI),
totDiv: Math.round(totDiv),
coi: Math.round(coi * 12),
dist, totDist: Math.round(totDist),
age: Math.floor(curAge),
isCrisis, paying,
totalIn: Math.round(cumIn),
});
```

}
return d;
}

// ─── SVG Line Chart ───
function RHChart({ data, idx, field, baseValue, width, height, onHover, hoverIdx, ghostField }) {
if (!data.length || width <= 0) return null;
const end = Math.max(2, idx + 1);
const pts = data.slice(0, end);
const allV = pts.flatMap(d => { const v = [d[field]]; if (ghostField) v.push(d[ghostField]); return v; });
const min = Math.min(…allV) * 0.97, max = Math.max(…allV) * 1.03;
const rng = max - min || 1;
const tX = i => (i / Math.max(1, pts.length - 1)) * width;
const tY = v => height - ((v - min) / rng) * height;

const path = pts.map((d, i) => `${i===0?'M':'L'}${tX(i).toFixed(1)},${tY(d[field]).toFixed(1)}`).join(’ ’);
const last = pts[pts.length-1]?.[field]||0;
const clr = last >= baseValue ? “#00C805” : “#FF5000”;
const fill = path + ` L${tX(pts.length-1).toFixed(1)},${height} L0,${height} Z`;

let ghostPath = null;
if (ghostField) ghostPath = pts.map((d, i) => `${i===0?'M':'L'}${tX(i).toFixed(1)},${tY(d[ghostField]).toFixed(1)}`).join(’ ’);

const hi = hoverIdx !== null ? Math.min(hoverIdx, pts.length-1) : null;

return (
<svg width={width} height={height} style={{ display:“block”, overflow:“visible” }}
onMouseMove={e => { const r = e.currentTarget.getBoundingClientRect(); const x = e.clientX-r.left; onHover?.(Math.max(0,Math.min(Math.round((x/width)*(pts.length-1)),pts.length-1))); }}
onMouseLeave={() => onHover?.(null)}
onTouchMove={e => { const r = e.currentTarget.getBoundingClientRect(); const x = e.touches[0].clientX-r.left; onHover?.(Math.max(0,Math.min(Math.round((x/width)*(pts.length-1)),pts.length-1))); }}
onTouchEnd={() => onHover?.(null)}>
<defs>
<linearGradient id={`g-${field}`} x1=“0” y1=“0” x2=“0” y2=“1”>
<stop offset="0%" stopColor={clr} stopOpacity={0.08}/>
<stop offset="100%" stopColor={clr} stopOpacity={0}/>
</linearGradient>
</defs>
{CRISES.map(c => { if(c.start>=pts.length)return null; return <rect key={c.name} x={tX(c.start)} y={0} width={Math.max(1,tX(Math.min(c.end,pts.length-1))-tX(c.start))} height={height} fill="rgba(255,80,0,0.035)"/>; })}
<path d={fill} fill={`url(#g-${field})`}/>
{ghostPath && <path d={ghostPath} fill="none" stroke="rgba(255,255,255,0.1)" strokeWidth={1.5} strokeDasharray="5 4"/>}
<path d={path} fill="none" stroke={clr} strokeWidth={2.2} strokeLinecap="round" strokeLinejoin="round"/>
{hi !== null ? <>
<line x1={tX(hi)} y1={0} x2={tX(hi)} y2={height} stroke="rgba(255,255,255,0.1)" strokeWidth={1} strokeDasharray="3 3"/>
<circle cx={tX(hi)} cy={tY(pts[hi]?.[field]||0)} r={5} fill={clr} stroke="#1a1d23" strokeWidth={2}/>
{ghostField && <circle cx={tX(hi)} cy={tY(pts[hi]?.[ghostField]||0)} r={4} fill="rgba(255,255,255,0.2)" stroke="#1a1d23" strokeWidth={2}/>}
</> : <circle cx={tX(pts.length-1)} cy={tY(last)} r={4} fill={clr}><animate attributeName="r" values="4;6;4" dur="2s" repeatCount="indefinite"/></circle>}
</svg>
);
}

const $ = v => `$${Math.round(Math.abs(v)).toLocaleString()}`;
const $s = v => { const a=Math.abs(v); return a>=1e6?`$${(v/1e6).toFixed(2)}M`:`$${Math.round(v).toLocaleString()}`; };

export default function App() {
const [initial, setInitial] = useState(200000);
const [monthly, setMonthly] = useState(1500);
const [age, setAge] = useState(35);
const [premium, setPremium] = useState(12000);
const [payTo, setPayTo] = useState(65);
const [idx, setIdx] = useState(-1);
const [playing, setPlaying] = useState(false);
const [speed, setSpeed] = useState(1);
const [hover, setHover] = useState(null);
const [setup, setSetup] = useState(false);
const [mode, setMode] = useState(“blend”);
const tmr = useRef(null);
const cBox = useRef(null);
const [cw, setCw] = useState(600);

const data = useMemo(() => simulate({ initial, monthly, age, premium, payTo }), [initial, monthly, age, premium, payTo]);

useEffect(() => { const m=()=>{if(cBox.current)setCw(cBox.current.offsetWidth)};m();window.addEventListener(“resize”,m);return()=>window.removeEventListener(“resize”,m); }, []);

useEffect(() => {
if (!playing) { clearTimeout(tmr.current); return; }
const tick = () => {
setIdx(prev => {
const next = prev+1;
if (next >= data.length) { setPlaying(false); return data.length-1; }
const r = SP500[next];
const crisis = CRISES.some(c => next>=c.start && next<=c.end);
let del;
if (crisis && r < -0.08) del = 1500/speed;
else if (crisis && r < -0.05) del = 1000/speed;
else if (crisis && r < -0.02) del = 650/speed;
else if (crisis) del = 420/speed;
else if (r < -0.04) del = 500/speed;
else del = 50/speed;
tmr.current = setTimeout(tick, del);
return next;
});
};
tmr.current = setTimeout(tick, 50/speed);
return () => clearTimeout(tmr.current);
}, [playing, speed, data.length]);

const vi = hover!==null ? Math.min(hover, idx>=0?idx:data.length-1) : (idx>=0?idx:data.length-1);
const cur = data[vi]||data[data.length-1];
const moPrem = premium/12;

const purePnL = cur.pure - cur.totalIn;
const blendPnL = cur.total - cur.totalIn;
const purePct = (purePnL/cur.totalIn)*100;
const blendPct = (blendPnL/cur.totalIn)*100;
const bufAdv = blendPnL - purePnL;

const mainVal = mode===“pure”?cur.pure:cur.total;
const mainPnL = mode===“pure”?purePnL:blendPnL;
const mainPct = mode===“pure”?purePct:blendPct;
const up = mainPnL>=0;
const G=”#00C805”, R=”#FF5000”;
const acc = up?G:R;
const inC = cur.isCrisis;
const bigL = cur.r < -0.05;

return (
<div style={{ minHeight:“100vh”, background:”#1a1d23”, color:”#fff”, fontFamily:”‘Inter’,-apple-system,sans-serif” }}>
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800;900&display=swap" rel="stylesheet"/>
<style>{`*{box-sizing:border-box;margin:0;padding:0} input[type=range]{-webkit-appearance:none;appearance:none;background:rgba(255,255,255,0.08);border-radius:4px;height:3px;outline:none;cursor:pointer} input[type=range]::-webkit-slider-thumb{-webkit-appearance:none;width:14px;height:14px;border-radius:50%;background:#fff;border:none} @keyframes flash{0%{background:rgba(255,80,0,0.12)}100%{background:transparent}} @keyframes up{from{opacity:0;transform:translateY(4px)}to{opacity:1;transform:translateY(0)}} @keyframes pulse{0%,100%{opacity:1}50%{opacity:.4}}`}</style>

```
{/* NAV */}
<div style={{ display:"flex", alignItems:"center", justifyContent:"space-between", padding:"10px 16px", borderBottom:"1px solid rgba(255,255,255,0.05)" }}>
<div style={{ display:"flex", alignItems:"center", gap:8 }}>
<div style={{ width:26, height:26, borderRadius:6, background:"linear-gradient(135deg,#1a6b3c,#2fa84d)", display:"flex", alignItems:"center", justifyContent:"center" }}>
<span style={{ fontSize:10, fontWeight:900, color:"#fff" }}>NM</span>
</div>
<div>
<div style={{ fontSize:13, fontWeight:700 }}>Volatility Buffer</div>
<div style={{ fontSize:9, color:"rgba(255,255,255,0.25)", fontWeight:500 }}>AAA/Aaa/A++ · {(NM_DIV*100).toFixed(2)}% Dividend</div>
</div>
</div>
<button onClick={()=>setSetup(!setup)} style={{ background:"rgba(255,255,255,0.06)", border:"none", borderRadius:20, padding:"5px 14px", color:"rgba(255,255,255,0.5)", fontSize:11, fontWeight:600, cursor:"pointer" }}>
{setup?"Done":"Edit"}
</button>
</div>

{/* SETTINGS */}
{setup && (
<div style={{ padding:"14px 16px", borderBottom:"1px solid rgba(255,255,255,0.05)", background:"rgba(0,0,0,0.2)", animation:"up 0.2s ease-out" }}>
<div style={{ display:"grid", gridTemplateColumns:"repeat(auto-fit, minmax(150px, 1fr))", gap:14 }}>
{[
{ l:"Initial SPX Investment", v:initial, s:setInitial, min:25000, max:1e6, step:25000 },
{ l:"Monthly → SPX", v:monthly, s:setMonthly, min:250, max:10000, step:250 },
{ l:"NM Annual Premium → WL", v:premium, s:setPremium, min:3000, max:50000, step:1000 },
{ l:"Client Start Age", v:age, s:setAge, min:25, max:55, step:1, isAge:true },
].map(p => (
<div key={p.l}>
<div style={{ display:"flex", justifyContent:"space-between", marginBottom:3 }}>
<span style={{ fontSize:9, color:"rgba(255,255,255,0.3)", fontWeight:500 }}>{p.l}</span>
<span style={{ fontSize:12, fontWeight:800 }}>{p.isAge?p.v:$(p.v)}</span>
</div>
<input type="range" min={p.min} max={p.max} step={p.step} value={p.v} onChange={e=>p.s(+e.target.value)} style={{ width:"100%" }}/>
</div>
))}
<div>
<div style={{ fontSize:9, color:"rgba(255,255,255,0.3)", fontWeight:500, marginBottom:5 }}>NM Policy Type</div>
<div style={{ display:"flex", gap:4 }}>
{[65,100].map(p=>(
<button key={p} onClick={()=>setPayTo(p)} style={{ flex:1, padding:"6px", borderRadius:6, fontSize:11, fontWeight:700, cursor:"pointer", background:payTo===p?"rgba(0,200,5,0.1)":"rgba(255,255,255,0.03)", border:`1px solid ${payTo===p?"rgba(0,200,5,0.25)":"rgba(255,255,255,0.06)"}`, color:payTo===p?G:"rgba(255,255,255,0.3)" }}>
Pay-to-{p}
</button>
))}
</div>
</div>
</div>
<div style={{ marginTop:12, padding:"10px 12px", background:"rgba(255,255,255,0.02)", borderRadius:8, border:"1px solid rgba(255,255,255,0.04)" }}>
<div style={{ fontSize:10, fontWeight:700, color:"rgba(255,255,255,0.4)", marginBottom:4 }}>How It Works — Same Total Dollars, Different Strategy</div>
<div style={{ fontSize:10, color:"rgba(255,255,255,0.22)", lineHeight:1.7 }}>
<strong style={{ color:"rgba(255,255,255,0.5)" }}>Market + NM WL:</strong> {$(monthly)}/mo → S&P 500 &nbsp;·&nbsp; {$(moPrem)}/mo → NM Whole Life Premium<br/>
<strong style={{ color:"rgba(255,255,255,0.5)" }}>100% Market:</strong> {$(monthly)}/mo + {$(moPrem)}/mo → ALL into S&P 500 (premium redirected)<br/>
Both invest <strong style={{ color:"#fff" }}>{$(monthly + moPrem)}/mo</strong> total. Apples-to-apples.
</div>
</div>
</div>
)}

<div style={{ maxWidth:940, margin:"0 auto", padding:"0 16px" }}>
{/* HEADER */}
<div style={{ paddingTop:20 }}>
<div style={{ fontSize:11, color:"rgba(255,255,255,0.3)", fontWeight:500, marginBottom:2 }}>
{mode==="pure"?"100% S&P 500 Portfolio":mode==="blend"?"S&P 500 + NM Whole Life Portfolio":"Side by Side Comparison"}
</div>
{mode !== "split" ? (
<>
<div style={{ fontSize:38, fontWeight:900, lineHeight:1, letterSpacing:-1.5 }}>{$(mainVal)}</div>
<div style={{ display:"flex", alignItems:"center", gap:6, marginTop:4, flexWrap:"wrap" }}>
<span style={{ fontSize:15, fontWeight:700, color:acc }}>{mainPnL>=0?"+":"-"}{$(Math.abs(mainPnL))}</span>
<span style={{ fontSize:13, fontWeight:600, color:acc }}>({mainPct>=0?"+":""}{mainPct.toFixed(2)}%)</span>
<span style={{ fontSize:11, color:"rgba(255,255,255,0.2)", marginLeft:4 }}>{cur.label} · Age {cur.age}</span>
{inC && <span style={{ fontSize:9, fontWeight:800, color:R, background:"rgba(255,80,0,0.1)", padding:"2px 7px", borderRadius:3, animation:"pulse 1s infinite" }}>CRISIS</span>}
</div>
</>
) : (
<div style={{ display:"flex", gap:20, alignItems:"baseline", flexWrap:"wrap" }}>
<div>
<div style={{ fontSize:9, color:"rgba(255,255,255,0.2)" }}>100% S&P 500</div>
<div style={{ fontSize:26, fontWeight:900, lineHeight:1 }}>{$s(cur.pure)}</div>
<div style={{ fontSize:12, fontWeight:700, color:purePnL>=0?G:R }}>{purePnL>=0?"+":"-"}{$(Math.abs(purePnL))} ({purePct>=0?"+":""}{purePct.toFixed(1)}%)</div>
</div>
<div style={{ fontSize:18, color:"rgba(255,255,255,0.08)" }}>vs</div>
<div>
<div style={{ fontSize:9, color:"rgba(255,255,255,0.2)" }}>S&P + NM WL</div>
<div style={{ fontSize:26, fontWeight:900, lineHeight:1 }}>{$s(cur.total)}</div>
<div style={{ fontSize:12, fontWeight:700, color:blendPnL>=0?G:R }}>{blendPnL>=0?"+":"-"}{$(Math.abs(blendPnL))} ({blendPct>=0?"+":""}{blendPct.toFixed(1)}%)</div>
</div>
<div>
<div style={{ fontSize:9, color:"rgba(255,255,255,0.2)" }}>NM Buffer</div>
<div style={{ fontSize:20, fontWeight:900, color:bufAdv>=0?G:R, lineHeight:1 }}>{bufAdv>=0?"+":"-"}{$(Math.abs(bufAdv))}</div>
</div>
</div>
)}
</div>

{/* MODE TABS */}
<div style={{ display:"flex", gap:0, margin:"12px 0 8px", borderRadius:8, overflow:"hidden", border:"1px solid rgba(255,255,255,0.06)", width:"fit-content" }}>
{[
{ k:"blend", l:"S&P + NM WL" },
{ k:"pure", l:"100% S&P 500" },
{ k:"split", l:"Side by Side" },
].map(m=>(
<button key={m.k} onClick={()=>setMode(m.k)} style={{ padding:"7px 16px", border:"none", cursor:"pointer", fontSize:11, fontWeight:600, background:mode===m.k?"rgba(255,255,255,0.08)":"transparent", color:mode===m.k?"#fff":"rgba(255,255,255,0.2)" }}>{m.l}</button>
))}
</div>

{/* CHART */}
<div ref={cBox} style={{ position:"relative", marginTop:4 }}>
{mode==="split" ? (
<div style={{ display:"flex", flexDirection:"column", gap:4 }}>
{[
{ f:"pure", l:"100% S&P 500 — all dollars into market", v:cur.pure, pnl:purePnL, pct:purePct },
{ f:"total", l:"S&P 500 + NM Cash Value combined", v:cur.total, pnl:blendPnL, pct:blendPct },
{ f:"cv", l:`NM Cash Value (${(NM_DIV*100).toFixed(2)}% dividend · steady growth)`, v:cur.cv, pnl:cur.cv-cur.totPrem, pct:cur.totPrem>0?((cur.cv-cur.totPrem)/cur.totPrem*100):0 },
].map((ch,ci)=>(
<div key={ch.f}>
<div style={{ display:"flex", justifyContent:"space-between", alignItems:"baseline", marginBottom:2, marginTop:ci>0?8:0 }}>
<span style={{ fontSize:9, color:"rgba(255,255,255,0.2)", fontWeight:500 }}>{ch.l}</span>
<div>
<span style={{ fontSize:13, fontWeight:800 }}>{$(ch.v)}</span>
<span style={{ fontSize:10, fontWeight:700, color:ch.pnl>=0?G:R, marginLeft:6 }}>{ch.pnl>=0?"+":"-"}{$(Math.abs(ch.pnl))}</span>
</div>
</div>
<RHChart data={data} idx={idx>=0?idx:data.length-1} field={ch.f} baseValue={ch.f==="cv"?0:initial} width={cw} height={ch.f==="cv"?80:125} onHover={setHover} hoverIdx={hover}/>
</div>
))}
</div>
) : (
<>
<RHChart data={data} idx={idx>=0?idx:data.length-1} field={mode==="pure"?"pure":"total"} ghostField={mode==="blend"?"pure":null} baseValue={initial} width={cw} height={280} onHover={setHover} hoverIdx={hover}/>
{mode==="blend" && <div style={{ fontSize:9, color:"rgba(255,255,255,0.12)", marginTop:4 }}>━ S&P + NM WL &nbsp;&nbsp; ┅ 100% S&P (reference)</div>}
</>
)}
</div>

{/* PLAYBACK */}
<div style={{ display:"flex", alignItems:"center", gap:10, marginTop:14, padding:"10px 0", borderTop:"1px solid rgba(255,255,255,0.04)" }}>
<button onClick={()=>{if(idx>=data.length-1||idx<0)setIdx(0);setPlaying(!playing);}} style={{ width:38, height:38, borderRadius:"50%", border:"none", cursor:"pointer", flexShrink:0, background:playing?"rgba(255,80,0,0.15)":G, color:playing?R:"#000", fontSize:14, fontWeight:900, display:"flex", alignItems:"center", justifyContent:"center" }}>
{playing?"❚❚":"▶"}
</button>
<div style={{ flex:1, position:"relative", height:4, background:"rgba(255,255,255,0.05)", borderRadius:2, cursor:"pointer" }} onClick={e=>{const r=e.currentTarget.getBoundingClientRect();setIdx(Math.floor(((e.clientX-r.left)/r.width)*(data.length-1)));setPlaying(false);}}>
<div style={{ position:"absolute", top:0, left:0, height:"100%", borderRadius:2, width:`${((idx>=0?idx:data.length-1)/(data.length-1))*100}%`, background:acc, transition:playing?"none":"width 0.15s" }}/>
</div>
{[0.5,1,2,3].map(s=>(
<button key={s} onClick={()=>setSpeed(s)} style={{ padding:"3px 8px", borderRadius:4, border:"none", cursor:"pointer", fontSize:10, fontWeight:700, background:speed===s?"rgba(255,255,255,0.1)":"transparent", color:speed===s?"#fff":"rgba(255,255,255,0.2)" }}>{s}x</button>
))}
<button onClick={()=>{setIdx(-1);setPlaying(false);setHover(null);}} style={{ padding:"3px 10px", borderRadius:4, border:"1px solid rgba(255,255,255,0.06)", background:"transparent", color:"rgba(255,255,255,0.25)", fontSize:10, fontWeight:600, cursor:"pointer" }}>Reset</button>
</div>

{/* LIVE CRISIS P&L */}
{inC && idx>=0 && (
<div style={{ padding:"14px 16px", borderRadius:10, marginTop:4, background:bigL?"rgba(255,80,0,0.05)":"rgba(255,255,255,0.015)", border:`1px solid ${bigL?"rgba(255,80,0,0.12)":"rgba(255,255,255,0.04)"}`, animation:bigL?"flash 0.5s ease-out":"none" }}>
<div style={{ display:"flex", alignItems:"center", gap:5, marginBottom:8 }}>
<div style={{ width:5, height:5, borderRadius:"50%", background:R, animation:"pulse 1s infinite" }}/>
<span style={{ fontSize:10, fontWeight:700, color:"rgba(255,255,255,0.4)" }}>LIVE P&L · {cur.label} · S&P: {cur.r>=0?"+":""}{(cur.r*100).toFixed(1)}%</span>
</div>
<div style={{ display:"flex", flexWrap:"wrap" }}>
{[
{ l:"100% S&P 500", v:cur.pure, pnl:purePnL, pct:purePct, note:`${$(monthly)}+${$(moPrem)}/mo all in SPX` },
{ l:"S&P + NM WL", v:cur.total, pnl:blendPnL, pct:blendPct, note:`${$(monthly)}/mo SPX · ${$(moPrem)}/mo NM premium` },
].map((p,i)=>(
<div key={i} style={{ flex:1, minWidth:150, padding:"6px 14px", borderRight:i===0?"1px solid rgba(255,255,255,0.04)":"none" }}>
<div style={{ fontSize:9, color:"rgba(255,255,255,0.22)" }}>{p.l}</div>
<div style={{ fontSize:20, fontWeight:900, lineHeight:1.1 }}>{$(p.v)}</div>
<div style={{ fontSize:13, fontWeight:700, color:p.pnl>=0?G:R }}>{p.pnl>=0?"+":"-"}{$(Math.abs(p.pnl))} ({p.pct>=0?"+":""}{p.pct.toFixed(1)}%)</div>
<div style={{ fontSize:8, color:"rgba(255,255,255,0.12)", marginTop:2 }}>{p.note}</div>
</div>
))}
<div style={{ flex:1, minWidth:130, padding:"6px 14px" }}>
<div style={{ fontSize:9, color:"rgba(255,255,255,0.22)" }}>NM Buffer</div>
<div style={{ fontSize:20, fontWeight:900, color:bufAdv>=0?G:"rgba(255,255,255,0.4)", lineHeight:1.1 }}>{bufAdv>=0?"+":"-"}{$(Math.abs(bufAdv))}</div>
<div style={{ fontSize:9, color:"rgba(255,255,255,0.18)", marginTop:2 }}>CV: {$(cur.cv)} · DD: {cur.blendDD.toFixed(1)}% vs {cur.pureDD.toFixed(1)}%</div>
</div>
</div>
{cur.dist>0 && <div style={{ marginTop:8, padding:"6px 10px", background:"rgba(0,200,5,0.05)", borderRadius:6, border:"1px solid rgba(0,200,5,0.1)", fontSize:10, fontWeight:600, color:G }}>
Tax-free policy loan: {$(cur.dist)} — avoided selling equities at {(cur.r*100).toFixed(1)}%
</div>}
</div>
)}

{/* STATS */}
<div style={{ display:"grid", gridTemplateColumns:"repeat(auto-fit, minmax(130px, 1fr))", gap:1, margin:"12px 0 0", background:"rgba(255,255,255,0.02)", borderRadius:10, overflow:"hidden", border:"1px solid rgba(255,255,255,0.03)" }}>
{[
{ l:"NM Cash Value", v:$(cur.cv), s:`Age ${cur.age}`, c:G },
{ l:"Death Benefit", v:$s(cur.db), s:"Tax-free", c:"rgba(255,255,255,0.25)" },
{ l:"Premiums Paid", v:$(cur.totPrem), s:cur.paying?`${$(moPrem)}/mo`:"Paid up", c:cur.paying?"rgba(255,255,255,0.25)":G },
{ l:"Dividends Earned", v:$(cur.totDiv), s:`${(NM_DIV*100).toFixed(2)}%`, c:"#fbbf24" },
{ l:"COI Charges", v:$(cur.totCOI), s:`${$(cur.coi)}/yr now`, c:R },
{ l:"Distributions", v:$(cur.totDist), s:"Tax-free loans", c:G },
{ l:"Total Invested", v:$(cur.totalIn), s:"Same both strategies", c:"rgba(255,255,255,0.25)" },
{ l:"NM Rating", v:"AAA", s:"Highest possible", c:G },
].map((s,i)=>(
<div key={i} style={{ padding:"11px 13px", background:"#1a1d23" }}>
<div style={{ fontSize:9, color:"rgba(255,255,255,0.18)", fontWeight:500, marginBottom:1 }}>{s.l}</div>
<div style={{ fontSize:14, fontWeight:800 }}>{s.v}</div>
<div style={{ fontSize:9, color:s.c, fontWeight:600, marginTop:1 }}>{s.s}</div>
</div>
))}
</div>

{/* COI */}
<div style={{ margin:"12px 0", padding:"14px 16px", background:"rgba(255,255,255,0.015)", borderRadius:10, border:"1px solid rgba(255,255,255,0.03)" }}>
<div style={{ fontSize:11, fontWeight:700, color:"rgba(255,255,255,0.35)", marginBottom:10 }}>Cost of Insurance · Pay-to-{payTo} · Face: {$s(premium*18)}</div>
<div style={{ display:"flex", gap:5, flexWrap:"wrap" }}>
{COI_T.filter(([a])=>a>=age&&a<=age+35).map(([a,r])=>{
const c=(premium*18/1000)*r;
const now=cur.age>=a&&cur.age<a+5;
return <div key={a} style={{ padding:"6px 10px", borderRadius:6, minWidth:75, textAlign:"center", background:now?"rgba(255,80,0,0.06)":"rgba(255,255,255,0.015)", border:`1px solid ${now?"rgba(255,80,0,0.15)":"rgba(255,255,255,0.03)"}` }}>
<div style={{ fontSize:9, color:now?"#fff":"rgba(255,255,255,0.2)", fontWeight:600 }}>{a}–{a+4}</div>
<div style={{ fontSize:12, fontWeight:800, color:now?R:"rgba(255,255,255,0.35)" }}>{$(c)}</div>
<div style={{ fontSize:8, color:"rgba(255,255,255,0.1)" }}>{(c/premium*100).toFixed(1)}%</div>
</div>;
})}
</div>
</div>

{/* EY */}
<div style={{ margin:"0 0 20px", padding:"16px", background:"rgba(251,191,36,0.02)", borderRadius:10, border:"1px solid rgba(251,191,36,0.06)" }}>
<div style={{ display:"flex", alignItems:"center", gap:6, marginBottom:10 }}>
<span style={{ background:"#fbbf24", color:"#000", padding:"2px 7px", borderRadius:3, fontSize:9, fontWeight:900 }}>EY</span>
<span style={{ fontSize:11, fontWeight:700, color:"rgba(255,255,255,0.35)" }}>Research Highlights</span>
</div>
<div style={{ display:"grid", gridTemplateColumns:"repeat(auto-fit, minmax(180px, 1fr))", gap:14 }}>
{[
{ m:"5–8 yrs", d:"Extended portfolio longevity drawing from WL during bear markets" },
{ m:"+0.5–1%", d:"Higher sustainable withdrawal rate with guaranteed CV floor" },
{ m:"30–40%", d:"Reduced max drawdown — the volatility buffer effect" },
{ m:"15–25%", d:"Optimal WL allocation for risk-adjusted returns from AAA carriers" },
].map((e,i)=>(
<div key={i}>
<div style={{ fontSize:20, fontWeight:900, color:"#fbbf24" }}>{e.m}</div>
<div style={{ fontSize:10, color:"rgba(255,255,255,0.18)", lineHeight:1.5, marginTop:2 }}>{e.d}</div>
</div>
))}
</div>
</div>

<div style={{ padding:"10px 0 20px", fontSize:8, color:"rgba(255,255,255,0.1)", lineHeight:1.6 }}>
S&P 500 monthly returns 2000–2024. NM policy at {(NM_DIV*100).toFixed(2)}% dividend scale (not guaranteed). COI illustrative. "100% Market" redirects premium into SPX — same total dollars invested. Not financial advice. Consult an NM representative.
</div>
</div>
</div>
```

);
}
