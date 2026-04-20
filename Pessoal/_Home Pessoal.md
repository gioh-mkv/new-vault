---
## tags: [homepage, pessoal] cssclasses: [home-page]
---

```dataviewjs
// ============================================================
//  HOME PESSOAL — DataviewJS
//  Seções: frase do dia · biblioteca · nota diária recente
// ============================================================

// ---------- HELPERS ----------
const hoje = new Date();
const pad   = n => String(n).padStart(2, "0");
const dataHoje   = `${hoje.getFullYear()}-${pad(hoje.getMonth()+1)}-${pad(hoje.getDate())}`;
const meses      = ["janeiro","fevereiro","março","abril","maio","junho","julho","agosto","setembro","outubro","novembro","dezembro"];
const dataExtenso = `${hoje.getDate()} de ${meses[hoje.getMonth()]} de ${hoje.getFullYear()}`;

function toDate(val) {
  if (!val) return null;
  if (val instanceof Date) return val;
  if (val && val.toJSDate) return val.toJSDate();
  return new Date(String(val));
}

// Link interno clicável no Obsidian
function ilink(path) {
  return `data-href="${path}" href="${path}" class="internal-link"`;
}

// Escolhe item de array usando o dia do ano como seed (muda a cada dia, consistente no dia)
function itemDoDia(arr) {
  const inicio = new Date(hoje.getFullYear(), 0, 0);
  const diaDono = Math.floor((hoje - inicio) / 86400000);
  return arr[diaDono % arr.length];
}

// ---------- FRASES DO DIA ----------
// Edite esta lista à vontade — adicione suas frases favoritas
const FRASES = [
  { texto: "A vida não é medida pelo número de respirações que damos, mas pelos momentos que nos tiram o fôlego.", autor: "Maya Angelou" },
  { texto: "Não é a espécie mais forte que sobrevive, nem a mais inteligente, mas a mais adaptável à mudança.", autor: "Charles Darwin" },
  { texto: "O único modo de fazer um excelente trabalho é amar o que você faz.", autor: "Steve Jobs" },
  { texto: "Seja a mudança que você deseja ver no mundo.", autor: "Mahatma Gandhi" },
  { texto: "Não importa o quão devagar você vá, desde que não pare.", autor: "Confúcio" },
  { texto: "O sucesso é a soma de pequenos esforços repetidos dia após dia.", autor: "Robert Collier" },
  { texto: "Você não pode voltar atrás e mudar o começo, mas pode começar agora e mudar o final.", autor: "C.S. Lewis" },
  { texto: "A persistência é o caminho do êxito.", autor: "Charles Chaplin" },
  { texto: "Tudo parece impossível até que seja feito.", autor: "Nelson Mandela" },
  { texto: "Viva como se fosse morrer amanhã. Aprenda como se fosse viver para sempre.", autor: "Mahatma Gandhi" },
  { texto: "A imaginação é mais importante que o conhecimento.", autor: "Albert Einstein" },
  { texto: "O tempo que você gosta de desperdiçar não é desperdiçado.", autor: "Bertrand Russell" },
  { texto: "Errar é humano, mas insistir no erro é burrice.", autor: "Cícero" },
  { texto: "Não tenha medo de desistir do bom para perseguir o ótimo.", autor: "John D. Rockefeller" },
  { texto: "A disciplina é a ponte entre metas e realizações.", autor: "Jim Rohn" },
  { texto: "Você é o resultado médio das cinco pessoas com quem mais convive.", autor: "Jim Rohn" },
  { texto: "O conhecimento fala, mas a sabedoria escuta.", autor: "Jimi Hendrix" },
  { texto: "A única limitação é aquela que você mesmo impõe.", autor: "Napoleon Hill" },
  { texto: "Faça o que você pode, com o que você tem, onde você está.", autor: "Theodore Roosevelt" },
  { texto: "Grandes realizações são possíveis quando você dá importância aos pequenos começos.", autor: "Lao Tsé" },
  { texto: "Quem não arrisca não petisca.", autor: "Provérbio popular" },
  { texto: "A leitura é para a mente o que o exercício é para o corpo.", autor: "Joseph Addison" },
  { texto: "Escrever é a pintura da voz.", autor: "Voltaire" },
  { texto: "Um livro é um sonho que você segura nas mãos.", autor: "Neil Gaiman" },
  { texto: "Não existe amigo mais leal do que um livro.", autor: "Ernest Hemingway" },
  { texto: "Quanto mais você lê, mais coisas você saberá. Quanto mais aprende, mais lugares você irá.", autor: "Dr. Seuss" },
  { texto: "Ler é viajar sem sair do lugar.", autor: "Provérbio popular" },
  { texto: "Se você quer conhecer o mundo, leia. Se quer mudar o mundo, escreva.", autor: "Anônimo" },
  { texto: "O segredo para avançar é começar.", autor: "Mark Twain" },
  { texto: "Cuide do seu corpo. É o único lugar que você tem para viver.", autor: "Jim Rohn" },
];

const fraseHoje = itemDoDia(FRASES);

// ---------- DADOS DA BIBLIOTECA ----------
const leituras = dv.pages('"Pessoal/Biblioteca"').where(p => p.file.name !== "_Home Pessoal");

const emAndamento = leituras.where(p => (p.status || "").toLowerCase() === "lendo").sort(p => p.file.mtime, "desc");
const concluidos  = leituras.where(p => (p.status || "").toLowerCase() === "concluído" || (p.status || "").toLowerCase() === "concluido").sort(p => p.data_conclusao, "desc");
const listaDesejo = leituras.where(p => (p.status || "").toLowerCase() === "quero ler").sort(p => p.file.ctime, "desc");

// ---------- NOTA DIÁRIA RECENTE ----------
const notaRecente = dv.pages('"MeuCofre/DailyNotes"').sort(p => p.file.day, "desc").first();

// Últimas 5 notas para o histórico
const ultimasNotas = dv.pages('"MeuCofre/DailyNotes"').sort(p => p.file.day, "desc").limit(5);

// ---------- FUNÇÕES DE RENDERIZAÇÃO ----------

// Estrelas de avaliação (1–5)
function estrelas(n) {
  if (!n || isNaN(n)) return "";
  const total = 5;
  const cheias = Math.round(n);
  return "★".repeat(cheias) + "☆".repeat(Math.max(0, total - cheias));
}

// Badge de status do livro
function badgeLivro(status) {
  const s = (status || "").toLowerCase();
  const mapa = {
    "lendo":       { bg:"#E6F1FB", cor:"#185FA5", label:"Lendo" },
    "concluído":   { bg:"#E1F5EE", cor:"#085041", label:"Concluído" },
    "concluido":   { bg:"#E1F5EE", cor:"#085041", label:"Concluído" },
    "quero ler":   { bg:"#FAEEDA", cor:"#854F0B", label:"Quero ler" },
    "pausado":     { bg:"#F1EFE8", cor:"#5F5E5A", label:"Pausado" },
    "abandonado":  { bg:"#FCEBEB", cor:"#A32D2D", label:"Abandonado" },
  };
  const cfg = mapa[s] || { bg:"#F1EFE8", cor:"#5F5E5A", label: status || "?" };
  return `<span style="font-size:10px;padding:2px 7px;border-radius:4px;font-weight:600;background:${cfg.bg};color:${cfg.cor};">${cfg.label}</span>`;
}

// Card de livro
function cardLivro(p) {
  const titulo  = p.titulo  || p.file.name;
  const autor   = p.autor   || "";
  const genero  = p.genero  || "";
  const nota    = p.nota    || null;
  const paginas = p.paginas || null;
  const paginasLidas = p.paginas_lidas || null;
  const progresso = (paginas && paginasLidas) ? Math.min(100, Math.round((paginasLidas / paginas) * 100)) : null;

  let card = `
  <a ${ilink(p.file.path)} style="display:block;border:1px solid var(--background-modifier-border);border-radius:10px;padding:12px;text-decoration:none;background:var(--background-primary);">
    <div style="display:flex;justify-content:space-between;align-items:flex-start;gap:8px;margin-bottom:6px;">
      <div style="font-size:13px;font-weight:600;color:var(--text-normal);line-height:1.3;">${titulo}</div>
      ${badgeLivro(p.status)}
    </div>`;

  if (autor) card += `<div style="font-size:11px;color:var(--text-faint);margin-bottom:4px;">${autor}</div>`;
  if (genero) card += `<div style="font-size:11px;color:var(--text-muted);">${genero}</div>`;

  if (progresso !== null) {
    card += `
    <div style="margin-top:8px;">
      <div style="display:flex;justify-content:space-between;margin-bottom:3px;">
        <span style="font-size:10px;color:var(--text-faint);">${paginasLidas} / ${paginas} pág.</span>
        <span style="font-size:10px;color:var(--text-faint);">${progresso}%</span>
      </div>
      <div style="height:3px;background:var(--background-modifier-border);border-radius:2px;">
        <div style="height:3px;width:${progresso}%;background:#378ADD;border-radius:2px;"></div>
      </div>
    </div>`;
  }

  if (nota) {
    card += `<div style="font-size:12px;color:#EF9F27;margin-top:6px;letter-spacing:1px;">${estrelas(nota)}</div>`;
  }

  card += `</a>`;
  return card;
}

// ---------- BUILD HTML ----------
let h = `<div style="max-width:860px;">`;

// ── Cabeçalho ───────────────────────────────────────────────────────────────
h += `
<div style="padding-bottom:16px;border-bottom:1px solid var(--background-modifier-border);margin-bottom:24px;display:flex;align-items:flex-start;justify-content:space-between;gap:12px;">
  <div>
    <div style="font-size:22px;font-weight:700;color:var(--text-normal);line-height:1.2;">Pessoal</div>
    <div style="font-size:13px;color:var(--text-faint);margin-top:4px;">${dataExtenso}</div>
  </div>
  <a ${ilink("Home")} style="font-size:12px;color:var(--interactive-accent);text-decoration:none;margin-top:4px;">← voltar para Home</a>
