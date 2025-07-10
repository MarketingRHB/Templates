<%* 
const id = "123456";
const botoes = `
BUTTON[Editar Perfil]
\`\`\`meta-bind-button
label: Editar Perfil
icon: ""
hidden: false
class: ""
tooltip: "Abrir Feedz"
id: "Editar Perfil"
style: primary
actions:
  - open: https://app.feedz.com.br/empresa/colaboradores/${id}
\`\`\`

BUTTON[Ver Perfil]
\`\`\`meta-bind-button
label: Ver Perfil
icon: ""
hidden: false
class: ""
tooltip: "Ver Perfil Feedz"
id: "Ver Perfil"
style: secondary
actions:
  - open: https://app.feedz.com.br/colaboradores/${id}/perfil
\`\`\`
`;

tR += botoes;
%>
