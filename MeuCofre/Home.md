---
## tags: [homepage] cssclasses: [home-page]
---
```dataviewjs
// ============================================================
//  HOME PRINCIPAL — DataviewJS
//  Navegação via links internos do Obsidian (href="obsidian://...")
//  Botão de nota diária via app.workspace (chamado dentro do dv.el)
// ============================================================

const SETORES = [
  { nome: "Pessoal",       path: "Pessoal/_Home Pessoal",                       emoji: "✦", cor: "#534AB7", bg: "#EEEDFE" },
  { nome: "Saúde",         path: "Pessoal/Saúde/_Home Saúde",                           emoji: "◈", cor: "#0F6E56", bg: "#E1F5EE" },
  { nome: "Universidade",  path: "Universidade/_Home Universidade",              emoji: "◎", cor: "#185FA5", bg: "#E6F1FB" },
  { nome: "Conhecimento",  path: "Conhecimento Independente/_Home Conhecimento", emoji: "⬡", cor: "#854F0B", bg: "#FAEEDA" },
  { nome: "Finanças",      path: "Pessoal/Finanças/_Home Finanças",                     emoji: "◇", cor: "#3B6D11", bg: "#EAF3DE" },
  { nome: "Projetos",      path: "Projetos/_Home Projetos",                     emoji: "⬢", cor: "#993C1D", bg: "#FAECE7" },
  { nome: "Profissional",  path: "Profissional/_Home Profissional",              emoji: "◉", cor: "#993556", bg: "#FBEAF0" },
];

// ---------- HELPERS ----------
const hoje = new Date();
const pad   = n => String(n).padStart(2, "0");
const dataHoje    = `${hoje.getFullYear()}-${pad(hoje.getMonth()+1)}-${pad(hoje.getDate())}`;
const diasSemana  = ["domingo","segunda-feira","terça-feira","quarta-feira","quinta-feira","sexta-feira","sábado"];
const meses       = ["janeiro","fevereiro","março","abril","maio","junho","julho","agosto","setembro","outubro","novembro","dezembro"];
const dataExtenso = `${hoje.getDate()} de ${meses[hoje.getMonth()]} de ${hoje.getFullYear()}`;

function toDate(val) {
  if (!val) return null;
  if (val instanceof Date) return val;
  if (val && val.toJSDate) return val.toJSDate();
  return new Date(String(val));
}
function diasAte(val) {
  const d = toDate(val);
  if (!d || isNaN(d)) return null;
  return Math.ceil((d - hoje) / 86400000);
}
function labelDias(n) {
  if (n === 0) return "Hoje";
  if (n === 1) return "Amanhã";
  if (n > 1 && n < 8) return `Em ${n} dias`;
  if (n < 0) return `Há ${Math.abs(n)} dias`;
  return `${n} dias`;
}
function contarNotas(pasta, filtro) {
  try {
    const pages = dv.pages(`"${pasta}"`);
    return filtro ? pages.where(filtro).length : pages.length;
  } catch(e) { return 0; }
}

// Gera href de link interno do Obsidian que o Dataview respeita
// Usa data-href que o Obsidian intercepta nativamente nos elementos renderizados
function obsLink(path) {
  return `data-href="${path}" href="${path}" class="internal-link"`;
}

// ---------- DADOS ----------
const eventosUniv = dv.pages('"Universidade/Eventos"')
  .where(p => p.date && diasAte(p.date) >= 0)
  .sort(p => toDate(p.date), "asc").limit(3);

const eventosProf = dv.pages('"Profissional/Freelances"')
  .where(p => p.date_fim && p.status !== "concluido" && diasAte(p.date_fim) >= 0)
  .sort(p => toDate(p.date_fim), "asc").limit(2);

const eventosProj = dv.pages('"Projetos"')
  .where(p => p.file.name !== "_Home Projetos" && p.deadline && diasAte(p.deadline) >= 0)
  .sort(p => toDate(p.deadline), "asc").limit(2);

const todosEventos = [
  ...eventosUniv.map(p => ({ titulo: p.file.name, data: toDate(p.date),     tipo: "univ", path: p.file.path })),
  ...eventosProf.map(p => ({ titulo: p.file.name, data: toDate(p.date_fim), tipo: "prof", path: p.file.path })),
  ...eventosProj.map(p => ({ titulo: p.file.name, data: toDate(p.deadline), tipo: "proj", path: p.file.path })),
].sort((a,b) => a.data - b.data).slice(0, 5);

const pendencias = dv.pages().where(p => p.pendente === true).limit(6);

const notaRecente = dv.pages('"DailyNotes"').sort(p => p.file.day, "desc").first();

const acadSemana = dv.pages('"DailyNotes"').where(p => {
  const d = toDate(p.file.day);
  return d && (hoje - d) < 7 * 86400000 && p.academia === true;
}).length;

const contagens = {
  "Pessoal":      `${contarNotas("Pessoal/Biblioteca")} leituras`,
  "Saúde":        `${acadSemana}× esta semana`,
  "Universidade": `${contarNotas("Universidade/Eventos", p => p.date && diasAte(p.date) >= 0)} eventos`,
  "Conhecimento": `${contarNotas("Conhecimento Independente")} notas`,
  "Finanças":     `ver transações`,
  "Projetos":     `${contarNotas("Projetos", p => p.file.name !== "_Home Projetos" && p.status === "ativo")} ativos`,
  "Profissional": `${contarNotas("Profissional/Freelances", p => p.status !== "concluido")} freelances`,
};

const corEvento = { univ:"#378ADD", prof:"#D4537E", proj:"#D85A30" };
const tagEvento  = { univ:"Universidade", prof:"Profissional", proj:"Projetos" };
const tagCores   = {
  pessoal:      { bg:"#EEEDFE", cor:"#534AB7" },
  saude:        { bg:"#E1F5EE", cor:"#085041" },
  universidade: { bg:"#E6F1FB", cor:"#185FA5" },
  financas:     { bg:"#EAF3DE", cor:"#3B6D11" },
  projetos:     { bg:"#FAECE7", cor:"#993C1D" },
  profissional: { bg:"#FBEAF0", cor:"#993556" },
};

// ---------- BUILD HTML ----------
let h = `<div style="max-width:860px;">`;

// ── Cabeçalho ──────────────────────────────────────────────────────────────
h += `
<div style="padding-bottom:18px;border-bottom:1px solid var(--background-modifier-border);margin-bottom:24px;">
  <div style="display:flex;align-items:flex-start;justify-content:space-between;gap:12px;">
    <div>
      <div style="font-size:24px;font-weight:700;color:var(--text-normal);line-height:1.2;">
        Bom dia, <span style="color:var(--color-green);">bem-vindo</span> de volta.
      </div>
      <div style="font-size:13px;color:var(--text-faint);margin-top:4px;">Seu espaço de controle pessoal</div>
    </div>
    <div style="text-align:right;flex-shrink:0;">
      <div style="font-size:13px;font-weight:500;color:var(--text-muted);">${dataExtenso}</div>
      <div style="font-size:11px;color:var(--text-faint);margin-top:2px;">${diasSemana[hoje.getDay()]}</div>
    </div>
  </div>
