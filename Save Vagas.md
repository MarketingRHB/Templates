<%*
const folderPath = "Vagas"; // 📂 Pasta onde o JSON será salvo
const today = tp.date.now("YYYY-MM-DD");
const fileName = `vagas-rh-${today}.json`;
const filePath = `${folderPath}/${fileName}`;

// 📋 Pega o conteúdo do clipboard diretamente
const clipboardText = await tp.system.clipboard();

if (!clipboardText) {
  tR += "❌ Nenhum texto encontrado no clipboard!";
  return;
}

// 🛠️ Tenta converter para JSON válido
let jsonData;
try {
  jsonData = JSON.parse(clipboardText);
} catch (e) {
  tR += "❌ O conteúdo do clipboard não é um JSON válido!";
  return;
}

// 📂 Criar pasta "Vagas" se não existir
if (!app.vault.getAbstractFileByPath(folderPath)) {
  await app.vault.createFolder(folderPath);
}

// 💾 Salvar o JSON no arquivo
await app.vault.create(filePath, JSON.stringify(jsonData, null, 2));

tR += `✅ JSON salvo em: \`${filePath}\``;
%>
