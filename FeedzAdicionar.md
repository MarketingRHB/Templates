<%* 
let Paste = await tp.system.clipboard();
let data = Paste;

// Helper function to safely extract data
function extractData(regex, fallback = "") {
    const match = data.match(regex);
    return match ? match[1].trim() : fallback;
}

// Helper function to capitalize the first letter of each word
function capitalizeWords(text) {
    return text.toLowerCase().replace(/\b\w/g, char => char.toUpperCase());
}

// Helper function to format CPF
function formatCPF(cpfRaw) {
    const digits = cpfRaw.replace(/\D/g, ''); // Remove non-numeric characters
    if (digits.length !== 11) return cpfRaw; // Return as is if not 11 digits
    return `${digits.slice(0, 3)}.${digits.slice(3, 6)}.${digits.slice(6, 9)}-${digits.slice(9, 11)}`;
}

// Extract information from the clipboard content
let nome = capitalizeWords(extractData(/Nome - (.+?)$/m));
let setor = capitalizeWords(extractData(/setor (.+?) da unidade/m));
let unidade = capitalizeWords(extractData(/Unidade - (.+?) -/m));
let unidade2 = capitalizeWords(extractData(/RHBRASIL - (.+?)$/m));
let email = extractData(/E-mail - (.+?)$/m);
let cpf = formatCPF(extractData(/CPF - (.+?)$/m)); // Apply CPF formatting
let dataDeNascimento = extractData(/Data de Nascimento - (.+?)$/m);
let dataDeAdmisao = extractData(/Admissão - (.+?)$/m);
let cargo = capitalizeWords(extractData(/Cargo - (.+?)$/m));
let gestor = capitalizeWords(extractData(/Gestor - (.+?)$/m));
-%>
---
name: <% nome %>
<%* 
function formatDateToISO(date) {
    const [day, month, year] = date.split('/');
    return `${year}-${month}-${day}`;
}

let formattedDataDeNascimento = formatDateToISO(dataDeNascimento);
let formattedDataDeAdmisao = formatDateToISO(dataDeAdmisao);
-%>
birthday: <% formattedDataDeNascimento %>
cpf: <% cpf %>
email: <% email %>
data_de_admissao: <% formattedDataDeAdmisao %>
cargo: <% cargo %>
departamento: <% setor %>
unidade: <% unidade %> - <% unidade2 %>
gestor: <% gestor %>
situacao: Ativo
papel: Colaborador
genero: 
ultimo_acesso: 

---
<% tp.file.move("Colaboradores/" + nome) %>