</div>`;

// ── Frase do dia ─────────────────────────────────────────────────────────────
h += `
<div style="background:var(--background-secondary);border-left:3px solid #7F77DD;border-radius:0 10px 10px 0;padding:16px 20px;margin-bottom:28px;">
  <div style="font-size:11px;font-weight:600;letter-spacing:.08em;text-transform:uppercase;color:#7F77DD;margin-bottom:8px;">Frase do dia</div>
  <div style="font-size:15px;color:var(--text-normal);line-height:1.6;font-style:italic;">"${fraseHoje.texto}"</div>
  <div style="font-size:12px;color:var(--text-faint);margin-top:8px;">— ${fraseHoje.autor}</div>
</div>`;

// ── Biblioteca ───────────────────────────────────────────────────────────────
h += `
<div style="display:flex;justify-content:space-between;align-items:center;margin-bottom:12px;">
  <p style="font-size:11px;font-weight:600;letter-spacing:.08em;text-transform:uppercase;color:var(--text-faint);margin:0;">Biblioteca</p>
  <div style="display:flex;gap:12px;">
    <span style="font-size:12px;color:var(--text-faint);">${emAndamento.length} lendo</span>
    <span style="font-size:12px;color:var(--text-faint);">${concluidos.length} concluídos</span>
    <span style="font-size:12px;color:var(--text-faint);">${listaDesejo.length} na fila</span>
  </div>
