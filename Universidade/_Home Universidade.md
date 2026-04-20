---
## tags: [homepage, universidade] cssclasses: [home-page]
---

```dataviewjs
// ============================================================
//  HOME UNIVERSIDADE — DataviewJS
//  Seções: próximos eventos · disciplinas em andamento · histórico
// ============================================================

// ---------- HELPERS ----------
const hoje       = new Date();
const pad        = n => String(n).padStart(2, "0");
const dataHoje   = `${hoje.getFullYear()}-${pad(hoje.getMonth()+1)}-${pad(hoje.getDate())}`;
const meses      = ["janeiro","fevereiro","março","abril","maio","junho","julho","agosto","setembro","outubro","novembro","dezembro"];

function toDate(val) {
  if (!val) return null;
  if (val instanceof Date) return val;
  if (val && val.toJSDate) return val.toJSDate();
  return new Date(String(val));
}
function ilink(path) {
  return `data-href="${path}" href="${path}" class="internal-link"`;
}
function diasAte(val) {
  const d = toDate(val);
  if (!d || isNaN(d)) return null;
  return Math.ceil((d - hoje) / 86400000);
}
function labelDias(n) {
  if (n === 0) return "Hoje";
  if (n === 1) return "Amanhã";
  if (n > 0 && n < 8) return `Em ${n} dias`;
  if (n < 0) return `Há ${Math.abs(n)} dias`;
  const d = toDate(null);
  return `${n} dias`;
}
function fmtData(val) {
  const d = toDate(val);
  if (!d || isNaN(d)) return "—";
  return `${d.getDate()} de ${meses[d.getMonth()]} de ${d.getFullYear()}`;
}

// ---------- CORES POR TIPO DE EVENTO ----------
const TIPO_CFG = {
  prova:     { bg:"#FCEBEB", cor:"#A32D2D", label:"Prova" },
  trabalho:  { bg:"#FAEEDA", cor:"#854F0B", label:"Trabalho" },
  palestra:  { bg:"#E6F1FB", cor:"#185FA5", label:"Palestra" },
  seminario: { bg:"#EEEDFE", cor:"#534AB7", label:"Seminário" },
  prazo:     { bg:"#FAECE7", cor:"#993C1D", label:"Prazo" },
  aula:      { bg:"#EAF3DE", cor:"#3B6D11", label:"Aula" },
  outro:     { bg:"#F1EFE8", cor:"#5F5E5A", label:"Outro" },
};

function badgeTipo(tipo) {
  const t = (tipo || "outro").toLowerCase();
  const cfg = TIPO_CFG[t] || TIPO_CFG["outro"];
  return `<span style="font-size:10px;padding:2px 7px;border-radius:4px;font-weight:600;background:${cfg.bg};color:${cfg.cor};">${cfg.label}</span>`;
}

function badgeStatus(status) {
  const s = (status || "").toLowerCase();
  const mapa = {
    "pendente":   { bg:"#FAEEDA", cor:"#854F0B" },
    "concluido":  { bg:"#E1F5EE", cor:"#085041" },
    "cancelado":  { bg:"#FCEBEB", cor:"#A32D2D" },
  };
  const cfg = mapa[s] || { bg:"#F1EFE8", cor:"#5F5E5A" };
  const label = s.charAt(0).toUpperCase() + s.slice(1) || "—";
  return `<span style="font-size:10px;padding:2px 7px;border-radius:4px;font-weight:600;background:${cfg.bg};color:${cfg.cor};">${label}</span>`;
}

// ---------- DADOS ----------
// Todos os eventos
const todosEventos = dv.pages('"Universidade/Eventos"')
  .where(p => p.date)
  .sort(p => toDate(p.date), "asc");

// Próximos (a partir de hoje, inclusive eventos de hoje)
const proximosEventos = todosEventos
  .where(p => diasAte(p.date) >= 0)
  .limit(10);

// Eventos passados recentes
const eventosPassados = todosEventos
  .where(p => diasAte(p.date) < 0)
  .sort(p => toDate(p.date), "desc")
  .limit(5);

// Disciplinas em andamento
const disciplinasAtivas = dv.pages('"Universidade/Disciplinas/Em andamento"')
  .where(p => p.file.name !== "_Home Universidade")
  .sort(p => p.file.name, "asc");

// Semestres anteriores — agrupa por pasta
const semestresAnteriores = dv.pages('"Universidade/Disciplinas/Semestres anteriores"')
  .where(p => p.file.name !== "_Home Universidade")
  .sort(p => p.file.name, "asc");

// Agrupa disciplinas por semestre (campo: semestre no frontmatter)
const porSemestre = {};
for (const d of semestresAnteriores) {
  const sem = d.semestre || d.file.folder.split("/").pop() || "Sem semestre";
  if (!porSemestre[sem]) porSemestre[sem] = [];
  porSemestre[sem].push(d);
}
const semestresOrdenados = Object.keys(porSemestre).sort().reverse();

// ---------- CARD DE DISCIPLINA ----------
function cardDisciplina(p) {
  const nome       = p.nome       || p.file.name;
  const professor  = p.professor  || "";
  const creditos   = p.creditos   || null;
  const horario    = p.horario    || "";
  const nota       = p.nota_final || null;
  const status     = p.status     || "";

  const statusCfg = {
    "aprovado":   { bg:"#E1F5EE", cor:"#085041" },
    "reprovado":  { bg:"#FCEBEB", cor:"#A32D2D" },
    "cursando":   { bg:"#E6F1FB", cor:"#185FA5" },
    "trancado":   { bg:"#F1EFE8", cor:"#5F5E5A" },
  };
  const sc = statusCfg[(status||"").toLowerCase()] || { bg:"#F1EFE8", cor:"#5F5E5A" };
  const statusBadge = status
    ? `<span style="font-size:10px;padding:2px 7px;border-radius:4px;font-weight:600;background:${sc.bg};color:${sc.cor};">${status}</span>`
    : "";

  return `
  <a ${ilink(p.file.path)} style="display:block;border:1px solid var(--background-modifier-border);border-radius:10px;padding:12px;text-decoration:none;background:var(--background-primary);">
    <div style="display:flex;justify-content:space-between;align-items:flex-start;gap:6px;margin-bottom:6px;">
      <div style="font-size:13px;font-weight:600;color:var(--text-normal);line-height:1.3;">${nome}</div>
      ${statusBadge}
    </div>
    ${professor ? `<div style="font-size:11px;color:var(--text-faint);margin-bottom:3px;">${professor}</div>` : ""}
    <div style="display:flex;gap:10px;flex-wrap:wrap;margin-top:4px;">
      ${creditos ? `<span style="font-size:11px;color:var(--text-muted);">${creditos} créditos</span>` : ""}
      ${horario  ? `<span style="font-size:11px;color:var(--text-muted);">${horario}</span>` : ""}
      ${nota !== null ? `<span style="font-size:11px;font-weight:600;color:#185FA5;">Nota: ${nota}</span>` : ""}
    </div>
  </a>`;
}

// ---------- BUILD HTML ----------
let h = `<div style="max-width:860px;">`;

// Cabeçalho
h += `
<div style="padding-bottom:16px;border-bottom:1px solid var(--background-modifier-border);margin-bottom:24px;display:flex;align-items:flex-start;justify-content:space-between;">
  <div>
    <div style="font-size:22px;font-weight:700;color:var(--text-normal);">Universidade</div>
    <div style="font-size:13px;color:var(--text-faint);margin-top:4px;">${dataHoje}</div>
  </div>
  <a ${ilink("Home")} style="font-size:12px;color:var(--interactive-accent);text-decoration:none;margin-top:4px;">← voltar para Home</a>
