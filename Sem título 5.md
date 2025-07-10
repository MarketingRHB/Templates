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

for (const unidade of rhBrasilData) {
  for (const local of unidade.LOCAIS_ATUACAO) {
    const [cidade, estado] = local.LOCAL_ATUACAO.split('/').map(e => e.trim());
    const key = `${cidade}/${estado}`;
    contagemPorCidade[key] = (contagemPorCidade[key] || 0) + local.VAGAS.length;
  }
}

// Ordenar por número de vagas (decrescente) e pegar as 10 primeiras
const top10 = Object.entries(contagemPorCidade)
  .sort((a, b) => b[1] - a[1])
  .slice(0, 10);

if (top10.length === 0) {
  tR += "⚠️ Nenhuma cidade com vagas encontrada.";
} else {
  tR += `🏆 **Top 10 cidades com mais vagas:**\n\n`;
  top10.forEach(([cidadeUF, total], idx) => {
    tR += `${idx + 1}. ${cidadeUF} — ${total} vaga(s)\n`;
  });
}
%>
