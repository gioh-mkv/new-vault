---
## tags: [homepage, saude] cssclasses: [home-page]
---
```dataviewjs
// ============================================================
//  HOME SAÚDE — DataviewJS
//  Heatmap de academia · pesagem com formulário inline
// ============================================================

// ---------- HELPERS ----------
const hoje       = new Date();
const pad        = n => String(n).padStart(2, "0");
const dataHoje   = `${hoje.getFullYear()}-${pad(hoje.getMonth()+1)}-${pad(hoje.getDate())}`;
const meses      = ["janeiro","fevereiro","março","abril","maio","junho","julho","agosto","setembro","outubro","novembro","dezembro"];
const mesesAbrev = ["jan","fev","mar","abr","mai","jun","jul","ago","set","out","nov","dez"];
const diasSemana = ["Dom","Seg","Ter","Qua","Qui","Sex","Sáb"];

function toDate(val) {
  if (!val) return null;
  if (val instanceof Date) return val;
  if (val && val.toJSDate) return val.toJSDate();
  return new Date(String(val));
}
function ilink(path) {
  return `data-href="${path}" href="${path}" class="internal-link"`;
}
function isoDate(d) {
  return `${d.getFullYear()}-${pad(d.getMonth()+1)}-${pad(d.getDate())}`;
}

// ---------- CONFIGURAÇÃO ----------
// Edite aqui os nomes e a meta semanal
const NOME_P1     = "Pessoa 1";
const NOME_P2     = "Pessoa 2";
const COR_P1      = "#378ADD";
const COR_P2      = "#D4537E";
const META_SEMANAL = 4;
const JSON_PATH   = "Pessoal/Saúde/pesagem.json";

// ---------- DADOS: ACADEMIA ----------
const DIAS_HEATMAP = 365;
const inicioHeatmap = new Date(hoje);
inicioHeatmap.setDate(inicioHeatmap.getDate() - DIAS_HEATMAP);

const notasDiarias = dv.pages('"MeuCofre/DailyNotes"').where(p => {
  const d = toDate(p.file.day);
  return d && d >= inicioHeatmap;
});

const presencas = new Set();
for (const nota of notasDiarias) {
  if (nota.academia === true) {
    const d = toDate(nota.file.day);
    if (d) presencas.add(isoDate(d));
  }
}

const hoje7   = new Date(hoje); hoje7.setDate(hoje7.getDate() - 7);
const hoje30  = new Date(hoje); hoje30.setDate(hoje30.getDate() - 30);
const hoje365 = new Date(hoje); hoje365.setDate(hoje365.getDate() - 365);

let totalSemana = 0, totalMes = 0, totalAno = 0;
for (const iso of presencas) {
  const d = new Date(iso + "T00:00:00");
  if (d >= hoje7)   totalSemana++;
  if (d >= hoje30)  totalMes++;
  if (d >= hoje365) totalAno++;
}

let streak = 0;
const cursorStreak = new Date(hoje);
while (true) {
  const iso = isoDate(cursorStreak);
  if (presencas.has(iso)) {
    streak++;
    cursorStreak.setDate(cursorStreak.getDate() - 1);
  } else {
    if (iso === dataHoje && streak === 0) {
      cursorStreak.setDate(cursorStreak.getDate() - 1);
      continue;
    }
    break;
  }
}

// ---------- DADOS: PESAGEM ----------
let pesagens = [];
try {
  const raw = await app.vault.adapter.read(JSON_PATH);
  pesagens = JSON.parse(raw);
} catch(e) {
  pesagens = [];
}
pesagens.sort((a, b) => new Date(a.data) - new Date(b.data));

const ultimaPesagem    = pesagens.length > 0 ? pesagens[pesagens.length - 1] : null;
const penultimaPesagem = pesagens.length > 1 ? pesagens[pesagens.length - 2] : null;

function diffPeso(key) {
  if (!ultimaPesagem || !penultimaPesagem) return null;
  return (ultimaPesagem[key] || 0) - (penultimaPesagem[key] || 0);
}
function formatDiff(diff) {
  if (diff === null) return "";
  const sinal = diff > 0 ? "+" : "";
  const cor   = diff > 0 ? "#E24B4A" : "#1D9E75";
  return `<span style="font-size:11px;color:${cor};font-weight:600;margin-left:6px;">${sinal}${diff.toFixed(1)} kg</span>`;
}

// ---------- HEATMAP SVG ----------
function buildHeatmap() {
  const start = new Date(inicioHeatmap);
  start.setDate(start.getDate() - start.getDay());

  const semanas = [];
  let semanaAtual = [];
  const cursor = new Date(start);
  while (cursor <= hoje) {
    if (cursor.getDay() === 0 && semanaAtual.length > 0) {
      semanas.push(semanaAtual);
      semanaAtual = [];
    }
    semanaAtual.push(new Date(cursor));
    cursor.setDate(cursor.getDate() + 1);
  }
  if (semanaAtual.length > 0) semanas.push(semanaAtual);

  const labelMeses = [];
  semanas.forEach((semana, idx) => {
    const prim = semana[0];
    if (prim.getDate() <= 7) labelMeses.push({ idx, mes: prim.getMonth() });
  });

  const CELL = 11, GAP = 2, PASSO = CELL + GAP;
  const OT = 20, OL = 28;
  const largura = semanas.length * PASSO + OL;
  const altura  = 7 * PASSO + OT + 4;

  let svg = `<svg width="${largura}" height="${altura}" style="overflow:visible;">`;

  [1, 3, 5].forEach(dia => {
    const y = OT + dia * PASSO + CELL * 0.75;
    svg += `<text x="0" y="${y}" style="font-size:9px;fill:var(--text-faint);font-family:var(--font-interface);">${diasSemana[dia]}</text>`;
  });

  labelMeses.forEach(({ idx, mes }) => {
    svg += `<text x="${OL + idx * PASSO}" y="${OT - 6}" style="font-size:9px;fill:var(--text-faint);font-family:var(--font-interface);">${mesesAbrev[mes]}</text>`;
  });

  semanas.forEach((semana, col) => {
    semana.forEach((dia, row) => {
      const iso     = isoDate(dia);
      const futuro  = dia > hoje;
      const presente = presencas.has(iso);
      const x = OL + col * PASSO;
      const y = OT + row * PASSO;
      const cor = futuro ? "transparent" : presente ? "#1D9E75" : "var(--background-modifier-border)";
      svg += `<rect x="${x}" y="${y}" width="${CELL}" height="${CELL}" rx="2" fill="${cor}"/>`;
    });
  });

  svg += `</svg>`;
  return svg;
}

// ---------- GRÁFICO DE PESO SVG ----------
function buildGraficoPeso(dados) {
  if (!dados || dados.length < 2) {
    return `<p style="font-size:12px;color:var(--text-faint);font-style:italic;padding:8px 0;">Adicione ao menos 2 registros para ver o gráfico.</p>`;
  }

  const W = 580, H = 180;
  const PAD = { top: 16, right: 16, bottom: 28, left: 38 };
  const iW = W - PAD.left - PAD.right;
  const iH = H - PAD.top  - PAD.bottom;

  const vals = dados.flatMap(d => [d.pessoa1, d.pessoa2]).filter(v => v != null);
  if (vals.length === 0) return "";
  const minV = Math.floor(Math.min(...vals) - 2);
  const maxV = Math.ceil( Math.max(...vals) + 2);

  const xOf = i => PAD.left + (i / (dados.length - 1)) * iW;
  const yOf = v => PAD.top  + (1 - (v - minV) / (maxV - minV)) * iH;

  let svg = `<svg width="100%" viewBox="0 0 ${W} ${H}" style="overflow:visible;">`;

  // Grades
  for (let i = 0; i <= 4; i++) {
    const v = minV + ((maxV - minV) / 4) * i;
    const y = yOf(v);
    svg += `<line x1="${PAD.left}" y1="${y}" x2="${W - PAD.right}" y2="${y}" stroke="var(--background-modifier-border)" stroke-width="0.5"/>`;
    svg += `<text x="${PAD.left - 4}" y="${y + 4}" text-anchor="end" style="font-size:9px;fill:var(--text-faint);font-family:var(--font-interface);">${Math.round(v)}</text>`;
  }

  // Labels X
  const step = Math.max(1, Math.floor(dados.length / 6));
  dados.forEach((d, i) => {
    if (i % step !== 0 && i !== dados.length - 1) return;
    svg += `<text x="${xOf(i)}" y="${H - PAD.bottom + 14}" text-anchor="middle" style="font-size:9px;fill:var(--text-faint);font-family:var(--font-interface);">${(d.data||"").slice(5)}</text>`;
  });

  // Linhas e pontos
  function plotLine(key, cor) {
    const pts = dados.map((d, i) => d[key] != null ? `${xOf(i).toFixed(1)},${yOf(d[key]).toFixed(1)}` : null).filter(Boolean);
    if (pts.length < 2) return;
    svg += `<polyline points="${pts.join(" ")}" fill="none" stroke="${cor}" stroke-width="2" stroke-linejoin="round" stroke-linecap="round"/>`;
    dados.forEach((d, i) => {
      if (d[key] == null) return;
      svg += `<circle cx="${xOf(i).toFixed(1)}" cy="${yOf(d[key]).toFixed(1)}" r="3" fill="${cor}"/>`;
    });
  }
  plotLine("pessoa1", COR_P1);
  plotLine("pessoa2", COR_P2);

  svg += `</svg>`;
  return svg;
}

// ---------- BUILD HTML ----------
let h = `<div style="max-width:860px;">`;

// Cabeçalho
h += `
<div style="padding-bottom:16px;border-bottom:1px solid var(--background-modifier-border);margin-bottom:24px;display:flex;align-items:flex-start;justify-content:space-between;">
  <div>
    <div style="font-size:22px;font-weight:700;color:var(--text-normal);">Saúde</div>
    <div style="font-size:13px;color:var(--text-faint);margin-top:4px;">${dataHoje}</div>
  </div>
  <a ${ilink("Home")} style="font-size:12px;color:var(--interactive-accent);text-decoration:none;margin-top:4px;">← voltar para Home</a>
