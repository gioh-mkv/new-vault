---
## tags: [homepage, financas] cssclasses: [home-page]
---

```dataviewjs
// ============================================================
//  HOME FINANÇAS — DataviewJS
//  Abas: Mensal (navegável) · Anual · Metas
//  Extras: bloco de anotações · editar/deletar transação · editar meta
// ============================================================

// ---------- HELPERS ----------
const hoje     = new Date();
const pad      = n => String(n).padStart(2, "0");
const dataHoje = `${hoje.getFullYear()}-${pad(hoje.getMonth()+1)}-${pad(hoje.getDate())}`;
const mesAtual = hoje.getMonth();
const anoAtual = hoje.getFullYear();
const MESES       = ["Janeiro","Fevereiro","Março","Abril","Maio","Junho","Julho","Agosto","Setembro","Outubro","Novembro","Dezembro"];
const MESES_ABREV = ["Jan","Fev","Mar","Abr","Mai","Jun","Jul","Ago","Set","Out","Nov","Dez"];

function ilink(path) { return `data-href="${path}" href="${path}" class="internal-link"`; }
function brl(v) { return "R$ " + Number(v||0).toLocaleString("pt-BR",{minimumFractionDigits:2,maximumFractionDigits:2}); }

// ---------- CAMINHOS ----------
const PATH_TX    = "01. Pessoal/Finanças/transacoes.json";
const PATH_METAS = "01. Pessoal/Finanças/metas.json";
const PATH_NOTAS = "01. Pessoal/Finanças/anotacoes.json";

// ---------- CATEGORIAS ----------
const CATEGORIAS = ["Alimentação","Moradia","Transporte","Saúde","Educação","Lazer","Vestuário","Assinaturas","Investimentos","Salário","Freelance","Outros"];
const COR_CAT = {
  "Alimentação":"#E24B4A","Moradia":"#185FA5","Transporte":"#854F0B",
  "Saúde":"#0F6E56","Educação":"#534AB7","Lazer":"#993556",
  "Vestuário":"#D4537E","Assinaturas":"#5F5E5A","Investimentos":"#3B6D11",
  "Salário":"#1D9E75","Freelance":"#378ADD","Outros":"#888780",
};

// ---------- LEITURA ----------
let transacoes = [];
let metas      = {};
let anotacoes  = "";

try { transacoes = JSON.parse(await app.vault.adapter.read(PATH_TX));    } catch(e) { transacoes = []; }
try { metas      = JSON.parse(await app.vault.adapter.read(PATH_METAS)); } catch(e) { metas = {};      }
try {
  const rawN = await app.vault.adapter.read(PATH_NOTAS);
  anotacoes = JSON.parse(rawN).texto || "";
} catch(e) { anotacoes = ""; }

// Adiciona índice original para permitir edição/exclusão
transacoes = transacoes.map((t, i) => ({ ...t, _idx: i }));
transacoes.sort((a, b) => new Date(b.data) - new Date(a.data));

// ---------- CÁLCULOS ----------
function txDoMes(mes, ano) {
  return transacoes.filter(t => {
    const d = new Date(t.data + "T00:00:00");
    return d.getMonth() === mes && d.getFullYear() === ano;
  });
}
function txDoAno(ano) {
  return transacoes.filter(t => new Date(t.data+"T00:00:00").getFullYear() === ano);
}
function somaReceitas(lista) { return lista.filter(t=>t.tipo==="receita").reduce((s,t)=>s+(t.valor||0),0); }
function somaDespesas(lista) { return lista.filter(t=>t.tipo==="despesa").reduce((s,t)=>s+(t.valor||0),0); }
function porCategoria(lista) {
  const map = {};
  for (const t of lista.filter(t=>t.tipo==="despesa")) {
    const c=t.categoria||"Outros"; map[c]=(map[c]||0)+(t.valor||0);
  }
  return Object.entries(map).sort((a,b)=>b[1]-a[1]);
}

const anosDisponiveis = [...new Set([anoAtual,...transacoes.map(t=>new Date(t.data+"T00:00:00").getFullYear())])].sort((a,b)=>b-a);

// ---------- SALVAR HELPERS ----------
async function salvarTransacoes(lista) {
  const clean = lista.map(({_idx,...t})=>t);
  clean.sort((a,b)=>new Date(b.data)-new Date(a.data));
  await app.vault.adapter.write(PATH_TX, JSON.stringify(clean,null,2));
}
async function salvarMetas(obj) {
  await app.vault.adapter.write(PATH_METAS, JSON.stringify(obj,null,2));
}
async function salvarAnotacoes(texto) {
  await app.vault.adapter.write(PATH_NOTAS, JSON.stringify({texto},null,2));
}

// ---------- GRÁFICO PIZZA ----------
function graficoPizza(cats) {
  if (!cats.length) return "";
  const total=cats.reduce((s,[,v])=>s+v,0);
  if (!total) return "";
  const R=60,CX=75,CY=75;
  let svg=`<svg width="150" height="150" viewBox="0 0 150 150">`;
  let ang=-Math.PI/2;
  cats.slice(0,6).forEach(([cat,val])=>{
    const frac=val/total,sweep=frac*2*Math.PI;
    const x1=CX+R*Math.cos(ang),y1=CY+R*Math.sin(ang);
    ang+=sweep;
    const x2=CX+R*Math.cos(ang),y2=CY+R*Math.sin(ang);
    const large=sweep>Math.PI?1:0;
    const cor=COR_CAT[cat]||"#888";
    svg+=`<path d="M${CX},${CY} L${x1.toFixed(2)},${y1.toFixed(2)} A${R},${R} 0 ${large} 1 ${x2.toFixed(2)},${y2.toFixed(2)} Z" fill="${cor}" stroke="var(--background-primary)" stroke-width="1.5"/>`;
  });
  svg+=`<circle cx="${CX}" cy="${CY}" r="32" fill="var(--background-primary)"/>`;
  svg+=`<text x="${CX}" y="${CY-6}" text-anchor="middle" style="font-size:9px;fill:var(--text-faint);font-family:var(--font-interface);">Gastos</text>`;
  svg+=`<text x="${CX}" y="${CY+8}" text-anchor="middle" style="font-size:9px;font-weight:600;fill:var(--text-normal);font-family:var(--font-interface);">${brl(total).replace("R$ ","")}</text>`;
  return svg+`</svg>`;
}

// ---------- GRÁFICO BARRAS ANUAL ----------
function graficoBarre(dadosMensais, mesDestaque) {
  const W=580,H=160,PAD={top:10,right:10,bottom:24,left:46};
  const iW=W-PAD.left-PAD.right,iH=H-PAD.top-PAD.bottom;
  const maxVal=Math.max(...dadosMensais.flatMap(d=>[d.receita,d.despesa]),1);
  const barW=Math.floor(iW/12/2)-2,grupoW=Math.floor(iW/12);
  let svg=`<svg width="100%" viewBox="0 0 ${W} ${H}" style="overflow:visible;">`;
  for(let i=0;i<=4;i++){
    const v=maxVal/4*i,y=PAD.top+iH-(v/maxVal)*iH;
    svg+=`<line x1="${PAD.left}" y1="${y}" x2="${W-PAD.right}" y2="${y}" stroke="var(--background-modifier-border)" stroke-width="0.5"/>`;
    if(i>0) svg+=`<text x="${PAD.left-4}" y="${y+4}" text-anchor="end" style="font-size:8px;fill:var(--text-faint);font-family:var(--font-interface);">${(v/1000).toFixed(0)}k</text>`;
  }
  dadosMensais.forEach((d,i)=>{
    const xBase=PAD.left+i*grupoW+2;
    const hRec=(d.receita/maxVal)*iH,hDesp=(d.despesa/maxVal)*iH;
    const dest=i===mesDestaque;
    svg+=`<rect x="${xBase}" y="${PAD.top+iH-hRec}" width="${barW}" height="${hRec}" fill="#1D9E75" opacity="${dest?1:0.45}" rx="2"/>`;
    svg+=`<rect x="${xBase+barW+1}" y="${PAD.top+iH-hDesp}" width="${barW}" height="${hDesp}" fill="#E24B4A" opacity="${dest?1:0.45}" rx="2"/>`;
    svg+=`<text x="${xBase+barW}" y="${H-PAD.bottom+14}" text-anchor="middle" style="font-size:8px;fill:${dest?'var(--text-normal)':'var(--text-faint)'};font-family:var(--font-interface);font-weight:${dest?'600':'400'};">${MESES_ABREV[i]}</text>`;
  });
  return svg+`</svg>`;
}

// ================================================================
//  RENDERIZA ABA MENSAL (função reutilizável para navegação)
// ================================================================
function renderMensal(painel, mes, ano) {
  const tx       = txDoMes(mes, ano);
  const receitas = somaReceitas(tx);
  const despesas = somaDespesas(tx);
  const saldoMes = receitas - despesas;
  const cats     = porCategoria(tx);

  // Limpa painel
  painel.innerHTML = "";

  // ── Barra de navegação ────────────────────────────────────────
  const nav = painel.createEl("div", {
    attr: { style: "display:flex;align-items:center;justify-content:space-between;gap:10px;margin-bottom:20px;flex-wrap:wrap;" }
  });

  const btnPrev = nav.createEl("button", { text:"←", attr:{ style:"padding:6px 14px;border:1px solid var(--background-modifier-border);border-radius:7px;background:var(--background-secondary);color:var(--text-normal);font-size:14px;cursor:pointer;" } });

  const label = nav.createEl("div", {
    attr: { style:"font-size:16px;font-weight:700;color:var(--text-normal);flex:1;text-align:center;" }
  });
  label.textContent = `${MESES[mes]} ${ano}`;

  const selMes = nav.createEl("select", { attr:{ style:"padding:6px 10px;border:1px solid var(--background-modifier-border);border-radius:7px;background:var(--background-secondary);color:var(--text-normal);font-size:13px;cursor:pointer;" } });
  MESES.forEach((m,i)=>{ const o=selMes.createEl("option",{text:m,attr:{value:i}}); if(i===mes) o.selected=true; });

  const selAno = nav.createEl("select", { attr:{ style:"padding:6px 10px;border:1px solid var(--background-modifier-border);border-radius:7px;background:var(--background-secondary);color:var(--text-normal);font-size:13px;cursor:pointer;" } });
  anosDisponiveis.forEach(a=>{ const o=selAno.createEl("option",{text:String(a),attr:{value:a}}); if(a===ano) o.selected=true; });

  const btnNext = nav.createEl("button", { text:"→", attr:{ style:`padding:6px 14px;border:1px solid var(--background-modifier-border);border-radius:7px;background:var(--background-secondary);color:var(--text-normal);font-size:14px;cursor:pointer;${mes===mesAtual&&ano===anoAtual?"opacity:0.4;cursor:default;":""}` } });
  if (mes===mesAtual && ano===anoAtual) btnNext.disabled = true;

  btnPrev.addEventListener("click",  ()=>{ let m=mes-1,a=ano; if(m<0){m=11;a--;} renderMensal(painel,m,a); });
  btnNext.addEventListener("click",  ()=>{ if(mes===mesAtual&&ano===anoAtual)return; let m=mes+1,a=ano; if(m>11){m=0;a++;} renderMensal(painel,m,a); });
  selMes.addEventListener("change",  ()=>renderMensal(painel,parseInt(selMes.value),ano));
  selAno.addEventListener("change",  ()=>renderMensal(painel,mes,parseInt(selAno.value)));

  // ── Cards resumo ─────────────────────────────────────────────
  const cards = painel.createEl("div", { attr:{style:"display:grid;grid-template-columns:repeat(3,minmax(0,1fr));gap:10px;margin-bottom:20px;"} });
  for (const [lbl,val,cor] of [["Receitas",receitas,"#1D9E75"],["Despesas",despesas,"#E24B4A"],["Saldo",saldoMes,saldoMes>=0?"#1D9E75":"#E24B4A"]]) {
    const c = cards.createEl("div",{attr:{style:"background:var(--background-secondary);border-radius:10px;padding:14px;"}});
    c.createEl("div",{text:lbl,attr:{style:"font-size:11px;color:var(--text-faint);margin-bottom:6px;text-transform:uppercase;letter-spacing:.06em;"}});
    c.createEl("div",{text:brl(val),attr:{style:`font-size:20px;font-weight:700;color:${cor};`}});
  }

  // ── Pizza ─────────────────────────────────────────────────────
  if (cats.length > 0) {
    const totalCats = cats.reduce((s,[,v])=>s+v,0);
    painel.createEl("p",{text:"Gastos por categoria",attr:{style:"font-size:11px;font-weight:600;color:var(--text-faint);text-transform:uppercase;letter-spacing:.06em;margin:0 0 10px;"}});
    const pizzaWrap = painel.createEl("div",{attr:{style:"display:flex;gap:20px;align-items:flex-start;margin-bottom:20px;flex-wrap:wrap;"}});
    pizzaWrap.innerHTML += graficoPizza(cats);
    const legenda = pizzaWrap.createEl("div",{attr:{style:"flex:1;min-width:160px;"}});
    for (const [cat,val] of cats) {
      const pct=totalCats>0?Math.round((val/totalCats)*100):0;
      const row=legenda.createEl("div",{attr:{style:"display:flex;align-items:center;gap:8px;margin-bottom:6px;"}});
      row.createEl("div",{attr:{style:`width:8px;height:8px;border-radius:50%;background:${COR_CAT[cat]||'#888'};flex-shrink:0;`}});
      row.createEl("span",{text:cat,attr:{style:"font-size:12px;color:var(--text-normal);flex:1;"}});
      row.createEl("span",{text:`${pct}%`,attr:{style:"font-size:11px;color:var(--text-faint);"}});
      row.createEl("span",{text:brl(val),attr:{style:"font-size:12px;font-weight:500;color:var(--text-normal);"}});
    }
  }

  // ── Tabela de transações ──────────────────────────────────────
  painel.createEl("p",{text:`Transações — ${MESES[mes]} ${ano}`,attr:{style:"font-size:11px;font-weight:600;color:var(--text-faint);text-transform:uppercase;letter-spacing:.06em;margin:0 0 10px;"}});

  if (tx.length === 0) {
    painel.createEl("div",{attr:{style:"border:1px dashed var(--background-modifier-border);border-radius:10px;padding:20px;text-align:center;"}})
      .createEl("div",{text:"Nenhuma transação neste período.",attr:{style:"font-size:13px;color:var(--text-faint);"}});
    return;
  }

  const tabela = painel.createEl("div",{attr:{style:"border:1px solid var(--background-modifier-border);border-radius:10px;overflow:hidden;"}});

  // Cabeçalho
  const thead = tabela.createEl("div",{attr:{style:"display:grid;grid-template-columns:80px 1fr 110px 100px 72px;background:var(--background-secondary);padding:8px 14px;border-bottom:1px solid var(--background-modifier-border);"}});
  for (const t of ["Data","Descrição","Categoria","Valor",""]) {
    thead.createEl("span",{text:t,attr:{style:`font-size:11px;font-weight:600;color:var(--text-faint);${t===""?"text-align:right;":""}`}});
  }

  for (const t of tx) {
    const cor   = t.tipo==="receita"?"#1D9E75":"#E24B4A";
    const sinal = t.tipo==="receita"?"+":"−";

    const row = tabela.createEl("div",{attr:{style:"display:grid;grid-template-columns:80px 1fr 110px 100px 72px;padding:8px 14px;border-bottom:1px solid var(--background-modifier-border);align-items:center;"}});
    row.createEl("span",{text:(t.data||"").slice(5).replace("-","/"),attr:{style:"font-size:12px;color:var(--text-faint);"}});
    row.createEl("span",{text:t.descricao||"—",attr:{style:"font-size:12px;color:var(--text-normal);"}});
    row.createEl("span",{text:t.categoria||"—",attr:{style:"font-size:11px;color:var(--text-faint);"}});
    row.createEl("span",{text:`${sinal} ${brl(t.valor).replace("R$ ","")}`,attr:{style:`font-size:12px;font-weight:600;color:${cor};`}});

    // Botões editar/deletar
    const acoes = row.createEl("div",{attr:{style:"display:flex;gap:4px;justify-content:flex-end;"}});

    const btnEdit = acoes.createEl("button",{text:"✎",attr:{style:"padding:3px 7px;border:1px solid var(--background-modifier-border);border-radius:5px;background:var(--background-secondary);color:var(--text-muted);font-size:12px;cursor:pointer;"}});
    const btnDel  = acoes.createEl("button",{text:"✕",attr:{style:"padding:3px 7px;border:1px solid var(--background-modifier-border);border-radius:5px;background:var(--background-secondary);color:#E24B4A;font-size:12px;cursor:pointer;"}});

    // ── Deletar ──
    btnDel.addEventListener("click", async () => {
      if (!confirm(`Excluir "${t.descricao}" (${brl(t.valor)})?`)) return;
      // Remove pelo índice original
      const novas = transacoes.filter(x => x._idx !== t._idx);
      await salvarTransacoes(novas);
      dv.index.touch(app.vault.getAbstractFileByPath(dv.currentFilePath));
    });

    // ── Editar (inline) ──
    btnEdit.addEventListener("click", () => {
      // Substitui a linha por um formulário inline
      row.innerHTML = "";
      row.style.gridTemplateColumns = "1fr";
      row.style.padding = "10px 14px";

      const editGrid = row.createEl("div",{attr:{style:"display:grid;grid-template-columns:120px 1fr 110px 140px 110px auto;gap:8px;align-items:end;"}});

      const fStyle = "width:100%;padding:5px 8px;border-radius:6px;border:1px solid var(--background-modifier-border);background:var(--background-primary);color:var(--text-normal);font-size:12px;";

      const eData = editGrid.createEl("input",{attr:{type:"date",value:t.data||"",style:fStyle}});
      const eDesc = editGrid.createEl("input",{attr:{type:"text",value:t.descricao||"",style:fStyle}});

      const eTipo = editGrid.createEl("select",{attr:{style:fStyle}});
      eTipo.createEl("option",{text:"Despesa",attr:{value:"despesa"}}); 
      eTipo.createEl("option",{text:"Receita",attr:{value:"receita"}});
      eTipo.value = t.tipo||"despesa";

      const eCat = editGrid.createEl("select",{attr:{style:fStyle}});
      for (const c of CATEGORIAS) { const o=eCat.createEl("option",{text:c,attr:{value:c}}); if(c===t.categoria)o.selected=true; }

      const eVal = editGrid.createEl("input",{attr:{type:"number",step:"0.01",value:t.valor||"",style:fStyle}});

      const eBtns = editGrid.createEl("div",{attr:{style:"display:flex;gap:4px;"}});
      const btnOk  = eBtns.createEl("button",{text:"✓",attr:{style:"padding:5px 9px;border:none;border-radius:6px;background:var(--interactive-accent);color:var(--text-on-accent);font-size:13px;cursor:pointer;font-weight:700;"}});
      const btnCancel = eBtns.createEl("button",{text:"✕",attr:{style:"padding:5px 9px;border:1px solid var(--background-modifier-border);border-radius:6px;background:var(--background-secondary);color:var(--text-muted);font-size:13px;cursor:pointer;"}});

      btnCancel.addEventListener("click", () => renderMensal(painel, mes, ano));

      btnOk.addEventListener("click", async () => {
        const novoValor = parseFloat(eVal.value);
        if (!eData.value||!eDesc.value.trim()||isNaN(novoValor)||novoValor<=0) return;
        const novas = transacoes.map(x => {
          if (x._idx !== t._idx) return x;
          return { ...x, data:eData.value, descricao:eDesc.value.trim(), tipo:eTipo.value, categoria:eCat.value, valor:parseFloat(novoValor.toFixed(2)) };
        });
        await salvarTransacoes(novas);
        dv.index.touch(app.vault.getAbstractFileByPath(dv.currentFilePath));
      });
    });
  }
}

// ================================================================
//  HTML ANUAL
// ================================================================
function htmlAnual() {
  const txAno=txDoAno(anoAtual);
  const recAnual=somaReceitas(txAno),despAnual=somaDespesas(txAno),saldoAnual=recAnual-despAnual;
  const dadosMensais=MESES.map((_,i)=>{const t=txDoMes(i,anoAtual);return{receita:somaReceitas(t),despesa:somaDespesas(t)};});
  return `
  <div style="display:grid;grid-template-columns:repeat(3,minmax(0,1fr));gap:10px;margin-bottom:20px;">
    <div style="background:var(--background-secondary);border-radius:10px;padding:14px;">
      <div style="font-size:11px;color:var(--text-faint);margin-bottom:6px;text-transform:uppercase;letter-spacing:.06em;">Receitas ${anoAtual}</div>
      <div style="font-size:20px;font-weight:700;color:#1D9E75;">${brl(recAnual)}</div>
    </div>
    <div style="background:var(--background-secondary);border-radius:10px;padding:14px;">
      <div style="font-size:11px;color:var(--text-faint);margin-bottom:6px;text-transform:uppercase;letter-spacing:.06em;">Despesas ${anoAtual}</div>
      <div style="font-size:20px;font-weight:700;color:#E24B4A;">${brl(despAnual)}</div>
    </div>
    <div style="background:var(--background-secondary);border-radius:10px;padding:14px;">
      <div style="font-size:11px;color:var(--text-faint);margin-bottom:6px;text-transform:uppercase;letter-spacing:.06em;">Saldo ${anoAtual}</div>
      <div style="font-size:20px;font-weight:700;color:${saldoAnual>=0?'#1D9E75':'#E24B4A'};">${brl(saldoAnual)}</div>
    </div>
  </div>
  <div style="border:1px solid var(--background-modifier-border);border-radius:10px;padding:16px;margin-bottom:20px;">
    <div style="display:flex;align-items:center;justify-content:space-between;margin-bottom:12px;">
      <span style="font-size:12px;font-weight:600;color:var(--text-muted);">Receitas vs Despesas — ${anoAtual}</span>
      <div style="display:flex;gap:12px;">
        <span style="display:flex;align-items:center;gap:5px;font-size:11px;color:var(--text-faint);"><span style="width:10px;height:10px;border-radius:2px;background:#1D9E75;display:inline-block;"></span>Receitas</span>
        <span style="display:flex;align-items:center;gap:5px;font-size:11px;color:var(--text-faint);"><span style="width:10px;height:10px;border-radius:2px;background:#E24B4A;display:inline-block;"></span>Despesas</span>
      </div>
    </div>
    ${graficoBarre(dadosMensais,mesAtual)}
  </div>
  <div style="border:1px solid var(--background-modifier-border);border-radius:10px;overflow:hidden;">
    <div style="display:grid;grid-template-columns:1fr 1fr 1fr 1fr;background:var(--background-secondary);padding:8px 14px;border-bottom:1px solid var(--background-modifier-border);">
      <span style="font-size:11px;font-weight:600;color:var(--text-faint);">Mês</span>
      <span style="font-size:11px;font-weight:600;color:var(--text-faint);text-align:right;">Receitas</span>
      <span style="font-size:11px;font-weight:600;color:var(--text-faint);text-align:right;">Despesas</span>
      <span style="font-size:11px;font-weight:600;color:var(--text-faint);text-align:right;">Saldo</span>
    </div>
    ${dadosMensais.map((d,i)=>{
      const sal=d.receita-d.despesa,dest=i===mesAtual;
      return `<div style="display:grid;grid-template-columns:1fr 1fr 1fr 1fr;padding:8px 14px;border-bottom:1px solid var(--background-modifier-border);background:${dest?'var(--background-secondary)':'transparent'};">
        <span style="font-size:12px;color:var(--text-normal);font-weight:${dest?'600':'400'};">${MESES[i]}${dest?" ←":""}</span>
        <span style="font-size:12px;color:#1D9E75;text-align:right;">${d.receita>0?brl(d.receita):"—"}</span>
        <span style="font-size:12px;color:#E24B4A;text-align:right;">${d.despesa>0?brl(d.despesa):"—"}</span>
        <span style="font-size:12px;font-weight:600;color:${sal>=0?'#1D9E75':'#E24B4A'};text-align:right;">${(d.receita>0||d.despesa>0)?brl(sal):"—"}</span>
      </div>`;
    }).join("")}
  </div>`;
}

// ================================================================
//  RENDERIZA ABA METAS (com edição inline)
// ================================================================
function renderMetas(painel) {
  painel.innerHTML = "";

  if (!Object.keys(metas).length) {
    painel.createEl("div",{attr:{style:"border:1px dashed var(--background-modifier-border);border-radius:10px;padding:24px;text-align:center;"}})
      .createEl("div",{text:"Nenhuma meta cadastrada. Crie o arquivo Finanças/metas.json.",attr:{style:"font-size:13px;color:var(--text-faint);"}});
    return;
  }

  const txMes = txDoMes(mesAtual, anoAtual);
  const tabela = painel.createEl("div",{attr:{style:"border:1px solid var(--background-modifier-border);border-radius:10px;overflow:hidden;margin-bottom:16px;"}});

  const thead = tabela.createEl("div",{attr:{style:"display:grid;grid-template-columns:1fr 130px 130px 70px 60px;background:var(--background-secondary);padding:8px 14px;border-bottom:1px solid var(--background-modifier-border);"}});
  for (const t of ["Categoria","Meta mensal","Gasto este mês","%",""]) {
    thead.createEl("span",{text:t,attr:{style:`font-size:11px;font-weight:600;color:var(--text-faint);${t===""||t==="%"?"text-align:right;":t!=="Categoria"?"text-align:right;":""}`}});
  }

  for (const [cat, metaVal] of Object.entries(metas)) {
    const gasto = txMes.filter(t=>t.tipo==="despesa"&&t.categoria===cat).reduce((s,t)=>s+(t.valor||0),0);
    const pct   = metaVal>0?Math.round((gasto/metaVal)*100):0;
    const cor   = pct>=100?"#E24B4A":pct>=80?"#EF9F27":"#1D9E75";

    const rowWrap = tabela.createEl("div",{attr:{style:"border-bottom:1px solid var(--background-modifier-border);"}});
    const row = rowWrap.createEl("div",{attr:{style:"display:grid;grid-template-columns:1fr 130px 130px 70px 60px;padding:8px 14px;align-items:center;"}});

    // Categoria
    const catCell = row.createEl("span",{attr:{style:"display:flex;align-items:center;gap:8px;font-size:12px;color:var(--text-normal);"}});
    catCell.createEl("span",{attr:{style:`width:8px;height:8px;border-radius:50%;background:${COR_CAT[cat]||'#888'};flex-shrink:0;`}});
    catCell.createSpan({text:cat});

    row.createEl("span",{text:brl(metaVal),attr:{style:"font-size:12px;color:var(--text-faint);text-align:right;"}});
    row.createEl("span",{text:brl(gasto),   attr:{style:`font-size:12px;font-weight:600;color:${cor};text-align:right;`}});
    row.createEl("span",{text:`${pct}%`,    attr:{style:`font-size:12px;font-weight:600;color:${cor};text-align:right;`}});

    // Botão editar meta
    const btnEditar = row.createEl("button",{text:"✎",attr:{style:"padding:3px 7px;border:1px solid var(--background-modifier-border);border-radius:5px;background:var(--background-secondary);color:var(--text-muted);font-size:12px;cursor:pointer;float:right;"}});

    // Barra de progresso
    const barraWrap = rowWrap.createEl("div",{attr:{style:"padding:0 14px 10px;"}});
    const barra = barraWrap.createEl("div",{attr:{style:"height:4px;background:var(--background-modifier-border);border-radius:2px;"}});
    barra.createEl("div",{attr:{style:`height:4px;width:${Math.min(100,pct)}%;background:${cor};border-radius:2px;`}});

    // ── Editar meta inline ──
    btnEditar.addEventListener("click", () => {
      row.innerHTML = "";
      row.style.gridTemplateColumns = "1fr";
      row.style.padding = "10px 14px";

      const editRow = row.createEl("div",{attr:{style:"display:flex;align-items:center;gap:10px;flex-wrap:wrap;"}});
      editRow.createEl("span",{text:cat,attr:{style:"font-size:13px;font-weight:600;color:var(--text-normal);flex:1;"}});
      editRow.createEl("label",{text:"Nova meta (R$):",attr:{style:"font-size:12px;color:var(--text-faint);"}});
      const inputMeta = editRow.createEl("input",{attr:{type:"number",step:"1",min:"0",value:metaVal,style:"width:120px;padding:5px 8px;border-radius:6px;border:1px solid var(--background-modifier-border);background:var(--background-primary);color:var(--text-normal);font-size:13px;"}});

      const btnOk     = editRow.createEl("button",{text:"✓",attr:{style:"padding:5px 10px;border:none;border-radius:6px;background:var(--interactive-accent);color:var(--text-on-accent);font-size:13px;cursor:pointer;font-weight:700;"}});
      const btnCancel = editRow.createEl("button",{text:"✕",attr:{style:"padding:5px 10px;border:1px solid var(--background-modifier-border);border-radius:6px;background:var(--background-secondary);color:var(--text-muted);font-size:13px;cursor:pointer;"}});

      btnCancel.addEventListener("click", ()=>renderMetas(painel));
      btnOk.addEventListener("click", async ()=>{
        const novoVal = parseFloat(inputMeta.value);
        if (isNaN(novoVal)||novoVal<0) return;
        metas[cat] = novoVal;
        await salvarMetas(metas);
        renderMetas(painel);
      });
    });
  }

  // Botão adicionar nova categoria de meta
  const addWrap = painel.createEl("div",{attr:{style:"margin-top:12px;"}});
  const btnAdd  = addWrap.createEl("button",{text:"+ Adicionar meta",attr:{style:"padding:7px 14px;border:1px dashed var(--background-modifier-border);border-radius:8px;background:transparent;color:var(--text-muted);font-size:12px;cursor:pointer;"}});

  btnAdd.addEventListener("click",()=>{
    btnAdd.style.display="none";
    const formAdd = addWrap.createEl("div",{attr:{style:"display:flex;align-items:center;gap:8px;flex-wrap:wrap;margin-top:8px;"}});

    const selCatAdd = formAdd.createEl("select",{attr:{style:"padding:6px 10px;border:1px solid var(--background-modifier-border);border-radius:7px;background:var(--background-primary);color:var(--text-normal);font-size:13px;"}});
    for (const c of CATEGORIAS) {
      if (!(c in metas)) selCatAdd.createEl("option",{text:c,attr:{value:c}});
    }
    const inputAdd = formAdd.createEl("input",{attr:{type:"number",step:"1",min:"0",placeholder:"Meta (R$)",style:"width:130px;padding:6px 10px;border-radius:7px;border:1px solid var(--background-modifier-border);background:var(--background-primary);color:var(--text-normal);font-size:13px;"}});
    const btnSave   = formAdd.createEl("button",{text:"Salvar",attr:{style:"padding:6px 14px;border:none;border-radius:7px;background:var(--interactive-accent);color:var(--text-on-accent);font-size:13px;cursor:pointer;font-weight:600;"}});
    const btnCancel = formAdd.createEl("button",{text:"Cancelar",attr:{style:"padding:6px 12px;border:1px solid var(--background-modifier-border);border-radius:7px;background:var(--background-secondary);color:var(--text-muted);font-size:13px;cursor:pointer;"}});

    btnCancel.addEventListener("click",()=>renderMetas(painel));
    btnSave.addEventListener("click",async()=>{
      const novoVal=parseFloat(inputAdd.value);
      if(isNaN(novoVal)||novoVal<=0||!selCatAdd.value)return;
      metas[selCatAdd.value]=novoVal;
      await salvarMetas(metas);
      renderMetas(painel);
    });
  });
}

// ================================================================
//  MONTA ESTRUTURA PRINCIPAL
// ================================================================

// Cabeçalho estático
dv.el("div",`<div style="max-width:860px;padding-bottom:16px;border-bottom:1px solid var(--background-modifier-border);margin-bottom:24px;display:flex;align-items:flex-start;justify-content:space-between;">
  <div>
    <div style="font-size:22px;font-weight:700;color:var(--text-normal);">Finanças</div>
    <div style="font-size:13px;color:var(--text-faint);margin-top:4px;">${MESES[mesAtual]} de ${anoAtual}</div>
  </div>
  <a ${ilink("Home")} style="font-size:12px;color:var(--interactive-accent);text-decoration:none;margin-top:4px;">← voltar para Home</a>
