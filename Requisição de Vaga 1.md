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
    const url = 'https://gist.githubusercontent.com/letanure/3012978/raw/6938daa8ba69bcafa89a8c719690225641e39586/estados-cidades.json';
    try {
        const response = await fetch(url);
        const data = await response.json();
        let estados = data.estados.map(estado => ({
            sigla: estado.sigla,
            nome: estado.nome,
            cidades: estado.cidades
        }));
        for (let i = 0; i < estados.length; i++) {
          const { sigla, nome, cidades } = estados[i];
         // Função para remover caracteres especiais
          function removeAccents(str) {
              return str.normalize("NFD").replace(/[\u0300-\u036f]/g, "");
          }
          // Remover caracteres especiais da cidade e das cidades do estado
          const normalizedCidade = removeAccents(Cidade);
          const normalizedCidades = cidades.map(cidade => removeAccents(cidade));
          // Verificar se cidades contém a cidade
          if (normalizedCidades.includes(normalizedCidade)) {
              console.log(`A cidade ${Cidade} está localizada no estado ${nome} (${sigla}).`);
              Sigla = sigla;
              Estado = estados[i];
            for (let j = 0; j < cidades.length; j++) {
                if (removeAccents(cidades[j]) === normalizedCidade) {
                    Cidade = cidades[j]; // Corrigir a ortografia de Cidade
                    break;
                }
            }
          }
          // verificar se cidades contem city
          if (cidades.includes(Cidade)) {
              console.log(`A cidade ${Cidade} está localizada no estado ${nome} (${sigla}).`);
                 Sigla = sigla;
                 Estado = estados[i];
          }
        }
    } catch (error) {
        console.error('Erro ao buscar os dados:', error);
    }
}

