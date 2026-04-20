# new-vault

# Guia do Cofre -- Documentacao e Configuracao

> Este arquivo serve como referencia central do seu cofre no Obsidian. Leia com calma antes de comecar a usar -- vai poupar muito tempo.

---

## O que e este cofre

Este cofre foi projetado para ser um **sistema de controle pessoal completo**, dividido em sete setores da sua vida. Cada setor tem uma homepage interativa construida em DataviewJS que le seus dados, exibe visualizacoes e permite registrar informacoes sem sair do Obsidian.

A ideia central e simples: **tudo em um so lugar, sem depender de apps externos**.

---

## Estrutura de pastas

```
MeuCofre/
|
|-- Home.md                        <- homepage principal
|-- DailyNotes/                    <- notas diarias (YYYY-MM-DD.md)
|-- _Templates/                    <- templates do Templater
|   |-- Nota Diaria.md
|   |-- Evento Universidade.md
|   |-- Disciplina.md
|   |-- Projeto.md
|   |-- Freelance.md
|   |-- Concurso.md
|   |-- Curso.md
|   |-- Estudo Independente.md
|   `-- Area de Conhecimento.md
|-- _Attachments/                  <- imagens e arquivos anexados
|
|-- Pessoal/
|   |-- _Home Pessoal.md
|   |-- Biblioteca/
|   |   |-- Leitura em andamento/
|   |   |-- Leituras concluidas/
|   |   `-- Lista de desejos/
|   `-- frases.json                <- nao necessario (frases embutidas no codigo)
|
|-- Saude/
|   |-- _Home Saude.md
|   |-- pesagem.json               <- registros de peso de duas pessoas
|   `-- academia.json              <- nao necessario (lido das notas diarias)
|
|-- Universidade/
|   |-- _Home Universidade.md
|   |-- Eventos/                   <- uma nota por evento (prova, palestra...)
|   `-- Disciplinas/
|       |-- Em andamento/          <- semestre atual
|       `-- Semestres anteriores/
|           |-- 2024.1/
|           `-- 2024.2/
|
|-- Conhecimento Independente/
|   |-- _Home Conhecimento.md
|   |-- Concursos Publicos/
|   |-- Cursos Profissionalizantes/
|   |-- Estudos Independentes/
|   `-- Areas de Conhecimento/
|
|-- Financas/
|   |-- _Home Financas.md
|   |-- transacoes.json            <- todas as transacoes financeiras
|   |-- metas.json                 <- limites mensais por categoria
|   `-- anotacoes.json             <- bloco de notas de financas
|
|-- Projetos/
|   |-- _Home Projetos.md
|   |-- anotacoes.json             <- bloco de notas de projetos
|   `-- [Nome do Projeto].md       <- uma nota por projeto
|
`-- Profissional/
    |-- _Home Profissional.md
    |-- anotacoes.json             <- bloco de notas de freelances
    `-- Freelances/
        `-- [Nome do Cliente].md   <- uma nota por trabalho
