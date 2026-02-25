import { useState, useMemo, useEffect } from "react";

const INITIAL_USERS = [
  { email:"admin@innotel.com",    pass:"Admin2026!", name:"Administrador", role:"admin",  dept:null },
  { email:"movistar@innotel.com", pass:"Mov2026!",   name:"Usr. Movistar", role:"editor", dept:"Movistar" },
  { email:"entel@innotel.com",    pass:"Ent2026!",   name:"Usr. Entel",    role:"editor", dept:"Entel" },
  { email:"wom@innotel.com",      pass:"Wom2026!",   name:"Usr. Wom",      role:"editor", dept:"Wom" },
];
const MONTHS  = ["Enero","Febrero","Marzo","Abril","Mayo","Junio","Julio","Agosto","Septiembre","Octubre","Noviembre","Diciembre"];
const CATS    = ["Comisiones/Ventas","Gastos Internos","Cargos Externos","Nómina/Sueldos","Gastos Operacionales","RRHH"];
const CCOLORS = ["#4f7fff","#7c5cfc","#22c55e","#f97316","#eab308","#ef4444"];
const DCOLORS = { Movistar:"#00b5e2", Entel:"#ff6b35", Wom:"#a78bfa" };
const DEPTS   = ["Movistar","Entel","Wom"];

const fmtMoney = n => (n < 0 ? "-$" : "$") + Math.abs(Math.round(n)).toLocaleString("es-CL");
const deptColor = d => DCOLORS[d] || "#4f7fff";
const deptRgb   = d => d==="Movistar"?"0,181,226":d==="Entel"?"255,107,53":"167,139,250";

// Week helpers
function getWeekRange(date) {
  const d = new Date(date);
  const day = d.getDay(); // 0=sun
  const mon = new Date(d); mon.setDate(d.getDate() - (day===0?6:day-1));
  const sun = new Date(mon); sun.setDate(mon.getDate()+6);
  return { mon, sun };
}
function isSameWeek(dateStr, ref) {
  const { mon, sun } = getWeekRange(ref);
  const d = new Date(dateStr);
  return d >= mon && d <= sun;
}
function weekLabel(date) {
  const { mon, sun } = getWeekRange(date);
  const fmt = d => d.toLocaleDateString("es-CL",{day:"2-digit",month:"2-digit"});
  return `${fmt(mon)} – ${fmt(sun)}`;
}

// Demo data with commission fields
const DEMO = [
  // Febrero 2026 — semana 1 (1-7 feb)
  {id:"d1",date:"2026-02-03",dept:"Movistar",cat:"Comisiones/Ventas",desc:"Pago diario vendedores",worker:"Juan Pérez",month:"Febrero",amount:880000,by:"admin",notes:"",
   isComision:true,qtTotal:22,qtCorrectas:20,qtIncorrectas:2,valorUnitario:40000,netoComision:800000},
  {id:"d2",date:"2026-02-03",dept:"Movistar",cat:"Gastos Internos",desc:"Arriendo oficina",worker:"—",month:"Febrero",amount:320000,by:"admin",notes:"Mes completo"},
  // Febrero 2026 — semana 3 (15-21 feb)
  {id:"d3",date:"2026-02-17",dept:"Entel",cat:"Comisiones/Ventas",desc:"Pago diario vendedores",worker:"María González",month:"Febrero",amount:600000,by:"entel",notes:"",
   isComision:true,qtTotal:15,qtCorrectas:14,qtIncorrectas:1,valorUnitario:40000,netoComision:560000},
  {id:"d4",date:"2026-02-07",dept:"Entel",cat:"Nómina/Sueldos",desc:"Sueldo base vendedor",worker:"Carlos Rojas",month:"Febrero",amount:450000,by:"admin",notes:""},
  {id:"d5",date:"2026-02-10",dept:"Wom",cat:"Gastos Operacionales",desc:"Materiales de oficina",worker:"—",month:"Febrero",amount:45000,by:"admin",notes:""},
  {id:"d6",date:"2026-02-18",dept:"Wom",cat:"Comisiones/Ventas",desc:"Pago diario vendedores",worker:"Luis Vega",month:"Febrero",amount:480000,by:"admin",notes:"",
   isComision:true,qtTotal:12,qtCorrectas:11,qtIncorrectas:1,valorUnitario:40000,netoComision:440000},
  {id:"d7",date:"2026-02-12",dept:"Wom",cat:"Cargos Externos",desc:"Proveedor logística",worker:"—",month:"Febrero",amount:180000,by:"admin",notes:"Factura #3421"},
  // Febrero 2026 — semana 4 (22-28 feb) — 74 ventas Movistar
  {id:"d8",date:"2026-02-24",dept:"Movistar",cat:"Comisiones/Ventas",desc:"Pago diario vendedores",worker:"Ana Torres",month:"Febrero",amount:4440000,by:"admin",notes:"",
   isComision:true,qtTotal:74,qtCorrectas:74,qtIncorrectas:0,valorUnitario:60000,netoComision:4440000},
  // Enero 2026
  {id:"d9",date:"2026-01-15",dept:"Movistar",cat:"Nómina/Sueldos",desc:"Sueldo vendedor enero",worker:"Ana Torres",month:"Enero",amount:380000,by:"admin",notes:""},
  {id:"d10",date:"2026-01-20",dept:"Entel",cat:"Gastos Internos",desc:"Servicios básicos",worker:"—",month:"Enero",amount:65000,by:"entel",notes:""},
  {id:"d11",date:"2026-01-25",dept:"Wom",cat:"Comisiones/Ventas",desc:"Pago diario vendedores",worker:"Pedro Soto",month:"Enero",amount:320000,by:"admin",notes:"",
   isComision:true,qtTotal:8,qtCorrectas:8,qtIncorrectas:0,valorUnitario:40000,netoComision:320000},
];

const DEMO_BILLING = [
  {id:"b1",dept:"Movistar",month:"Febrero",
   baf:{qty:620,val:52000},tv:{qty:95,val:32000},voz:{qty:0,val:0},
   totalCalculado:620*52000+95*32000,totalFactura:37430000,notas:"Liquidación Feb 2026"},
  {id:"b2",dept:"Entel",month:"Febrero",
   baf:{qty:380,val:48000},tv:{qty:80,val:28000},voz:{qty:20,val:25000},
   totalCalculado:380*48000+80*28000+20*25000,totalFactura:21740000,notas:""},
  {id:"b3",dept:"Wom",month:"Febrero",
   baf:{qty:250,val:45000},tv:{qty:45,val:26000},voz:{qty:15,val:22000},
   totalCalculado:250*45000+45*26000+15*22000,totalFactura:13920000,notas:""},
  {id:"b4",dept:"Movistar",month:"Enero",
   baf:{qty:580,val:52000},tv:{qty:80,val:32000},voz:{qty:20,val:30000},
   totalCalculado:580*52000+80*32000+20*30000,totalFactura:34760000,notas:""},
];

// ── CALIDAD helpers ──
const PAGO = "Cliente pagó";
const parseCalidadExcel = (rows) => {
  // rows[0] = headers, rows[1..] = data
  const headers = rows[0].map(h => String(h||"").trim().toUpperCase());
  const idx = {
    rut:     headers.findIndex(h=>h.includes("RUT")),
    orden:   headers.findIndex(h=>h.includes("ORDEN")),
    cuenta:  headers.findIndex(h=>h.includes("CUENTA")),
    region:  headers.findIndex(h=>h.includes("REGION")||h.includes("REGIÓN")),
    comuna:  headers.findIndex(h=>h.includes("COMUNA")),
    usuario: headers.findIndex(h=>h.includes("USUARIO")||h.includes("CRM")),
    mes1:    headers.findIndex(h=>h.includes("ESTADO MES 1")||h.includes("MES 1")),
    mes2:    headers.findIndex(h=>h.includes("ESTADO MES 2")||h.includes("MES 2")),
    mes3:    headers.findIndex(h=>h.includes("ESTADO MES 3")||h.includes("MES 3")),
    mes:     headers.findIndex(h=>h==="MES"),
  };
  return rows.slice(1).filter(r=>r&&r[idx.rut]).map((r,i)=>({
    id: `q-${i}`,
    rut:     String(r[idx.rut]||""),
    orden:   String(r[idx.orden]||""),
    cuenta:  String(r[idx.cuenta]||""),
    region:  String(r[idx.region]||""),
    comuna:  String(r[idx.comuna]||""),
    usuario: String(r[idx.usuario]||"Sin asignar").trim(),
    mes1:    String(r[idx.mes1]||"Sin datos"),
    mes2:    String(r[idx.mes2]||"Sin datos"),
    mes3:    String(r[idx.mes3]||"Sin datos"),
    camada:  String(r[idx.mes]||"Sin datos"),
  }));
};

// ── SVG DONUT ──
function DonutChart({ data, total }) {
  if (!total) return <div style={{color:"#4a5168",fontSize:13,textAlign:"center",padding:"60px 0",width:"100%"}}>Sin datos para este período</div>;
  const R=78,r=48,cx=110,cy=110;
  let sa=-Math.PI/2;
  const slices = data.map((item,i) => {
    const pct=item.value/total, ang=pct*2*Math.PI, ea=sa+ang, lg=ang>Math.PI?1:0;
    const x1=cx+R*Math.cos(sa),y1=cy+R*Math.sin(sa),x2=cx+R*Math.cos(ea),y2=cy+R*Math.sin(ea);
    const ix1=cx+r*Math.cos(sa),iy1=cy+r*Math.sin(sa),ix2=cx+r*Math.cos(ea),iy2=cy+r*Math.sin(ea);
    const mid=sa+ang/2,lx=cx+(R+r)/2*Math.cos(mid),ly=cy+(R+r)/2*Math.sin(mid),ps=(pct*100).toFixed(1);
    const el=(<g key={i}><path d={`M${x1} ${y1} A${R} ${R} 0 ${lg} 1 ${x2} ${y2} L${ix2} ${iy2} A${r} ${r} 0 ${lg} 0 ${ix1} ${iy1}Z`} fill={item.color} stroke="#0a0b0e" strokeWidth="2"><title>{item.label}: {fmtMoney(item.value)} ({ps}%)</title></path>{pct>0.06&&<text x={lx} y={ly} textAnchor="middle" dominantBaseline="middle" fontSize="10" fill="white" fontFamily="monospace" fontWeight="600">{ps}%</text>}</g>);
    sa=ea; return el;
  });
  return (
    <div style={{display:"flex",alignItems:"center",gap:16,width:"100%"}}>
      <svg width="220" height="220" viewBox="0 0 220 220" style={{flexShrink:0}}>
        {slices}
        <text x={cx} y={cy-9} textAnchor="middle" fontSize="11" fill="#7a8399" fontFamily="sans-serif">Total</text>
        <text x={cx} y={cy+10} textAnchor="middle" fontSize="13" fill="#e8ecf4" fontFamily="sans-serif" fontWeight="700">{fmtMoney(total)}</text>
      </svg>
      <div style={{flex:1,overflow:"hidden"}}>
        {data.map((item,i)=>(
          <div key={i} style={{display:"flex",alignItems:"center",gap:7,marginBottom:9}}>
            <div style={{width:10,height:10,borderRadius:2,background:item.color,flexShrink:0}}/>
            <span style={{fontSize:11,color:"#7a8399",overflow:"hidden",textOverflow:"ellipsis",whiteSpace:"nowrap"}}>{item.label}</span>
          </div>
        ))}
      </div>
    </div>
  );
}