</div>`;

// Lendo agora
if (emAndamento.length > 0) {
  h += `<p style="font-size:12px;font-weight:600;color:var(--text-muted);margin:0 0 8px;">Lendo agora</p>`;
  h += `<div style="display:grid;grid-template-columns:repeat(3,minmax(0,1fr));gap:10px;margin-bottom:20px;">`;
  for (const p of emAndamento) { h += cardLivro(p); }
  h += `</div>`;
}

// Concluídos recentes (máx 6)
if (concluidos.length > 0) {
  h += `<p style="font-size:12px;font-weight:600;color:var(--text-muted);margin:0 0 8px;">Concluídos recentemente</p>`;
  h += `<div style="display:grid;grid-template-columns:repeat(3,minmax(0,1fr));gap:10px;margin-bottom:20px;">`;
  for (const p of concluidos.limit(6)) { h += cardLivro(p); }
  h += `</div>`;
  if (concluidos.length > 6) {
    h += `<a ${ilink("01. Pessoal/Biblioteca")} style="font-size:12px;color:var(--interactive-accent);text-decoration:none;display:block;margin-bottom:20px;">ver todos os ${concluidos.length} livros concluídos →</a>`;
  }
}

// Lista de desejo (máx 4)
if (listaDesejo.length > 0) {
  h += `<p style="font-size:12px;font-weight:600;color:var(--text-muted);margin:0 0 8px;">Quero ler</p>`;
  h += `<div style="display:grid;grid-template-columns:repeat(3,minmax(0,1fr));gap:10px;margin-bottom:28px;">`;
  for (const p of listaDesejo.limit(3)) { h += cardLivro(p); }
  h += `</div>`;
}

if (leituras.length === 0) {
  h += `
  <div style="border:1px dashed var(--background-modifier-border);border-radius:10px;padding:24px;text-align:center;margin-bottom:28px;">
    <div style="font-size:13px;color:var(--text-faint);">Nenhum livro cadastrado ainda.</div>
    <div style="font-size:12px;color:var(--text-faint);margin-top:4px;">Crie uma nota em <code>01. Pessoal/Biblioteca/</code> com o frontmatter abaixo.</div>
  </div>`;
}

// ── Nota diária mais recente ─────────────────────────────────────────────────
h += `<p style="font-size:11px;font-weight:600;letter-spacing:.08em;text-transform:uppercase;color:var(--text-faint);margin:0 0 12px;">Nota diária</p>`;

if (notaRecente) {
  const d  = toDate(notaRecente.file.day);
  const ds = d ? `${d.getDate()} de ${meses[d.getMonth()]} de ${d.getFullYear()}` : notaRecente.file.name;
  const academiaOk = notaRecente.academia === true;
  const humor  = notaRecente.humor || null;
  const tarefas = notaRecente.tarefas_concluidas || null;

  // Card da nota mais recente
  h += `
  <a ${ilink(notaRecente.file.path)} style="display:block;background:var(--background-secondary);border:1px solid var(--background-modifier-border);border-radius:10px;padding:16px;text-decoration:none;margin-bottom:12px;">
    <div style="font-size:11px;color:var(--text-faint);text-transform:uppercase;letter-spacing:.07em;margin-bottom:4px;">${ds} · mais recente</div>
    <div style="font-size:17px;font-weight:600;color:var(--text-normal);margin-bottom:10px;">${notaRecente.titulo || notaRecente.file.name}</div>
    <div style="display:flex;gap:6px;flex-wrap:wrap;">
      <span style="font-size:11px;padding:3px 9px;border-radius:20px;${academiaOk ? 'background:#E1F5EE;border:1px solid #9FE1CB;color:#085041;' : 'background:var(--background-primary);border:1px solid var(--background-modifier-border);color:var(--text-muted);'}">${academiaOk ? "Academia ✓" : "Academia —"}</span>
      ${humor   ? `<span style="font-size:11px;padding:3px 9px;border-radius:20px;background:var(--background-primary);border:1px solid var(--background-modifier-border);color:var(--text-muted);">Humor: ${humor}</span>` : ""}
      ${tarefas ? `<span style="font-size:11px;padding:3px 9px;border-radius:20px;background:var(--background-primary);border:1px solid var(--background-modifier-border);color:var(--text-muted);">${tarefas} tarefas</span>` : ""}
    </div>
  </a>`;

  // Histórico das últimas notas
  if (ultimasNotas.length > 1) {
    h += `<div style="border:1px solid var(--background-modifier-border);border-radius:10px;overflow:hidden;">`;
    let primeiro = true;
    for (const nota of ultimasNotas) {
      if (primeiro) { primeiro = false; continue; } // pula a mais recente (já mostrada)
      const dd = toDate(nota.file.day);
      const ddStr = dd ? `${dd.getDate()} de ${meses[dd.getMonth()]}` : nota.file.name;
      const acad = nota.academia === true;
      h += `
      <a ${ilink(nota.file.path)} style="display:flex;align-items:center;justify-content:space-between;padding:8px 14px;border-bottom:1px solid var(--background-modifier-border);text-decoration:none;">
        <div>
          <span style="font-size:12px;color:var(--text-normal);">${nota.titulo || nota.file.name}</span>
          <span style="font-size:11px;color:var(--text-faint);margin-left:8px;">${ddStr}</span>
        </div>
        <span style="font-size:10px;padding:2px 7px;border-radius:4px;${acad ? 'background:#E1F5EE;color:#085041;' : 'background:var(--background-modifier-border);color:var(--text-faint);'}">${acad ? "Academia ✓" : "—"}</span>
      </a>`;
    }
    h += `</div>`;
  }
} else {
  h += `<div style="background:var(--background-secondary);border:1px solid var(--background-modifier-border);border-radius:10px;padding:16px;">
    <p style="font-size:12px;color:var(--text-faint);font-style:italic;margin:0;">Nenhuma nota diária encontrada.</p>
  </div>`;
}

h += `</div>`;
dv.el("div", h);

// ── Botão criar nota diária (via createEl para funcionar no Obsidian) ─────────
const wrap = dv.el("div", "", { attr: { style: "margin:16px 0 0;" } });
wrap.createEl("a", {
  text: `+ Criar nota de hoje — ${dataHoje}`,
  attr: {
    "data-href": `DailyNotes/${dataHoje}`,
    "href":      `DailyNotes/${dataHoje}`,
    "class":     "internal-link",
    "style":     "display:inline-flex;align-items:center;gap:8px;background:var(--interactive-accent);color:var(--text-on-accent);border:none;border-radius:8px;padding:9px 18px;font-size:13px;font-weight:600;text-decoration:none;"
  }
});
```

---

## Frontmatter para notas de livro

Crie notas em `Pessoal/Biblioteca/` com este modelo:

```yaml
---
titulo: Nome do Livro
autor: Nome do Autor
genero: Romance
status: lendo          # lendo | concluído | quero ler | pausado | abandonado
paginas: 320
paginas_lidas: 150     # só para livros com status "lendo"
data_inicio: 2025-04-01
data_conclusao:        # preencher ao concluir
nota: 4                # avaliação de 1 a 5 (opcional)
pendente: false
setor: Pessoal
---
```