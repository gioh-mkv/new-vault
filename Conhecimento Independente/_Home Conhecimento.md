---
## tags: [homepage, conhecimento] cssclasses: [home-page]
---

```dataviewjs
// ============================================================
//  HOME CONHECIMENTO INDEPENDENTE — DataviewJS
//  Abas via createEl + addEventListener (sem <script> inline)
// ============================================================

// ---------- HELPERS ----------
const hoje     = new Date();
const pad      = n => String(n).padStart(2, "0");
const dataHoje = `${hoje.getFullYear()}-${pad(hoje.getMonth()+1)}-${pad(hoje.getDate())}`;

function toDate(val) {
  if (!val) return null;
  if (val instanceof Date) return val;
  if (val && val.toJSDate) return val.toJSDate();
  return new Date(String(val));
}
function ilink(path) {
  return `data-href="${path}" href="${path}" class="internal-link"`;
}
function fmtData(val) {
  const d = toDate(val);
  if (!d || isNaN(d)) return "—";
  return `${d.getDate()}/${pad(d.getMonth()+1)}/${d.getFullYear()}`;
}
function diasAte(val) {
  const d = toDate(val);
  if (!d || isNaN(d)) return null;
  return Math.ceil((d - hoje) / 86400000);
}
function badge(texto, bg, cor) {
  return `<span style="font-size:10px;padding:2px 7px;border-radius:4px;font-weight:600;background:${bg};color:${cor};">${texto}</span>`;
}
function barraProgresso(valor, cor) {
  const pct = Math.min(100, Math.max(0, valor || 0));
  return `<div style="margin-top:8px;">
    <div style="display:flex;justify-content:space-between;margin-bottom:3px;">
      <span style="font-size:10px;color:var(--text-faint);">Progresso</span>
      <span style="font-size:10px;color:var(--text-faint);">${pct}%</span>
    </div>
    <div style="height:3px;background:var(--background-modifier-border);border-radius:2px;">
      <div style="height:3px;width:${pct}%;background:${cor};border-radius:2px;"></div>
    </div>
  </div>`;
}

// ---------- STATUS BADGES ----------
const STATUS_CONCURSO = {
  "inscricoes abertas": { bg:"#E1F5EE", cor:"#085041" },
  "em andamento":       { bg:"#E6F1FB", cor:"#185FA5" },
  "aguardando edital":  { bg:"#FAEEDA", cor:"#854F0B" },
  "encerrado":          { bg:"#F1EFE8", cor:"#5F5E5A" },
  "resultado":          { bg:"#EEEDFE", cor:"#534AB7" },
};
const STATUS_CURSO = {
  "em andamento": { bg:"#E6F1FB", cor:"#185FA5" },
  "concluido":    { bg:"#E1F5EE", cor:"#085041" },
  "concluído":    { bg:"#E1F5EE", cor:"#085041" },
  "pausado":      { bg:"#FAEEDA", cor:"#854F0B" },
  "planejado":    { bg:"#F1EFE8", cor:"#5F5E5A" },
};
const STATUS_ESTUDO = {
  "ativo":      { bg:"#E6F1FB", cor:"#185FA5" },
  "concluido":  { bg:"#E1F5EE", cor:"#085041" },
  "concluído":  { bg:"#E1F5EE", cor:"#085041" },
  "pausado":    { bg:"#FAEEDA", cor:"#854F0B" },
  "planejado":  { bg:"#F1EFE8", cor:"#5F5E5A" },
};

function badgeStatus(status, mapa) {
  const key = (status || "").toLowerCase();
  const cfg = mapa[key] || { bg:"#F1EFE8", cor:"#5F5E5A" };
  const label = status ? status.charAt(0).toUpperCase() + status.slice(1) : "—";
  return badge(label, cfg.bg, cfg.cor);
}

// ---------- DADOS ----------
const concursos = dv.pages('"Conhecimento Independente/Concursos Públicos"').sort(p => p.file.mtime, "desc");
const cursos    = dv.pages('"Conhecimento Independente/Cursos Profissionalizantes"').sort(p => p.file.mtime, "desc");
const estudos   = dv.pages('"Conhecimento Independente/Estudos Independentes"').sort(p => p.file.mtime, "desc");
const areas     = dv.pages('"Conhecimento Independente/Áreas de Conhecimento"').sort(p => p.file.name, "asc");

const concursosAtivos = concursos.where(p => ["inscricoes abertas","em andamento"].includes((p.status||"").toLowerCase())).length;
const cursosAtivos    = cursos.where(p => (p.status||"").toLowerCase() === "em andamento").length;
const estudosAtivos   = estudos.where(p => (p.status||"").toLowerCase() === "ativo").length;

// ---------- HTML DOS PAINÉIS ----------
function htmlConcursos() {
  if (concursos.length === 0) return `<div style="border:1px dashed var(--background-modifier-border);border-radius:10px;padding:24px;text-align:center;"><div style="font-size:13px;color:var(--text-faint);">Nenhum concurso cadastrado. Crie notas em <code>Conhecimento Independente/Concursos Públicos/</code></div></div>`;

  const grupos = [
    { label: "Em andamento",     lista: concursos.where(p => ["inscricoes abertas","em andamento"].includes((p.status||"").toLowerCase())) },
    { label: "Aguardando edital",lista: concursos.where(p => (p.status||"").toLowerCase() === "aguardando edital") },
    { label: "Outros",           lista: concursos.where(p => !["inscricoes abertas","em andamento","aguardando edital"].includes((p.status||"").toLowerCase())) },
  ];

  let out = "";
  for (const g of grupos) {
    if (g.lista.length === 0) continue;
    out += `<p style="font-size:11px;font-weight:600;color:var(--text-faint);text-transform:uppercase;letter-spacing:.06em;margin:0 0 8px;">${g.label}</p>`;
    for (const p of g.lista) {
      const prova   = p.data_prova || null;
      const nDias   = prova ? diasAte(prova) : null;
      const urgente = nDias !== null && nDias >= 0 && nDias <= 30;
      out += `
      <a ${ilink(p.file.path)} style="display:block;border:1px solid var(--background-modifier-border);${urgente?"border-left:3px solid #E24B4A;":""}border-radius:10px;padding:14px;margin-bottom:10px;text-decoration:none;background:var(--background-primary);">
        <div style="display:flex;justify-content:space-between;align-items:flex-start;gap:8px;margin-bottom:6px;">
          <div>
            <div style="font-size:13px;font-weight:600;color:var(--text-normal);">${p.orgao || p.file.name}</div>
            ${p.cargo ? `<div style="font-size:12px;color:var(--text-faint);margin-top:2px;">${p.cargo}</div>` : ""}
          </div>
          ${badgeStatus(p.status, STATUS_CONCURSO)}
        </div>
        <div style="display:flex;gap:14px;flex-wrap:wrap;margin-top:6px;">
          ${p.edital    ? `<span style="font-size:11px;color:var(--text-faint);">Edital: ${fmtData(p.edital)}</span>` : ""}
          ${prova       ? `<span style="font-size:11px;color:${urgente?"#E24B4A":"var(--text-faint)"};">Prova: ${fmtData(prova)}${nDias !== null && nDias >= 0 ? ` (${nDias===0?"hoje":nDias===1?"amanhã":`em ${nDias} dias`})` : ""}</span>` : ""}
          ${p.vagas     ? `<span style="font-size:11px;color:var(--text-faint);">${p.vagas} vagas</span>` : ""}
          ${p.salario   ? `<span style="font-size:11px;color:var(--text-faint);">R$ ${p.salario}</span>` : ""}
        </div>
        ${p.progresso != null ? barraProgresso(p.progresso, "#185FA5") : ""}
      </a>`;
    }
    out += `<div style="margin-bottom:16px;"></div>`;
  }
  return out;
}

function htmlCursos() {
  if (cursos.length === 0) return `<div style="border:1px dashed var(--background-modifier-border);border-radius:10px;padding:24px;text-align:center;"><div style="font-size:13px;color:var(--text-faint);">Nenhum curso cadastrado. Crie notas em <code>Conhecimento Independente/Cursos Profissionalizantes/</code></div></div>`;
  let out = `<div style="display:grid;grid-template-columns:repeat(2,minmax(0,1fr));gap:10px;">`;
  for (const p of cursos) {
    out += `
    <a ${ilink(p.file.path)} style="display:block;border:1px solid var(--background-modifier-border);border-radius:10px;padding:12px;text-decoration:none;background:var(--background-primary);">
      <div style="display:flex;justify-content:space-between;align-items:flex-start;gap:6px;margin-bottom:6px;">
        <div style="font-size:13px;font-weight:600;color:var(--text-normal);line-height:1.3;">${p.titulo || p.file.name}</div>
        ${badgeStatus(p.status, STATUS_CURSO)}
      </div>
      ${p.plataforma ? `<div style="font-size:11px;color:var(--text-faint);margin-bottom:2px;">${p.plataforma}</div>` : ""}
      ${p.instrutor  ? `<div style="font-size:11px;color:var(--text-faint);margin-bottom:2px;">${p.instrutor}</div>` : ""}
      <div style="display:flex;gap:10px;flex-wrap:wrap;margin-top:4px;">
        ${p.carga_horaria ? `<span style="font-size:11px;color:var(--text-muted);">${p.carga_horaria}h</span>` : ""}
        ${p.certificado === true ? badge("Certificado ✓","#E1F5EE","#085041") : ""}
      </div>
      ${p.progresso != null ? barraProgresso(p.progresso, "#378ADD") : ""}
    </a>`;
  }
  return out + `</div>`;
}

function htmlEstudos() {
  if (estudos.length === 0) return `<div style="border:1px dashed var(--background-modifier-border);border-radius:10px;padding:24px;text-align:center;"><div style="font-size:13px;color:var(--text-faint);">Nenhum estudo cadastrado. Crie notas em <code>Conhecimento Independente/Estudos Independentes/</code></div></div>`;
  let out = `<div style="display:grid;grid-template-columns:repeat(2,minmax(0,1fr));gap:10px;">`;
  for (const p of estudos) {
    const dd = toDate(p.ultima_revisao || p.file.mtime);
    const ddStr = dd ? `${dd.getDate()}/${pad(dd.getMonth()+1)}/${dd.getFullYear()}` : "—";
    out += `
    <a ${ilink(p.file.path)} style="display:block;border:1px solid var(--background-modifier-border);border-radius:10px;padding:12px;text-decoration:none;background:var(--background-primary);">
      <div style="display:flex;justify-content:space-between;align-items:flex-start;gap:6px;margin-bottom:6px;">
        <div style="font-size:13px;font-weight:600;color:var(--text-normal);line-height:1.3;">${p.topico || p.file.name}</div>
        ${badgeStatus(p.status, STATUS_ESTUDO)}
      </div>
      ${p.area ? `<div style="font-size:11px;color:var(--text-faint);margin-bottom:4px;">${p.area}</div>` : ""}
      <div style="display:flex;justify-content:space-between;align-items:center;margin-top:4px;">
        <span style="font-size:11px;color:var(--text-faint);">Atualizado: ${ddStr}</span>
        ${p.recursos ? `<span style="font-size:11px;color:var(--text-faint);">${p.recursos} recursos</span>` : ""}
      </div>
      ${p.progresso != null ? barraProgresso(p.progresso, "#854F0B") : ""}
    </a>`;
  }
  return out + `</div>`;
}

function htmlAreas() {
  if (areas.length === 0) return `<div style="border:1px dashed var(--background-modifier-border);border-radius:10px;padding:24px;text-align:center;"><div style="font-size:13px;color:var(--text-faint);">Nenhuma área cadastrada. Crie notas em <code>Conhecimento Independente/Áreas de Conhecimento/</code></div></div>`;
  const CORES = ["#534AB7","#0F6E56","#185FA5","#854F0B","#993C1D","#993556","#3B6D11"];
  const BGS   = ["#EEEDFE","#E1F5EE","#E6F1FB","#FAEEDA","#FAECE7","#FBEAF0","#EAF3DE"];
  let out = `<div style="display:grid;grid-template-columns:repeat(3,minmax(0,1fr));gap:10px;">`;
  areas.forEach((p, i) => {
    const cor = CORES[i % CORES.length];
    const bg  = BGS[i % BGS.length];
    out += `
    <a ${ilink(p.file.path)} style="display:block;border:1px solid var(--background-modifier-border);border-radius:10px;padding:14px;text-decoration:none;background:var(--background-primary);">
      <div style="width:32px;height:32px;border-radius:8px;background:${bg};color:${cor};display:flex;align-items:center;justify-content:center;font-size:15px;font-weight:700;margin-bottom:8px;">${p.file.name.charAt(0).toUpperCase()}</div>
      <div style="font-size:13px;font-weight:600;color:var(--text-normal);margin-bottom:4px;">${p.file.name}</div>
      ${p.descricao ? `<div style="font-size:11px;color:var(--text-faint);line-height:1.4;">${p.descricao}</div>` : ""}
      <div style="display:flex;gap:10px;margin-top:8px;flex-wrap:wrap;">
        ${p.subtopicos ? `<span style="font-size:11px;color:var(--text-faint);">${p.subtopicos} subtópicos</span>` : ""}
        ${p.recursos   ? `<span style="font-size:11px;color:var(--text-faint);">${p.recursos} recursos</span>` : ""}
      </div>
    </a>`;
  });
  return out + `</div>`;
}

// ---------- DEFINIÇÃO DAS ABAS ----------
const ABAS = [
  { id: "concursos", label: "Concursos",  count: concursos.length, html: htmlConcursos() },
  { id: "cursos",    label: "Cursos",     count: cursos.length,    html: htmlCursos()    },
  { id: "estudos",   label: "Estudos",    count: estudos.length,   html: htmlEstudos()   },
  { id: "areas",     label: "Áreas",      count: areas.length,     html: htmlAreas()     },
];

// ---------- BUILD HTML (cabeçalho + cards + painéis) ----------
let h = `<div style="max-width:860px;">`;

// Cabeçalho
h += `
<div style="padding-bottom:16px;border-bottom:1px solid var(--background-modifier-border);margin-bottom:24px;display:flex;align-items:flex-start;justify-content:space-between;">
  <div>
    <div style="font-size:22px;font-weight:700;color:var(--text-normal);">Conhecimento Independente</div>
    <div style="font-size:13px;color:var(--text-faint);margin-top:4px;">${dataHoje}</div>
  </div>
  <a ${ilink("Home")} style="font-size:12px;color:var(--interactive-accent);text-decoration:none;margin-top:4px;">← voltar para Home</a>