// ── MODAL ──
function Modal({ onClose, onSave, entry, userDept }) {
  const [dept,    setDept]    = useState(entry?.dept   || userDept || "Movistar");
  const [cat,     setCat]     = useState(entry?.cat    || CATS[0]);
  const [desc,    setDesc]    = useState(entry?.desc   || "");
  const [worker,  setWorker]  = useState(entry?.worker === "—" ? "" : (entry?.worker || ""));
  const [month,   setMonth]   = useState(entry?.month  || MONTHS[new Date().getMonth()]);
  const [amount,  setAmount]  = useState(entry?.isComision ? "" : (entry?.amount || ""));
  const [date,    setDate]    = useState(entry?.date   || new Date().toISOString().split("T")[0]);
  const [notes,   setNotes]   = useState(entry?.notes  || "");
  // Commission fields
  const [qtTotal,       setQtTotal]       = useState(entry?.qtTotal       || "");
  const [qtCorrectas,   setQtCorrectas]   = useState(entry?.qtCorrectas   || "");
  const [qtIncorrectas, setQtIncorrectas] = useState(entry?.qtIncorrectas || "");
  const [valorUnit,     setValorUnit]     = useState(entry?.valorUnitario || "");

  const isComision = cat === "Comisiones/Ventas";
  const totalBruto = isComision ? (parseFloat(qtTotal)||0) * (parseFloat(valorUnit)||0) : parseFloat(amount)||0;
  const netoComision = isComision ? ((parseFloat(qtCorrectas)||0) - (parseFloat(qtIncorrectas)||0)) * (parseFloat(valorUnit)||0) : 0;
  const incorrectasMonto = isComision ? (parseFloat(qtIncorrectas)||0) * (parseFloat(valorUnit)||0) : 0;

  const save = () => {
    if (!desc.trim()||!date) { alert("Completa los campos obligatorios"); return; }
    if (userDept && dept !== userDept) { alert("Sin permiso para este departamento"); return; }
    if (isComision) {
      if (!qtTotal||!valorUnit) { alert("Completa cantidad y valor unitario"); return; }
      onSave({ dept,cat,desc:desc.trim(),worker:worker.trim()||"—",month,date,notes:notes.trim(),
        amount: totalBruto, isComision:true,
        qtTotal:parseFloat(qtTotal), qtCorrectas:parseFloat(qtCorrectas)||0,
        qtIncorrectas:parseFloat(qtIncorrectas)||0, valorUnitario:parseFloat(valorUnit),
        netoComision });
    } else {
      if (!amount) { alert("Completa el monto"); return; }
      onSave({ dept,cat,desc:desc.trim(),worker:worker.trim()||"—",month,amount:parseFloat(amount),date,notes:notes.trim(),isComision:false });
    }
  };

  const fi = { width:"100%",background:"#181b22",border:"1px solid #1e2330",borderRadius:8,padding:"9px 12px",color:"#e8ecf4",fontFamily:"inherit",fontSize:13,outline:"none" };
  const lbl = txt => <label style={{fontSize:11,color:"#7a8399",textTransform:"uppercase",letterSpacing:".08em",display:"block",marginBottom:6}}>{txt}</label>;

  return (
    <div onClick={e=>e.target===e.currentTarget&&onClose()} style={{position:"fixed",inset:0,background:"rgba(0,0,0,.75)",display:"flex",alignItems:"center",justifyContent:"center",zIndex:200,backdropFilter:"blur(4px)"}}>
      <div style={{background:"#111318",border:"1px solid #2a3045",borderRadius:16,padding:32,width:520,maxWidth:"95vw",maxHeight:"90vh",overflowY:"auto"}}>
        <div style={{fontWeight:700,fontSize:18,marginBottom:24}}>{entry?"Editar Gasto":"Agregar Gasto"}</div>

        <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:16}}>
          <div>{lbl("Departamento")}<select style={fi} value={dept} onChange={e=>setDept(e.target.value)}>{DEPTS.map(d=><option key={d} disabled={!!userDept&&d!==userDept}>{d}</option>)}</select></div>
          <div>{lbl("Categoría")}<select style={fi} value={cat} onChange={e=>setCat(e.target.value)}>{CATS.map(c=><option key={c}>{c}</option>)}</select></div>
          <div style={{gridColumn:"1/-1"}}>{lbl("Descripción *")}<input style={fi} value={desc} onChange={e=>setDesc(e.target.value)} placeholder="Ej: Pago diario vendedores"/></div>
          <div>{lbl("Trabajador")}<input style={fi} value={worker} onChange={e=>setWorker(e.target.value)} placeholder="Nombre"/></div>
          <div>{lbl("Mes")}<select style={fi} value={month} onChange={e=>setMonth(e.target.value)}>{MONTHS.map(m=><option key={m}>{m}</option>)}</select></div>
          <div>{lbl("Fecha *")}<input style={fi} type="date" value={date} onChange={e=>setDate(e.target.value)}/></div>

          {/* COMISIONES FIELDS */}
          {isComision ? (
            <div style={{gridColumn:"1/-1",background:"rgba(79,127,255,.06)",border:"1px solid rgba(79,127,255,.15)",borderRadius:10,padding:16}}>
                <div style={{fontSize:12,color:"#4f7fff",fontWeight:600,marginBottom:14}}>📊 Detalle de Comisiones</div>
                <div style={{display:"grid",gridTemplateColumns:"1fr 1fr 1fr",gap:12,marginBottom:14}}>
                  <div>
                    {lbl("Total ventas")}
                    <input style={fi} type="number" min="0" value={qtTotal} onChange={e=>setQtTotal(e.target.value)} placeholder="22"/>
                  </div>
                  <div>
                    {lbl("✅ Correctas")}
                    <input style={{...fi,borderColor:"rgba(34,197,94,.3)"}} type="number" min="0" value={qtCorrectas} onChange={e=>setQtCorrectas(e.target.value)} placeholder="20"/>
                  </div>
                  <div>
                    {lbl("❌ Incorrectas")}
                    <input style={{...fi,borderColor:"rgba(239,68,68,.3)"}} type="number" min="0" value={qtIncorrectas} onChange={e=>setQtIncorrectas(e.target.value)} placeholder="2"/>
                  </div>
                </div>
                <div style={{marginBottom:14}}>
                  {lbl("Valor unitario ($)")}
                  <input style={fi} type="number" min="0" value={valorUnit} onChange={e=>setValorUnit(e.target.value)} placeholder="40000"/>
                </div>
                {/* Live calculation */}
                {qtTotal && valorUnit && (
                  <div style={{background:"#0a0b0e",borderRadius:8,padding:12,display:"grid",gridTemplateColumns:"1fr 1fr 1fr",gap:8,textAlign:"center"}}>
                    <div>
                      <div style={{fontSize:10,color:"#7a8399",textTransform:"uppercase",letterSpacing:".06em",marginBottom:4}}>Total pagado</div>
                      <div style={{fontWeight:700,fontSize:15,color:"#e8ecf4"}}>{fmtMoney(totalBruto)}</div>
                    </div>
                    <div>
                      <div style={{fontSize:10,color:"#7a8399",textTransform:"uppercase",letterSpacing:".06em",marginBottom:4}}>Incorrectas</div>
                      <div style={{fontWeight:700,fontSize:15,color:"#ef4444"}}>-{fmtMoney(incorrectasMonto)}</div>
                    </div>
                    <div>
                      <div style={{fontSize:10,color:"#7a8399",textTransform:"uppercase",letterSpacing:".06em",marginBottom:4}}>Neto real</div>
                      <div style={{fontWeight:700,fontSize:15,color:"#22c55e"}}>{fmtMoney(netoComision)}</div>
                    </div>
                  </div>
                )}
              </div>
          ) : (
            <div>{lbl("Monto ($) *")}<input style={fi} type="number" value={amount} onChange={e=>setAmount(e.target.value)} placeholder="0"/></div>
          )}

          <div style={{gridColumn:"1/-1"}}>{lbl("Notas")}<input style={fi} value={notes} onChange={e=>setNotes(e.target.value)} placeholder="Observaciones adicionales"/></div>
        </div>

        <div style={{display:"flex",gap:10,justifyContent:"flex-end",marginTop:24}}>
          <button onClick={onClose} style={{background:"#181b22",border:"1px solid #2a3045",borderRadius:8,padding:"9px 20px",color:"#7a8399",cursor:"pointer",fontFamily:"inherit",fontSize:13}}>Cancelar</button>
          <button onClick={save} style={{background:"#4f7fff",border:"none",borderRadius:8,padding:"9px 20px",color:"#fff",cursor:"pointer",fontWeight:600,fontSize:13}}>Guardar</button>
        </div>
      </div>
    </div>
  );
}

// ── BILLING MODAL ──
function BillingModal({ onClose, onSave, entry }) {
  const [dept,   setDept]   = useState(entry?.dept  || "Movistar");
  const [month,  setMonth]  = useState(entry?.month || MONTHS[new Date().getMonth()]);
  const [bafQty, setBafQty] = useState(entry?.baf?.qty || "");
  const [bafVal, setBafVal] = useState(entry?.baf?.val || "");
  const [tvQty,  setTvQty]  = useState(entry?.tv?.qty  || "");
  const [tvVal,  setTvVal]  = useState(entry?.tv?.val  || "");
  const [vozQty, setVozQty] = useState(entry?.voz?.qty || "");
  const [vozVal, setVozVal] = useState(entry?.voz?.val || "");
  const [totalFactura, setTotalFactura] = useState(entry?.totalFactura || "");
  const [notas,  setNotas]  = useState(entry?.notas || "");
  const [tieneReliq, setTieneReliq] = useState(entry?.reliquidacion != null ? true : false);
  const [reliqMonto, setReliqMonto] = useState(entry?.reliquidacion?.monto || "");

  const bafTotal = (parseFloat(bafQty)||0) * (parseFloat(bafVal)||0);
  const tvTotal  = (parseFloat(tvQty)||0)  * (parseFloat(tvVal)||0);
  const vozTotal = (parseFloat(vozQty)||0) * (parseFloat(vozVal)||0);
  const totalCalculado = bafTotal + tvTotal + vozTotal;
  const diff = (parseFloat(totalFactura)||0) - totalCalculado;
  const totalFinal = (parseFloat(totalFactura)||0) + (tieneReliq ? (parseFloat(reliqMonto)||0) : 0);

  const fi  = { width:"100%",background:"#181b22",border:"1px solid #1e2330",borderRadius:8,padding:"9px 12px",color:"#e8ecf4",fontFamily:"inherit",fontSize:13,outline:"none",boxSizing:"border-box" };
  const lbl = txt => <label style={{fontSize:10,color:"#7a8399",textTransform:"uppercase",letterSpacing:".08em",display:"block",marginBottom:5}}>{txt}</label>;

  const save = () => {
    if (!totalFactura) { alert("Ingresa el Total de la Factura"); return; }
    if (tieneReliq && !reliqMonto) { alert("Ingresa el monto de la reliquidación"); return; }
    onSave({
      dept, month, notas,
      baf: { qty: parseFloat(bafQty)||0, val: parseFloat(bafVal)||0 },
      tv:  { qty: parseFloat(tvQty)||0,  val: parseFloat(tvVal)||0  },
      voz: { qty: parseFloat(vozQty)||0, val: parseFloat(vozVal)||0 },
      totalCalculado,
      totalFactura: parseFloat(totalFactura)||0,
      reliquidacion: tieneReliq ? { monto: parseFloat(reliqMonto)||0 } : null,
      totalFinal,
    });
  };

  const rowStyle = { display:"grid", gridTemplateColumns:"1fr 1fr 1fr", gap:10, marginBottom:12, alignItems:"end" };
  const badge = (label, color) => <div style={{background:`rgba(${color},.12)`,border:`1px solid rgba(${color},.3)`,borderRadius:6,padding:"5px 10px",fontSize:11,fontWeight:700,color:`rgb(${color})`,textAlign:"center"}}>{label}</div>;

  return (
    <div onClick={e=>e.target===e.currentTarget&&onClose()} style={{position:"fixed",inset:0,background:"rgba(0,0,0,.75)",display:"flex",alignItems:"center",justifyContent:"center",zIndex:200,backdropFilter:"blur(4px)"}}>
      <div style={{background:"#111318",border:"1px solid #2a3045",borderRadius:16,padding:28,width:520,maxWidth:"95vw",maxHeight:"90vh",overflowY:"auto"}}>
        <div style={{fontWeight:700,fontSize:18,marginBottom:20}}>{entry?"Editar Facturación":"Registrar Facturación"}</div>

        {/* Dept + Mes */}
        <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:14,marginBottom:16}}>
          <div>{lbl("Departamento")}<select style={fi} value={dept} onChange={e=>setDept(e.target.value)}>{DEPTS.map(d=><option key={d}>{d}</option>)}</select></div>
          <div>{lbl("Mes")}<select style={fi} value={month} onChange={e=>setMonth(e.target.value)}>{MONTHS.map(m=><option key={m}>{m}</option>)}</select></div>
        </div>

        {/* Header labels */}
        <div style={{display:"grid",gridTemplateColumns:"1fr 1fr 1fr",gap:10,marginBottom:6}}>
          <div>{badge("BAF","0,181,226")}</div>
          <div>{badge("TV","167,139,250")}</div>
          <div>{badge("VOZ","34,197,94")}</div>
        </div>

        {/* Qty row */}
        <div style={rowStyle}>
          <div>{lbl("Cantidad BAF")}<input style={fi} type="number" value={bafQty} onChange={e=>setBafQty(e.target.value)} placeholder="620"/></div>
          <div>{lbl("Cantidad TV")}<input style={fi} type="number" value={tvQty} onChange={e=>setTvQty(e.target.value)} placeholder="95"/></div>
          <div>{lbl("Cantidad VOZ")}<input style={fi} type="number" value={vozQty} onChange={e=>setVozQty(e.target.value)} placeholder="0"/></div>
        </div>

        {/* Val row */}
        <div style={rowStyle}>
          <div>{lbl("Valor Unit. BAF ($)")}<input style={fi} type="number" value={bafVal} onChange={e=>setBafVal(e.target.value)} placeholder="52000"/></div>
          <div>{lbl("Valor Unit. TV ($)")}<input style={fi} type="number" value={tvVal} onChange={e=>setTvVal(e.target.value)} placeholder="32000"/></div>
          <div>{lbl("Valor Unit. VOZ ($)")}<input style={fi} type="number" value={vozVal} onChange={e=>setVozVal(e.target.value)} placeholder="30000"/></div>
        </div>

        {/* Subtotals */}
        {(bafQty||tvQty||vozQty) && (
          <div style={{display:"grid",gridTemplateColumns:"1fr 1fr 1fr",gap:10,marginBottom:16}}>
            {[["BAF",bafTotal,"0,181,226"],[" TV",tvTotal,"167,139,250"],["VOZ",vozTotal,"34,197,94"]].map(([lbl,val,c])=>(
              <div key={lbl} style={{background:"#0a0b0e",borderRadius:7,padding:"8px 10px",textAlign:"center"}}>
                <div style={{fontSize:10,color:"#7a8399",marginBottom:2}}>{lbl}</div>
                <div style={{fontWeight:700,fontSize:13,color:`rgb(${c})`}}>{fmtMoney(val)}</div>
              </div>
            ))}
          </div>
        )}

        {/* Total calculado */}
        <div style={{background:"#0a0b0e",borderRadius:8,padding:"10px 14px",marginBottom:14,display:"flex",justifyContent:"space-between",alignItems:"center"}}>
          <span style={{fontSize:12,color:"#7a8399"}}>Total calculado (BAF+TV+VOZ)</span>
          <span style={{fontWeight:700,fontSize:16,color:"#e8ecf4"}}>{fmtMoney(totalCalculado)}</span>
        </div>

        {/* Total factura real */}
        <div style={{marginBottom:14}}>
          {lbl("Total de la Factura ($)")}
          <input style={{...fi,fontSize:16,fontWeight:700,color:"#22c55e"}} type="number" value={totalFactura} onChange={e=>setTotalFactura(e.target.value)} placeholder="Ingresa el total real de la factura"/>
        </div>

        {/* Diferencia */}
        {totalFactura && totalCalculado > 0 && (
          <div style={{background:Math.abs(diff)<1000?"rgba(34,197,94,.08)":"rgba(239,68,68,.08)",border:`1px solid ${Math.abs(diff)<1000?"rgba(34,197,94,.2)":"rgba(239,68,68,.2)"}`,borderRadius:8,padding:"10px 14px",marginBottom:14,display:"flex",justifyContent:"space-between",alignItems:"center"}}>
            <span style={{fontSize:12,color:"#7a8399"}}>Diferencia (factura - calculado)</span>
            <span style={{fontWeight:700,fontSize:14,color:Math.abs(diff)<1000?"#22c55e":"#ef4444"}}>{diff>=0?"+":""}{fmtMoney(diff)}</span>
          </div>
        )}

        <div style={{marginBottom:20}}>{lbl("Notas")}<input style={fi} value={notas} onChange={e=>setNotas(e.target.value)} placeholder="Ej: Liquidación Febrero"/></div>

        {/* Reliquidación */}
        <div style={{border:"1px solid #2a3045",borderRadius:10,padding:16,marginBottom:20}}>
          <div style={{display:"flex",alignItems:"center",justifyContent:"space-between",marginBottom: tieneReliq ? 14 : 0}}>
            <div>
              <div style={{fontWeight:600,fontSize:13,color:"#e8ecf4"}}>⚖️ Reliquidación</div>
              <div style={{fontSize:11,color:"#4a5168",marginTop:2}}>Monto adicional por apelación</div>
            </div>
            <div style={{display:"flex",alignItems:"center",gap:8}}>
              <span style={{fontSize:12,color:"#7a8399"}}>{tieneReliq?"Sí":"No"}</span>
              <div onClick={()=>setTieneReliq(p=>!p)} style={{width:40,height:22,borderRadius:11,background:tieneReliq?"#f97316":"#2a3045",cursor:"pointer",position:"relative",transition:"background .2s"}}>
                <div style={{position:"absolute",top:3,left:tieneReliq?20:3,width:16,height:16,borderRadius:"50%",background:"#fff",transition:"left .2s"}}/>
              </div>
            </div>
          </div>
          {tieneReliq && (
            <div>
              {lbl("Monto reliquidación ($)")}
              <input style={{...fi,color:"#f97316",fontWeight:700,fontSize:15}} type="number" value={reliqMonto} onChange={e=>setReliqMonto(e.target.value)} placeholder="Ej: 1.250.000"/>
              {reliqMonto && (
                <div style={{marginTop:12,background:"rgba(249,115,22,.08)",border:"1px solid rgba(249,115,22,.25)",borderRadius:8,padding:"10px 14px",display:"flex",justifyContent:"space-between",alignItems:"center"}}>
                  <span style={{fontSize:12,color:"#7a8399"}}>Total Final (Factura + Reliquidación)</span>
                  <span style={{fontWeight:800,fontSize:16,color:"#f97316"}}>{fmtMoney(totalFinal)}</span>
                </div>
              )}
            </div>
          )}
        </div>

        <div style={{display:"flex",gap:10,justifyContent:"flex-end"}}>
          <button onClick={onClose} style={{background:"#181b22",border:"1px solid #2a3045",borderRadius:8,padding:"9px 20px",color:"#7a8399",cursor:"pointer",fontFamily:"inherit",fontSize:13}}>Cancelar</button>
          <button onClick={save} style={{background:"#22c55e",border:"none",borderRadius:8,padding:"9px 20px",color:"#fff",cursor:"pointer",fontWeight:600,fontSize:13}}>Guardar</button>
        </div>
      </div>
    </div>
  );
}