</div>`);

const root = dv.el("div","",{attr:{style:"max-width:860px;"}});

// ── Abas ──
const ABAS = [
  { id:"mensal",  label:"Mensal"  },
  { id:"anual",   label:"Anual"   },
  { id:"metas",   label:"Metas"   },
];

const tabBar  = root.createEl("div",{attr:{style:"display:flex;gap:4px;border-bottom:1px solid var(--background-modifier-border);margin-bottom:20px;"}});
const paineis = {};
const tabBtns = {};

for (const aba of ABAS) {
  const btn = tabBar.createEl("button",{attr:{style:"padding:8px 16px;border:none;border-bottom:2px solid transparent;background:transparent;color:var(--text-muted);font-size:13px;font-weight:400;cursor:pointer;border-radius:0;"}});
  btn.textContent = aba.label;
  tabBtns[aba.id] = btn;
  const painel = root.createEl("div",{attr:{style:"display:none;"}});
  paineis[aba.id] = painel;
}

paineis["anual"].innerHTML = htmlAnual();

function mostrarAba(id) {
  for (const aba of ABAS) {
    const ativo=aba.id===id;
    paineis[aba.id].style.display      = ativo?"block":"none";
    tabBtns[aba.id].style.borderBottom = ativo?"2px solid var(--interactive-accent)":"2px solid transparent";
    tabBtns[aba.id].style.color        = ativo?"var(--interactive-accent)":"var(--text-muted)";
    tabBtns[aba.id].style.fontWeight   = ativo?"600":"400";
  }
}

renderMensal(paineis["mensal"], mesAtual, anoAtual);
renderMetas(paineis["metas"]);
mostrarAba("mensal");
for (const aba of ABAS) {
  tabBtns[aba.id].addEventListener("click",()=>{
    if(aba.id==="metas") renderMetas(paineis["metas"]);
    mostrarAba(aba.id);
  });
}

// ================================================================
//  FORMULÁRIO DE NOVA TRANSAÇÃO
// ================================================================
const sepForm = root.createEl("div",{attr:{style:"margin-top:28px;border-top:1px solid var(--background-modifier-border);padding-top:20px;"}});
sepForm.createEl("p",{text:"Registrar nova transação",attr:{style:"font-size:11px;font-weight:600;letter-spacing:.08em;text-transform:uppercase;color:var(--text-faint);margin:0 0 14px;"}});

const fGrid = sepForm.createEl("div",{attr:{style:"display:grid;grid-template-columns:130px 1fr 110px 150px 120px;gap:10px;align-items:end;"}});
const fS = "width:100%;padding:7px 10px;border-radius:7px;border:1px solid var(--background-modifier-border);background:var(--background-primary);color:var(--text-normal);font-size:13px;";
const lS = "font-size:11px;color:var(--text-faint);display:block;margin-bottom:4px;";

const c1=fGrid.createEl("div"); c1.createEl("label",{text:"Data",attr:{style:lS}});
const inputData=c1.createEl("input",{attr:{type:"date",value:dataHoje,style:fS}});

const c2=fGrid.createEl("div"); c2.createEl("label",{text:"Descrição",attr:{style:lS}});
const inputDesc=c2.createEl("input",{attr:{type:"text",placeholder:"ex: Almoço, Salário...",style:fS}});

const c3=fGrid.createEl("div"); c3.createEl("label",{text:"Tipo",attr:{style:lS}});
const selTipo=c3.createEl("select",{attr:{style:fS}});
selTipo.createEl("option",{text:"Despesa",attr:{value:"despesa"}});
selTipo.createEl("option",{text:"Receita",attr:{value:"receita"}});

const c4=fGrid.createEl("div"); c4.createEl("label",{text:"Categoria",attr:{style:lS}});
const selCat=c4.createEl("select",{attr:{style:fS}});
for (const cat of CATEGORIAS) selCat.createEl("option",{text:cat,attr:{value:cat}});

const c5=fGrid.createEl("div"); c5.createEl("label",{text:"Valor (R$)",attr:{style:lS}});
const inputValor=c5.createEl("input",{attr:{type:"number",step:"0.01",min:"0",placeholder:"0,00",style:fS}});

const fFooter = sepForm.createEl("div",{attr:{style:"display:flex;align-items:center;gap:12px;margin-top:12px;"}});
const fbk      = fFooter.createEl("span",{attr:{style:"font-size:12px;color:var(--text-faint);flex:1;"}});
const btnSalvar = fFooter.createEl("button",{text:"Salvar transação",attr:{style:"background:var(--interactive-accent);color:var(--text-on-accent);border:none;border-radius:8px;padding:9px 18px;font-size:13px;font-weight:600;cursor:pointer;"}});

btnSalvar.addEventListener("click",async()=>{
  const data=inputData.value.trim(),desc=inputDesc.value.trim(),tipo=selTipo.value,cat=selCat.value,vRaw=inputValor.value.trim();
  if(!data){fbk.textContent="Informe a data.";fbk.style.color="var(--text-error)";return;}
  if(!desc){fbk.textContent="Informe a descrição.";fbk.style.color="var(--text-error)";return;}
  if(!vRaw){fbk.textContent="Informe o valor.";fbk.style.color="var(--text-error)";return;}
  const valor=parseFloat(vRaw);
  if(isNaN(valor)||valor<=0){fbk.textContent="Valor inválido.";fbk.style.color="var(--text-error)";return;}
  btnSalvar.disabled=true; fbk.textContent="Salvando..."; fbk.style.color="var(--text-faint)";
  try {
    let regs=[];
    try{regs=JSON.parse(await app.vault.adapter.read(PATH_TX));}catch(e){}
    regs.push({data,descricao:desc,tipo,categoria:cat,valor:parseFloat(valor.toFixed(2))});
    regs.sort((a,b)=>new Date(b.data)-new Date(a.data));
    await app.vault.adapter.write(PATH_TX,JSON.stringify(regs,null,2));
    fbk.textContent=`✓ ${tipo==="receita"?"Receita":"Despesa"} de ${brl(valor)} registrada!`;
    fbk.style.color="#1D9E75";
    inputDesc.value=""; inputValor.value=""; inputData.value=dataHoje;
    setTimeout(()=>dv.index.touch(app.vault.getAbstractFileByPath(dv.currentFilePath)),1500);
  } catch(err){fbk.textContent=`Erro: ${err.message}`;fbk.style.color="var(--text-error)";btnSalvar.disabled=false;}
});

// ================================================================
//  BLOCO DE ANOTAÇÕES
// ================================================================
const sepNotas = root.createEl("div",{attr:{style:"margin-top:28px;border-top:1px solid var(--background-modifier-border);padding-top:20px;"}});
sepNotas.createEl("p",{text:"Anotações",attr:{style:"font-size:11px;font-weight:600;letter-spacing:.08em;text-transform:uppercase;color:var(--text-faint);margin:0 0 10px;"}});

const textarea = sepNotas.createEl("textarea",{
  attr:{
    placeholder:"Use este espaço para anotações, lembretes financeiros, metas de longo prazo...",
    style:"width:100%;min-height:120px;padding:12px;border-radius:10px;border:1px solid var(--background-modifier-border);background:var(--background-secondary);color:var(--text-normal);font-size:13px;font-family:var(--font-interface);line-height:1.6;resize:vertical;box-sizing:border-box;"
  }
});
textarea.value = anotacoes;

const notasFooter = sepNotas.createEl("div",{attr:{style:"display:flex;align-items:center;gap:10px;margin-top:8px;"}});
const notasFbk    = notasFooter.createEl("span",{attr:{style:"font-size:12px;color:var(--text-faint);flex:1;"}});
const btnSalvarNotas = notasFooter.createEl("button",{text:"Salvar anotações",attr:{style:"padding:7px 16px;border:1px solid var(--background-modifier-border);border-radius:8px;background:var(--background-secondary);color:var(--text-normal);font-size:12px;font-weight:600;cursor:pointer;"}});

btnSalvarNotas.addEventListener("click",async()=>{
  try {
    await salvarAnotacoes(textarea.value);
    notasFbk.textContent="✓ Salvo!"; notasFbk.style.color="#1D9E75";
    setTimeout(()=>{notasFbk.textContent="";},2000);
  } catch(err) {
    notasFbk.textContent=`Erro: ${err.message}`; notasFbk.style.color="var(--text-error)";
  }
});
```

---

## Arquivos JSON necessários

### `Finanças/transacoes.json`

```json
[]
```

### `Finanças/metas.json`

```json
{
  "Alimentação":  800,
  "Transporte":   300,
  "Moradia":     1500,
  "Lazer":        400,
  "Saúde":        200,
  "Assinaturas":  150
}
```

### `Finanças/anotacoes.json`

```json
{ "texto": "" }
```