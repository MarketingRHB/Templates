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

const contagemPorVaga = {};
let totalVagas = 0;

for (const unidade of rhBrasilData) {
  for (const local of unidade.LOCAIS_ATUACAO) {
    for (const vaga of local.VAGAS) {
      const nome = vaga.NOME_VAGA.trim().toUpperCase();
      contagemPorVaga[nome] = (contagemPorVaga[nome] || 0) + 1;
      totalVagas++;
    }
  }
}

// Ordenar por número de vagas (decrescente)
const vagasOrdenadas = Object.entries(contagemPorVaga)
  .sort((a, b) => b[1] - a[1]);

if (vagasOrdenadas.length === 0) {
  tR += "⚠️ Nenhuma vaga encontrada.";
} else {
  tR += `🗂️ **Vagas agrupadas por nome:**\n\n`;
  vagasOrdenadas.forEach(([nome, qtd]) => {
    tR += `- ${nome}: ${qtd} vaga(s)\n`;
  });
  tR += `\n📊 **Total geral de vagas:** ${totalVagas}\n`;
}
%>