</div>`;

// Cards de resumo
h += `<div style="display:grid;grid-template-columns:repeat(4,minmax(0,1fr));gap:10px;margin-bottom:28px;">
<div style="background:var(--background-secondary);border-radius:10px;padding:14px;">
  <div style="font-size:11px;color:var(--text-faint);margin-bottom:6px;text-transform:uppercase;letter-spacing:.06em;">Concursos ativos</div>
  <div style="font-size:26px;font-weight:700;color:var(--text-normal);line-height:1;">${concursosAtivos}</div>
  <div style="font-size:11px;color:var(--text-faint);margin-top:4px;">de ${concursos.length} registrados</div>
</div>
<div style="background:var(--background-secondary);border-radius:10px;padding:14px;">
  <div style="font-size:11px;color:var(--text-faint);margin-bottom:6px;text-transform:uppercase;letter-spacing:.06em;">Cursos em andamento</div>
  <div style="font-size:26px;font-weight:700;color:var(--text-normal);line-height:1;">${cursosAtivos}</div>
  <div style="font-size:11px;color:var(--text-faint);margin-top:4px;">de ${cursos.length} registrados</div>
</div>
<div style="background:var(--background-secondary);border-radius:10px;padding:14px;">
  <div style="font-size:11px;color:var(--text-faint);margin-bottom:6px;text-transform:uppercase;letter-spacing:.06em;">Estudos ativos</div>
  <div style="font-size:26px;font-weight:700;color:var(--text-normal);line-height:1;">${estudosAtivos}</div>
  <div style="font-size:11px;color:var(--text-faint);margin-top:4px;">de ${estudos.length} registrados</div>