</div>`;

// Cards de resumo
h += `<div style="display:grid;grid-template-columns:repeat(3,minmax(0,1fr));gap:10px;margin-bottom:28px;">`;
h += `
<div style="background:var(--background-secondary);border-radius:10px;padding:14px;">
  <div style="font-size:11px;color:var(--text-faint);margin-bottom:6px;text-transform:uppercase;letter-spacing:.06em;">Próximos eventos</div>
  <div style="font-size:26px;font-weight:700;color:var(--text-normal);line-height:1;">${proximosEventos.length}</div>
  <div style="font-size:11px;color:var(--text-faint);margin-top:4px;">agendados</div>
</div>
<div style="background:var(--background-secondary);border-radius:10px;padding:14px;">
  <div style="font-size:11px;color:var(--text-faint);margin-bottom:6px;text-transform:uppercase;letter-spacing:.06em;">Disciplinas ativas</div>
  <div style="font-size:26px;font-weight:700;color:var(--text-normal);line-height:1;">${disciplinasAtivas.length}</div>
  <div style="font-size:11px;color:var(--text-faint);margin-top:4px;">este semestre</div>
</div>
<div style="background:var(--background-secondary);border-radius:10px;padding:14px;">
  <div style="font-size:11px;color:var(--text-faint);margin-bottom:6px;text-transform:uppercase;letter-spacing:.06em;">Semestres</div>
  <div style="font-size:26px;font-weight:700;color:var(--text-normal);line-height:1;">${semestresOrdenados.length}</div>
  <div style="font-size:11px;color:var(--text-faint);margin-top:4px;">concluídos</div>