</div>`;

// Cards de resumo academia
const pctMeta = Math.min(100, Math.round((totalSemana / META_SEMANAL) * 100));
h += `<div style="display:grid;grid-template-columns:repeat(4,minmax(0,1fr));gap:10px;margin-bottom:28px;">`;
h += `
<div style="background:var(--background-secondary);border-radius:10px;padding:14px;">
  <div style="font-size:11px;color:var(--text-faint);margin-bottom:6px;text-transform:uppercase;letter-spacing:.06em;">Esta semana</div>
  <div style="font-size:26px;font-weight:700;color:var(--text-normal);line-height:1;">${totalSemana}</div>
  <div style="font-size:11px;color:var(--text-faint);margin-top:4px;">meta: ${META_SEMANAL}×</div>
  <div style="height:3px;background:var(--background-modifier-border);border-radius:2px;margin-top:8px;">
    <div style="height:3px;width:${pctMeta}%;background:#1D9E75;border-radius:2px;"></div>
  </div>
</div>
<div style="background:var(--background-secondary);border-radius:10px;padding:14px;">
  <div style="font-size:11px;color:var(--text-faint);margin-bottom:6px;text-transform:uppercase;letter-spacing:.06em;">Este mês</div>
  <div style="font-size:26px;font-weight:700;color:var(--text-normal);line-height:1;">${totalMes}</div>
  <div style="font-size:11px;color:var(--text-faint);margin-top:4px;">idas à academia</div>