</div>
<div style="background:var(--background-secondary);border-radius:10px;padding:14px;">
  <div style="font-size:11px;color:var(--text-faint);margin-bottom:6px;text-transform:uppercase;letter-spacing:.06em;">Áreas mapeadas</div>
  <div style="font-size:26px;font-weight:700;color:var(--text-normal);line-height:1;">${areas.length}</div>
  <div style="font-size:11px;color:var(--text-faint);margin-top:4px;">de conhecimento</div>
</div>
</div>`;

h += `</div>`; // fecha max-width div
dv.el("div", h);

// ---------- ABAS VIA createEl (único jeito confiável no Obsidian) ----------
const root = dv.el("div", "", { attr: { style: "max-width:860px;" } });

// Barra de abas
const tabBar = root.createEl("div", {
  attr: { style: "display:flex;gap:4px;border-bottom:1px solid var(--background-modifier-border);margin-bottom:20px;" }
});

// Painéis
const paineis = {};
const tabBtns = {};

for (const aba of ABAS) {
  // Botão da aba
  const btn = tabBar.createEl("button", {
    attr: {
      style: `padding:8px 16px;border:none;border-bottom:2px solid transparent;background:transparent;color:var(--text-muted);font-size:13px;font-weight:400;cursor:pointer;border-radius:0;display:inline-flex;align-items:center;gap:6px;`
    }
  });
  btn.createSpan({ text: aba.label });
  const pill = btn.createEl("span", {
    text: String(aba.count),
    attr: { style: "font-size:10px;padding:1px 6px;border-radius:10px;background:var(--background-modifier-border);color:var(--text-faint);" }
  });
  tabBtns[aba.id] = btn;

  // Painel da aba
  const painel = root.createEl("div", {
    attr: { style: "display:none;" }
  });
  painel.innerHTML = aba.html;
  paineis[aba.id] = painel;
}

// Função de troca de aba
function mostrarAba(idAtivo) {
  for (const aba of ABAS) {
    const ativo = aba.id === idAtivo;
    paineis[aba.id].style.display = ativo ? "block" : "none";
    tabBtns[aba.id].style.borderBottom = ativo
      ? "2px solid var(--interactive-accent)"
      : "2px solid transparent";
    tabBtns[aba.id].style.color      = ativo ? "var(--interactive-accent)" : "var(--text-muted)";
    tabBtns[aba.id].style.fontWeight = ativo ? "600" : "400";
  }
}

// Ativa a primeira aba e adiciona listeners
mostrarAba(ABAS[0].id);
for (const aba of ABAS) {
  tabBtns[aba.id].addEventListener("click", () => mostrarAba(aba.id));
}

// ---------- BOTÕES DE NOVA NOTA ----------
const botoesWrap = root.createEl("div", {
  attr: { style: "margin-top:20px;display:flex;gap:10px;flex-wrap:wrap;" }
});

const botoes = [
  { label: "+ Novo concurso", path: "Conhecimento Independente/Concursos Públicos/Novo Concurso",          primario: true  },
  { label: "+ Novo curso",    path: "Conhecimento Independente/Cursos Profissionalizantes/Novo Curso",      primario: false },
  { label: "+ Novo estudo",   path: "Conhecimento Independente/Estudos Independentes/Novo Estudo",          primario: false },
  { label: "+ Nova área",     path: "Conhecimento Independente/Áreas de Conhecimento/Nova Área",            primario: false },
];

for (const b of botoes) {
  botoesWrap.createEl("a", {
    text: b.label,
    attr: {
      "data-href": b.path,
      "href":      b.path,
      "class":     "internal-link",
      "style":     b.primario
        ? "display:inline-flex;align-items:center;border-radius:8px;padding:8px 14px;font-size:12px;font-weight:600;text-decoration:none;background:var(--interactive-accent);color:var(--text-on-accent);"
        : "display:inline-flex;align-items:center;border-radius:8px;padding:8px 14px;font-size:12px;font-weight:600;text-decoration:none;background:var(--background-secondary);color:var(--text-normal);border:1px solid var(--background-modifier-border);"
    }
  });
}
```

