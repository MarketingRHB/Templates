<%*let empresas = ["Whirlpool", "Britânia", "AB Plast", "Docol", "Krona", "Schulz", "Cipla","Wetzel", "Outra..."];
let EmpresaNome = await tp.system.suggester(empresas, empresas, true, "Escolha uma empresa ou selecione 'Outra...' para digitar um nome diferente");
if (EmpresaNome === "Outra...") {
    EmpresaNome = await tp.system.prompt("Digite o nome da empresa:");
}-%>
<%* let Cidade = await tp.system.prompt("Cidade") -%>
<%* let Cargo = await tp.system.prompt("Cargo")-%>
<%* let Quantidade = await tp.system.prompt("Quantidade de Vagas") -%>
<%* let WhatsApp = await tp.system.prompt("WhatsApp") -%>
<%* let Data = await tp.system.suggester(["Segunda à Sexta", "Outra data"], ["Segunda à Sexta", "Outra data"], false, "Data") 
if (Data === "Outra data") {
    Data = await tp.system.prompt("Digite a data:");
}-%>
<%* let Horario = await tp.system.suggester(["Das 7h30 às 17h", "Outro horário"], ["Das 7h30 às 17h", "Outro horário"], false, "Horário") 
if (Horario === "Outro horário") {
    Horario = await tp.system.prompt("Digite o horário:");
}-%>
<%* let Beneficios = await tp.system.prompt("Benefícios") -%>
<%* let Requisitos = await tp.system.prompt("Requisitos") -%>
<%* let Endereco = await tp.system.suggester(["RHBrasil Joinville", "Britânia CD", "Britânia Fábrica", "Wetzel", "Outro Endereço"], ["RHBrasil Joinville", "Britânia CD", "Britânia Fábrica", "Wetzel", "Outro Endereço"], false, "Endereço") || "RHBrasil Joinville"
let EnderecoTexto = "";
let EnderecoLink = "";
if (Endereco === "Outro Endereço") {
    EnderecoTexto = await tp.system.prompt("Digite o endereço completo:");
    EnderecoLink = await tp.system.prompt("Digite o link do Google Maps:");
} -%>
<%* 
// Função para remover caracteres inválidos para nome de arquivo
function removeInvalidFileNameChars(str) {
    return str.replace(/[\\/:*?"<>|]/g, '');
}
// Corrigir o nome da empresa para remover caracteres inválidos
EmpresaNome = removeInvalidFileNameChars(EmpresaNome);
-%>
<%* let Sigla = ""-%>
<%* let Estado = ""-%>
<%* async function fetchData() {
    // ...existing fetchData function code...
}
await fetchData();
-%>

🚀 Mutirão do Emprego na <% EmpresaNome %>  - <% Quantidade %> vagas de <% Cargo %>! 🚀
<%* if (WhatsApp.includes("(") || WhatsApp.includes(")") || WhatsApp.includes(" ") || WhatsApp.includes("-")) { WhatsApp = WhatsApp.replace(/[()\s-]/g, ""); } -%>
<%* if (WhatsApp.length === 10) { WhatsApp = WhatsApp.slice(0, 2) + "9" + WhatsApp.slice(2); } -%>
Quer participar? Entre em contato com a RHBrasil pelo WhatsApp:  (<% WhatsApp.slice(0, 2) %>) <% WhatsApp.slice(2, 3) %> <% WhatsApp.slice(3, 7) %>-<% WhatsApp.slice(7, 11) %> e garanta a sua chance!

📞 https://wa.me/55<% WhatsApp -%>

<%* switch (Endereco) { 
case "RHBrasil Joinville": -%>
⚠️ Compareça para o processo seletivo, munido de documentos.
🗓️ Data: <% Data %>
🕒 Horário: <% Horario %>
📍 Endereço: RHBrasil – Rua Blumenau, 295 – Centro – Joinville/SC
🗺️ https://maps.app.goo.gl/9EYrkD72fqQBeimG6
<%* break; case "Britânia CD": -%>
⚠️ Compareça para o processo seletivo, munido de documentos.
🗓️ Data: <% Data %>
🕒 Horário: <% Horario %>
📍 Local: Britânia CD – Endereço específico
🗺️ Link do Google Maps específico
<%* break; case "Britânia Fábrica": -%>
⚠️ Compareça para o processo seletivo, munido de documentos.
🗓️ Data: <% Data %>
🕒 Horário: <% Horario %>
📍 Local: Britânia Fábrica – Endereço específico
🗺️ Link do Google Maps específico
<%* break; case "Wetzel": -%>
⚠️ Compareça para o processo seletivo, munido de documentos.
🗓️ Data: <% Data %>
🕒 Horário: <% Horario %>
📍 Local: R. Dona Francisca, 8300 - Distrito Industrial
🗺️ https://maps.app.goo.gl/48nKN7XWE8xULH9FA
<%* break; case "Outro Endereço": -%>
⚠️ Compareça para o processo seletivo, munido de documentos.
🗓️ Data: <% Data %>
🕒 Horário: <% Horario %>
📍 Local: <% EnderecoTexto %>
🗺️ <% EnderecoLink %>
<%* break; } -%>

✨ RHBrasil, há 30 anos transformando vidas através do emprego. ✨

𝐁𝐨𝐚 𝐬𝐨𝐫𝐭𝐞!
𝐂𝐨𝐧𝐡𝐞𝐜𝐞 𝐚𝐥𝐠𝐮é𝐦 𝐪𝐮𝐞 𝐩𝐨𝐬𝐬𝐚 𝐬𝐞 𝐢𝐧𝐭𝐞𝐫𝐞𝐬𝐬𝐚𝐫? 𝐂𝐨𝐦𝐩𝐚𝐫𝐭𝐢𝐥𝐡𝐞 𝐞𝐬𝐭𝐞 𝐩𝐨𝐬𝐭!

RHBrasil, empresa líder em recrutamento e seleção de pessoas no país.

#RHBrasil #VagasDeEmprego #MutirãoDoEmprego #OportunidadeDeEmprego #<% EmpresaNome.replace(/\s+/g, '') -%> #Vagas<% Cidade.replace(/\s+/g, '') -%> #<%- Cargo.replace(/\s+/g, '') %>

https://wa.me/55<%- WhatsApp -%>?text=Olá,%20tenho%20interesse%20nas%20vagas%20de%20<%- Cargo.replace(/\s+/g, '%20') %>%20para%20<%-EmpresaNome.replace(/\s+/g, '%20') -%>

[[<% EmpresaNome %>]]  [[<%- Cargo -%>]] [[<%- Cidade -%>]] 


<%* 
// Função para verificar se o arquivo já existe e adicionar um número no final se necessário
async function ensureUniqueFileName(fileName) {
    let baseName = fileName;
    let counter = 1;
    while (await tp.file.exists(fileName)) {
        fileName = `${baseName} (${counter})`;
        counter++;
    }
    return fileName;
}

let newFileName = await ensureUniqueFileName("Requisição Mutirão " + tp.date.now("DDMMYYYYHHmm") + " " + EmpresaNome);
await tp.file.rename(newFileName);
await tp.file.move("/Requisição de Arte/" + newFileName);
-%>

[[<%- Cargo -%>]] [[<%- Cidade -%>]] [[<% EmpresaNome %>]]