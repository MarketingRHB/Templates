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

const contagemPorUnidade = {};
let totalGeral = 0;

for (const unidade of rhBrasilData) {
  const nomeUnidade = unidade.UNIDADE_RHBRASIL.trim();
  let totalVagasUnidade = 0;

  for (const local of unidade.LOCAIS_ATUACAO) {
    totalVagasUnidade += local.VAGAS.length;
  }

  contagemPorUnidade[nomeUnidade] = totalVagasUnidade;
  totalGeral += totalVagasUnidade;
}

const unidadesOrdenadas = Object.entries(contagemPorUnidade)
  .sort((a, b) => b[1] - a[1]);

if (unidadesOrdenadas.length === 0) {
  tR += "⚠️ Nenhuma unidade com vagas encontrada.";
} else {
  tR += `🏢 **Vagas por unidade RHBrasil:**\n\n`;
  unidadesOrdenadas.forEach(([unidade, total]) => {
    tR += `- ${unidade}: ${total} vaga(s)\n`;
  });
  tR += `\n📊 **Total geral de vagas:** ${totalGeral}\n`;
}
%>