---

## Templates para o Templater

### `_Templates/Concurso.md`

```yaml
---
orgao: 
cargo: 
status: aguardando edital
edital: 
data_prova: 
vagas: 
salario: 
progresso: 0
tags: [conhecimento, concurso]
pendente: true
setor: Conhecimento
---
```

### `_Templates/Curso.md`

```yaml
---
titulo: 
plataforma: 
instrutor: 
carga_horaria: 
status: planejado
progresso: 0
certificado: false
tags: [conhecimento, curso]
setor: Conhecimento
---
```

### `_Templates/Estudo Independente.md`

```yaml
---
topico: 
area: 
status: ativo
progresso: 0
ultima_revisao: <% tp.date.now("YYYY-MM-DD") %>
recursos: 0
tags: [conhecimento, estudo]
setor: Conhecimento
---
```

### `_Templates/Área de Conhecimento.md`

```yaml
---
descricao: 
subtopicos: 0
recursos: 0
tags: [conhecimento, area]
setor: Conhecimento
---
```

### Configurar no Templater (Folder Templates)

|Pasta|Template|
|---|---|
|`Conhecimento Independente/Concursos Públicos`|`_Templates/Concurso`|
|`Conhecimento Independente/Cursos Profissionalizantes`|`_Templates/Curso`|
|`Conhecimento Independente/Estudos Independentes`|`_Templates/Estudo Independente`|
|`Conhecimento Independente/Áreas de Conhecimento`|`_Templates/Área de Conhecimento`|