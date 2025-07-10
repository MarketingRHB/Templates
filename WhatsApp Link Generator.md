
# WhatsApp Link Generator

<%*let WhatsApp = await tp.system.prompt("WhatsApp") -%>
<%*let Vaga = await tp.system.suggester(["Sim", "Não"],["Sim", "Não"], true, "É vaga?") -%>
<%*let Cargo = Vaga === "Sim" ? await tp.system.prompt("Cargo") : "" -%>
<%*let EmpresaNome = Vaga === "Sim" ? await tp.system.prompt("Nome da Empresa") : "" -%>
<%*let Mensagem = await tp.system.prompt("Mensagem") -%>
<%*if (WhatsApp.includes("(") || WhatsApp.includes(")") || WhatsApp.includes(" ") || WhatsApp.includes("-")) { WhatsApp = WhatsApp.replace(/[()\s-]/g, ""); } -%>
<%* if (WhatsApp.length < 10) { -%>

# Verificar o número se está correto

<%* if (WhatsApp.length === 10) { WhatsApp = WhatsApp.slice(0, 2) + "9" + WhatsApp.slice(2); } -%>

<%*} -%>
<%*if (WhatsApp.length === 10) { WhatsApp = WhatsApp.slice(0, 2) + "9" + WhatsApp.slice(2); } -%>
<%-Mensagem -%>
<%* 
let link = Vaga === "Sim" ? `https://wa.me/55${WhatsApp}?text=Olá,%20tenho%20interesse%20nas%20vagas%20de%20${Cargo.replace(/\s+/g, '%20')}%20para%20${EmpresaNome.replace(/\s+/g, '%20')}` : `https://wa.me/55${WhatsApp}?text=${Mensagem.replace(/\s+/g, '%20')}`;
-%> 
<%- link -%>
<%*await tp.file.move("/Whatsapp Link Gerados/" + tp.file.title) -%>
<%*await tp.file.rename("Whats Link "+" "+ tp.date.now("DD-MM-YYYY-HHmm")) -%>