</div>
<div style="background:var(--background-secondary);border-radius:10px;padding:14px;">
  <div style="font-size:11px;color:var(--text-faint);margin-bottom:6px;text-transform:uppercase;letter-spacing:.06em;">Sequência</div>
  <div style="font-size:26px;font-weight:700;color:${streak > 0 ? '#1D9E75' : 'var(--text-normal)'};line-height:1;">${streak}</div>
  <div style="font-size:11px;color:var(--text-faint);margin-top:4px;">${streak === 1 ? "dia seguido" : "dias seguidos"}</div>
</div>
<div style="background:var(--background-secondary);border-radius:10px;padding:14px;">
  <div style="font-size:11px;color:var(--text-faint);margin-bottom:6px;text-transform:uppercase;letter-spacing:.06em;">Últimos 365d</div>
  <div style="font-size:26px;font-weight:700;color:var(--text-normal);line-height:1;">${totalAno}</div>
  <div style="font-size:11px;color:var(--text-faint);margin-top:4px;">idas à academia</div>
</div>`;
h += `</div>`;

// Heatmap
h += `
<div style="margin-bottom:28px;">
  <div style="display:flex;justify-content:space-between;align-items:center;margin-bottom:12px;">
    <p style="font-size:11px;font-weight:600;letter-spacing:.08em;text-transform:uppercase;color:var(--text-faint);margin:0;">Frequência — últimos 365 dias</p>
    <div style="display:flex;align-items:center;gap:5px;">
      <span style="font-size:10px;color:var(--text-faint);">menos</span>
      <div style="width:10px;height:10px;border-radius:2px;background:var(--background-modifier-border);"></div>
      <div style="width:10px;height:10px;border-radius:2px;background:#1D9E75;"></div>
      <span style="font-size:10px;color:var(--text-faint);">mais</span>
    </div>
  </div>
  <div style="overflow-x:auto;">${buildHeatmap()}</div>