</div>`;
h += `</div>`;

// ── Próximos eventos ──
h += `
<div style="display:flex;justify-content:space-between;align-items:center;margin-bottom:12px;">
  <p style="font-size:11px;font-weight:600;letter-spacing:.08em;text-transform:uppercase;color:var(--text-faint);margin:0;">Próximos eventos</p>
</div>`;

if (proximosEventos.length === 0) {
  h += `<div style="border:1px dashed var(--background-modifier-border);border-radius:10px;padding:20px;text-align:center;margin-bottom:24px;">
    <div style="font-size:13px;color:var(--text-faint);">Nenhum evento próximo registrado.</div>
  </div>`;
} else {
  // Separa: hoje/urgente (≤3 dias), próximos (4–14 dias), futuros (>14 dias)
  const urgentes  = proximosEventos.where(p => diasAte(p.date) <= 3);
  const proximos  = proximosEventos.where(p => diasAte(p.date) > 3 && diasAte(p.date) <= 14);
  const futuros   = proximosEventos.where(p => diasAte(p.date) > 14);

  function listaEventos(eventos, destaque) {
    let out = "";
    for (const ev of eventos) {
      const n    = diasAte(ev.date);
      const tipo = (ev.type || ev.tipo || "outro").toLowerCase();
      const disc = ev.disciplina || "";
      const local = ev.local || "";
      const borda = destaque ? "border-left:3px solid #E24B4A;" : "border-left:3px solid var(--background-modifier-border);";

      out += `
      <a ${ilink(ev.file.path)} style="display:flex;align-items:flex-start;gap:12px;padding:10px 14px;border-bottom:1px solid var(--background-modifier-border);text-decoration:none;${borda}">
        <div style="flex-shrink:0;text-align:center;min-width:44px;">
          <div style="font-size:18px;font-weight:700;color:${destaque ? '#E24B4A' : 'var(--text-normal)'};">${toDate(ev.date).getDate()}</div>
          <div style="font-size:10px;color:var(--text-faint);text-transform:uppercase;">${meses[toDate(ev.date).getMonth()].slice(0,3)}</div>
        </div>
        <div style="flex:1;min-width:0;">
          <div style="display:flex;align-items:center;gap:6px;margin-bottom:3px;flex-wrap:wrap;">
            <span style="font-size:13px;font-weight:600;color:var(--text-normal);">${ev.file.name}</span>
            ${badgeTipo(tipo)}
            ${ev.status ? badgeStatus(ev.status) : ""}
          </div>
          <div style="display:flex;gap:10px;flex-wrap:wrap;">
            ${disc  ? `<span style="font-size:11px;color:var(--text-faint);">${disc}</span>` : ""}
            ${local ? `<span style="font-size:11px;color:var(--text-faint);">${local}</span>` : ""}
          </div>
        </div>
        <div style="flex-shrink:0;font-size:11px;font-weight:600;color:${destaque ? '#E24B4A' : 'var(--text-faint)'};">${labelDias(n)}</div>
      </a>`;
    }
    return out;
  }

  h += `<div style="border:1px solid var(--background-modifier-border);border-radius:10px;overflow:hidden;margin-bottom:24px;">`;
  if (urgentes.length > 0)  h += listaEventos(urgentes, true);
  if (proximos.length > 0)  h += listaEventos(proximos, false);
  if (futuros.length > 0)   h += listaEventos(futuros, false);
  h += `</div>`;
}

// Eventos passados recentes
if (eventosPassados.length > 0) {
  h += `<p style="font-size:11px;font-weight:600;letter-spacing:.08em;text-transform:uppercase;color:var(--text-faint);margin:0 0 10px;">Eventos recentes</p>`;
  h += `<div style="border:1px solid var(--background-modifier-border);border-radius:10px;overflow:hidden;margin-bottom:28px;">`;
  for (const ev of eventosPassados) {
    const tipo = (ev.type || ev.tipo || "outro").toLowerCase();
    h += `
    <a ${ilink(ev.file.path)} style="display:flex;align-items:center;gap:10px;padding:8px 14px;border-bottom:1px solid var(--background-modifier-border);text-decoration:none;opacity:0.7;">
      <div style="flex:1;">
        <span style="font-size:12px;color:var(--text-normal);">${ev.file.name}</span>
        ${ev.disciplina ? `<span style="font-size:11px;color:var(--text-faint);margin-left:8px;">${ev.disciplina}</span>` : ""}
      </div>
      ${badgeTipo(tipo)}
      ${ev.status ? badgeStatus(ev.status) : ""}
      <span style="font-size:11px;color:var(--text-faint);flex-shrink:0;">${fmtData(ev.date)}</span>
    </a>`;
  }
  h += `</div>`;
}

// ── Disciplinas em andamento ──
h += `
<div style="display:flex;justify-content:space-between;align-items:center;margin-bottom:12px;">
  <p style="font-size:11px;font-weight:600;letter-spacing:.08em;text-transform:uppercase;color:var(--text-faint);margin:0;">Disciplinas — semestre atual</p>