```

---

## Plugins necessarios

### Obrigatorios

Sem estes plugins, as homepages nao funcionam.

#### Dataview

- **O que faz:** permite consultar e exibir dados de notas usando JavaScript. E o motor de tudo.
- **Instalacao:** Configuracoes > Plugins da comunidade > Procurar "Dataview" > Instalar > Ativar
- **Configuracao obrigatoria:** apos ativar, va em Configuracoes > Dataview e habilite a opcao **"Enable JavaScript Queries"** (DataviewJS). Sem isso, os blocos `dataviewjs` nao executam.

#### Templater

- **O que faz:** aplica templates automaticamente ao criar notas em pastas especificas, preenchendo frontmatter com data, campos padrao e estrutura de secoes.
- **Instalacao:** Configuracoes > Plugins da comunidade > Procurar "Templater" > Instalar > Ativar
- **Configuracao:** veja a secao "Configurando o Templater" mais abaixo.

### Recomendados

Nao obrigatorios, mas melhoram muito a experiencia.

#### Calendar

- **O que faz:** adiciona um calendario na barra lateral para navegar entre notas diarias.
- **Instalacao:** Configuracoes > Plugins da comunidade > Procurar "Calendar" > Instalar > Ativar
- **Configuracao:** apos ativar, configure para usar a pasta `DailyNotes` como local das notas.

#### Style Settings

- **O que faz:** permite personalizar temas com opcoes visuais sem editar CSS manualmente.
- **Instalacao:** Configuracoes > Plugins da comunidade > Procurar "Style Settings" > Instalar > Ativar

---

## Configurando o Templater

O Templater e responsavel por aplicar o frontmatter correto automaticamente em cada pasta.

### Passo 1 -- Configuracoes basicas

Configuracoes > Templater:

- **Template folder location:** `_Templates`
- Ativar: **"Trigger Templater on new file creation"**

### Passo 2 -- Folder Templates

Ainda em Configuracoes > Templater, role ate **"Folder Templates"** e adicione:

|Pasta|Template|
|---|---|
|`DailyNotes`|`_Templates/Nota Diaria`|
|`Universidade/Eventos`|`_Templates/Evento Universidade`|
|`Universidade/Disciplinas/Em andamento`|`_Templates/Disciplina`|
|`Projetos`|`_Templates/Projeto`|
|`Profissional/Freelances`|`_Templates/Freelance`|
|`Conhecimento Independente/Concursos Publicos`|`_Templates/Concurso`|
|`Conhecimento Independente/Cursos Profissionalizantes`|`_Templates/Curso`|
|`Conhecimento Independente/Estudos Independentes`|`_Templates/Estudo Independente`|
|`Conhecimento Independente/Areas de Conhecimento`|`_Templates/Area de Conhecimento`|

### Passo 3 -- Templates das notas diarias

Crie o arquivo `_Templates/Nota Diaria.md` com este conteudo:

```
---
date: <% tp.date.now("YYYY-MM-DD") %>
titulo: <% tp.date.now("dddd, D [de] MMMM") %>
academia: false
humor: neutro
tarefas_concluidas: 0
pendente: false
setor: Pessoal
tags: [diario]
---

## O que aconteceu hoje


## Tarefas
- [ ] 

## Reflexao