</div>`;

// Seção pesagem
h += `
<div style="display:flex;justify-content:space-between;align-items:center;margin-bottom:12px;">
  <p style="font-size:11px;font-weight:600;letter-spacing:.08em;text-transform:uppercase;color:var(--text-faint);margin:0;">Controle de peso</p>
  <div style="display:flex;align-items:center;gap:14px;font-size:11px;color:var(--text-faint);">
    <span style="display:flex;align-items:center;gap:5px;"><span style="width:12px;height:2px;background:${COR_P1};display:inline-block;border-radius:1px;"></span>${NOME_P1}</span>
    <span style="display:flex;align-items:center;gap:5px;"><span style="width:12px;height:2px;background:${COR_P2};display:inline-block;border-radius:1px;"></span>${NOME_P2}</span>
  </div>
</div>`;

// Cards peso atual
if (ultimaPesagem) {
  h += `<div style="display:grid;grid-template-columns:1fr 1fr;gap:12px;margin-bottom:20px;">`;
  for (const [key, nome, cor] of [["pessoa1",NOME_P1,COR_P1],["pessoa2",NOME_P2,COR_P2]]) {
    const diff = diffPeso(key);
    h += `
    <div style="border:1px solid var(--background-modifier-border);border-radius:10px;padding:14px;">
      <div style="display:flex;justify-content:space-between;margin-bottom:4px;">
        <span style="font-size:12px;font-weight:600;color:var(--text-muted);">${nome}</span>
        <span style="width:10px;height:10px;border-radius:50%;background:${cor};display:inline-block;margin-top:2px;"></span>
      </div>
      <div style="display:flex;align-items:baseline;gap:4px;">
        <span style="font-size:28px;font-weight:700;color:var(--text-normal);">${ultimaPesagem[key] != null ? ultimaPesagem[key].toFixed(1) : "—"}</span>
        <span style="font-size:13px;color:var(--text-faint);">kg</span>
        ${diff !== null ? formatDiff(diff) : ""}
      </div>
      <div style="font-size:11px;color:var(--text-faint);margin-top:4px;">Último registro: ${ultimaPesagem.data}</div>
    </div>`;
  }
  h += `</div>`;
}

// Gráfico
h += `
<div style="border:1px solid var(--background-modifier-border);border-radius:10px;padding:16px;margin-bottom:20px;">
  ${buildGraficoPeso(pesagens.slice(-24))}
