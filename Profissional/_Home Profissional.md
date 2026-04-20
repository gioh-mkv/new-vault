---
## tags: [homepage, profissional] cssclasses: [home-page]
---
```dataviewjs
// HOME PROFISSIONAL - DataviewJS
// Acompanhamento de freelances: status, ganhos, registro e anotacoes

// ---------- HELPERS ----------
const hoje     = new Date();
const pad      = n => String(n).padStart(2, "0");
const dataHoje = hoje.getFullYear()+"-"+pad(hoje.getMonth()+1)+"-"+pad(hoje.getDate());
const mesAtual = hoje.getMonth();
const anoAtual = hoje.getFullYear();
const MESES    = ["Janeiro","Fevereiro","Marco","Abril","Maio","Junho","Julho","Agosto","Setembro","Outubro","Novembro","Dezembro"];
const mesesAbrev = ["jan","fev","mar","abr","mai","jun","jul","ago","set","out","nov","dez"];

function toDate(val) {
  if (!val) return null;
  if (val instanceof Date) return val;
  if (val && val.toJSDate) return val.toJSDate();
  return new Date(String(val));
}
function ilink(path) {
  return 'data-href="'+path+'" href="'+path+'" class="internal-link"';
}
function fmtData(val) {
  const d = toDate(val);
  if (!d || isNaN(d)) return "-";
  return d.getDate()+" "+mesesAbrev[d.getMonth()]+" "+d.getFullYear();
}
function diasAte(val) {
  const d = toDate(val);
  if (!d || isNaN(d)) return null;
  return Math.ceil((d - hoje) / 86400000);
}
function brl(v) {
  return "R$ "+Number(v||0).toLocaleString("pt-BR",{minimumFractionDigits:2,maximumFractionDigits:2});
}

// ---------- STATUS CONFIG ----------
const STATUS_CFG = {
  "prospecto":    { bg:"#F1EFE8", cor:"#5F5E5A", label:"Prospecto"   },
  "negociacao":   { bg:"#FAEEDA", cor:"#854F0B", label:"Negociacao"  },
  "em andamento": { bg:"#E6F1FB", cor:"#185FA5", label:"Em andamento"},
  "revisao":      { bg:"#EEEDFE", cor:"#534AB7", label:"Revisao"     },
  "concluido":    { bg:"#E1F5EE", cor:"#085041", label:"Concluido"   },
  "cancelado":    { bg:"#FCEBEB", cor:"#A32D2D", label:"Cancelado"   },
  "pausado":      { bg:"#F1EFE8", cor:"#5F5E5A", label:"Pausado"     },
};
const STATUS_LISTA = [
  ["prospecto","Prospecto"],["negociacao","Negociacao"],
  ["em andamento","Em andamento"],["revisao","Revisao"],
  ["concluido","Concluido"],["cancelado","Cancelado"],["pausado","Pausado"]
];

function badgeStatus(status) {
  const key = (status||"").toLowerCase();
  const cfg = STATUS_CFG[key] || { bg:"#F1EFE8", cor:"#5F5E5A", label:status||"-" };
  return '<span style="font-size:10px;padding:2px 8px;border-radius:4px;font-weight:600;background:'+cfg.bg+';color:'+cfg.cor+';">'+cfg.label+'</span>';
}

// ---------- DADOS ----------
const todos = dv.pages('"Profissional/Freelances"')
  .where(p => p.file.name !== "_Home Profissional")
  .sort(p => p.file.mtime, "desc");

function valorNum(p) { return parseFloat(p.valor||0)||0; }

const ativos    = todos.where(p => ["em andamento","revisao"].includes((p.status||"").toLowerCase()));
const prospects = todos.where(p => ["prospecto","negociacao"].includes((p.status||"").toLowerCase()));
const concluidos = todos.where(p => (p.status||"").toLowerCase() === "concluido");
const cancelados = todos.where(p => (p.status||"").toLowerCase() === "cancelado");

const ganhoTotal  = concluidos.array().reduce((s,p)=>s+valorNum(p),0);
const ganhoMes    = concluidos.where(p=>{
  const d=toDate(p.data_fim);
  return d && d.getMonth()===mesAtual && d.getFullYear()===anoAtual;
}).array().reduce((s,p)=>s+valorNum(p),0);
const valorAberto = ativos.array().reduce((s,p)=>s+valorNum(p),0);

const ganhosPorMes = MESES.map((_,i)=>
  concluidos.where(p=>{
    const d=toDate(p.data_fim);
    return d && d.getMonth()===i && d.getFullYear()===anoAtual;
  }).array().reduce((s,p)=>s+valorNum(p),0)
);

// ---------- GRAFICO DE BARRAS ----------
function graficoGanhos(dados) {
  const W=580, H=140;
  const PAD={top:10,right:10,bottom:24,left:50};
  const iW=W-PAD.left-PAD.right, iH=H-PAD.top-PAD.bottom;
  const maxVal=Math.max(...dados,1);
  const barW=Math.floor(iW/12)-4;
  let svg='<svg width="100%" viewBox="0 0 '+W+' '+H+'" style="overflow:visible;">';
  for (let i=0;i<=4;i++){
    const v=maxVal/4*i, y=PAD.top+iH-(v/maxVal)*iH;
    svg+='<line x1="'+PAD.left+'" y1="'+y+'" x2="'+(W-PAD.right)+'" y2="'+y+'" stroke="var(--background-modifier-border)" stroke-width="0.5"/>';
    if(i>0) svg+='<text x="'+(PAD.left-4)+'" y="'+(y+4)+'" text-anchor="end" style="font-size:8px;fill:var(--text-faint);font-family:var(--font-interface);">'+(v/1000).toFixed(0)+'k</text>';
  }
  dados.forEach((val,i)=>{
    const xBase=PAD.left+i*(barW+4)+2;
    const barH=val>0?(val/maxVal)*iH:0;
    const dest=i===mesAtual;
    svg+='<rect x="'+xBase+'" y="'+(PAD.top+iH-barH)+'" width="'+barW+'" height="'+barH+'" fill="'+(dest?"#1D9E75":"#9FE1CB")+'" rx="2"/>';
    svg+='<text x="'+(xBase+barW/2)+'" y="'+(H-PAD.bottom+14)+'" text-anchor="middle" style="font-size:8px;fill:'+(dest?'var(--text-normal)':'var(--text-faint)')+';font-family:var(--font-interface);font-weight:'+(dest?'600':'400')+';">'+mesesAbrev[i]+'</text>';
  });
  return svg+'</svg>';
}

// ---------- BUILD HTML ----------
let h = '<div style="max-width:860px;">';

h += '<div style="padding-bottom:16px;border-bottom:1px solid var(--background-modifier-border);margin-bottom:24px;display:flex;align-items:flex-start;justify-content:space-between;">';
h += '<div><div style="font-size:22px;font-weight:700;color:var(--text-normal);">Profissional</div>';
h += '<div style="font-size:13px;color:var(--text-faint);margin-top:4px;">'+dataHoje+'</div></div>';
h += '<a '+ilink("Home")+' style="font-size:12px;color:var(--interactive-accent);text-decoration:none;margin-top:4px;">voltar para Home</a>';
h += '</div>';

// Cards
const resumos = [
  { lbl:"Ganhos no mes",  val:brl(ganhoMes),    cor:"#1D9E75" },
  { lbl:"Total ganho",    val:brl(ganhoTotal),  cor:"#1D9E75" },
  { lbl:"Em aberto",      val:brl(valorAberto), cor:"#185FA5" },
  { lbl:"Em andamento",   val:String(ativos.length), cor:"#185FA5" },
];
h += '<div style="display:grid;grid-template-columns:repeat(4,minmax(0,1fr));gap:10px;margin-bottom:28px;">';
for (const r of resumos) {
  h += '<div style="background:var(--background-secondary);border-radius:10px;padding:14px;">';
  h += '<div style="font-size:11px;color:var(--text-faint);margin-bottom:6px;text-transform:uppercase;letter-spacing:.06em;">'+r.lbl+'</div>';
  h += '<div style="font-size:18px;font-weight:700;color:'+r.cor+';line-height:1.2;">'+r.val+'</div>';
  h += '</div>';
}
h += '</div>';

// Grafico
h += '<div style="border:1px solid var(--background-modifier-border);border-radius:10px;padding:16px;margin-bottom:28px;">';
h += '<div style="display:flex;align-items:center;justify-content:space-between;margin-bottom:12px;">';
h += '<span style="font-size:12px;font-weight:600;color:var(--text-muted);">Ganhos mensais '+anoAtual+'</span>';
h += '<span style="font-size:11px;color:var(--text-faint);">apenas concluidos</span>';
h += '</div>'+graficoGanhos(ganhosPorMes)+'</div>';

// Listas
const grupos = [
  { label:"Em andamento", lista: ativos },
  { label:"Prospectos",   lista: prospects },
  { label:"Concluidos",   lista: concluidos.sort(p=>toDate(p.data_fim),"desc").limit(10) },
  { label:"Cancelados",   lista: cancelados },
];

for (const grupo of grupos) {
  if (grupo.lista.length === 0) continue;
  h += '<p style="font-size:11px;font-weight:600;color:var(--text-faint);text-transform:uppercase;letter-spacing:.06em;margin:0 0 10px;">'+grupo.label+'</p>';
  h += '<div style="border:1px solid var(--background-modifier-border);border-radius:10px;overflow:hidden;margin-bottom:24px;">';

  for (const p of grupo.lista) {
    const cliente = p.cliente || p.file.name;
    const desc    = p.descricao || "";
    const valor   = valorNum(p);
    const prazo   = p.prazo || null;
    const n       = prazo ? diasAte(prazo) : null;
    const urgente = n !== null && n >= 0 && n <= 7;

    h += '<a '+ilink(p.file.path)+' style="display:flex;align-items:flex-start;gap:12px;padding:12px 14px;border-bottom:1px solid var(--background-modifier-border);text-decoration:none;'+(urgente?'border-left:3px solid #E24B4A;':'')+'">';
    h += '<div style="flex:1;min-width:0;">';
    h += '<div style="display:flex;align-items:center;gap:8px;margin-bottom:4px;flex-wrap:wrap;">';
    h += '<span style="font-size:13px;font-weight:600;color:var(--text-normal);">'+cliente+'</span>';
    h += badgeStatus(p.status);
    h += '</div>';
    if (desc) h += '<div style="font-size:12px;color:var(--text-faint);line-height:1.4;margin-bottom:4px;">'+desc+'</div>';
    h += '<div style="display:flex;gap:14px;flex-wrap:wrap;">';
    if (p.data_inicio) h += '<span style="font-size:11px;color:var(--text-faint);">Inicio: '+fmtData(p.data_inicio)+'</span>';
    if (p.data_fim)    h += '<span style="font-size:11px;color:var(--text-faint);">Fim: '+fmtData(p.data_fim)+'</span>';
    if (prazo && n !== null) {
      const prazoStr = n===0?"hoje":n===1?"amanha":n<0?"encerrado":"em "+n+" dias";
      h += '<span style="font-size:11px;color:'+(urgente?'#E24B4A':'var(--text-faint)')+';">Prazo: '+prazoStr+'</span>';
    }
    h += '</div></div>';
    if (valor > 0) {
      h += '<div style="text-align:right;flex-shrink:0;">';
      h += '<div style="font-size:14px;font-weight:700;color:'+((p.status||"")==="concluido"?"#1D9E75":"var(--text-normal)")+';">'+brl(valor)+'</div>';
      h += '</div>';
    }
    h += '</a>';
  }
  h += '</div>';
}

if (todos.length === 0) {
  h += '<div style="border:1px dashed var(--background-modifier-border);border-radius:10px;padding:32px;text-align:center;margin-bottom:24px;">';
  h += '<div style="font-size:14px;font-weight:600;color:var(--text-normal);margin-bottom:6px;">Nenhum trabalho cadastrado</div>';
  h += '<div style="font-size:13px;color:var(--text-faint);">Use o formulario abaixo para registrar seu primeiro freelance.</div>';
  h += '</div>';
}

h += '</div>';
dv.el("div", h);

// ================================================================
// PAINEL INTERATIVO
// ================================================================
const root = dv.el("div","",{attr:{style:"max-width:860px;"}});

// -- ATUALIZAR STATUS --
const secStatus = root.createEl("div",{attr:{style:"border-top:1px solid var(--background-modifier-border);padding-top:20px;margin-top:4px;margin-bottom:28px;"}});
secStatus.createEl("p",{text:"Atualizar status",attr:{style:"font-size:11px;font-weight:600;letter-spacing:.08em;text-transform:uppercase;color:var(--text-faint);margin:0 0 12px;"}});

if (todos.length > 0) {
  const fS2 = "width:100%;padding:7px 10px;border-radius:7px;border:1px solid var(--background-modifier-border);background:var(--background-primary);color:var(--text-normal);font-size:13px;";
  const lS2 = "font-size:11px;color:var(--text-faint);display:block;margin-bottom:4px;";
  const statusGrid = secStatus.createEl("div",{attr:{style:"display:grid;grid-template-columns:1fr 180px 140px;gap:10px;align-items:end;"}});

  const cSel = statusGrid.createEl("div");
  cSel.createEl("label",{text:"Trabalho",attr:{style:lS2}});
  const selTrabalho = cSel.createEl("select",{attr:{style:fS2}});
  for (const p of todos) {
    selTrabalho.createEl("option",{text:(p.cliente||p.file.name)+" ("+(p.status||"-")+")",attr:{value:p.file.path}});
  }

  const cNS = statusGrid.createEl("div");
  cNS.createEl("label",{text:"Novo status",attr:{style:lS2}});
  const selNovoStatus = cNS.createEl("select",{attr:{style:fS2}});
  for (const [val,lbl] of STATUS_LISTA) selNovoStatus.createEl("option",{text:lbl,attr:{value:val}});

  const cBtn = statusGrid.createEl("div");
  const fbkStatus = secStatus.createEl("span",{attr:{style:"font-size:12px;color:var(--text-faint);display:block;margin-top:8px;"}});
  const btnAtualizar = cBtn.createEl("button",{text:"Atualizar",attr:{style:"width:100%;padding:8px 14px;border:none;border-radius:8px;background:var(--interactive-accent);color:var(--text-on-accent);font-size:13px;font-weight:600;cursor:pointer;margin-top:20px;"}});

  btnAtualizar.addEventListener("click", async () => {
    const caminho    = selTrabalho.value;
    const novoStatus = selNovoStatus.value;
    if (!caminho) return;
    btnAtualizar.disabled = true;
    fbkStatus.textContent = "Atualizando...";
    fbkStatus.style.color = "var(--text-faint)";
    try {
      const file = app.vault.getAbstractFileByPath(caminho);
      if (!file) throw new Error("Arquivo nao encontrado.");
      await app.fileManager.processFrontMatter(file, fm => {
        fm.status = novoStatus;
        if (novoStatus==="concluido" && !fm.data_fim) fm.data_fim = dataHoje;
        if (novoStatus==="em andamento" && !fm.data_inicio) fm.data_inicio = dataHoje;
      });
      fbkStatus.textContent = "Status atualizado: "+novoStatus;
      fbkStatus.style.color = "#1D9E75";
      setTimeout(()=>dv.index.touch(app.vault.getAbstractFileByPath(dv.currentFilePath)),1000);
    } catch(err) {
      fbkStatus.textContent = "Erro: "+err.message;
      fbkStatus.style.color = "var(--text-error)";
      btnAtualizar.disabled = false;
    }
  });
} else {
  secStatus.createEl("div",{text:"Nenhum trabalho para atualizar.",attr:{style:"font-size:12px;color:var(--text-faint);font-style:italic;"}});
}

// -- FORMULARIO NOVO FREELANCE --
const sepForm = root.createEl("div",{attr:{style:"border-top:1px solid var(--background-modifier-border);padding-top:20px;margin-bottom:28px;"}});
const formHeader = sepForm.createEl("div",{attr:{style:"display:flex;align-items:center;justify-content:space-between;"}});
formHeader.createEl("p",{text:"Novo trabalho",attr:{style:"font-size:11px;font-weight:600;letter-spacing:.08em;text-transform:uppercase;color:var(--text-faint);margin:0;"}});
const btnToggle = formHeader.createEl("button",{text:"+ Abrir formulario",attr:{style:"padding:6px 14px;border:1px solid var(--background-modifier-border);border-radius:7px;background:var(--background-secondary);color:var(--text-muted);font-size:12px;cursor:pointer;"}});
const formBody = sepForm.createEl("div",{attr:{style:"display:none;margin-top:14px;"}});
btnToggle.addEventListener("click",()=>{
  const ab=formBody.style.display!=="none";
  formBody.style.display=ab?"none":"block";
  btnToggle.textContent=ab?"+ Abrir formulario":"- Fechar";
});

const fS = "width:100%;padding:7px 10px;border-radius:7px;border:1px solid var(--background-modifier-border);background:var(--background-primary);color:var(--text-normal);font-size:13px;";
const lS = "font-size:11px;color:var(--text-faint);display:block;margin-bottom:4px;";

const l1 = formBody.createEl("div",{attr:{style:"display:grid;grid-template-columns:1fr 160px 140px;gap:10px;margin-bottom:10px;"}});
const cCli=l1.createEl("div"); cCli.createEl("label",{text:"Cliente",attr:{style:lS}});
const inputCliente=cCli.createEl("input",{attr:{type:"text",placeholder:"Nome do cliente ou empresa",style:fS}});
const cSt=l1.createEl("div"); cSt.createEl("label",{text:"Status inicial",attr:{style:lS}});
const selSt=cSt.createEl("select",{attr:{style:fS}});
for(const[val,lbl]of STATUS_LISTA) selSt.createEl("option",{text:lbl,attr:{value:val}});
const cVal=l1.createEl("div"); cVal.createEl("label",{text:"Valor (R$)",attr:{style:lS}});
const inputValor=cVal.createEl("input",{attr:{type:"number",min:"0",step:"0.01",placeholder:"0,00",style:fS}});

const lD=formBody.createEl("div",{attr:{style:"margin-bottom:10px;"}});
lD.createEl("label",{text:"Descricao do trabalho",attr:{style:lS}});
const inputDesc=lD.createEl("textarea",{attr:{placeholder:"Descreva o escopo do trabalho...",style:fS+"min-height:60px;resize:vertical;line-height:1.5;"}});

const l3=formBody.createEl("div",{attr:{style:"display:grid;grid-template-columns:1fr 1fr 1fr;gap:10px;margin-bottom:14px;"}});
const cDI=l3.createEl("div"); cDI.createEl("label",{text:"Data de inicio",attr:{style:lS}});
const inputDI=cDI.createEl("input",{attr:{type:"date",value:dataHoje,style:fS}});
const cDF=l3.createEl("div"); cDF.createEl("label",{text:"Previsao de entrega",attr:{style:lS}});
const inputDF=cDF.createEl("input",{attr:{type:"date",style:fS}});
const cPrazo=l3.createEl("div"); cPrazo.createEl("label",{text:"Prazo limite",attr:{style:lS}});
const inputPrazo=cPrazo.createEl("input",{attr:{type:"date",style:fS}});

const fFooter=formBody.createEl("div",{attr:{style:"display:flex;align-items:center;gap:12px;"}});
const fbk=fFooter.createEl("span",{attr:{style:"font-size:12px;color:var(--text-faint);flex:1;"}});
const btnSalvar=fFooter.createEl("button",{text:"Criar trabalho",attr:{style:"background:var(--interactive-accent);color:var(--text-on-accent);border:none;border-radius:8px;padding:9px 20px;font-size:13px;font-weight:600;cursor:pointer;"}});

btnSalvar.addEventListener("click", async()=>{
  const cliente=inputCliente.value.trim();
  if(!cliente){fbk.textContent="Informe o cliente.";fbk.style.color="var(--text-error)";return;}
  btnSalvar.disabled=true; fbk.textContent="Criando nota..."; fbk.style.color="var(--text-faint)";
  try {
    const valor=parseFloat(inputValor.value)||0;
    const linhas=[
      "---",
      'cliente: "'+cliente+'"',
      'descricao: "'+inputDesc.value.trim().replace(/"/g,"'")+'"',
      'status: '+selSt.value,
      'valor: '+valor.toFixed(2),
      'data_inicio: '+(inputDI.value||dataHoje),
      'data_fim: '+(inputDF.value||""),
      'prazo: '+(inputPrazo.value||""),
      'pendente: true',
      'setor: Profissional',
      'tags: [freelance]',
      "---","",
      "## "+cliente,"",
      "## Escopo","",
      "## Entregas","",
      "## Comunicacao","",
      "## Financeiro",
      "- Valor acordado: "+brl(valor),"",
      "## Notas","",
    ];
    const caminho="Profissional/Freelances/"+cliente+".md";
    if(app.vault.getAbstractFileByPath(caminho)){
      fbk.textContent='Ja existe um trabalho para "'+cliente+'".';
      fbk.style.color="var(--text-error)"; btnSalvar.disabled=false; return;
    }
    await app.vault.create(caminho,linhas.join("\n"));
    fbk.textContent='Trabalho "'+cliente+'" criado!'; fbk.style.color="#1D9E75";
    inputCliente.value=""; inputDesc.value=""; inputValor.value="";
    inputDI.value=dataHoje; inputDF.value=""; inputPrazo.value="";
    setTimeout(async()=>{
      await app.workspace.openLinkText(caminho,"",false);
      dv.index.touch(app.vault.getAbstractFileByPath(dv.currentFilePath));
    },1000);
  } catch(err){fbk.textContent="Erro: "+err.message;fbk.style.color="var(--text-error)";btnSalvar.disabled=false;}
});

// -- ANOTACOES --
const PATH_NOTAS="Profissional/anotacoes.json";
let anotTxt="";
try{const raw=await app.vault.adapter.read(PATH_NOTAS);anotTxt=JSON.parse(raw).texto||"";}catch(e){}

const sepNotas=root.createEl("div",{attr:{style:"border-top:1px solid var(--background-modifier-border);padding-top:20px;"}});
sepNotas.createEl("p",{text:"Anotacoes",attr:{style:"font-size:11px;font-weight:600;letter-spacing:.08em;text-transform:uppercase;color:var(--text-faint);margin:0 0 10px;"}});
const textarea=sepNotas.createEl("textarea",{attr:{placeholder:"Contatos, tarifas de referencia, ideias de prospeccao...",style:"width:100%;min-height:100px;padding:12px;border-radius:10px;border:1px solid var(--background-modifier-border);background:var(--background-secondary);color:var(--text-normal);font-size:13px;font-family:var(--font-interface);line-height:1.6;resize:vertical;box-sizing:border-box;"}});
textarea.value=anotTxt;
const nFooter=sepNotas.createEl("div",{attr:{style:"display:flex;align-items:center;gap:10px;margin-top:8px;"}});
const nFbk=nFooter.createEl("span",{attr:{style:"font-size:12px;color:var(--text-faint);flex:1;"}});
const btnSalvarN=nFooter.createEl("button",{text:"Salvar anotacoes",attr:{style:"padding:7px 16px;border:1px solid var(--background-modifier-border);border-radius:8px;background:var(--background-secondary);color:var(--text-normal);font-size:12px;font-weight:600;cursor:pointer;"}});
btnSalvarN.addEventListener("click",async()=>{
  try{
    await app.vault.adapter.write(PATH_NOTAS,JSON.stringify({texto:textarea.value},null,2));
    nFbk.textContent="Salvo!"; nFbk.style.color="#1D9E75";
    setTimeout(()=>{nFbk.textContent="";},2000);
  }catch(err){nFbk.textContent="Erro: "+err.message;nFbk.style.color="var(--text-error)";}
});
```

---

## Frontmatter da nota de freelance

```yaml
---
cliente: "Nome do Cliente"
descricao: "Descricao do trabalho"
status: em andamento
valor: 1500.00
data_inicio: 2025-04-01
data_fim: 
prazo: 2025-05-01
pendente: true
setor: Profissional
tags: [freelance]
---
```

### Status disponiveis

- `prospecto` - lead ainda nao confirmado
- `negociacao` - em processo de fechamento
- `em andamento` - trabalho em execucao
- `revisao` - aguardando aprovacao do cliente
- `concluido` - entregue e pago
- `cancelado` - nao realizado
- `pausado` - temporariamente interrompido

### Arquivos necessarios

- `Profissional/anotacoes.json` com `{ "texto": "" }`