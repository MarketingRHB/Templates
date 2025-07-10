<%*
let empresa = await tp.system.prompt("Informe o nome da empresa:")
let orcamento = await tp.system.prompt("Informe o orçamento total:")
let dataInicio = await tp.system.prompt("Informe a data de início:")
let dataTermino = await tp.system.prompt("Informe a data de término:")
let localizacao = await tp.system.prompt("Informe a localização:")
let idade = await tp.system.prompt("Informe a faixa etária:")
let genero = await tp.system.prompt("Informe o gênero:")
let link = await tp.system.prompt("Informe o WhatsApp:")

// Obter todas as notas do Vault
let arquivos = app.vault.getMarkdownFiles()
let nomesNotas = arquivos.map(arquivo => arquivo.path)

// Criar sugestão de seleção
let notaLegenda = await tp.system.suggester(nomesNotas, nomesNotas, true)

// Ler o conteúdo da nota escolhida
let arquivoLegenda = app.vault.getAbstractFileByPath(notaLegenda)
let conteudoLegenda = ""

if (arquivoLegenda) {
  conteudoLegenda = await app.vault.read(arquivoLegenda)
} else {
  conteudoLegenda = "⚠️ Nota não encontrada: " + notaLegenda
}

tR += `**Empresa:** ${empresa}  

**Orçamento total:** ${orcamento}  
**Data de início:** ${dataInicio}  
**Data de término:** ${dataTermino}  
**Localização:** ${localizacao}  
**Idade:** ${idade}  
**Gênero:** ${genero}  

**Link:** ${link}  

---

Legenda:
${conteudoLegenda}

---

### 🏷️ Legenda (link):
[[${notaLegenda}]]
`
-%>
