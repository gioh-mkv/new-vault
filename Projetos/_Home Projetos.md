---
## tags: [homepage, projetos] cssclasses: [home-page]
---


```dataviewjs
// HOME PROJETOS - DataviewJS
// Visualizacao de projetos com status, linguagem, GitHub e formulario

// ---------- HELPERS ----------
const hoje     = new Date();
const pad      = n => String(n).padStart(2, "0");
const dataHoje = hoje.getFullYear()+"-"+pad(hoje.getMonth()+1)+"-"+pad(hoje.getDate());
const meses    = ["jan","fev","mar","abr","mai","jun","jul","ago","set","out","nov","dez"];

function toDate(val) {
  if (!val) return null;
  if (val instanceof Date) return val;
  if (val && val.toJSDate) return val.toJSDate();
  return new Date(String(val));
}
function ilink(path) {
  return 'data-href="' + path + '" href="' + path + '" class="internal-link"';
}
function fmtData(val) {
  const d = toDate(val);
  if (!d || isNaN(d)) return "-";
  return d.getDate() + " " + meses[d.getMonth()] + " " + d.getFullYear();
}
function diasAtras(val) {
  const d = toDate(val);
  if (!d || isNaN(d)) return null;
  return Math.floor((hoje - d) / 86400000);
}

// ---------- DADOS ----------
const projetos = dv.pages('"Projetos"')
  .where(p => p.file.name !== "_Home Projetos")
  .sort(p => p.file.mtime, "desc");

// ---------- STATUS CONFIG ----------
const STATUS_CFG = {
  "ativo":               { bg:"#E6F1FB", cor:"#185FA5", label:"Ativo"           },
  "pausado":             { bg:"#FAEEDA", cor:"#854F0B", label:"Pausado"         },
  "concluido":           { bg:"#E1F5EE", cor:"#085041", label:"Concluido"       },
  "concluido":           { bg:"#E1F5EE", cor:"#085041", label:"Concluido"       },
  "arquivado":           { bg:"#F1EFE8", cor:"#5F5E5A", label:"Arquivado"       },
  "em desenvolvimento":  { bg:"#EEEDFE", cor:"#534AB7", label:"Em dev."         },
};

// ---------- LINGUAGEM CORES ----------
const LANG_COR = {
  "JavaScript":"#EF9F27","TypeScript":"#378ADD","Python":"#185FA5",
  "Rust":"#D85A30","Go":"#5DCAA5","Java":"#E24B4A",
  "C":"#888780","C++":"#534AB7","C#":"#7F77DD",
  "HTML/CSS":"#D85A30","PHP":"#7F77DD","Ruby":"#E24B4A",
  "Swift":"#EF9F27","Kotlin":"#7F77DD","Dart":"#5DCAA5",
  "Shell":"#3B6D11","Vue":"#1D9E75","React":"#5DCAA5","Outros":"#888780",
};

function badgeStatus(status) {
  const key = (status||"").toLowerCase();
  const cfg = STATUS_CFG[key] || { bg:"#F1EFE8", cor:"#5F5E5A", label: status||"-" };
  return '<span style="font-size:10px;padding:2px 8px;border-radius:4px;font-weight:600;background:'+cfg.bg+';color:'+cfg.cor+';">'+cfg.label+'</span>';
}

function dotLinguagem(lang) {
  if (!lang) return "";
  const cor = LANG_COR[lang] || LANG_COR["Outros"];
  return '<span style="display:inline-flex;align-items:center;gap:5px;font-size:11px;color:var(--text-faint);"><span style="width:9px;height:9px;border-radius:50%;background:'+cor+';flex-shrink:0;"></span>'+lang+'</span>';
}

// ---------- CONTAGENS ----------
const ativos     = projetos.where(p => ["ativo","em desenvolvimento"].includes((p.status||"").toLowerCase())).length;
const concluidos = projetos.where(p => ["concluido","concluido"].includes((p.status||"").toLowerCase())).length;
const pausados   = projetos.where(p => (p.status||"").toLowerCase() === "pausado").length;

const grupos = [
  { label: "Em andamento", ids: ["ativo","em desenvolvimento"] },
  { label: "Pausados",     ids: ["pausado"] },
  { label: "Concluidos",   ids: ["concluido","concluido"] },
  { label: "Arquivados",   ids: ["arquivado"] },
];

// ---------- BUILD HTML ----------
let h = '<div style="max-width:860px;">';

h += '<div style="padding-bottom:16px;border-bottom:1px solid var(--background-modifier-border);margin-bottom:24px;display:flex;align-items:flex-start;justify-content:space-between;">';
h += '<div><div style="font-size:22px;font-weight:700;color:var(--text-normal);">Projetos</div>';
h += '<div style="font-size:13px;color:var(--text-faint);margin-top:4px;">'+dataHoje+'</div></div>';
h += '<a '+ilink("Home")+' style="font-size:12px;color:var(--interactive-accent);text-decoration:none;margin-top:4px;">voltar para Home</a>';
h += '</div>';

// Cards de resumo
h += '<div style="display:grid;grid-template-columns:repeat(4,minmax(0,1fr));gap:10px;margin-bottom:28px;">';
const resumos = [
  { lbl:"Total",      val:projetos.length, cor:"var(--text-normal)", sub:"projetos"     },
  { lbl:"Ativos",     val:ativos,          cor:"#185FA5",            sub:"em andamento"  },
  { lbl:"Concluidos", val:concluidos,      cor:"#1D9E75",            sub:"finalizados"   },
  { lbl:"Pausados",   val:pausados,        cor:"var(--text-normal)", sub:"em espera"     },
];
for (const r of resumos) {
  h += '<div style="background:var(--background-secondary);border-radius:10px;padding:14px;">';
  h += '<div style="font-size:11px;color:var(--text-faint);margin-bottom:6px;text-transform:uppercase;letter-spacing:.06em;">'+r.lbl+'</div>';
  h += '<div style="font-size:26px;font-weight:700;color:'+r.cor+';line-height:1;">'+r.val+'</div>';
  h += '<div style="font-size:11px;color:var(--text-faint);margin-top:4px;">'+r.sub+'</div>';
  h += '</div>';
}
h += '</div>';

// Grid de projetos
if (projetos.length === 0) {
  h += '<div style="border:1px dashed var(--background-modifier-border);border-radius:10px;padding:32px;text-align:center;margin-bottom:24px;">';
  h += '<div style="font-size:14px;font-weight:600;color:var(--text-normal);margin-bottom:6px;">Nenhum projeto cadastrado</div>';
  h += '<div style="font-size:13px;color:var(--text-faint);">Use o formulario abaixo para registrar seu primeiro projeto.</div>';
  h += '</div>';
} else {
  for (const grupo of grupos) {
    const lista = projetos.where(p => grupo.ids.includes((p.status||"").toLowerCase()));
    if (lista.length === 0) continue;

    h += '<p style="font-size:11px;font-weight:600;color:var(--text-faint);text-transform:uppercase;letter-spacing:.06em;margin:0 0 10px;">'+grupo.label+'</p>';
    h += '<div style="display:grid;grid-template-columns:repeat(2,minmax(0,1fr));gap:10px;margin-bottom:24px;">';

    for (const p of lista) {
      const nome      = p.nome      || p.file.name;
      const desc      = p.descricao || "";
      const linguagem = p.linguagem || "";
      const ghurl     = p.github_url || "";
      const ultima    = p.ultima_atualizacao || p.file.mtime;
      const da        = diasAtras(ultima);
      const daStr     = da === null ? "-" : da === 0 ? "hoje" : da === 1 ? "ontem" : "ha " + da + " dias";
      const deadline  = p.deadline  || null;
      const progresso = p.progresso != null ? p.progresso : null;
      const techs     = Array.isArray(p.tecnologias) ? p.tecnologias : (p.tecnologias ? [p.tecnologias] : []);

      h += '<a '+ilink(p.file.path)+' style="display:block;border:1px solid var(--background-modifier-border);border-radius:10px;padding:14px;text-decoration:none;background:var(--background-primary);">';

      h += '<div style="display:flex;justify-content:space-between;align-items:flex-start;gap:8px;margin-bottom:8px;">';
      h += '<div style="font-size:14px;font-weight:600;color:var(--text-normal);line-height:1.3;">'+nome+'</div>';
      h += badgeStatus(p.status);
      h += '</div>';

      if (desc) {
        h += '<div style="font-size:12px;color:var(--text-faint);line-height:1.5;margin-bottom:8px;">'+desc+'</div>';
      }

      h += '<div style="display:flex;align-items:center;gap:12px;flex-wrap:wrap;margin-bottom:8px;">';
      h += dotLinguagem(linguagem);
      for (const t of techs.slice(0,3)) {
        h += '<span style="font-size:10px;padding:1px 7px;border-radius:4px;background:var(--background-modifier-border);color:var(--text-faint);">'+t+'</span>';
      }
      h += '</div>';

      if (progresso !== null) {
        const pct = Math.min(100, progresso);
        h += '<div style="margin-bottom:8px;">';
        h += '<div style="display:flex;justify-content:space-between;margin-bottom:3px;">';
        h += '<span style="font-size:10px;color:var(--text-faint);">Progresso</span>';
        h += '<span style="font-size:10px;color:var(--text-faint);">'+pct+'%</span>';
        h += '</div>';
        h += '<div style="height:3px;background:var(--background-modifier-border);border-radius:2px;">';
        h += '<div style="height:3px;width:'+pct+'%;background:#378ADD;border-radius:2px;"></div>';
        h += '</div></div>';
      }

      h += '<div style="display:flex;align-items:center;justify-content:space-between;margin-top:8px;padding-top:8px;border-top:1px solid var(--background-modifier-border);">';
      h += '<span style="font-size:11px;color:var(--text-faint);">Atualizado: '+daStr+'</span>';
      if (deadline) h += '<span style="font-size:11px;color:var(--text-faint);">Prazo: '+fmtData(deadline)+'</span>';
      if (ghurl)    h += '<span style="font-size:11px;color:var(--interactive-accent);">GitHub</span>';
      h += '</div>';

      h += '</a>';
    }
    h += '</div>';
  }
}

h += '</div>';
dv.el("div", h);

// ================================================================
// FORMULARIO DE NOVO PROJETO
// ================================================================
const root = dv.el("div", "", { attr: { style: "max-width:860px;" } });

const sepForm = root.createEl("div", { attr: { style: "border-top:1px solid var(--background-modifier-border);padding-top:20px;margin-top:4px;" } });

const formHeader = sepForm.createEl("div", { attr: { style: "display:flex;align-items:center;justify-content:space-between;margin-bottom:0;" } });
formHeader.createEl("p", { text: "Novo projeto", attr: { style: "font-size:11px;font-weight:600;letter-spacing:.08em;text-transform:uppercase;color:var(--text-faint);margin:0;" } });
const btnToggle = formHeader.createEl("button", { text: "+ Abrir formulario", attr: { style: "padding:6px 14px;border:1px solid var(--background-modifier-border);border-radius:7px;background:var(--background-secondary);color:var(--text-muted);font-size:12px;cursor:pointer;" } });

const formBody = sepForm.createEl("div", { attr: { style: "display:none;margin-top:14px;" } });

btnToggle.addEventListener("click", () => {
  const aberto = formBody.style.display !== "none";
  formBody.style.display = aberto ? "none" : "block";
  btnToggle.textContent  = aberto ? "+ Abrir formulario" : "- Fechar";
});

const fS = "width:100%;padding:7px 10px;border-radius:7px;border:1px solid var(--background-modifier-border);background:var(--background-primary);color:var(--text-normal);font-size:13px;";
const lS = "font-size:11px;color:var(--text-faint);display:block;margin-bottom:4px;";

const linha1 = formBody.createEl("div", { attr: { style: "display:grid;grid-template-columns:1fr 150px 160px;gap:10px;margin-bottom:10px;" } });

const cNome = linha1.createEl("div");
cNome.createEl("label", { text: "Nome do projeto", attr: { style: lS } });
const inputNome = cNome.createEl("input", { attr: { type: "text", placeholder: "ex: Portfolio, API REST...", style: fS } });

const cLang = linha1.createEl("div");
cLang.createEl("label", { text: "Linguagem", attr: { style: lS } });
const selLang = cLang.createEl("select", { attr: { style: fS } });
const LINGUAGENS = ["","JavaScript","TypeScript","Python","Rust","Go","Java","C","C++","C#","HTML/CSS","PHP","Ruby","Swift","Kotlin","Dart","Shell","Vue","React","Outros"];
for (const l of LINGUAGENS) selLang.createEl("option", { text: l || "-- selecione --", attr: { value: l } });

const cStatus = linha1.createEl("div");
cStatus.createEl("label", { text: "Status", attr: { style: lS } });
const selStatus = cStatus.createEl("select", { attr: { style: fS } });
const STATUS_OPTS = [["ativo","Ativo"],["em desenvolvimento","Em desenvolvimento"],["pausado","Pausado"],["concluido","Concluido"],["arquivado","Arquivado"]];
for (const [val, lbl] of STATUS_OPTS) selStatus.createEl("option", { text: lbl, attr: { value: val } });

const linha2 = formBody.createEl("div", { attr: { style: "margin-bottom:10px;" } });
linha2.createEl("label", { text: "Descricao", attr: { style: lS } });
const inputDesc = linha2.createEl("textarea", { attr: { placeholder: "Descreva brevemente o projeto...", style: fS + "min-height:60px;resize:vertical;line-height:1.5;" } });

const linha3 = formBody.createEl("div", { attr: { style: "display:grid;grid-template-columns:1fr 150px 130px;gap:10px;margin-bottom:10px;" } });

const cGH = linha3.createEl("div");
cGH.createEl("label", { text: "URL do GitHub", attr: { style: lS } });
const inputGH = cGH.createEl("input", { attr: { type: "url", placeholder: "https://github.com/usuario/repo", style: fS } });

const cDL = linha3.createEl("div");
cDL.createEl("label", { text: "Prazo (opcional)", attr: { style: lS } });
const inputDL = cDL.createEl("input", { attr: { type: "date", style: fS } });

const cProg = linha3.createEl("div");
cProg.createEl("label", { text: "Progresso (%)", attr: { style: lS } });
const inputProg = cProg.createEl("input", { attr: { type: "number", min: "0", max: "100", step: "1", placeholder: "0", style: fS } });

const linha4 = formBody.createEl("div", { attr: { style: "margin-bottom:14px;" } });
linha4.createEl("label", { text: "Tecnologias (separadas por virgula)", attr: { style: lS } });
const inputTechs = linha4.createEl("input", { attr: { type: "text", placeholder: "ex: React, Node.js, PostgreSQL", style: fS } });

const fFooter   = formBody.createEl("div", { attr: { style: "display:flex;align-items:center;gap:12px;" } });
const fbk       = fFooter.createEl("span", { attr: { style: "font-size:12px;color:var(--text-faint);flex:1;" } });
const btnSalvar = fFooter.createEl("button", { text: "Criar projeto", attr: { style: "background:var(--interactive-accent);color:var(--text-on-accent);border:none;border-radius:8px;padding:9px 20px;font-size:13px;font-weight:600;cursor:pointer;" } });

btnSalvar.addEventListener("click", async () => {
  const nome = inputNome.value.trim();
  if (!nome) { fbk.textContent = "Informe o nome do projeto."; fbk.style.color = "var(--text-error)"; return; }

  btnSalvar.disabled = true;
  fbk.textContent = "Criando nota...";
  fbk.style.color = "var(--text-faint)";

  try {
    const techs = inputTechs.value.trim()
      ? inputTechs.value.split(",").map(t => t.trim()).filter(Boolean)
      : [];

    const linhasFM = [
      "---",
      'nome: "' + nome + '"',
      'descricao: "' + inputDesc.value.trim().replace(/"/g, "'") + '"',
      'linguagem: "' + selLang.value + '"',
      'status: ' + selStatus.value,
      'github_url: "' + inputGH.value.trim() + '"',
      'ultima_atualizacao: ' + dataHoje,
      'deadline: ' + (inputDL.value || ""),
      'progresso: ' + (inputProg.value || 0),
      'tecnologias: [' + techs.map(t => '"' + t + '"').join(", ") + ']',
      'pendente: false',
      'setor: Projetos',
      'tags: [projeto]',
      "---",
      "",
      "## " + nome,
      "",
      "## Objetivo",
      "",
      "## Tecnologias",
      "",
      "## Progresso",
      "",
      "## Links",
      inputGH.value.trim() ? "- [Repositorio GitHub](" + inputGH.value.trim() + ")" : "",
      "",
      "## Notas",
      "",
    ];
    const frontmatter = linhasFM.join("\n");
    const caminho = "Projetos/" + nome + ".md";

    const existe = app.vault.getAbstractFileByPath(caminho);
    if (existe) {
      fbk.textContent = 'Ja existe um projeto com o nome "' + nome + '".';
      fbk.style.color = "var(--text-error)";
      btnSalvar.disabled = false;
      return;
    }

    await app.vault.create(caminho, frontmatter);
    fbk.textContent = 'Projeto "' + nome + '" criado!';
    fbk.style.color = "#1D9E75";

    inputNome.value = ""; inputDesc.value = ""; inputGH.value = "";
    inputDL.value = ""; inputProg.value = ""; inputTechs.value = "";

    setTimeout(async () => {
      await app.workspace.openLinkText(caminho, "", false);
      dv.index.touch(app.vault.getAbstractFileByPath(dv.currentFilePath));
    }, 1000);

  } catch(err) {
    fbk.textContent = "Erro: " + err.message;
    fbk.style.color = "var(--text-error)";
    btnSalvar.disabled = false;
  }
});

// ================================================================
// ANOTACOES
// ================================================================
const PATH_NOTAS_PROJ = "Projetos/anotacoes.json";
let anotacoesProj = "";
try {
  const raw = await app.vault.adapter.read(PATH_NOTAS_PROJ);
  anotacoesProj = JSON.parse(raw).texto || "";
} catch(e) {}

const sepNotas = root.createEl("div", { attr: { style: "margin-top:28px;border-top:1px solid var(--background-modifier-border);padding-top:20px;" } });
sepNotas.createEl("p", { text: "Anotacoes", attr: { style: "font-size:11px;font-weight:600;letter-spacing:.08em;text-transform:uppercase;color:var(--text-faint);margin:0 0 10px;" } });

const textarea = sepNotas.createEl("textarea", { attr: {
  placeholder: "Ideias de projetos, links uteis, referencias...",
  style: "width:100%;min-height:100px;padding:12px;border-radius:10px;border:1px solid var(--background-modifier-border);background:var(--background-secondary);color:var(--text-normal);font-size:13px;font-family:var(--font-interface);line-height:1.6;resize:vertical;box-sizing:border-box;"
}});
textarea.value = anotacoesProj;

const notasFooter    = sepNotas.createEl("div", { attr: { style: "display:flex;align-items:center;gap:10px;margin-top:8px;" } });
const notasFbk       = notasFooter.createEl("span", { attr: { style: "font-size:12px;color:var(--text-faint);flex:1;" } });
const btnSalvarNotas = notasFooter.createEl("button", { text: "Salvar anotacoes", attr: { style: "padding:7px 16px;border:1px solid var(--background-modifier-border);border-radius:8px;background:var(--background-secondary);color:var(--text-normal);font-size:12px;font-weight:600;cursor:pointer;" } });

btnSalvarNotas.addEventListener("click", async () => {
  try {
    await app.vault.adapter.write(PATH_NOTAS_PROJ, JSON.stringify({ texto: textarea.value }, null, 2));
    notasFbk.textContent = "Salvo!";
    notasFbk.style.color = "#1D9E75";
    setTimeout(() => { notasFbk.textContent = ""; }, 2000);
  } catch(err) {
    notasFbk.textContent = "Erro: " + err.message;
    notasFbk.style.color = "var(--text-error)";
  }
});
```

---

## Frontmatter da nota de projeto

```yaml
---
nome: Nome do Projeto
descricao: Descricao breve
linguagem: TypeScript
status: ativo
github_url: https://github.com/usuario/repo
ultima_atualizacao: 2025-04-19
deadline: 
progresso: 30
tecnologias: ["React", "Node.js", "PostgreSQL"]
pendente: false
setor: Projetos
tags: [projeto]
---
```

### Status disponiveis

- `ativo`
- `em desenvolvimento`
- `pausado`
- `concluido`
- `arquivado`