function Toast({ msg, type }) {
  if (!msg) return null;
  return <div style={{position:"fixed",bottom:24,right:24,background:"#181b22",border:`1px solid ${type==="error"?"rgba(239,68,68,.3)":"rgba(34,197,94,.3)"}`,borderRadius:8,padding:"12px 20px",fontSize:13,color:type==="error"?"#ef4444":"#22c55e",zIndex:300}}>{msg}</div>;
}

// ── CONFIRM MODAL ──
function ConfirmModal({ msg, onConfirm, onCancel }) {
  return (
    <div style={{position:"fixed",inset:0,background:"rgba(0,0,0,.75)",display:"flex",alignItems:"center",justifyContent:"center",zIndex:400,backdropFilter:"blur(4px)"}}>
      <div style={{background:"#111318",border:"1px solid #2a3045",borderRadius:14,padding:28,width:320,maxWidth:"90vw",textAlign:"center"}}>
        <div style={{fontSize:28,marginBottom:12}}>🗑️</div>
        <div style={{fontWeight:600,fontSize:15,color:"#e8ecf4",marginBottom:8}}>¿Eliminar?</div>
        <div style={{fontSize:13,color:"#7a8399",marginBottom:24}}>{msg||"Esta acción no se puede deshacer."}</div>
        <div style={{display:"flex",gap:10,justifyContent:"center"}}>
          <button onClick={onCancel} style={{background:"#181b22",border:"1px solid #2a3045",borderRadius:8,padding:"9px 24px",color:"#7a8399",cursor:"pointer",fontFamily:"inherit",fontSize:13}}>Cancelar</button>
          <button onClick={onConfirm} style={{background:"#ef4444",border:"none",borderRadius:8,padding:"9px 24px",color:"#fff",cursor:"pointer",fontWeight:600,fontSize:13}}>Eliminar</button>
        </div>
      </div>
    </div>
  );
}

// ── USER MODAL ──
function UserModal({ onClose, onSave, entry }) {
  const [name,  setName]  = useState(entry?.name  || "");
  const [email, setEmail] = useState(entry?.email || "");
  const [pass,  setPass]  = useState(entry?.pass  || "");
  const [role,  setRole]  = useState(entry?.role  || "editor");
  const [dept,  setDept]  = useState(entry?.dept  || "Movistar");
  const [showPass, setShowPass] = useState(false);

  const fi = { width:"100%",background:"#181b22",border:"1px solid #1e2330",borderRadius:8,padding:"9px 12px",color:"#e8ecf4",fontFamily:"inherit",fontSize:13,outline:"none" };
  const lbl = txt => <label style={{fontSize:11,color:"#7a8399",textTransform:"uppercase",letterSpacing:".08em",display:"block",marginBottom:6}}>{txt}</label>;

  const save = () => {
    if (!name.trim()||!email.trim()||!pass.trim()) { alert("Completa todos los campos"); return; }
    onSave({ name:name.trim(), email:email.trim(), pass:pass.trim(), role, dept: role==="admin" ? null : dept });
  };

  return (
    <div onClick={e=>e.target===e.currentTarget&&onClose()} style={{position:"fixed",inset:0,background:"rgba(0,0,0,.75)",display:"flex",alignItems:"center",justifyContent:"center",zIndex:200,backdropFilter:"blur(4px)"}}>
      <div style={{background:"#111318",border:"1px solid #2a3045",borderRadius:16,padding:32,width:420,maxWidth:"95vw"}}>
        <div style={{fontWeight:700,fontSize:18,marginBottom:24}}>{entry?"Editar Usuario":"Nuevo Usuario"}</div>
        <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:16}}>
          <div style={{gridColumn:"1/-1"}}>{lbl("Nombre completo")}<input style={fi} value={name} onChange={e=>setName(e.target.value)} placeholder="Ej: Juan Pérez"/></div>
          <div style={{gridColumn:"1/-1"}}>{lbl("Correo")}<input style={fi} type="email" value={email} onChange={e=>setEmail(e.target.value)} placeholder="correo@innotel.com"/></div>
          <div style={{gridColumn:"1/-1"}}>
            {lbl("Contraseña")}
            <div style={{position:"relative",overflow:"hidden",borderRadius:8}}>
              <input style={{...fi,paddingRight:42,boxSizing:"border-box"}} type={showPass?"text":"password"} value={pass} onChange={e=>setPass(e.target.value)} placeholder="Contraseña segura"/>
              <button onClick={()=>setShowPass(p=>!p)} style={{position:"absolute",right:10,top:"50%",transform:"translateY(-50%)",background:"none",border:"none",cursor:"pointer",color:"#7a8399",fontSize:14,lineHeight:1}}>{showPass?"🙈":"👁"}</button>
            </div>
          </div>
          <div>
            {lbl("Rol")}
            <select style={fi} value={role} onChange={e=>setRole(e.target.value)}>
              <option value="admin">Administrador</option>
              <option value="editor">Editor</option>
              <option value="reader">Lector</option>
            </select>
          </div>
          {role !== "admin" && (
            <div>
              {lbl("Departamento")}
              <select style={fi} value={dept} onChange={e=>setDept(e.target.value)}>
                {["Movistar","Entel","Wom"].map(d=><option key={d}>{d}</option>)}
              </select>
            </div>
          )}
        </div>
        <div style={{display:"flex",gap:10,justifyContent:"flex-end",marginTop:24}}>
          <button onClick={onClose} style={{background:"#181b22",border:"1px solid #2a3045",borderRadius:8,padding:"9px 20px",color:"#7a8399",cursor:"pointer",fontFamily:"inherit",fontSize:13}}>Cancelar</button>
          <button onClick={save} style={{background:"#4f7fff",border:"none",borderRadius:8,padding:"9px 20px",color:"#fff",cursor:"pointer",fontWeight:600,fontSize:13}}>Guardar</button>
        </div>
      </div>
    </div>
  );
}