</div>`;

// Tabela de registros
if (pesagens.length > 0) {
  h += `
  <div style="border:1px solid var(--background-modifier-border);border-radius:10px;overflow:hidden;margin-bottom:20px;">
    <div style="display:grid;grid-template-columns:1fr 1fr 1fr;background:var(--background-secondary);padding:8px 14px;border-bottom:1px solid var(--background-modifier-border);">
      <span style="font-size:11px;font-weight:600;color:var(--text-faint);">Data</span>
      <span style="font-size:11px;font-weight:600;color:var(--text-faint);">${NOME_P1}</span>
      <span style="font-size:11px;font-weight:600;color:var(--text-faint);">${NOME_P2}</span>
    </div>`;
  for (const reg of pesagens.slice(-8).reverse()) {
    h += `
    <div style="display:grid;grid-template-columns:1fr 1fr 1fr;padding:8px 14px;border-bottom:1px solid var(--background-modifier-border);">
      <span style="font-size:12px;color:var(--text-faint);">${reg.data}</span>
      <span style="font-size:12px;color:var(--text-normal);font-weight:500;">${reg.pessoa1 != null ? reg.pessoa1.toFixed(1)+" kg" : "—"}</span>
      <span style="font-size:12px;color:var(--text-normal);font-weight:500;">${reg.pessoa2 != null ? reg.pessoa2.toFixed(1)+" kg" : "—"}</span>
    </div>`;
  }
  h += `</div>`;
}

h += `</div>`;
dv.el("div", h);

// ============================================================
//  FORMULÁRIO DE NOVA PESAGEM
//  Usa createEl para ter acesso ao contexto do Obsidian
// ============================================================
const form = dv.el("div", "", {
  attr: {
    style: "border:1px solid var(--background-modifier-border);border-radius:10px;padding:16px;margin-top:4px;"
  }
});

// Título do formulário
form.createEl("p", {
  text: "Registrar nova pesagem",
  attr: { style: "font-size:11px;font-weight:600;letter-spacing:.08em;text-transform:uppercase;color:var(--text-faint);margin:0 0 14px;" }
});

// Grid de campos
const grid = form.createEl("div", {
  attr: { style: "display:grid;grid-template-columns:1fr 1fr 1fr;gap:10px;align-items:end;" }
});

// Campo: data
const colData = grid.createEl("div");
colData.createEl("label", {
  text: "Data",
  attr: { style: "font-size:11px;color:var(--text-faint);display:block;margin-bottom:4px;" }
});
const inputData = colData.createEl("input", {
  attr: {
    type: "date",
    value: dataHoje,
    id: "peso-data",
    style: "width:100%;padding:7px 10px;border-radius:7px;border:1px solid var(--background-modifier-border);background:var(--background-primary);color:var(--text-normal);font-size:13px;"
  }
});

// Campo: pessoa 1
const colP1 = grid.createEl("div");
colP1.createEl("label", {
  text: NOME_P1 + " (kg)",
  attr: { style: "font-size:11px;color:var(--text-faint);display:block;margin-bottom:4px;" }
});
const inputP1 = colP1.createEl("input", {
  attr: {
    type: "number",
    step: "0.1",
    placeholder: "ex: 80.5",
    id: "peso-p1",
    style: "width:100%;padding:7px 10px;border-radius:7px;border:1px solid var(--background-modifier-border);background:var(--background-primary);color:var(--text-normal);font-size:13px;"
  }
});

// Campo: pessoa 2
const colP2 = grid.createEl("div");
colP2.createEl("label", {
  text: NOME_P2 + " (kg)",
  attr: { style: "font-size:11px;color:var(--text-faint);display:block;margin-bottom:4px;" }
});
const inputP2 = colP2.createEl("input", {
  attr: {
    type: "number",
    step: "0.1",
    placeholder: "ex: 65.0",
    id: "peso-p2",
    style: "width:100%;padding:7px 10px;border-radius:7px;border:1px solid var(--background-modifier-border);background:var(--background-primary);color:var(--text-normal);font-size:13px;"
  }
});

// Área de feedback + botão
const footer = form.createEl("div", {
  attr: { style: "display:flex;align-items:center;gap:12px;margin-top:12px;" }
});

const feedback = footer.createEl("span", {
  attr: { style: "font-size:12px;color:var(--text-faint);flex:1;" }
});

const btnSalvar = footer.createEl("button", {
  text: "Salvar pesagem",
  attr: {
    style: "background:var(--interactive-accent);color:var(--text-on-accent);border:none;border-radius:8px;padding:9px 18px;font-size:13px;font-weight:600;cursor:pointer;"
  }
});

// Lógica de salvar
btnSalvar.addEventListener("click", async () => {
  const data = inputData.value.trim();
  const p1raw = inputP1.value.trim();
  const p2raw = inputP2.value.trim();

  // Validação
  if (!data) {
    feedback.textContent = "Informe a data.";
    feedback.style.color = "var(--text-error)";
    return;
  }
  if (!p1raw && !p2raw) {
    feedback.textContent = `Informe o peso de pelo menos uma pessoa.`;
    feedback.style.color = "var(--text-error)";
    return;
  }

  const p1 = p1raw !== "" ? parseFloat(p1raw) : null;
  const p2 = p2raw !== "" ? parseFloat(p2raw) : null;

  if (p1raw !== "" && isNaN(p1)) {
    feedback.textContent = `Peso de ${NOME_P1} inválido.`;
    feedback.style.color = "var(--text-error)";
    return;
  }
  if (p2raw !== "" && isNaN(p2)) {
    feedback.textContent = `Peso de ${NOME_P2} inválido.`;
    feedback.style.color = "var(--text-error)";
    return;
  }

  btnSalvar.disabled = true;
  feedback.textContent = "Salvando...";
  feedback.style.color = "var(--text-faint)";

  try {
    // Lê o JSON atual (ou inicia array vazio)
    let registros = [];
    try {
      const raw = await app.vault.adapter.read(JSON_PATH);
      registros = JSON.parse(raw);
    } catch(e) { registros = []; }

    // Remove entrada com a mesma data se existir (sobrescreve)
    registros = registros.filter(r => r.data !== data);

    // Adiciona novo registro
    const novoRegistro = { data };
    if (p1 !== null) novoRegistro.pessoa1 = p1;
    if (p2 !== null) novoRegistro.pessoa2 = p2;
    registros.push(novoRegistro);

    // Ordena por data
    registros.sort((a, b) => new Date(a.data) - new Date(b.data));

    // Salva de volta no JSON
    await app.vault.adapter.write(JSON_PATH, JSON.stringify(registros, null, 2));

    feedback.textContent = `✓ Pesagem de ${data} salva com sucesso!`;
    feedback.style.color = "#1D9E75";

    // Limpa campos
    inputP1.value = "";
    inputP2.value = "";
    inputData.value = dataHoje;

    // Recarrega a view após 1.5s para refletir os novos dados
    setTimeout(() => {
      dv.index.touch(app.vault.getAbstractFileByPath(dv.currentFilePath));
    }, 1500);

  } catch(err) {
    feedback.textContent = `Erro ao salvar: ${err.message}`;
    feedback.style.color = "var(--text-error)";
    btnSalvar.disabled = false;
  }
});
```

---

## Arquivo `Saúde/pesagem.json`

Crie este arquivo antes de usar o formulário. Pode iniciar vazio ou com dados históricos:

```json
[]
```

## Configuração rápida

No topo do código, edite as constantes:

```js
const NOME_P1      = "Seu nome";
const NOME_P2      = "Nome da outra pessoa";
const META_SEMANAL = 4;  // idas à academia por semana
```