</div>`;

if (disciplinasAtivas.length === 0) {
  h += `<div style="border:1px dashed var(--background-modifier-border);border-radius:10px;padding:20px;text-align:center;margin-bottom:28px;">
    <div style="font-size:13px;color:var(--text-faint);">Nenhuma disciplina cadastrada em <code>Universidade/Disciplinas/Em andamento/</code>.</div>
  </div>`;
} else {
  h += `<div style="display:grid;grid-template-columns:repeat(3,minmax(0,1fr));gap:10px;margin-bottom:28px;">`;
  for (const d of disciplinasAtivas) { h += cardDisciplina(d); }
  h += `</div>`;
}

// ── Semestres anteriores ──
h += `<p style="font-size:11px;font-weight:600;letter-spacing:.08em;text-transform:uppercase;color:var(--text-faint);margin:0 0 12px;">Semestres anteriores</p>`;

if (semestresOrdenados.length === 0) {
  h += `<div style="border:1px dashed var(--background-modifier-border);border-radius:10px;padding:20px;text-align:center;margin-bottom:24px;">
    <div style="font-size:13px;color:var(--text-faint);">Nenhum semestre anterior cadastrado em <code>Universidade/Disciplinas/Semestres anteriores/</code>.</div>
  </div>`;
} else {
  for (const sem of semestresOrdenados) {
    const discs = porSemestre[sem];
    // Calcula média do semestre
    const notas = discs.map(d => d.nota_final).filter(n => n != null && !isNaN(n));
    const media = notas.length > 0 ? (notas.reduce((a,b)=>a+b,0)/notas.length).toFixed(1) : null;

    h += `
    <div style="margin-bottom:16px;">
      <div style="display:flex;align-items:center;justify-content:space-between;margin-bottom:8px;">
        <span style="font-size:13px;font-weight:600;color:var(--text-normal);">${sem}</span>
        <div style="display:flex;gap:10px;align-items:center;">
          ${media ? `<span style="font-size:11px;color:var(--text-faint);">Média: <strong style="color:var(--text-normal);">${media}</strong></span>` : ""}
          <span style="font-size:11px;color:var(--text-faint);">${discs.length} disciplina${discs.length !== 1 ? "s":""}</span>
        </div>
      </div>
      <div style="display:grid;grid-template-columns:repeat(3,minmax(0,1fr));gap:8px;">`;
    for (const d of discs) { h += cardDisciplina(d); }
    h += `</div></div>`;
  }
}

h += `</div>`;
dv.el("div", h);

// ── Botão novo evento (via createEl) ──
const wrap = dv.el("div", "", { attr: { style: "margin:16px 0 0;display:flex;gap:10px;" } });

wrap.createEl("a", {
  text: "+ Novo evento",
  attr: {
    "data-href": `Universidade/Eventos/Novo Evento`,
    "href":      `Universidade/Eventos/Novo Evento`,
    "class":     "internal-link",
    "style":     "display:inline-flex;align-items:center;background:var(--interactive-accent);color:var(--text-on-accent);border:none;border-radius:8px;padding:9px 18px;font-size:13px;font-weight:600;text-decoration:none;"
  }
});

wrap.createEl("a", {
  text: "+ Nova disciplina",
  attr: {
    "data-href": `Universidade/Disciplinas/Em andamento/Nova Disciplina`,
    "href":      `Universidade/Disciplinas/Em andamento/Nova Disciplina`,
    "class":     "internal-link",
    "style":     "display:inline-flex;align-items:center;background:var(--background-secondary);color:var(--text-normal);border:1px solid var(--background-modifier-border);border-radius:8px;padding:9px 18px;font-size:13px;font-weight:600;text-decoration:none;"
  }
});
```

---

## Templates para o Templater

### `_Templates/Evento Universidade.md`

```markdown
---
date: <% tp.date.now("YYYY-MM-DD") %>
type: prova
disciplina: 
local: 
status: pendente
tags: [universidade, evento]
pendente: true
setor: Universidade
---

## Descrição


## Anotações

```

### `_Templates/Disciplina.md`

```markdown
---
nome: 
professor: 
creditos: 
horario: 
semestre: <% tp.date.now("YYYY") %>.1
status: cursando
nota_final: 
tags: [universidade, disciplina]
---

## Ementa


## Anotações de aula


## Trabalhos e provas

```

### Configurar no Templater (Folder Templates)

|Pasta|Template|
|---|---|
|`Universidade/Eventos`|`_Templates/Evento Universidade`|
|`Universidade/Disciplinas/Em andamento`|`_Templates/Disciplina`|