</div>`;

// ── Botão nota diária (renderizado via dv.paragraph para usar o motor do Obsidian) ──
// Isso é feito FORA do bloco HTML principal, usando a API do Dataview diretamente
// O link [[DailyNotes/YYYY-MM-DD]] é reconhecido e clicável nativamente

// ── Botão "Criar nota de hoje" renderizado pelo motor nativo do Dataview ──
// Isso garante que o link [[...]] seja interceptado pelo Obsidian corretamente
const btnContainer = dv.el("div", "", {
  attr: { style: "margin: 10px 0 24px;" }
});

const btnLink = btnContainer.createEl("a", {
  text: `+ Criar nota de hoje — ${dataHoje}`,
  attr: {
    "data-href": `MeuCofre/DailyNotes/${dataHoje}`,
    "href":      `MeuCofre/DailyNotes/${dataHoje}`,
    "class":     "internal-link",
    "style":     "display:inline-flex;align-items:center;gap:8px;background:var(--interactive-accent);color:var(--text-on-accent);border:none;border-radius:8px;padding:9px 18px;font-size:13px;font-weight:600;text-decoration:none;"
  }
});

// ── Galeria de setores ──────────────────────────────────────────────────────
h += `<p style="font-size:11px;font-weight:600;letter-spacing:.08em;text-transform:uppercase;color:var(--text-faint);margin:0 0 10px;">Setores</p>`;
h += `<div style="display:grid;grid-template-columns:repeat(4,minmax(0,1fr));gap:10px;margin-bottom:28px;">`;

for (const s of SETORES) {
  h += `
  <a ${obsLink(s.path)} style="border:1px solid var(--background-modifier-border);border-radius:10px;padding:14px 12px;cursor:pointer;background:var(--background-primary);display:block;text-decoration:none;">
    <div style="width:30px;height:30px;border-radius:7px;background:${s.bg};color:${s.cor};display:flex;align-items:center;justify-content:center;font-size:15px;margin-bottom:8px;">${s.emoji}</div>
    <div style="font-size:13px;font-weight:600;color:var(--text-normal);">${s.nome}</div>
    <div style="font-size:11px;color:var(--text-faint);margin-top:2px;">${contagens[s.nome]}</div>
  </a>`;
}
h += `</div>`;

// ── Dois cards lado a lado ──────────────────────────────────────────────────
h += `<div style="display:grid;grid-template-columns:1fr 1fr;gap:12px;margin-bottom:24px;">`;

// Card eventos
h += `<div style="border:1px solid var(--background-modifier-border);border-radius:10px;padding:14px;">`;
h += `<div style="display:flex;justify-content:space-between;align-items:center;margin-bottom:12px;">
  <span style="font-size:11px;font-weight:600;letter-spacing:.06em;text-transform:uppercase;color:var(--text-faint);">Próximos eventos</span>
  <a ${obsLink("Universidade/_Home Universidade")} style="font-size:11px;color:var(--interactive-accent);text-decoration:none;">ver todos →</a>
