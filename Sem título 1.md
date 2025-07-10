<%*
const folderPath = "Vagas";
const files = app.vault.getFiles().filter(f => f.path.startsWith(folderPath) && f.name.endsWith(".json"));
files.sort((a, b) => b.name.localeCompare(a.name));
const latestFile = files[0];

if (!latestFile) {
  tR += "❌ Nenhum arquivo de vagas encontrado na pasta `Vagas/`.";
  return;
}

const content = await app.vault.read(latestFile);
const rhBrasilData = JSON.parse(content);

// Lista todos os estados disponíveis
const estadosUnicos = [...new Set(rhBrasilData.flatMap(unidade =>
  unidade.LOCAIS_ATUACAO.map(local => local.LOCAL_ATUACAO.split('/')[1].trim())
))];
let estadoSelecionado = await tp.system.suggester(estadosUnicos, estadosUnicos);

// Filtra as cidades desse estado
const cidades = new Set();
for (const unidade of rhBrasilData) {
  for (const local of unidade.LOCAIS_ATUACAO) {
    const [cidade, estado] = local.LOCAL_ATUACAO.split('/').map(e => e.trim());
    if (estado === estadoSelecionado) {
      cidades.add(cidade);
    }
  }
}

if (cidades.size === 0) {
  tR += `⚠️ Nenhuma cidade encontrada com vagas no estado ${estadoSelecionado}.`;
} else {
  tR += `🏙️ Cidades com vagas em **${estadoSelecionado}**:\n\n`;
  Array.from(cidades).sort().forEach(cidade => {
    tR += `- ${cidade}\n`;
  });
}
%>
