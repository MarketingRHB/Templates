<%*let Codigo = await tp.system.prompt("Código") -%> 
<%* let Cidade = await tp.system.prompt("Cidade") -%>
<%* let Cargo = await tp.system.prompt("Cargo") -%>
<%* let Beneficios = await tp.system.prompt("Benefícios") -%>
<%* let Requisitos = await tp.system.prompt("Requisitos") -%>
<%* let EmpresaNome = await tp.system.prompt("Nome da Empresa") -%>
<%* let Quantidade = await tp.system.prompt("Quantidade de Vagas") -%>
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
await fetchData();-%>
💼 𝐂𝐀𝐍𝐃𝐈𝐃𝐀𝐓𝐄-𝐒𝐄 𝐏𝐄𝐋𝐎 𝐋𝐈𝐍𝐊 𝐀𝐁𝐀𝐈𝐗𝐎
https://www.rhbrasil.com.br/portaldocandidato/view/ver-vagas.php?cod=<% Codigo %>

🌐 Ou pesquise pelo código, <% Codigo %> no Portal do Candidato  

<%* if (EmpresaNome) { -%>
A <%- EmpresaNome %>, por meio da RHBrasil, está com<%* if (Quantidade) { %> <%- Quantidade %> vagas<%* } else { %> vagas<%* } %> para <%- Cargo %> em <%- Cidade %>/<%- Sigla %>. 
<%* } else { %>
A RHBrasil está com<%* if (Quantidade) { %> <%- Quantidade %> vagas<%* } else { %> vagas<%* } -%> para <%- Cargo %> em <%- Cidade %>/<%- Sigla %>. 
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

#RHBrasil #OportunidadeDeEmprego<%* if (EmpresaNome) { -%>#<%- EmpresaNome.replace(/\s+/g, '') -%><%* } %> #Vagas<%- Cidade.replace(/\s+/g, '') %> #<%- Cargo.replace(/\s+/g, '') -%>


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

let newFileName = await ensureUniqueFileName("Requisição " + tp.date.now("DDMMYYYYHHmm") + " " + Codigo);
await tp.file.rename(newFileName);
await tp.file.move("/Requisição de Arte/" + newFileName);
-%>

[[<%- Cargo -%>]] [[<%- Cidade -%>]] [[<%- Estado.nome -%>]] [[<% EmpresaNome %>]]