</div>`;

if (todosEventos.length === 0) {
  h += `<p style="font-size:12px;color:var(--text-faint);font-style:italic;">Nenhum evento próximo.</p>`;
} else {
  for (const ev of todosEventos) {
    const n = diasAte(ev.data);
    h += `
    <a ${obsLink(ev.path)} style="display:flex;gap:10px;padding:6px 0;border-bottom:1px solid var(--background-modifier-border);text-decoration:none;">
      <div style="width:6px;height:6px;border-radius:50%;background:${corEvento[ev.tipo]||'#888'};flex-shrink:0;margin-top:5px;"></div>
      <div>
        <div style="font-size:12px;color:var(--text-normal);line-height:1.4;">${ev.titulo}</div>
        <div style="font-size:11px;color:var(--text-faint);margin-top:2px;">${n !== null ? labelDias(n) : "—"} · ${tagEvento[ev.tipo]||ev.tipo}</div>
      </div>
    </a>`;
  }
}
h += `</div>`;

// Card pendências
h += `<div style="border:1px solid var(--background-modifier-border);border-radius:10px;padding:14px;">`;
h += `<span style="font-size:11px;font-weight:600;letter-spacing:.06em;text-transform:uppercase;color:var(--text-faint);display:block;margin-bottom:12px;">Pendências</span>`;

if (pendencias.length === 0) {
  h += `<p style="font-size:12px;color:var(--text-faint);font-style:italic;">Sem pendências registradas.</p>`;
} else {
  for (const p of pendencias) {
    const setor = (p.setor||"").toLowerCase()
      .normalize("NFD").replace(/[\u0300-\u036f]/g,"").replace(/\s+/g,"");
    const tc = tagCores[setor] || { bg:"#F1EFE8", cor:"#5F5E5A" };
    h += `
    <a ${obsLink(p.file.path)} style="display:flex;align-items:center;gap:8px;padding:5px 0;border-bottom:1px solid var(--background-modifier-border);text-decoration:none;">
      <span style="font-size:12px;color:var(--text-normal);flex:1;">${p.file.name}</span>
      <span style="font-size:10px;padding:2px 7px;border-radius:4px;font-weight:600;background:${tc.bg};color:${tc.cor};">${p.setor||"Geral"}</span>
    </a>`;
  }
}
h += `</div>`;
h += `</div>`;

// ── Nota diária mais recente ────────────────────────────────────────────────
h += `<p style="font-size:11px;font-weight:600;letter-spacing:.08em;text-transform:uppercase;color:var(--text-faint);margin:0 0 10px;">Nota mais recente</p>`;

if (notaRecente) {
  const d  = toDate(notaRecente.file.day);
  const ds = d ? `${d.getDate()} de ${meses[d.getMonth()]} de ${d.getFullYear()}` : notaRecente.file.name;
  const academiaOk = notaRecente.academia === true;
  const humor  = notaRecente.humor || null;
  const tarefas = notaRecente.tarefas_concluidas || null;

  h += `
  <a ${obsLink(notaRecente.file.path)} style="display:block;background:var(--background-secondary);border:1px solid var(--background-modifier-border);border-radius:10px;padding:16px;text-decoration:none;">
    <div style="font-size:11px;color:var(--text-faint);text-transform:uppercase;letter-spacing:.07em;margin-bottom:4px;">${ds}</div>
    <div style="font-size:17px;font-weight:600;color:var(--text-normal);margin-bottom:10px;">${notaRecente.titulo || notaRecente.file.name}</div>
    <div style="display:flex;gap:6px;flex-wrap:wrap;">
      <span style="font-size:11px;padding:3px 9px;border-radius:20px;${academiaOk ? 'background:#E1F5EE;border:1px solid #9FE1CB;color:#085041;' : 'background:var(--background-primary);border:1px solid var(--background-modifier-border);color:var(--text-muted);'}">${academiaOk ? "Academia ✓" : "Academia —"}</span>
      ${humor   ? `<span style="font-size:11px;padding:3px 9px;border-radius:20px;background:var(--background-primary);border:1px solid var(--background-modifier-border);color:var(--text-muted);">Humor: ${humor}</span>` : ""}
      ${tarefas ? `<span style="font-size:11px;padding:3px 9px;border-radius:20px;background:var(--background-primary);border:1px solid var(--background-modifier-border);color:var(--text-muted);">${tarefas} tarefas concluídas</span>` : ""}
    </div>
  </a>`;
} else {
  h += `
  <div style="background:var(--background-secondary);border:1px solid var(--background-modifier-border);border-radius:10px;padding:16px;">
    <p style="font-size:12px;color:var(--text-faint);font-style:italic;margin:0;">Nenhuma nota diária encontrada ainda.</p>
  </div>`;
}

h += `</div>`;

// ── Renderiza tudo ──────────────────────────────────────────────────────────
dv.el("div", h);


```