export default function App() {
  const [currentUser, setCurrentUser] = useState(null);

  // Load SheetJS on mount
  useEffect(() => {
    if (!window.XLSX) {
      const s = document.createElement("script");
      s.src = "https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js";
      document.head.appendChild(s);
    }
  }, []);
  const [loginEmail, setLoginEmail]   = useState("");
  const [loginPass,  setLoginPass]    = useState("");
  const [loginErr,   setLoginErr]     = useState("");
  const [entries,    setEntries]      = useState([...DEMO]);
  const [billing,    setBilling]      = useState([...DEMO_BILLING]);
  const [view,       setView]         = useState("dashboard");
  const [activeDept, setActiveDept]   = useState("all");
  const [dashDept,   setDashDept]     = useState("all");
  const [chartMode,  setChartMode]    = useState("cat");
  const [dashMonth,  setDashMonth]    = useState(MONTHS[new Date().getMonth()]);
  const [search,     setSearch]       = useState("");
  const [filterCat,  setFilterCat]    = useState("");
  const [filterMonth,setFilterMonth]  = useState("");
  const [billMonth,  setBillMonth]    = useState(MONTHS[new Date().getMonth()]);
  const [calidadData,   setCalidadData]   = useState({}); // { "Noviembre": [...], "Diciembre": [...] }
  const [calidadCamada, setCalidadCamada] = useState("Noviembre");
  const [calidadSearch, setCalidadSearch] = useState("");
  const [calidadTab,    setCalidadTab]    = useState("resumen"); // resumen | vendedores | detalle
  const [selectedWeek, setSelectedWeek] = useState(0); // 0 = semana actual
  const [modal,      setModal]        = useState(null);
  const [billModal,  setBillModal]    = useState(null);
  const [toast,      setToast]        = useState(null);
  const [userModal,  setUserModal]    = useState(null); // null | "new" | user object
  const [confirmModal, setConfirmModal] = useState(null); // null | { msg, onConfirm }
  const [appUsers,   setAppUsers]     = useState([
    { id:"u1", email:"admin@innotel.com",    pass:"Admin2026!",  name:"Administrador", role:"admin",  dept:null },
    { id:"u2", email:"movistar@innotel.com", pass:"Mov2026!",    name:"Usr. Movistar", role:"editor", dept:"Movistar" },
    { id:"u3", email:"entel@innotel.com",    pass:"Ent2026!",    name:"Usr. Entel",    role:"editor", dept:"Entel" },
    { id:"u4", email:"wom@innotel.com",      pass:"Wom2026!",    name:"Usr. Wom",      role:"editor", dept:"Wom" },
  ]);

  // Reset week when month changes
  useEffect(() => { setSelectedWeek(0); }, [dashMonth]);

  const showToast = (msg, type="success") => { setToast({msg,type}); setTimeout(()=>setToast(null),2800); };
  const askConfirm = (msg, onConfirm) => setConfirmModal({ msg, onConfirm });
  const doLogin = () => {
    const u = appUsers.find(u=>u.email===loginEmail && u.pass===loginPass);
    if (!u) { setLoginErr("Credenciales incorrectas."); return; }
    setLoginErr(""); setCurrentUser({...u});
    setView(u.role==="admin" ? "dashboard" : "table");
    if (u.dept) { setActiveDept(u.dept); setDashDept(u.dept); }
  };
  const doLogout = () => { setCurrentUser(null); setView("dashboard"); };

  // Table data
  const tableData = useMemo(() => {
    let d = activeDept==="all" ? entries : entries.filter(e=>e.dept===activeDept);
    if (search)      d = d.filter(e=>e.desc.toLowerCase().includes(search.toLowerCase())||e.worker.toLowerCase().includes(search.toLowerCase())||e.cat.toLowerCase().includes(search.toLowerCase()));
    if (filterCat)   d = d.filter(e=>e.cat===filterCat);
    if (filterMonth) d = d.filter(e=>e.month===filterMonth);
    return d;
  }, [entries,activeDept,search,filterCat,filterMonth]);

  // Dashboard data
  const dashData = useMemo(() => {
    let d = [...entries];
    const dep = currentUser?.dept || dashDept;
    if (dep&&dep!=="all") d = d.filter(e=>e.dept===dep);
    if (dashMonth) d = d.filter(e=>e.month===dashMonth);
    return d;
  }, [entries,currentUser,dashDept,dashMonth]);

  // Build list of weeks for the selected month
  const weeksInMonth = useMemo(() => {
    const monthIdx = MONTHS.indexOf(dashMonth);
    if (monthIdx === -1) return [];
    const year = new Date().getFullYear();
    const fmt = dt => dt.toLocaleDateString("es-CL", {day:"2-digit", month:"2-digit"});
    const weeks = [];
    let weekNum = 1;
    // Start from day 1 of the month, advance 7 days at a time
    let start = new Date(year, monthIdx, 1);
    while (start.getMonth() === monthIdx) {
      // Monday of this chunk = start (we define weeks as blocks of 7 from day 1)
      const mon = new Date(start);
      const sun = new Date(start); sun.setDate(start.getDate() + 6);
      weeks.push({ label: `Semana ${weekNum} (${fmt(mon)} – ${fmt(sun)})`, mon, sun, num: weekNum });
      weekNum++;
      start.setDate(start.getDate() + 7);
    }
    return weeks;
  }, [dashMonth]);

  // Selected week range
  const currentWeekRange = useMemo(() => {
    if (!weeksInMonth.length) return null;
    const idx = Math.min(selectedWeek, weeksInMonth.length - 1);
    return weeksInMonth[idx];
  }, [weeksInMonth, selectedWeek]);

  // Weekly data filtered by selected week
  const weekData = useMemo(() => {
    if (!currentWeekRange) return [];
    let d = [...entries].filter(e => {
      const date = new Date(e.date);
      return date >= currentWeekRange.mon && date <= currentWeekRange.sun;
    });
    const dep = currentUser?.dept || dashDept;
    if (dep && dep !== "all") d = d.filter(e => e.dept === dep);
    return d;
  }, [entries, currentUser, dashDept, currentWeekRange]);

  const weekComisiones = weekData.filter(e=>e.isComision);
  const weekTotalPagado   = weekComisiones.reduce((s,e)=>s+e.amount,0);
  const weekTotalNeto     = weekComisiones.reduce((s,e)=>s+(e.netoComision||e.amount),0);
  const weekIncorrectas   = weekComisiones.reduce((s,e)=>s+(e.qtIncorrectas||0),0);
  const weekPerdida       = weekTotalPagado - weekTotalNeto;

  const dashGroups = useMemo(() => {
    const groups={}, colorMap={};
    if (chartMode==="cat") {
      dashData.forEach(e=>{groups[e.cat]=(groups[e.cat]||0)+e.amount;});
      Object.keys(groups).forEach((k,i)=>{colorMap[k]=CCOLORS[i%CCOLORS.length];});
    } else {
      dashData.forEach(e=>{groups[e.dept]=(groups[e.dept]||0)+e.amount;});
      Object.keys(groups).forEach(k=>{colorMap[k]=DCOLORS[k]||"#888";});
    }
    return Object.entries(groups).sort((a,b)=>b[1]-a[1]).map(([label,value])=>({label,value,color:colorMap[label]}));
  }, [dashData,chartMode]);

  const dashTotal = dashGroups.reduce((s,i)=>s+i.value,0);
  const maxEntry  = dashData.reduce((m,e)=>!m||e.amount>m.amount?e:m,null);
  const minEntry  = dashData.reduce((m,e)=>!m||e.amount<m.amount?e:m,null);

  const saveEntry = data => {
    if (modal==="new") { setEntries(p=>[...p,{id:"e"+Date.now(),...data,by:currentUser.email.split("@")[0]}]); showToast("Gasto registrado ✓"); }
    else               { setEntries(p=>p.map(e=>e.id===modal.id?{...e,...data}:e)); showToast("Gasto actualizado ✓"); }
    setModal(null);
  };
  const deleteEntry   = id => askConfirm("¿Eliminar este gasto?",   () => { setEntries(p=>p.filter(e=>e.id!==id)); showToast("Eliminado"); setConfirmModal(null); });
  const saveBilling = data => {
    if (billModal==="new") { setBilling(p=>[...p,{id:"b"+Date.now(),...data}]); showToast("Facturación registrada ✓"); }
    else                   { setBilling(p=>p.map(b=>b.id===billModal.id?{...b,...data}:b)); showToast("Facturación actualizada ✓"); }
    setBillModal(null);
  };
  const deleteBilling = id => askConfirm("¿Eliminar este registro de facturación?", () => { setBilling(p=>p.filter(b=>b.id!==id)); showToast("Eliminado"); setConfirmModal(null); });

  const depts = currentUser?.role==="admin"||!currentUser?.dept ? ["all","Movistar","Entel","Wom"] : [currentUser.dept];

  // ── EXPORT EXCEL ──
  const exportExcel = async () => {
    const XLSX = window.XLSX;
    if (!XLSX) { showToast("Cargando Excel... intenta de nuevo en 2 segundos","error"); return; }
    const wb = XLSX.utils.book_new();
    const month = new Date().toLocaleDateString("es-CL",{month:"long",year:"numeric"});

    // Helper: auto-width
    const autoWidth = ws => {
      const data = XLSX.utils.sheet_to_json(ws, {header:1});
      const cols = data.reduce((acc,row) => {
        row.forEach((cell,i) => { acc[i] = Math.max(acc[i]||8, String(cell||"").length+2); });
        return acc;
      }, []);
      ws["!cols"] = cols.map(w => ({wch: Math.min(w, 40)}));
    };

    // ── Hoja: Resumen ──
    const resumenRows = [
      ["GastosPro — Resumen Ejecutivo", "", "", "", "", ""],
      [`Exportado: ${new Date().toLocaleDateString("es-CL")}`, "", "", "", "", ""],
      [],
      ["Departamento","Total Gastos","Com. Pagadas","Neto Real","Ventas Incorrectas","Pérdida Incorrectas","Total Facturado","Margen"],
    ];
    DEPTS.forEach(dept => {
      const de = entries.filter(e=>e.dept===dept);
      const totalGastos   = de.reduce((s,e)=>s+e.amount,0);
      const comPagadas    = de.filter(e=>e.isComision).reduce((s,e)=>s+e.amount,0);
      const netoReal      = de.filter(e=>e.isComision).reduce((s,e)=>s+(e.netoComision||e.amount),0);
      const incorrectas   = de.filter(e=>e.isComision).reduce((s,e)=>s+(e.qtIncorrectas||0),0);
      const perdida       = comPagadas - netoReal;
      const facturado     = billing.filter(b=>b.dept===dept).reduce((s,b)=>s+b.total,0);
      const margen        = facturado - totalGastos;
      resumenRows.push([dept, totalGastos, comPagadas, netoReal, incorrectas, perdida, facturado, margen]);
    });
    const wsResumen = XLSX.utils.aoa_to_sheet(resumenRows);
    autoWidth(wsResumen);
    XLSX.utils.book_append_sheet(wb, wsResumen, "Resumen");

    // ── Hojas por departamento ──
    DEPTS.forEach(dept => {
      const de = entries.filter(e=>e.dept===dept);
      const rows = [
        [`Gastos — ${dept}`, "", "", "", "", "", "", "", "", "", ""],
        [],
        ["Fecha","Categoría","Descripción","Trabajador","Mes","Total Ventas","✅ Correctas","❌ Incorrectas","Valor Unitario","Total Pagado","Neto Real","Notas"],
      ];
      de.forEach(e => {
        rows.push([
          e.date, e.cat, e.desc, e.worker, e.month,
          e.isComision ? e.qtTotal      : "—",
          e.isComision ? e.qtCorrectas  : "—",
          e.isComision ? e.qtIncorrectas: "—",
          e.isComision ? e.valorUnitario: "—",
          e.amount,
          e.isComision ? (e.netoComision||e.amount) : e.amount,
          e.notes||"",
        ]);
      });
      rows.push(["TOTAL","","","","","","","","",
        de.reduce((s,e)=>s+e.amount,0),
        de.reduce((s,e)=>s+(e.isComision?(e.netoComision||e.amount):e.amount),0),
        ""
      ]);
      const ws = XLSX.utils.aoa_to_sheet(rows);
      autoWidth(ws);
      XLSX.utils.book_append_sheet(wb, ws, dept);
    });

    // ── Hoja: Facturación ──
    const billRows = [
      ["Facturación — Ingresos vs Gastos","","","","","","","","","","","",""],
      [],
      ["Departamento","Mes","BAF Qty","BAF Val","TV Qty","TV Val","VOZ Qty","VOZ Val","Total Calculado","Total Factura","Diferencia","Total Gastos","Margen"],
    ];
    billing.forEach(b => {
      const gastos   = entries.filter(e=>e.dept===b.dept&&e.month===b.month).reduce((s,e)=>s+e.amount,0);
      const totalFact = b.totalFactura||b.total||0;
      const diff      = totalFact - (b.totalCalculado||0);
      const margen    = totalFact - gastos;
      billRows.push([b.dept, b.month, b.baf?.qty||0, b.baf?.val||0, b.tv?.qty||0, b.tv?.val||0, b.voz?.qty||0, b.voz?.val||0, b.totalCalculado||0, totalFact, diff, gastos, margen]);
    });
    const wsBill = XLSX.utils.aoa_to_sheet(billRows);
    autoWidth(wsBill);
    XLSX.utils.book_append_sheet(wb, wsBill, "Facturación");

    XLSX.writeFile(wb, `GastosPro_${new Date().toISOString().split("T")[0]}.xlsx`);
    showToast("Excel exportado ✓");
  };

  // Billing tab data
  const filteredBilling = useMemo(()=>billing.filter(b=>b.month===billMonth),[billing,billMonth]);

  // ── LOGIN ──
  if (!currentUser) return (
    <div style={{background:"#0a0b0e",minHeight:"100vh",display:"flex",alignItems:"center",justifyContent:"center",color:"#e8ecf4",fontFamily:"system-ui,sans-serif"}}>
      <div style={{background:"#111318",border:"1px solid #1e2330",borderRadius:16,padding:48,width:380}}>
        <div style={{fontWeight:800,fontSize:24,marginBottom:8,background:"linear-gradient(135deg,#4f7fff,#7c5cfc)",WebkitBackgroundClip:"text",WebkitTextFillColor:"transparent"}}>Gestión de Gastos</div>
        <div style={{fontWeight:600,fontSize:14,color:"#7a8399",marginBottom:28}}>Innotel — Panel de Control</div>
        {[["Correo","email","text",loginEmail,setLoginEmail],["Contraseña","pass","password",loginPass,setLoginPass]].map(([l,k,t,v,s])=>(
          <div key={k}>
            <label style={{fontSize:11,color:"#7a8399",textTransform:"uppercase",letterSpacing:".08em",display:"block",marginBottom:6}}>{l}</label>
            <input type={t} value={v} onChange={e=>s(e.target.value)} onKeyDown={e=>e.key==="Enter"&&doLogin()} style={{width:"100%",background:"#181b22",border:"1px solid #1e2330",borderRadius:8,padding:"10px 14px",color:"#e8ecf4",fontFamily:"inherit",fontSize:14,outline:"none",marginBottom:16}}/>
          </div>
        ))}
        <button onClick={doLogin} style={{width:"100%",background:"#4f7fff",border:"none",borderRadius:8,padding:11,color:"#fff",fontWeight:600,fontSize:14,cursor:"pointer"}}>Ingresar al Sistema</button>
        {loginErr && <div style={{color:"#ef4444",fontSize:12,marginTop:10,textAlign:"center"}}>{loginErr}</div>}
      </div>
    </div>
  );

  const initials = currentUser.name.split(" ").map(w=>w[0]).join("").substring(0,2);
  const S = { surface:"#111318", surface2:"#181b22", border:"1px solid #1e2330", border2:"1px solid #2a3045" };

  return (
    <div style={{background:"#0a0b0e",minHeight:"100vh",color:"#e8ecf4",fontFamily:"system-ui,sans-serif"}}>

      {/* TOPBAR */}
      <div style={{height:56,background:S.surface,borderBottom:S.border,display:"flex",alignItems:"center",padding:"0 20px",gap:10,position:"sticky",top:0,zIndex:100}}>
        <div style={{fontWeight:800,fontSize:16,background:"linear-gradient(135deg,#4f7fff,#7c5cfc)",WebkitBackgroundClip:"text",WebkitTextFillColor:"transparent",marginRight:4,whiteSpace:"nowrap"}}>Gestión de Gastos</div>
        <div style={{display:"flex",gap:2,background:S.surface2,border:S.border,borderRadius:8,padding:3}}>
          {[["dashboard","📊 Dashboard"],["table","☰ Tabla"],["billing","💰 Facturación"],["calidad","⭐ Calidad"]]
            .filter(([v])=> currentUser.role==="admin" || v==="table")
            .map(([v,label])=>(
            <button key={v} onClick={()=>setView(v)} style={{padding:"5px 14px",borderRadius:5,fontSize:12,fontWeight:500,cursor:"pointer",border:"none",color:view===v?"#e8ecf4":"#7a8399",background:view===v?S.surface:"transparent",fontFamily:"inherit",boxShadow:view===v?"0 1px 3px rgba(0,0,0,.3)":"none",whiteSpace:"nowrap"}}>
              {label}
            </button>
          ))}
        </div>
        <div style={{display:"flex",gap:4,flex:1,overflowX:"auto"}}>
          {depts.map(d=>{
            const isActive=d===activeDept, color=deptColor(d);
            return <button key={d} onClick={()=>setActiveDept(d)} style={{padding:"6px 14px",borderRadius:6,fontSize:13,fontWeight:500,cursor:"pointer",border:`1px solid ${isActive?(d==="all"?"rgba(79,127,255,.3)":`rgba(${deptRgb(d)},.3)`):"transparent"}`,color:isActive?(d==="all"?"#4f7fff":color):"#7a8399",background:isActive?(d==="all"?"rgba(79,127,255,.12)":`rgba(${deptRgb(d)},.12)`):"transparent",fontFamily:"inherit",whiteSpace:"nowrap"}}>{d==="all"?"Todos":d}</button>;
          })}
        </div>
        <div style={{display:"flex",alignItems:"center",gap:10,marginLeft:"auto"}}>
          <div style={{display:"flex",alignItems:"center",gap:8,background:S.surface2,border:S.border,borderRadius:20,padding:"4px 12px 4px 4px"}}>
            <div style={{width:26,height:26,borderRadius:"50%",background:"linear-gradient(135deg,#4f7fff,#7c5cfc)",display:"flex",alignItems:"center",justifyContent:"center",fontSize:11,fontWeight:700,color:"#fff"}}>{initials}</div>
            <div><div style={{fontSize:12,fontWeight:500}}>{currentUser.name}</div><div style={{fontSize:10,color:"#7a8399"}}>{currentUser.role==="admin"?"Administrador":currentUser.role==="editor"?"Editor":"Lector"}</div></div>
          </div>
          {currentUser.role==="admin" && <button onClick={()=>setView("users")} title="Gestión de usuarios" style={{background:view==="users"?"rgba(79,127,255,.15)":"none",border:`1px solid ${view==="users"?"rgba(79,127,255,.4)":"#1e2330"}`,borderRadius:6,height:32,width:32,display:"flex",alignItems:"center",justifyContent:"center",cursor:"pointer",color:view==="users"?"#4f7fff":"#7a8399",fontSize:16}}>👤</button>}
          <button onClick={exportExcel} title="Exportar a Excel" style={{background:"#166534",border:"1px solid #22c55e",borderRadius:6,height:32,padding:"0 12px",display:"flex",alignItems:"center",gap:5,cursor:"pointer",color:"#22c55e",fontSize:12,fontWeight:600}}>↓ Excel</button>
          <button onClick={doLogout} style={{background:"none",border:S.border,borderRadius:6,width:32,height:32,display:"flex",alignItems:"center",justifyContent:"center",cursor:"pointer",color:"#7a8399",fontSize:15}}>⎋</button>
        </div>
      </div>

      {/* ── DASHBOARD ── */}
      {view==="dashboard" && (
        <div style={{padding:24}}>
          <div style={{display:"flex",alignItems:"center",gap:12,flexWrap:"wrap",marginBottom:20}}>
            <div>
              <span style={{fontWeight:700,fontSize:20}}>{(currentUser.dept||dashDept)==="all"?"Dashboard Global":"Dashboard "+(currentUser.dept||dashDept)}</span>
              <span style={{fontSize:13,color:"#7a8399",marginLeft:8}}>{dashMonth?dashMonth+" "+new Date().getFullYear():"Vista general"}</span>
            </div>
            {currentUser.role==="admin" && (
              <div style={{display:"flex",gap:6,flexWrap:"wrap"}}>
                {["all","Movistar","Entel","Wom"].map(d=>{
                  const active=d===dashDept,color=deptColor(d);
                  return <button key={d} onClick={()=>setDashDept(d)} style={{padding:"5px 14px",borderRadius:20,fontSize:12,fontWeight:500,cursor:"pointer",border:`1px solid ${active?(d==="all"?"rgba(79,127,255,.3)":`rgba(${deptRgb(d)},.3)`):"#2a3045"}`,color:active?(d==="all"?"#4f7fff":color):"#7a8399",background:active?(d==="all"?"rgba(79,127,255,.12)":`rgba(${deptRgb(d)},.12)`):"transparent",fontFamily:"inherit"}}>{d==="all"?"Global":d}</button>;
                })}
              </div>
            )}
            <div style={{marginLeft:"auto",display:"flex",alignItems:"center",gap:8}}>
              <label style={{fontSize:12,color:"#7a8399"}}>Mes:</label>
              <select value={dashMonth} onChange={e=>setDashMonth(e.target.value)} style={{background:S.surface,border:S.border,borderRadius:8,padding:"7px 12px",color:"#e8ecf4",fontFamily:"inherit",fontSize:13,outline:"none",cursor:"pointer"}}>
                <option value="">Todos</option>{MONTHS.map(m=><option key={m}>{m}</option>)}
              </select>
            </div>
          </div>

          {/* Metric cards */}
          <div style={{display:"grid",gridTemplateColumns:"repeat(auto-fit,minmax(200px,1fr))",gap:12,marginBottom:20}}>
            {[
              {bg:"linear-gradient(135deg,#4f7fff,#3a5fd4)",label:"Total Gastos",val:fmtMoney(dashTotal),sub:dashMonth||"Todos los meses"},
              {bg:"linear-gradient(135deg,#7c5cfc,#5a3dd4)",label:"Valor Máximo",val:maxEntry?fmtMoney(maxEntry.amount):"—",sub:maxEntry?.cat||""},
              {bg:"linear-gradient(135deg,#22c55e,#16a34a)",label:"Valor Mínimo",val:minEntry?fmtMoney(minEntry.amount):"—",sub:minEntry?.cat||""},
              {bg:"linear-gradient(135deg,#f97316,#ea6000)",label:"Registros",val:dashData.length,sub:"entradas"},
            ].map((m,i)=>(
              <div key={i} style={{background:m.bg,borderRadius:14,padding:"22px 24px",position:"relative",overflow:"hidden"}}>
                <div style={{fontSize:11,fontWeight:600,textTransform:"uppercase",letterSpacing:".1em",color:"rgba(255,255,255,.7)",marginBottom:10}}>{m.label}</div>
                <div style={{fontWeight:800,fontSize:26,color:"#fff",letterSpacing:-1,lineHeight:1,overflow:"hidden",textOverflow:"ellipsis",whiteSpace:"nowrap"}}>{m.val}</div>
                <div style={{fontSize:11,color:"rgba(255,255,255,.6)",marginTop:6}}>{m.sub}</div>
                <div style={{position:"absolute",top:16,right:16,width:28,height:28,background:"rgba(255,255,255,.15)",borderRadius:6,display:"flex",alignItems:"center",justifyContent:"center",fontSize:14}}>↗</div>
              </div>
            ))}
          </div>

          {/* Weekly summary */}
          <div style={{background:S.surface,border:S.border,borderRadius:14,padding:24,marginBottom:20}}>
            <div style={{display:"flex",alignItems:"center",justifyContent:"space-between",flexWrap:"wrap",gap:12,marginBottom:16}}>
              <div>
                <div style={{fontWeight:700,fontSize:15}}>📅 Resumen Semanal — Comisiones</div>
                <div style={{fontSize:12,color:"#7a8399",marginTop:2}}>{currentWeekRange?.label || "Selecciona un mes"}</div>
              </div>
              <div style={{display:"flex",alignItems:"center",gap:8}}>
                <label style={{fontSize:12,color:"#7a8399"}}>Semana:</label>
                <div style={{display:"flex",gap:4}}>
                  {weeksInMonth.map((w,i) => (
                    <button key={i} onClick={()=>setSelectedWeek(i)} style={{padding:"5px 12px",borderRadius:20,fontSize:12,fontWeight:500,cursor:"pointer",border:`1px solid ${selectedWeek===i?"rgba(79,127,255,.4)":"#2a3045"}`,color:selectedWeek===i?"#4f7fff":"#7a8399",background:selectedWeek===i?"rgba(79,127,255,.12)":"transparent",fontFamily:"inherit",whiteSpace:"nowrap"}}>
                      S{w.num}
                    </button>
                  ))}
                </div>
                <span style={{fontSize:11,color:"#7a8399",background:S.surface2,border:S.border2,borderRadius:20,padding:"4px 12px"}}>{weekComisiones.length} registros</span>
              </div>
            </div>
            <div style={{display:"grid",gridTemplateColumns:"repeat(auto-fit,minmax(160px,1fr))",gap:12,marginBottom:weekComisiones.length>0?20:0}}>
              {[
                {label:"Total pagado a vendedores",val:fmtMoney(weekTotalPagado),color:"#4f7fff"},
                {label:"Ventas incorrectas",val:weekIncorrectas+" ventas",color:"#ef4444"},
                {label:"Pérdida por incorrectas",val:"-"+fmtMoney(weekPerdida),color:"#ef4444"},
                {label:"Neto real semana",val:fmtMoney(weekTotalNeto),color:"#22c55e"},
              ].map((s,i)=>(
                <div key={i} style={{background:S.surface2,border:S.border2,borderRadius:10,padding:"14px 16px"}}>
                  <div style={{fontSize:10,color:"#7a8399",textTransform:"uppercase",letterSpacing:".08em",marginBottom:6}}>{s.label}</div>
                  <div style={{fontWeight:700,fontSize:18,color:s.color}}>{s.val}</div>
                </div>
              ))}
            </div>
            {weekComisiones.length > 0 ? (
              <table style={{width:"100%",borderCollapse:"collapse"}}>
                <thead><tr style={{borderBottom:S.border}}>
                  {["Fecha","Dept","Vendedor","Total ventas","✅ Correctas","❌ Incorrectas","Total pagado","Neto real"].map(h=>(
                    <th key={h} style={{padding:"8px 10px",fontSize:10,textTransform:"uppercase",letterSpacing:".06em",color:"#7a8399",fontWeight:600,textAlign:"left"}}>{h}</th>
                  ))}
                </tr></thead>
                <tbody>
                  {weekComisiones.map(e=>(
                    <tr key={e.id} style={{borderBottom:S.border}}>
                      <td style={{padding:"10px",fontSize:12,color:"#7a8399",fontFamily:"monospace"}}>{e.date}</td>
                      <td style={{padding:"10px"}}><span style={{background:`rgba(${deptRgb(e.dept)},.12)`,color:deptColor(e.dept),padding:"2px 8px",borderRadius:20,fontSize:11,fontWeight:600}}>{e.dept}</span></td>
                      <td style={{padding:"10px",fontSize:13}}>{e.worker}</td>
                      <td style={{padding:"10px",fontFamily:"monospace",fontWeight:600}}>{e.qtTotal}</td>
                      <td style={{padding:"10px",color:"#22c55e",fontFamily:"monospace"}}>{e.qtCorrectas}</td>
                      <td style={{padding:"10px",color:"#ef4444",fontFamily:"monospace"}}>{e.qtIncorrectas > 0 ? `-${e.qtIncorrectas}` : "0"}</td>
                      <td style={{padding:"10px",fontFamily:"monospace",fontWeight:600}}>{fmtMoney(e.amount)}</td>
                      <td style={{padding:"10px",fontFamily:"monospace",fontWeight:600,color:"#22c55e"}}>{fmtMoney(e.netoComision||e.amount)}</td>
                    </tr>
                  ))}
                </tbody>
              </table>
            ) : (
              <div style={{textAlign:"center",color:"#4a5168",fontSize:13,padding:"20px 0"}}>No hay registros de comisiones esta semana</div>
            )}
          </div>

          {/* Charts */}
          <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:16}}>
            <div style={{background:S.surface,border:S.border,borderRadius:14,padding:24}}>
              <div style={{display:"flex",alignItems:"flex-start",justifyContent:"space-between",marginBottom:4}}>
                <div><div style={{fontWeight:700,fontSize:15}}>Proporción por Valor</div><div style={{fontSize:12,color:"#7a8399",marginTop:2,marginBottom:18}}>{chartMode==="cat"?"Por categoría":"Por departamento"}</div></div>
                <div style={{display:"flex",gap:4,background:S.surface2,border:S.border,borderRadius:6,padding:2}}>
                  {[["cat","Categoría"],["dept","Departamento"]].map(([m,l])=>(
                    <button key={m} onClick={()=>setChartMode(m)} style={{padding:"4px 10px",borderRadius:4,fontSize:11,fontWeight:500,cursor:"pointer",border:"none",background:chartMode===m?S.surface:"transparent",color:chartMode===m?"#e8ecf4":"#7a8399",fontFamily:"inherit"}}>{l}</button>
                  ))}
                </div>
              </div>
              <div style={{minHeight:220,display:"flex",alignItems:"center"}}><DonutChart data={dashGroups} total={dashTotal}/></div>
            </div>
            <div style={{background:S.surface,border:S.border,borderRadius:14,padding:24}}>
              <div style={{fontWeight:700,fontSize:15,marginBottom:4}}>{chartMode==="cat"?"Monitoreo de Montos":"Gastos por Departamento"}</div>
              <div style={{fontSize:12,color:"#7a8399",marginBottom:18}}>{chartMode==="cat"?"Detalle por categoría":"Detalle por departamento"}</div>
              <table style={{width:"100%",borderCollapse:"collapse"}}>
                <thead><tr>{["Nombre","Valor","%"].map((h,i)=><th key={h} style={{padding:"7px 0",fontSize:10,textTransform:"uppercase",letterSpacing:".08em",color:"#7a8399",fontWeight:600,textAlign:i>0?"right":"left",borderBottom:S.border}}>{h}</th>)}</tr></thead>
                <tbody>
                  {dashGroups.map((item,i)=>{
                    const pct=dashTotal>0?(item.value/dashTotal*100).toFixed(1):0;
                    return <tr key={i}><td style={{padding:"9px 0",borderBottom:S.border}}><div style={{fontWeight:500}}>{item.label}</div><div style={{height:4,borderRadius:2,background:item.color,width:`${pct}%`,marginTop:5}}/></td><td style={{padding:"9px 0",borderBottom:S.border,textAlign:"right",fontFamily:"monospace",fontWeight:500}}>{fmtMoney(item.value)}</td><td style={{padding:"9px 0",borderBottom:S.border,textAlign:"right",color:"#7a8399",fontFamily:"monospace"}}>{pct}%</td></tr>;
                  })}
                  {!dashGroups.length&&<tr><td colSpan={3} style={{padding:"40px 0",textAlign:"center",color:"#4a5168",fontSize:13}}>Sin datos</td></tr>}
                </tbody>
              </table>
            </div>
          </div>
        </div>
      )}

      {/* ── TABLE ── */}
      {view==="table" && (
        <div style={{padding:24,display:"flex",flexDirection:"column",gap:20}}>
          <div style={{display:"grid",gridTemplateColumns:"repeat(auto-fit,minmax(170px,1fr))",gap:12}}>
            {[
              {color:"#4f7fff",label:"Total Gastos",val:fmtMoney(entries.reduce((s,e)=>s+e.amount,0)),sub:`${entries.length} registros`},
              {color:"#00b5e2",label:"Movistar",val:fmtMoney(entries.filter(e=>e.dept==="Movistar").reduce((s,e)=>s+e.amount,0)),sub:`${entries.filter(e=>e.dept==="Movistar").length} reg.`},
              {color:"#ff6b35",label:"Entel",val:fmtMoney(entries.filter(e=>e.dept==="Entel").reduce((s,e)=>s+e.amount,0)),sub:`${entries.filter(e=>e.dept==="Entel").length} reg.`},
              {color:"#a78bfa",label:"Wom",val:fmtMoney(entries.filter(e=>e.dept==="Wom").reduce((s,e)=>s+e.amount,0)),sub:`${entries.filter(e=>e.dept==="Wom").length} reg.`},
            ].map((s,i)=>(
              <div key={i} style={{background:S.surface,border:S.border,borderRadius:12,padding:"18px 20px",position:"relative",overflow:"hidden"}}>
                <div style={{position:"absolute",top:0,left:0,right:0,height:2,background:s.color}}/>
                <div style={{fontSize:10,textTransform:"uppercase",letterSpacing:".1em",color:"#7a8399",marginBottom:8,fontWeight:500}}>{s.label}</div>
                <div style={{fontWeight:700,fontSize:22,letterSpacing:-.5}}>{s.val}</div>
                <div style={{fontSize:11,color:"#4a5168",marginTop:4}}>{s.sub}</div>
              </div>
            ))}
          </div>

          <div style={{display:"flex",alignItems:"center",gap:10,flexWrap:"wrap"}}>
            <div style={{display:"flex",alignItems:"center",gap:8,background:S.surface,border:S.border,borderRadius:8,padding:"7px 12px",flex:1,minWidth:180,maxWidth:300}}>
              <span style={{color:"#4a5168",fontSize:15}}>⌕</span>
              <input value={search} onChange={e=>setSearch(e.target.value)} placeholder="Buscar..." style={{background:"none",border:"none",outline:"none",color:"#e8ecf4",fontFamily:"inherit",fontSize:13,width:"100%"}}/>
            </div>
            <select value={filterCat} onChange={e=>setFilterCat(e.target.value)} style={{background:S.surface,border:S.border,borderRadius:8,padding:"7px 12px",color:"#e8ecf4",fontFamily:"inherit",fontSize:13,outline:"none",cursor:"pointer"}}>
              <option value="">Todas las categorías</option>{CATS.map(c=><option key={c}>{c}</option>)}
            </select>
            <select value={filterMonth} onChange={e=>setFilterMonth(e.target.value)} style={{background:S.surface,border:S.border,borderRadius:8,padding:"7px 12px",color:"#e8ecf4",fontFamily:"inherit",fontSize:13,outline:"none",cursor:"pointer"}}>
              <option value="">Todos los meses</option>{MONTHS.map(m=><option key={m}>{m}</option>)}
            </select>
            <button onClick={()=>setModal("new")} disabled={currentUser.role==="reader"} style={{background:"#4f7fff",border:"none",borderRadius:8,padding:"8px 16px",color:"#fff",fontWeight:600,fontSize:13,cursor:"pointer",opacity:currentUser.role==="reader"?.4:1,whiteSpace:"nowrap"}}>+ Agregar Gasto</button>
            <button onClick={()=>{const h=["Fecha","Dept","Cat","Desc","Trabajador","Mes","Monto","Neto","Por"];const rows=tableData.map(e=>[e.date,e.dept,e.cat,`"${e.desc}"`,e.worker,e.month,e.amount,e.netoComision||e.amount,e.by]);const csv=[h,...rows].map(r=>r.join(",")).join("\n");const a=document.createElement("a");a.href=URL.createObjectURL(new Blob(["\uFEFF"+csv],{type:"text/csv;charset=utf-8;"}));a.download="gastos.csv";a.click();showToast("CSV exportado ✓");}} style={{background:S.surface2,border:S.border2,borderRadius:8,padding:"8px 16px",color:"#7a8399",fontFamily:"inherit",fontSize:13,cursor:"pointer",whiteSpace:"nowrap"}}>↓ CSV</button>
            <button onClick={exportExcel} style={{background:"#166534",border:"1px solid #22c55e",borderRadius:8,padding:"8px 16px",color:"#22c55e",fontFamily:"inherit",fontSize:13,fontWeight:600,cursor:"pointer",whiteSpace:"nowrap"}}>↓ Excel</button>
          </div>

          <div style={{background:S.surface,border:S.border,borderRadius:12,overflow:"hidden"}}>
            <table style={{width:"100%",borderCollapse:"collapse"}}>
              <thead><tr style={{background:S.surface2,borderBottom:S.border}}>
                {["Fecha","Dept","Categoría","Descripción / Detalle","Trabajador","Mes","Total pagado","Neto real","Por",""].map(h=><th key={h} style={{padding:"10px 12px",textAlign:"left",fontSize:10,fontWeight:600,textTransform:"uppercase",letterSpacing:".08em",color:"#7a8399",whiteSpace:"nowrap"}}>{h}</th>)}
              </tr></thead>
              <tbody>
                {!tableData.length&&<tr><td colSpan={10} style={{padding:"60px",textAlign:"center",color:"#4a5168"}}><div style={{fontSize:36,marginBottom:12}}>📋</div>Sin registros</td></tr>}
                {tableData.map(e=>(
                  <tr key={e.id} style={{borderBottom:S.border}} onMouseEnter={ev=>ev.currentTarget.style.background=S.surface2} onMouseLeave={ev=>ev.currentTarget.style.background="transparent"}>
                    <td style={{padding:"12px",fontSize:12,color:"#7a8399",fontFamily:"monospace"}}>{e.date}</td>
                    <td style={{padding:"12px"}}><span style={{background:`rgba(${deptRgb(e.dept)},.12)`,color:deptColor(e.dept),padding:"3px 10px",borderRadius:20,fontSize:11,fontWeight:600}}>{e.dept}</span></td>
                    <td style={{padding:"12px"}}><span style={{background:S.surface2,border:S.border2,padding:"2px 8px",borderRadius:4,fontSize:11,color:"#7a8399",fontFamily:"monospace"}}>{e.cat}</span></td>
                    <td style={{padding:"12px"}}>
                      <div>{e.desc}</div>
                      {e.isComision && (
                        <div style={{display:"flex",gap:8,marginTop:5,flexWrap:"wrap"}}>
                          <span style={{fontSize:11,color:"#7a8399"}}>Total: <strong style={{color:"#e8ecf4"}}>{e.qtTotal}</strong></span>
                          <span style={{fontSize:11,color:"#22c55e"}}>✅ {e.qtCorrectas}</span>
                          {e.qtIncorrectas>0 && <span style={{fontSize:11,color:"#ef4444"}}>❌ -{e.qtIncorrectas}</span>}
                          <span style={{fontSize:11,color:"#7a8399"}}>× {fmtMoney(e.valorUnitario)}</span>
                        </div>
                      )}
                      {e.notes&&<div style={{fontSize:11,color:"#4a5168",marginTop:2}}>{e.notes}</div>}
                    </td>
                    <td style={{padding:"12px",color:"#7a8399"}}>{e.worker}</td>
                    <td style={{padding:"12px",color:"#7a8399"}}>{e.month}</td>
                    <td style={{padding:"12px",fontFamily:"monospace",fontWeight:600}}>{fmtMoney(e.amount)}</td>
                    <td style={{padding:"12px",fontFamily:"monospace",fontWeight:600,color:e.isComision&&e.netoComision<e.amount?"#22c55e":"#e8ecf4"}}>
                      {e.isComision ? (
                        <div>
                          {fmtMoney(e.netoComision)}
                          {e.qtIncorrectas>0&&<div style={{fontSize:11,color:"#ef4444",marginTop:2}}>-{fmtMoney(e.qtIncorrectas*e.valorUnitario)}</div>}
                        </div>
                      ) : fmtMoney(e.amount)}
                    </td>
                    <td style={{padding:"12px",fontSize:11,color:"#4a5168"}}>{e.by}</td>
                    <td style={{padding:"12px"}}>
                      {currentUser.role!=="reader"&&(
                        <div style={{display:"flex",gap:4}}>
                          <button onClick={()=>setModal(e)} style={{background:S.surface2,border:S.border2,borderRadius:5,width:26,height:26,display:"flex",alignItems:"center",justifyContent:"center",cursor:"pointer",color:"#7a8399",fontSize:12}}>✎</button>
                          <button onClick={()=>deleteEntry(e.id)} style={{background:S.surface2,border:S.border2,borderRadius:5,width:26,height:26,display:"flex",alignItems:"center",justifyContent:"center",cursor:"pointer",color:"#7a8399",fontSize:12}}>✕</button>
                        </div>
                      )}
                    </td>
                  </tr>
                ))}
              </tbody>
            </table>
          </div>
        </div>
      )}

      {/* ── FACTURACIÓN ── */}
      {view==="billing" && (
        <div style={{padding:24}}>
          <div style={{display:"flex",alignItems:"center",gap:12,flexWrap:"wrap",marginBottom:20}}>
            <div><div style={{fontWeight:700,fontSize:20}}>💰 Facturación</div><div style={{fontSize:13,color:"#7a8399",marginTop:2}}>Ingresos vs Gastos por departamento</div></div>
            <div style={{marginLeft:"auto",display:"flex",alignItems:"center",gap:8}}>
              <label style={{fontSize:12,color:"#7a8399"}}>Mes:</label>
              <select value={billMonth} onChange={e=>setBillMonth(e.target.value)} style={{background:S.surface,border:S.border,borderRadius:8,padding:"7px 12px",color:"#e8ecf4",fontFamily:"inherit",fontSize:13,outline:"none",cursor:"pointer"}}>
                {MONTHS.map(m=><option key={m}>{m}</option>)}
              </select>
              {currentUser.role==="admin"&&<button onClick={()=>setBillModal("new")} style={{background:"#22c55e",border:"none",borderRadius:8,padding:"8px 16px",color:"#fff",fontWeight:600,fontSize:13,cursor:"pointer",whiteSpace:"nowrap"}}>+ Registrar Factura</button>}
            </div>
          </div>

          {/* Per-dept comparison cards */}
          <div style={{display:"grid",gridTemplateColumns:"repeat(auto-fit,minmax(300px,1fr))",gap:16,marginBottom:24}}>
            {DEPTS.map(dept=>{
              const billRec = filteredBilling.filter(b=>b.dept===dept);
              const totalFacturado = billRec.reduce((s,b)=>s+(b.totalFinal||b.totalFactura||b.total||0),0);
              const totalGastos    = entries.filter(e=>e.dept===dept&&e.month===billMonth).reduce((s,e)=>s+e.amount,0);
              const totalComPagado = entries.filter(e=>e.dept===dept&&e.month===billMonth&&e.isComision).reduce((s,e)=>s+e.amount,0);
              const totalNeto      = entries.filter(e=>e.dept===dept&&e.month===billMonth&&e.isComision).reduce((s,e)=>s+(e.netoComision||e.amount),0);
              const margen         = totalFacturado - totalGastos;
              const color          = deptColor(dept);
              return (
                <div key={dept} style={{background:S.surface,border:`1px solid rgba(${deptRgb(dept)},.25)`,borderRadius:14,padding:24,position:"relative",overflow:"hidden"}}>
                  <div style={{position:"absolute",top:0,left:0,right:0,height:3,background:color}}/>
                  <div style={{display:"flex",alignItems:"center",justifyContent:"space-between",marginBottom:16}}>
                    <span style={{fontWeight:700,fontSize:16,color}}>{dept}</span>
                    <span style={{fontSize:11,color:"#7a8399",background:S.surface2,border:S.border2,borderRadius:20,padding:"3px 10px"}}>{billMonth}</span>
                  </div>
                  <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:10,marginBottom:14}}>
                    {[
                      {label:"📥 Facturado",val:fmtMoney(totalFacturado),c:"#22c55e"},
                      {label:"📤 Total Gastos",val:fmtMoney(totalGastos),c:"#ef4444"},
                      {label:"💸 Com. pagadas",val:fmtMoney(totalComPagado),c:"#f97316"},
                      {label:"✅ Com. correctas",val:fmtMoney(totalNeto),c:"#4f7fff"},
                    ].map((item,i)=>(
                      <div key={i} style={{background:S.surface2,borderRadius:8,padding:"10px 12px"}}>
                        <div style={{fontSize:10,color:"#7a8399",marginBottom:4}}>{item.label}</div>
                        <div style={{fontWeight:700,fontSize:14,color:item.c}}>{item.val}</div>
                      </div>
                    ))}
                  </div>
                  <div style={{background: margen>=0?"rgba(34,197,94,.08)":"rgba(239,68,68,.08)",border:`1px solid ${margen>=0?"rgba(34,197,94,.2)":"rgba(239,68,68,.2)"}`,borderRadius:8,padding:"12px 14px",display:"flex",justifyContent:"space-between",alignItems:"center"}}>
                    <span style={{fontSize:12,color:"#7a8399"}}>Margen {billMonth}</span>
                    <span style={{fontWeight:800,fontSize:18,color:margen>=0?"#22c55e":"#ef4444"}}>{margen>=0?"+":""}{fmtMoney(margen)}</span>
                  </div>
                  {billRec.length===0&&<div style={{marginTop:10,fontSize:11,color:"#4a5168",textAlign:"center"}}>Sin facturación registrada</div>}
                </div>
              );
            })}
          </div>

          {/* Billing records table */}
          <div style={{background:S.surface,border:S.border,borderRadius:12,overflow:"hidden"}}>
            <div style={{display:"flex",alignItems:"center",justifyContent:"space-between",padding:"16px 20px",borderBottom:S.border}}>
              <div style={{fontWeight:600,fontSize:14}}>Registros de Facturación — {billMonth}</div>
            </div>
            <table style={{width:"100%",borderCollapse:"collapse"}}>
              <thead><tr style={{background:S.surface2,borderBottom:S.border}}>
                {["Depto","Mes","BAF","TV","VOZ","Total Calc.","Total Factura","Reliq.","Total Final","Notas",""].map(h=><th key={h} style={{padding:"10px 12px",textAlign:"left",fontSize:10,fontWeight:600,textTransform:"uppercase",letterSpacing:".08em",color:"#7a8399"}}>{h}</th>)}
              </tr></thead>
              <tbody>
                {!filteredBilling.length&&<tr><td colSpan={10} style={{padding:"40px",textAlign:"center",color:"#4a5168",fontSize:13}}>No hay registros de facturación para {billMonth}</td></tr>}
                {filteredBilling.map(b=>{
                  const diff = (b.totalFactura||0) - (b.totalCalculado||0);
                  const reliqMonto = b.reliquidacion?.monto || 0;
                  const totalFinal = (b.totalFinal) || (b.totalFactura||0) + reliqMonto;
                  return (
                  <tr key={b.id} style={{borderBottom:S.border}} onMouseEnter={ev=>ev.currentTarget.style.background=S.surface2} onMouseLeave={ev=>ev.currentTarget.style.background="transparent"}>
                    <td style={{padding:"12px 12px"}}><span style={{background:`rgba(${deptRgb(b.dept)},.12)`,color:deptColor(b.dept),padding:"3px 10px",borderRadius:20,fontSize:11,fontWeight:600}}>{b.dept}</span></td>
                    <td style={{padding:"12px 12px",color:"#7a8399",fontSize:12}}>{b.month}</td>
                    <td style={{padding:"12px 12px"}}>
                      <div style={{fontFamily:"monospace",fontWeight:600,color:"#00b5e2",fontSize:12}}>{(b.baf?.qty||0).toLocaleString("es-CL")}</div>
                      <div style={{fontSize:10,color:"#4a5168"}}>{fmtMoney(b.baf?.val||0)}/u</div>
                    </td>
                    <td style={{padding:"12px 12px"}}>
                      <div style={{fontFamily:"monospace",fontWeight:600,color:"#a78bfa",fontSize:12}}>{(b.tv?.qty||0).toLocaleString("es-CL")}</div>
                      <div style={{fontSize:10,color:"#4a5168"}}>{fmtMoney(b.tv?.val||0)}/u</div>
                    </td>
                    <td style={{padding:"12px 12px"}}>
                      <div style={{fontFamily:"monospace",fontWeight:600,color:"#22c55e",fontSize:12}}>{(b.voz?.qty||0).toLocaleString("es-CL")}</div>
                      <div style={{fontSize:10,color:"#4a5168"}}>{fmtMoney(b.voz?.val||0)}/u</div>
                    </td>
                    <td style={{padding:"12px 12px",fontFamily:"monospace",fontSize:12,color:"#7a8399"}}>{fmtMoney(b.totalCalculado||0)}</td>
                    <td style={{padding:"12px 12px",fontFamily:"monospace",fontWeight:700,color:"#22c55e",fontSize:13}}>{fmtMoney(b.totalFactura||0)}</td>
                    <td style={{padding:"12px 12px"}}>
                      {b.reliquidacion
                        ? <span style={{background:"rgba(249,115,22,.12)",color:"#f97316",padding:"3px 10px",borderRadius:20,fontSize:11,fontWeight:700}}>+{fmtMoney(reliqMonto)}</span>
                        : <span style={{color:"#4a5168",fontSize:11}}>—</span>
                      }
                    </td>
                    <td style={{padding:"12px 12px",fontFamily:"monospace",fontWeight:800,fontSize:13,color:b.reliquidacion?"#f97316":"#22c55e"}}>{fmtMoney(totalFinal)}</td>
                    <td style={{padding:"12px 12px",fontSize:11,color:"#4a5168"}}>{b.notas||"—"}</td>
                    <td style={{padding:"12px 12px"}}>
                      {currentUser.role==="admin"&&(
                        <div style={{display:"flex",gap:4}}>
                          <button onClick={()=>setBillModal(b)} style={{background:S.surface2,border:S.border2,borderRadius:5,width:26,height:26,display:"flex",alignItems:"center",justifyContent:"center",cursor:"pointer",color:"#7a8399",fontSize:12}}>✎</button>
                          <button onClick={()=>deleteBilling(b.id)} style={{background:S.surface2,border:S.border2,borderRadius:5,width:26,height:26,display:"flex",alignItems:"center",justifyContent:"center",cursor:"pointer",color:"#7a8399",fontSize:12}}>✕</button>
                        </div>
                      )}
                    </td>
                  </tr>
                  );
                })}
              </tbody>
            </table>
          </div>
        </div>
      )}

      {/* ── CALIDAD ── */}
      {view==="calidad" && (() => {
        const camadaRows = calidadData[calidadCamada] || [];
        const total = camadaRows.length;
        const pagM1 = camadaRows.filter(r=>r.mes1===PAGO).length;
        const pagM2 = camadaRows.filter(r=>r.mes2===PAGO).length;
        const pagM3 = camadaRows.filter(r=>r.mes3===PAGO).length;
        const pagTres = camadaRows.filter(r=>r.mes1===PAGO&&r.mes2===PAGO&&r.mes3===PAGO).length;
        const pctM1  = total ? Math.round(pagM1/total*100) : 0;
        const pctM2  = total ? Math.round(pagM2/total*100) : 0;
        const pctM3  = total ? Math.round(pagM3/total*100) : 0;
        const pctBono = total ? Math.round(pagTres/total*100) : 0;
        const tieneM3 = camadaRows.some(r=>r.mes3!=="Sin datos");
        const tieneM2 = camadaRows.some(r=>r.mes2!=="Sin datos");

        // Ranking vendedores
        const vendMap = {};
        camadaRows.forEach(r=>{
          const v = r.usuario||"Sin asignar";
          if(!vendMap[v]) vendMap[v]={nombre:v,total:0,m1:0,m2:0,m3:0,bono:0};
          vendMap[v].total++;
          if(r.mes1===PAGO) vendMap[v].m1++;
          if(r.mes2===PAGO) vendMap[v].m2++;
          if(r.mes3===PAGO) vendMap[v].m3++;
          if(r.mes1===PAGO&&r.mes2===PAGO&&r.mes3===PAGO) vendMap[v].bono++;
        });
        const vendedores = Object.values(vendMap).sort((a,b)=>
          tieneM3 ? (b.bono/b.total)-(a.bono/a.total) : (b.m1/b.total)-(a.m1/a.total)
        );

        // Detalle filtrado
        const detalle = camadaRows.filter(r=>
          !calidadSearch || r.usuario.toLowerCase().includes(calidadSearch.toLowerCase()) ||
          r.rut.includes(calidadSearch) || r.orden.includes(calidadSearch)
        );

        const estadoColor = e => e===PAGO?"#22c55e":e==="Sin datos"||e==="Pendiente de facturación"?"#7a8399":"#ef4444";
        const estadoBadge = e => (
          <span style={{background:`${estadoColor(e)}22`,color:estadoColor(e),padding:"2px 8px",borderRadius:20,fontSize:10,fontWeight:600,whiteSpace:"nowrap"}}>
            {e==="Cliente pagó"?"✅ Pagó":e==="Sin datos"?"—":e==="Pendiente de facturación"?"⏳ Pend.":e==="Cliente aún no paga"?"⏳ Aún no":e==="Cobrar"?"📋 Cobrar":e==="Sin boleta"?"📄 S/boleta":e==="Desconexión"?"🔌 Desconex.":"❌ No pagó"}
          </span>
        );

        const handleExcel = async (e) => {
          const file = e.target.files[0]; if(!file) return;
          const XLSX = window.XLSX;
          if(!XLSX){showToast("Cargando librería, intenta de nuevo","error");return;}
          const ab = await file.arrayBuffer();
          const wb = XLSX.read(ab);
          const ws = wb.Sheets[wb.SheetNames[0]];
          const rows = XLSX.utils.sheet_to_json(ws,{header:1,defval:""});
          const parsed = parseCalidadExcel(rows);
          if(!parsed.length){showToast("No se encontraron datos válidos","error");return;}
          // group by camada
          const byCamada = {};
          parsed.forEach(r=>{ if(!byCamada[r.camada]) byCamada[r.camada]=[]; byCamada[r.camada].push(r); });
          setCalidadData(prev=>({...prev,...byCamada}));
          const camadas = Object.keys(byCamada).join(", ");
          showToast(`${parsed.length} registros cargados (${camadas}) ✓`);
          if(Object.keys(byCamada)[0]) setCalidadCamada(Object.keys(byCamada)[0]);
          e.target.value="";
        };

        const camadas = Object.keys(calidadData);

        return (
          <div style={{padding:24}}>
            {/* Header */}
            <div style={{display:"flex",alignItems:"center",gap:12,flexWrap:"wrap",marginBottom:20}}>
              <div>
                <div style={{fontWeight:700,fontSize:20}}>⭐ Control de Calidad</div>
                <div style={{fontSize:13,color:"#7a8399",marginTop:2}}>Movistar — Seguimiento de pagos por camada</div>
              </div>
              <div style={{marginLeft:"auto",display:"flex",alignItems:"center",gap:8,flexWrap:"wrap"}}>
                {camadas.length>0 && (
                  <select value={calidadCamada} onChange={e=>setCalidadCamada(e.target.value)}
                    style={{background:S.surface,border:S.border,borderRadius:8,padding:"7px 12px",color:"#e8ecf4",fontFamily:"inherit",fontSize:13,outline:"none",cursor:"pointer"}}>
                    {camadas.map(c=><option key={c}>{c}</option>)}
                  </select>
                )}
                <label style={{background:"#4f7fff",border:"none",borderRadius:8,padding:"8px 16px",color:"#fff",fontWeight:600,fontSize:13,cursor:"pointer",whiteSpace:"nowrap"}}>
                  📂 Cargar Excel
                  <input type="file" accept=".xlsx,.xls" onChange={handleExcel} style={{display:"none"}}/>
                </label>
              </div>
            </div>

            {/* Sub-tabs */}
            <div style={{display:"flex",gap:4,marginBottom:20,borderBottom:"1px solid #1e2330",paddingBottom:0}}>
              {[["resumen","📊 Resumen"],["vendedores","🏆 Vendedores"],["detalle","📋 Detalle"]].map(([v,lbl])=>(
                <button key={v} onClick={()=>setCalidadTab(v)} style={{background:"none",border:"none",borderBottom:`2px solid ${calidadTab===v?"#4f7fff":"transparent"}`,borderRadius:0,padding:"8px 16px",color:calidadTab===v?"#e8ecf4":"#7a8399",fontWeight:calidadTab===v?600:400,fontSize:13,cursor:"pointer",fontFamily:"inherit",marginBottom:-1}}>{lbl}</button>
              ))}
            </div>

            {/* Empty state */}
            {!total && (
              <div style={{textAlign:"center",padding:"80px 20px",color:"#4a5168"}}>
                <div style={{fontSize:48,marginBottom:16}}>📂</div>
                <div style={{fontSize:16,fontWeight:600,color:"#7a8399",marginBottom:8}}>Sin datos cargados</div>
                <div style={{fontSize:13}}>Presiona "Cargar Excel" para importar el listado de Movistar</div>
              </div>
            )}

            {/* ── RESUMEN ── */}
            {total>0 && calidadTab==="resumen" && (
              <div>
                {/* KPI cards */}
                <div style={{display:"grid",gridTemplateColumns:"repeat(auto-fit,minmax(200px,1fr))",gap:14,marginBottom:24}}>
                  {[
                    {label:"Total clientes",val:total.toLocaleString("es-CL"),color:"#4f7fff",icon:"👥"},
                    {label:"Mes 1 — pagaron",val:`${pagM1.toLocaleString("es-CL")} (${pctM1}%)`,color:"#22c55e",icon:"1️⃣"},
                    {label:"Mes 2 — pagaron",val:tieneM2?`${pagM2.toLocaleString("es-CL")} (${pctM2}%)`:"Sin datos aún",color:tieneM2?"#22c55e":"#4a5168",icon:"2️⃣"},
                    {label:"Mes 3 — pagaron",val:tieneM3?`${pagM3.toLocaleString("es-CL")} (${pctM3}%)`:"Sin datos aún",color:tieneM3?"#22c55e":"#4a5168",icon:"3️⃣"},
                  ].map((k,i)=>(
                    <div key={i} style={{background:S.surface,border:S.border,borderRadius:12,padding:20}}>
                      <div style={{fontSize:11,color:"#7a8399",textTransform:"uppercase",letterSpacing:".08em",marginBottom:6}}>{k.icon} {k.label}</div>
                      <div style={{fontWeight:800,fontSize:22,color:k.color}}>{k.val}</div>
                    </div>
                  ))}
                </div>

                {/* Bono calidad card */}
                <div style={{background:S.surface,border:`1px solid ${pctBono>=70?"rgba(34,197,94,.3)":pctBono>=50?"rgba(234,179,8,.3)":"rgba(239,68,68,.3)"}`,borderRadius:14,padding:28,marginBottom:24,position:"relative",overflow:"hidden"}}>
                  <div style={{position:"absolute",top:0,left:0,right:0,height:4,background:pctBono>=70?"#22c55e":pctBono>=50?"#eab308":"#ef4444"}}/>
                  <div style={{display:"flex",alignItems:"center",justifyContent:"space-between",flexWrap:"wrap",gap:16}}>
                    <div>
                      <div style={{fontSize:12,color:"#7a8399",textTransform:"uppercase",letterSpacing:".08em",marginBottom:4}}>⭐ Calidad Trimestral — Bono Movistar</div>
                      <div style={{fontSize:13,color:"#4a5168",marginBottom:12}}>Camada <strong style={{color:"#e8ecf4"}}>{calidadCamada}</strong> — clientes que pagaron los 3 meses consecutivos</div>
                      <div style={{display:"flex",alignItems:"baseline",gap:10}}>
                        <span style={{fontWeight:900,fontSize:48,color:pctBono>=70?"#22c55e":pctBono>=50?"#eab308":"#ef4444"}}>{tieneM3?`${pctBono}%`:"—"}</span>
                        {tieneM3&&<span style={{fontSize:14,color:"#7a8399"}}>{pagTres} de {total} clientes</span>}
                      </div>
                      {!tieneM3 && <div style={{fontSize:13,color:"#7a8399",marginTop:4}}>Aún no hay datos del Mes 3 para esta camada</div>}
                    </div>
                    {/* Mini barra de progreso */}
                    {tieneM3 && (
                      <div style={{minWidth:180}}>
                        <div style={{fontSize:11,color:"#7a8399",marginBottom:8,textAlign:"right"}}>{pctBono>=70?"✅ Bono alcanzado":pctBono>=50?"⚠️ En riesgo":"❌ Bajo el mínimo"}</div>
                        <div style={{background:"#1e2330",borderRadius:8,height:14,overflow:"hidden",marginBottom:6}}>
                          <div style={{height:"100%",width:`${pctBono}%`,background:pctBono>=70?"#22c55e":pctBono>=50?"#eab308":"#ef4444",borderRadius:8,transition:"width .5s"}}/>
                        </div>
                        <div style={{display:"flex",justifyContent:"space-between",fontSize:10,color:"#4a5168"}}>
                          <span>0%</span><span style={{color:"#eab308"}}>50%</span><span style={{color:"#22c55e"}}>70%</span><span>100%</span>
                        </div>
                      </div>
                    )}
                  </div>
                </div>

                {/* Barras mensuales */}
                <div style={{background:S.surface,border:S.border,borderRadius:12,padding:24}}>
                  <div style={{fontWeight:600,fontSize:14,marginBottom:20}}>Progresión mensual — {calidadCamada}</div>
                  <div style={{display:"flex",flexDirection:"column",gap:16}}>
                    {[["Mes 1",pagM1,pctM1,true],["Mes 2",pagM2,pctM2,tieneM2],["Mes 3",pagM3,pctM3,tieneM3]].map(([lbl,pag,pct,tiene],i)=>(
                      <div key={i}>
                        <div style={{display:"flex",justifyContent:"space-between",marginBottom:6}}>
                          <span style={{fontSize:13,color:"#e8ecf4"}}>{lbl}</span>
                          <span style={{fontSize:13,fontWeight:700,color:tiene?( pct>=70?"#22c55e":pct>=50?"#eab308":"#ef4444"):"#4a5168"}}>
                            {tiene?`${pag.toLocaleString("es-CL")} pagaron (${pct}%)`:"Sin datos aún"}
                          </span>
                        </div>
                        <div style={{background:"#1e2330",borderRadius:6,height:10,overflow:"hidden"}}>
                          <div style={{height:"100%",width:tiene?`${pct}%`:"0%",background:pct>=70?"#22c55e":pct>=50?"#eab308":"#ef4444",borderRadius:6,transition:"width .5s"}}/>
                        </div>
                      </div>
                    ))}
                  </div>
                </div>
              </div>
            )}

            {/* ── VENDEDORES ── */}
            {total>0 && calidadTab==="vendedores" && (
              <div style={{background:S.surface,border:S.border,borderRadius:12,overflow:"hidden"}}>
                <div style={{padding:"16px 20px",borderBottom:S.border,fontWeight:600,fontSize:14}}>
                  🏆 Ranking Vendedores — {calidadCamada}
                  <span style={{fontSize:12,color:"#7a8399",fontWeight:400,marginLeft:8}}>
                    {tieneM3?"ordenado por % calidad (3 meses)":"ordenado por % Mes 1"}
                  </span>
                </div>
                <table style={{width:"100%",borderCollapse:"collapse"}}>
                  <thead><tr style={{background:S.surface2,borderBottom:S.border}}>
                    {["#","Vendedor","Clientes","Mes 1","Mes 2","Mes 3","Calidad (3M)"].map(h=>(
                      <th key={h} style={{padding:"10px 14px",textAlign:"left",fontSize:10,fontWeight:600,textTransform:"uppercase",letterSpacing:".08em",color:"#7a8399"}}>{h}</th>
                    ))}
                  </tr></thead>
                  <tbody>
                    {vendedores.map((v,i)=>{
                      const pm1 = v.total?Math.round(v.m1/v.total*100):0;
                      const pm2 = v.total?Math.round(v.m2/v.total*100):0;
                      const pm3 = v.total?Math.round(v.m3/v.total*100):0;
                      const pb  = v.total?Math.round(v.bono/v.total*100):0;
                      const pct = c => <span style={{fontWeight:700,color:c>=70?"#22c55e":c>=50?"#eab308":"#ef4444"}}>{c}%</span>;
                      return (
                        <tr key={v.nombre} style={{borderBottom:S.border}} onMouseEnter={ev=>ev.currentTarget.style.background=S.surface2} onMouseLeave={ev=>ev.currentTarget.style.background="transparent"}>
                          <td style={{padding:"12px 14px",color:"#4a5168",fontSize:12,fontWeight:700}}>
                            {i===0?"🥇":i===1?"🥈":i===2?"🥉":`${i+1}`}
                          </td>
                          <td style={{padding:"12px 14px",fontSize:13,fontWeight:600,color:"#e8ecf4",maxWidth:200}}>
                            <div style={{overflow:"hidden",textOverflow:"ellipsis",whiteSpace:"nowrap"}}>{v.nombre}</div>
                          </td>
                          <td style={{padding:"12px 14px",fontFamily:"monospace",color:"#7a8399"}}>{v.total}</td>
                          <td style={{padding:"12px 14px"}}>{pct(pm1)}<span style={{color:"#4a5168",fontSize:11,marginLeft:4}}>({v.m1})</span></td>
                          <td style={{padding:"12px 14px"}}>{tieneM2?<>{pct(pm2)}<span style={{color:"#4a5168",fontSize:11,marginLeft:4}}>({v.m2})</span></>:<span style={{color:"#4a5168"}}>—</span>}</td>
                          <td style={{padding:"12px 14px"}}>{tieneM3?<>{pct(pm3)}<span style={{color:"#4a5168",fontSize:11,marginLeft:4}}>({v.m3})</span></>:<span style={{color:"#4a5168"}}>—</span>}</td>
                          <td style={{padding:"12px 14px"}}>
                            {tieneM3?(
                              <div style={{display:"flex",alignItems:"center",gap:8}}>
                                <div style={{background:"#1e2330",borderRadius:4,height:6,width:80,overflow:"hidden"}}>
                                  <div style={{height:"100%",width:`${pb}%`,background:pb>=70?"#22c55e":pb>=50?"#eab308":"#ef4444",borderRadius:4}}/>
                                </div>
                                {pct(pb)}
                              </div>
                            ):<span style={{color:"#4a5168"}}>—</span>}
                          </td>
                        </tr>
                      );
                    })}
                  </tbody>
                </table>
              </div>
            )}

            {/* ── DETALLE ── */}
            {total>0 && calidadTab==="detalle" && (
              <div style={{background:S.surface,border:S.border,borderRadius:12,overflow:"hidden"}}>
                <div style={{padding:"16px 20px",borderBottom:S.border,display:"flex",alignItems:"center",gap:12,flexWrap:"wrap"}}>
                  <div style={{fontWeight:600,fontSize:14}}>📋 Detalle Clientes — {calidadCamada}</div>
                  <input value={calidadSearch} onChange={e=>setCalidadSearch(e.target.value)}
                    placeholder="Buscar por vendedor, RUT u orden..."
                    style={{marginLeft:"auto",background:S.surface2,border:S.border,borderRadius:8,padding:"7px 12px",color:"#e8ecf4",fontFamily:"inherit",fontSize:12,outline:"none",width:260}}/>
                </div>
                <div style={{maxHeight:480,overflowY:"auto"}}>
                  <table style={{width:"100%",borderCollapse:"collapse"}}>
                    <thead style={{position:"sticky",top:0,zIndex:1}}>
                      <tr style={{background:S.surface2,borderBottom:S.border}}>
                        {["RUT","Orden","Vendedor","Mes 1","Mes 2","Mes 3"].map(h=>(
                          <th key={h} style={{padding:"10px 14px",textAlign:"left",fontSize:10,fontWeight:600,textTransform:"uppercase",letterSpacing:".08em",color:"#7a8399"}}>{h}</th>
                        ))}
                      </tr>
                    </thead>
                    <tbody>
                      {!detalle.length&&<tr><td colSpan={6} style={{padding:"40px",textAlign:"center",color:"#4a5168",fontSize:13}}>Sin resultados</td></tr>}
                      {detalle.slice(0,200).map(r=>(
                        <tr key={r.id} style={{borderBottom:S.border}} onMouseEnter={ev=>ev.currentTarget.style.background=S.surface2} onMouseLeave={ev=>ev.currentTarget.style.background="transparent"}>
                          <td style={{padding:"10px 14px",fontFamily:"monospace",fontSize:12,color:"#7a8399"}}>{r.rut}</td>
                          <td style={{padding:"10px 14px",fontFamily:"monospace",fontSize:12,color:"#7a8399"}}>{r.orden}</td>
                          <td style={{padding:"10px 14px",fontSize:12,color:"#e8ecf4",maxWidth:180}}>
                            <div style={{overflow:"hidden",textOverflow:"ellipsis",whiteSpace:"nowrap"}}>{r.usuario}</div>
                          </td>
                          <td style={{padding:"10px 14px"}}>{estadoBadge(r.mes1)}</td>
                          <td style={{padding:"10px 14px"}}>{estadoBadge(r.mes2)}</td>
                          <td style={{padding:"10px 14px"}}>{estadoBadge(r.mes3)}</td>
                        </tr>
                      ))}
                      {detalle.length>200&&<tr><td colSpan={6} style={{padding:"12px",textAlign:"center",color:"#4a5168",fontSize:12}}>Mostrando 200 de {detalle.length} — usa el buscador para filtrar</td></tr>}
                    </tbody>
                  </table>
                </div>
              </div>
            )}
          </div>
        );
      })()}

      {/* ── USUARIOS ── */}
      {view==="users" && (
        <div style={{padding:24}}>
          <div style={{display:"flex",alignItems:"center",justifyContent:"space-between",marginBottom:20}}>
            <div>
              <div style={{fontWeight:700,fontSize:20}}>👤 Gestión de Usuarios</div>
              <div style={{fontSize:13,color:"#7a8399",marginTop:2}}>Administra accesos y permisos del sistema</div>
            </div>
            <button onClick={()=>setUserModal("new")} style={{background:"#4f7fff",border:"none",borderRadius:8,padding:"8px 18px",color:"#fff",fontWeight:600,fontSize:13,cursor:"pointer"}}>+ Nuevo Usuario</button>
          </div>
          <div style={{background:S.surface,border:S.border,borderRadius:12,overflow:"hidden"}}>
            <table style={{width:"100%",borderCollapse:"collapse"}}>
              <thead><tr style={{background:S.surface2,borderBottom:S.border}}>
                {["Nombre","Correo","Rol","Departamento","Contraseña",""].map(h=><th key={h} style={{padding:"10px 16px",textAlign:"left",fontSize:10,fontWeight:600,textTransform:"uppercase",letterSpacing:".08em",color:"#7a8399"}}>{h}</th>)}
              </tr></thead>
              <tbody>
                {appUsers.map((u,i)=>(
                  <tr key={u.id} style={{borderBottom:S.border}} onMouseEnter={ev=>ev.currentTarget.style.background=S.surface2} onMouseLeave={ev=>ev.currentTarget.style.background="transparent"}>
                    <td style={{padding:"14px 16px",fontWeight:500}}>{u.name}</td>
                    <td style={{padding:"14px 16px",color:"#7a8399",fontFamily:"monospace",fontSize:12}}>{u.email}</td>
                    <td style={{padding:"14px 16px"}}>
                      <span style={{background:u.role==="admin"?"rgba(79,127,255,.15)":u.role==="editor"?"rgba(34,197,94,.1)":"rgba(122,131,153,.1)",color:u.role==="admin"?"#4f7fff":u.role==="editor"?"#22c55e":"#7a8399",padding:"3px 10px",borderRadius:20,fontSize:11,fontWeight:600}}>
                        {u.role==="admin"?"Administrador":u.role==="editor"?"Editor":"Lector"}
                      </span>
                    </td>
                    <td style={{padding:"14px 16px"}}>
                      {u.dept ? <span style={{background:`rgba(${deptRgb(u.dept)},.12)`,color:deptColor(u.dept),padding:"3px 10px",borderRadius:20,fontSize:11,fontWeight:600}}>{u.dept}</span> : <span style={{color:"#4a5168",fontSize:12}}>Todos</span>}
                    </td>
                    <td style={{padding:"14px 16px",fontFamily:"monospace",fontSize:12,color:"#4a5168"}}>{"•".repeat(u.pass.length)}</td>
                    <td style={{padding:"14px 16px"}}>
                      <div style={{display:"flex",gap:4}}>
                        <button onClick={()=>setUserModal(u)} style={{background:S.surface2,border:S.border2,borderRadius:5,width:26,height:26,display:"flex",alignItems:"center",justifyContent:"center",cursor:"pointer",color:"#7a8399",fontSize:12}}>✎</button>
                        {u.id!==currentUser.id && <button onClick={()=>askConfirm("¿Eliminar usuario "+u.name+"?", ()=>{ setAppUsers(p=>p.filter(x=>x.id!==u.id)); showToast("Usuario eliminado"); setConfirmModal(null); })} style={{background:S.surface2,border:S.border2,borderRadius:5,width:26,height:26,display:"flex",alignItems:"center",justifyContent:"center",cursor:"pointer",color:"#ef4444",fontSize:12}}>✕</button>}
                      </div>
                    </td>
                  </tr>
                ))}
              </tbody>
            </table>
          </div>
        </div>
      )}

      {modal && <Modal onClose={()=>setModal(null)} onSave={saveEntry} entry={modal==="new"?null:modal} userDept={currentUser.dept}/>}
      {billModal && <BillingModal onClose={()=>setBillModal(null)} onSave={saveBilling} entry={billModal==="new"?null:billModal}/>}
      {userModal && <UserModal onClose={()=>setUserModal(null)} onSave={data=>{
        if(userModal==="new"){setAppUsers(p=>[...p,{id:"u"+Date.now(),...data}]);showToast("Usuario creado ✓");}
        else{setAppUsers(p=>p.map(u=>u.id===userModal.id?{...u,...data}:u));showToast("Usuario actualizado ✓");}
        setUserModal(null);
      }} entry={userModal==="new"?null:userModal}/>}
      {toast && <Toast msg={toast.msg} type={toast.type}/>}
      {confirmModal && <ConfirmModal msg={confirmModal.msg} onConfirm={confirmModal.onConfirm} onCancel={()=>setConfirmModal(null)}/>}
    </div>
  );
}