```

---

## Arquivos JSON necessarios

Alguns setores usam arquivos JSON para armazenar dados estruturados. Crie estes arquivos antes de usar as homepages:

|Arquivo|Conteudo inicial|Usado por|
|---|---|---|
|`Saude/pesagem.json`|`[]`|Home Saude|
|`Financas/transacoes.json`|`[]`|Home Financas|
|`Financas/metas.json`|veja abaixo|Home Financas|
|`Financas/anotacoes.json`|`{ "texto": "" }`|Home Financas|
|`Projetos/anotacoes.json`|`{ "texto": "" }`|Home Projetos|
|`Profissional/anotacoes.json`|`{ "texto": "" }`|Home Profissional|

### Conteudo inicial do `metas.json`

Edite os valores conforme sua realidade financeira:

```json
{
  "Alimentacao":  800,
  "Transporte":   300,
  "Moradia":     1500,
  "Lazer":        400,
  "Saude":        200,
  "Assinaturas":  150
}
```

---

## Como cada setor funciona

### Home principal

Exibe um painel geral com atalhos para todos os setores, proximos eventos (lidos de Universidade, Projetos e Profissional), pendencias (qualquer nota com `pendente: true` no frontmatter), e a nota diaria mais recente. O botao "Criar nota de hoje" abre a nota do dia atual em `DailyNotes/`.

### Pessoal

Exibe uma frase do dia (rotacionada pelo numero do dia do ano, muda a cada dia automaticamente), a biblioteca de leituras organizada por status (lendo, concluido, quero ler) com barra de progresso, e o historico recente de notas diarias. Para cadastrar um livro, crie uma nota em `Pessoal/Biblioteca/` com o frontmatter abaixo:

```yaml
---
titulo: Nome do Livro
autor: Nome do Autor
genero: Romance
status: lendo
paginas: 320
paginas_lidas: 150
nota: 4
---
```

### Saude

Exibe um heatmap de frequencia na academia (igual ao GitHub) gerado a partir do campo `academia: true` nas notas diarias, cards de resumo semanal e streak atual, e o grafico de linha de evolucao de peso para duas pessoas. O peso e registrado via formulario inline que salva diretamente no `pesagem.json`. Para marcar presenca na academia, basta abrir a nota diaria do dia e mudar `academia: false` para `academia: true`.

### Universidade

Exibe proximos eventos com destaque de urgencia (vermelho para eventos em ate 3 dias), disciplinas do semestre atual em galeria, e semestres anteriores agrupados com media calculada automaticamente. Eventos sao notas em `Universidade/Eventos/` com campo `date` no frontmatter.

### Conhecimento Independente

Quatro abas -- Concursos, Cursos, Estudos e Areas -- cada uma consultando sua respectiva pasta. Concursos com prova proxima ficam destacados em vermelho. As abas funcionam via JavaScript puro sem recarregar a pagina.

### Financas

Tres abas. **Mensal:** navegacao livre por qualquer mes e ano com selectores e setas, pizza de gastos por categoria e tabela de transacoes com edicao e exclusao inline. **Anual:** grafico de barras dos 12 meses e tabela de resumo. **Metas:** barras de progresso por categoria com edicao inline de valores. Formulario de nova transacao e bloco de anotacoes persistido em JSON.

### Projetos

Grid de projetos por status com linguagem colorida (paleta identica ao GitHub), barra de progresso opcional e contador de dias desde a ultima atualizacao. Formulario de novo projeto cria a nota com frontmatter e estrutura de secoes automaticamente, depois abre a nota criada.

### Profissional

Lista de freelances agrupada por status (Em andamento, Prospectos, Concluidos, Cancelados), grafico de ganhos mensais do ano, painel de atualizacao rapida de status (sem abrir a nota), formulario de novo trabalho e bloco de anotacoes. Ao marcar um trabalho como "concluido", o campo `data_fim` e preenchido automaticamente.

---

## Frontmatter de referencia

### Pendencias (qualquer nota)

Para que uma nota apareca no card de pendencias da Home principal, adicione ao frontmatter:

```yaml
pendente: true
setor: Financas   # Pessoal | Saude | Universidade | Financas | Projetos | Profissional
```

### Nota diaria

```yaml
---
date: 2025-04-19
titulo: Sabado produtivo
academia: false
humor: bom
tarefas_concluidas: 3
pendente: false
setor: Pessoal
tags: [diario]
---
```

### Evento de universidade

```yaml
---
date: 2025-04-25
type: prova
disciplina: Calculo II
local: Sala 201
status: pendente
tags: [universidade, evento]
pendente: true
setor: Universidade
---
```

### Disciplina

```yaml
---
nome: Calculo II
professor: Prof. Silva
creditos: 4
horario: Seg/Qua 10h
semestre: 2025.1
status: cursando
nota_final:
tags: [universidade, disciplina]
---
```

### Projeto

```yaml
---
nome: Portfolio Pessoal
descricao: Site portfolio com projetos e blog
linguagem: TypeScript
status: ativo
github_url: https://github.com/usuario/portfolio
ultima_atualizacao: 2025-04-19
deadline:
progresso: 60
tecnologias: ["React", "Next.js", "Tailwind"]
pendente: false
setor: Projetos
tags: [projeto]
---
```

### Freelance

```yaml
---
cliente: Empresa XYZ
descricao: Desenvolvimento de landing page
status: em andamento
valor: 2500.00
data_inicio: 2025-04-01
data_fim:
prazo: 2025-05-15
pendente: true
setor: Profissional
tags: [freelance]
---
```

---

## Solucao de problemas comuns

### A homepage exibe HTML em vez de renderizar

**Causa:** o arquivo `.md` contem caracteres Unicode fora do ASCII (como travessoes `--`, aspas curvas, ou acentos nos comentarios do codigo).

**Solucao:** abra o arquivo num editor de texto puro (VS Code, Notepad++), copie todo o conteudo, cole num novo arquivo `.md` no Obsidian. Isso elimina os caracteres invisiveis. Os arquivos entregues neste projeto foram validados como ASCII puro para evitar esse problema.

### O DataviewJS nao executa (mostra o codigo)

**Causa:** a opcao "Enable JavaScript Queries" nao esta ativada no plugin Dataview.

**Solucao:** Configuracoes > Dataview > ativar "Enable JavaScript Queries".

### Os links das homepages nao funcionam (nao abrem a nota)

**Causa:** o Obsidian nao intercepta `onclick` em HTML injetado pelo Dataview. Os links precisam usar `class="internal-link"` com `data-href`.

**Solucao:** os arquivos entregues ja usam o metodo correto. Se voce modificar algum link manualmente, use o formato:

```html
<a data-href="Pasta/Nota" href="Pasta/Nota" class="internal-link">texto</a>
```

### O formulario salva mas a view nao atualiza

**Causa:** o `dv.index.touch()` precisa de um arquivo valido para disparar o re-render.

**Solucao:** aguarde 1-2 segundos apos salvar. Se nao atualizar, feche e reabra a nota. Isso e uma limitacao do Dataview, nao um bug do codigo.

### O heatmap de academia aparece vazio

**Causa:** as notas diarias nao tem o campo `academia: true` ou estao em pasta diferente de `DailyNotes`.

**Solucao:** verifique se o template da nota diaria inclui `academia: false` no frontmatter e se a pasta configurada no Templater e exatamente `DailyNotes` (sem barra final, sem espaco).

### O grafico de peso nao aparece

**Causa:** o arquivo `Saude/pesagem.json` nao existe ou esta vazio.

**Solucao:** crie o arquivo com conteudo `[]` e adicione ao menos dois registros pelo formulario da Home Saude para o grafico ter pontos suficientes para tracelar a linha.

---

## Dicas de uso

**Crie sempre pelas homepages.** Os formularios embutidos garantem que o frontmatter esteja correto. Criar notas manualmente exige que voce lembre de todos os campos necessarios.

**Use o campo `pendente: true` com frequencia.** E a forma mais rapida de fazer algo aparecer no painel de pendencias da Home sem precisar de um app separado de tarefas.

**Mantenha o `ultima_atualizacao` atualizado nos projetos.** O card de projeto exibe "ha X dias" com base nesse campo -- e um lembrete visual de projetos que voce nao toca ha tempo.

**Para financas, registre no dia.** O formulario ja preenchde a data com hoje. Registrar transacoes no momento em que acontecem evita esquecer e torna os graficos mais precisos.

**Semestres anteriores:** quando terminar um semestre, mova as pastas de disciplinas de `Em andamento/` para `Semestres anteriores/YYYY.N/`. O campo `semestre` no frontmatter de cada disciplina e o que aparece como cabecalho no agrupamento da Home Universidade.

---

## Consideracoes finais

Este cofre foi construido para ser funcional imediatamente, mas tambem para ser facilmente expandido. Todas as homepages usam o mesmo padrao de arquitetura: HTML gerado como string para o conteudo estatico, e `createEl()` com `addEventListener()` para qualquer elemento interativo. Se voce quiser adicionar novos campos ou secoes, esse e o padrao a seguir.

Os dados sensiveis (transacoes, pesos, notas de freelances) ficam inteiramente no seu computador dentro do cofre. Nenhuma informacao e enviada para servicos externos -- tudo e lido e escrito localmente pelo Obsidian.

Se o cofre crescer muito e as consultas do Dataview ficarem lentas, considere mover notas antigas de `DailyNotes/` para uma subpasta de arquivo (ex: `DailyNotes/2024/`) e ajuste os caminhos nas queries das homepages que usam `dv.pages('"DailyNotes"')`.

Bom uso!