// Chamar a função
await fetchData();
-%>
<%* 
// Ensure EmpresaNome is defined and sanitized
if (!EmpresaNome) {
    EmpresaNome = "Empresa Não Definida";
}
EmpresaNome = EmpresaNome.replace(/[\\/:*?"<>|]/g, '');

// Ensure Cidade is defined
if (!Cidade) {
    Cidade = "Cidade Não Definida";
}

// Ensure Estado is defined
if (!Estado || !Estado.nome) {
    Estado = { nome: "Estado Não Definido" };
}

// Ensure WhatsApp is sanitized
if (WhatsApp) {
    WhatsApp = WhatsApp.replace(/[()\s-]/g, "");
    if (WhatsApp.length === 10) {
        WhatsApp = WhatsApp.slice(0, 2) + "9" + WhatsApp.slice(2);
    }
} else {
    WhatsApp = "0000000000";
}
-%>
<%* switch (EmpresaNome) { case "Whirlpool": -%>
💼 𝐈𝐍𝐒𝐂𝐑𝐄𝐕𝐀 𝐏𝐄𝐋𝐎 𝐖𝐇𝐀𝐓𝐒𝐀𝐏𝐏:
<%* break; case "Britânia": -%>
💼 𝐈𝐍𝐒𝐂𝐑𝐄𝐕𝐀 𝐏𝐄𝐋𝐎 𝐖𝐇𝐀𝐓𝐒𝐀𝐏𝐏:
<%* break; case "AB Plast": -%>
💼 𝐈𝐍𝐒𝐂𝐑𝐄𝐕𝐀 𝐏𝐄𝐋𝐎 𝐖𝐇𝐀𝐓𝐒𝐀𝐏𝐏:
<%* break; case "Docol": -%>
💼 𝐈𝐍𝐒𝐂𝐑𝐄𝐕𝐀 𝐏𝐄𝐋𝐎 𝐖𝐇𝐀𝐓𝐒𝐀𝐏𝐏:
<%* break; case "Krona": -%>
💼 𝐈𝐍𝐒𝐂𝐑𝐄𝐕𝐀 𝐏𝐄𝐋𝐎 𝐖𝐇𝐀𝐓𝐒𝐀𝐏𝐏:
<%* break; case "Schulz": -%>
💼 𝐈𝐍𝐒𝐂𝐑𝐄𝐕𝐀 𝐏𝐄𝐋𝐎 𝐖𝐇𝐀𝐓𝐒𝐀𝐏𝐏:
<%* break; case "Cipla": -%>
💼 𝐈𝐍𝐒𝐂𝐑𝐄𝐕𝐀 𝐏𝐄𝐋𝐎 𝐖𝐇𝐀𝐓𝐒𝐀𝐏𝐏:
<%* break; case "Wetzel": -%>
💼 𝐈𝐍𝐒𝐂𝐑𝐄𝐕𝐀 𝐏𝐄𝐋𝐎 𝐖𝐇𝐀𝐓𝐒𝐀𝐏𝐏:
<%* break; default: -%>
💼 𝐈𝐍𝐒𝐂𝐑𝐄𝐕𝐀 𝐏𝐄𝐋𝐎 𝐖𝐇𝐀𝐓𝐒𝐀𝐏𝐏:
<%* break; } -%>
📞 (<% WhatsApp.slice(0, 2) %>) <% WhatsApp.slice(2, 3) %> <% WhatsApp.slice(3, 7) %>-<% WhatsApp.slice(7, 11) %>: https://wa.me/55<% WhatsApp -%>

<%* switch (Endereco) { case "": break; case "RHBrasil Joinville": -%> 
⚠️ Ou Compareça para o processo seletivo, munido de documentos.
🗓️ Data: <% Data %>
🕒 Horário: <% Horario %>
📍 Endereço: RHBrasil – Rua Blumenau, 295 – Centro – Joinville/SC
🗺️ https://maps.app.goo.gl/9EYrkD72fqQBeimG6
<%* break; case "Britânia CD": -%> 
⚠️ Compareça para o processo seletivo, munido de documentos.
🗓️ Data: <% Data %>
🕒 Horário: <% Horario %>
📍 Endereço: Britânia CD – Rua Dona Francisca, 12340 – Próximo à Embraco Fundição
🗺️ https://maps.app.goo.gl/zcQNvMiMeKV2FNcn6
<%* break; case "Britânia Fábrica": -%> 
⚠️ Compareça para o processo seletivo, munido de documentos.
🗓️ Data: <% Data %>
🕒 Horário: <% Horario %>
📍 Local: Britânia Fábrica - Vergilio Prochnow 200  
🗺️ https://maps.app.goo.gl/ukKDbi6DeA6Ydx3F9
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

<%* if (EmpresaNome === "") { -%> 
A RHBrasil,está com <% Quantidade %> vagas para <% Cargo %> em <% Cidade %>/<%Sigla%>. 
<%* } else { %>
A <% EmpresaNome %>, por meio da RHBrasil está com<%* if (Quantidade === "") { -%> vagas para <% Cargo %> em <% Cidade %>/<%Sigla%>. <%* } else { %> <% Quantidade %> vagas para <% Cargo %> em <% Cidade %>/<%Sigla%>. <%* } -%>
<%* } -%>

<%* if (Requisitos === "") { -%><%* } else { -%> 
✅ Requisitos: 
- <% Requisitos %>  
<%* } -%>

<%* if (Beneficios === "") { -%> 
<%* } else { %> 
✨ Benefícios:
- <% Beneficios %>  
<%* } -%>

𝐁𝐨𝐚 𝐬𝐨𝐫𝐭𝐞!
𝐂𝐨𝐧𝐡𝐞𝐜𝐞 𝐚𝐥𝐠𝐮é𝐦 𝐪𝐮𝐞 𝐩𝐨𝐬𝐬𝐚 𝐬𝐞 𝐢𝐧𝐭𝐞𝐫𝐞𝐬𝐬𝐚𝐫? 𝐂𝐨𝐦𝐩𝐚𝐫𝐭𝐢𝐥𝐡𝐞 𝐞𝐬𝐭𝐞 𝐩𝐨𝐬𝐭!

RHBrasil, empresa líder em recrutamento e seleção de pessoas no país.

#RHBrasil #OportunidadeDeEmprego  #<% EmpresaNome.replace(/\s+/g, '') -%>  #Vagas<% Cidade.replace(/\s+/g, '') -%>  #<%- Cargo.replace(/\s+/g, '') %>

https://wa.me/55<%- WhatsApp -%>?text=Olá,%20tenho%20interesse%20nas%20vagas%20de%20<%- Cargo.replace(/\s+/g, '%20') %>%20para%20<%-EmpresaNome.replace(/\s+/g, '%20') -%>

<%*
// Function to normalize and sanitize folder names
function normalizeFolderName(str) {
    const semAcento = str.normalize("NFD").replace(/[\u0300-\u036f]/g, "");
    const capitalizado = semAcento.charAt(0).toUpperCase() + semAcento.slice(1).toLowerCase();
    return capitalizado.replace(/[\\/:*?"<>|]/g, '');
}

let folderName = normalizeFolderName(EmpresaNome);
let folderPath = `/Requisição de Arte/${folderName}`;

if (!(await tp.file.exists(folderPath))) {
    await tp.file.create_new("", folderPath + "/.placeholder");
}

// Function to ensure unique file names
async function ensureUniqueFileName(fileName) {
    let baseName = fileName;
    let counter = 1;
    while (await tp.file.exists(fileName)) {
        fileName = `${baseName} (${counter})`;
        counter++;
    }
    return fileName;
}

let newFileName = await ensureUniqueFileName("Requisição " + EmpresaNome + " " + tp.date.now("DD MM YYYY HH mm"));
await tp.file.rename(newFileName);
await tp.file.move(`${folderPath}/${newFileName}`);
-%>

[[<%- Cargo -%>]] [[<%- Cidade -%>]] [[<%- Estado.nome -%>]] [[<%- EmpresaNome -%>]]