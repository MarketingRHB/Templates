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

const contagemPorCidade = {};
let totalVagas = 0;

for (const unidade of rhBrasilData) {
  for (const local of unidade.LOCAIS_ATUACAO) {
    const [cidade, estado] = local.LOCAL_ATUACAO.split('/').map(e => e.trim());
    const key = `${cidade}/${estado}`;
    const quantidade = local.VAGAS.length;
    contagemPorCidade[key] = (contagemPorCidade[key] || 0) + quantidade;
    totalVagas += quantidade;
  }
}

// Ordenar por número de vagas (decrescente)
const cidadesOrdenadas = Object.entries(contagemPorCidade)
  .sort((a, b) => b[1] - a[1]);

if (cidadesOrdenadas.length === 0) {
  tR += "⚠️ Nenhuma cidade com vagas encontrada.";
} else {
  tR += `📍 **Cidades com vagas disponíveis:**\n\n`;
  cidadesOrdenadas.forEach(([cidadeUF, total]) => {
    tR += `- ${cidadeUF}: ${total} vaga(s)\n`;
  });
  tR += `\n📊 **Total geral de vagas:** ${totalVagas}\n`;
}
%>
