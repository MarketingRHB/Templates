<%*
const empresasFixas = [
  "Joinville",
  "Matriz",
  "Dayone System",
  "Trade Marketing",
  "Temporários Online"
];

const unidades = [
  "UNIDADE ARAÇATUBA",
  "UNIDADE CAMPINAS",
  "UNIDADE CUIABÁ",
  "UNIDADE JOÃO PESSOA",
  "UNIDADE RIO CLARO",
  "UNIDADE RIO DE JANEIRO",
  "UNIDADE SÃO BENTO DO SUL",
  "UNIDADE SÃO PAULO",
  "UNIDADE TAUBATÉ"
];

function parseValor(str) {
  return parseFloat(
    str.replace("R$", "")
       .replace(/\./g, "")
       .replace(",", ".")
       .trim()
  );
}

function ehSim(valor) {
  return ["sim", "s", "sm"].includes(valor.toLowerCase());
}

function ehNao(valor) {
  return ["nao", "não", "n", "na", "nn"].includes(valor.toLowerCase());
}

// Função para contar quantos colaboradores têm determinado relacionamento
async function contarRelacionamento(nome) {
  const arquivos = app.vault.getMarkdownFiles().filter(f => f.path.startsWith("Colaboradores/"));
  let count = 0;
  for (const file of arquivos) {
    const fm = app.metadataCache.getFileCache(file)?.frontmatter;
    if (!fm || !fm.relationship) continue;

    const rel = fm.relationship;
    if (Array.isArray(rel)) {
      if (rel.map(r => r.toLowerCase()).includes(nome.toLowerCase())) count++;
    } else if (rel.toLowerCase() === nome.toLowerCase()) {
      count++;
    }
  }
  return count;
}

// 🔷 Pergunta se é Feedz
let respostaFeedz = await tp.system.prompt("É rateio do Feedz? (sim/não)");
while (!ehSim(respostaFeedz) && !ehNao(respostaFeedz)) {
  respostaFeedz = await tp.system.prompt("Resposta inválida. Por favor, responda com 'sim' ou 'não'.");
}
const isFeedz = ehSim(respostaFeedz);

// 🟡 Valor total
const valorTotalStr = await tp.system.prompt("Qual o valor total a ser rateado? (ex: 123.456,78)");
const valorTotal = parseValor(valorTotalStr);

if (isNaN(valorTotal)) {
  tR += "❌ Valor total inválido inserido.";
  return;
}

const empresas = [];

// 🔵 Empresas Fixas com sugestão automática
for (const nome of empresasFixas) {
  const count = await contarRelacionamento(nome);
  const opcoes = [`${count} colaboradores (automático)`, "Outra quantidade?"];
  const escolha = await tp.system.suggester(opcoes, opcoes, false, `Quantidade de funcionários da empresa "${nome}"`);

  let funcionarios;
  if (escolha === "Outra quantidade?") {
    const inputManual = await tp.system.prompt(`Digite a quantidade de funcionários da empresa "${nome}":`);
    funcionarios = parseInt(inputManual);
  } else {
    funcionarios = count;
  }

  if (isNaN(funcionarios) || funcionarios < 0) {
    tR += `❌ Número inválido de funcionários para ${nome}.\n`;
    return;
  }

  empresas.push({ nome, funcionarios });
}

// 🔴 Unidades com desconto (somente se for Feedz)
let totalDescontadoParaMatriz = 0;
if (isFeedz) {
  for (const unidade of unidades) {
    const count = await contarRelacionamento(unidade);
    const opcoes = [`${count} colaboradores (automático)`, "Outra quantidade?"];
    const escolha = await tp.system.suggester(opcoes, opcoes, false, `Quantidade de funcionários da unidade "${unidade}"`);

    let funcionarios;
    if (escolha === "Outra quantidade?") {
      const inputManual = await tp.system.prompt(`Digite a quantidade de funcionários da unidade "${unidade}":`);
      funcionarios = parseInt(inputManual);
    } else {
      funcionarios = count;
    }

    if (isNaN(funcionarios) || funcionarios <= 0) {
      tR += `❌ Número inválido de funcionários para ${unidade}.\n`;
      return;
    }

    funcionarios -= 1;
    totalDescontadoParaMatriz += 1;

    empresas.push({ nome: unidade, funcionarios });
  }
}

// ➕ Adiciona os funcionários pagos pela Matriz
if (isFeedz && totalDescontadoParaMatriz > 0) {
  const matriz = empresas.find(e => e.nome === "Matriz");
  if (matriz) {
    matriz.funcionarios += totalDescontadoParaMatriz;
  } else {
    tR += "⚠️ Empresa 'Matriz' não encontrada para somar os funcionários cobertos.\n";
  }
}

// ⚖️ Soma e ajuste se houver referência maior
let totalFuncionarios = empresas.reduce((soma, emp) => soma + emp.funcionarios, 0);

// 🟠 Pergunta quantidade de referência se for Feedz
if (isFeedz) {
  const referenciaStr = await tp.system.prompt("Qual a quantidade de funcionários de referência do mês?");
  const referencia = parseInt(referenciaStr);
  if (!isNaN(referencia) && referencia > totalFuncionarios) {
    const excedente = referencia - totalFuncionarios;
    const joinville = empresas.find(e => e.nome === "Joinville");
    if (joinville) {
      joinville.funcionarios += excedente;
      totalFuncionarios += excedente;
    } else {
      tR += "⚠️ Empresa 'Joinville' não encontrada para somar o excedente de referência.\n";
    }
  }
}

// 📊 Rateio proporcional
let rateio = empresas.map(emp => {
  const proporcao = emp.funcionarios / totalFuncionarios;
  const valorEmpresa = proporcao * valorTotal;
  return {
    ...emp,
    valorRateado: parseFloat(valorEmpresa.toFixed(2))
  };
});

let somaRateio = rateio.reduce((soma, emp) => soma + emp.valorRateado, 0);
let diferenca = parseFloat((valorTotal - somaRateio).toFixed(2));

// 🔧 Ajuste final em Joinville
if (diferenca !== 0) {
  const indexJoinville = rateio.findIndex(emp => emp.nome.toLowerCase() === "joinville");
  if (indexJoinville !== -1) {
    rateio[indexJoinville].valorRateado = parseFloat((rateio[indexJoinville].valorRateado + diferenca).toFixed(2));
    somaRateio = rateio.reduce((soma, emp) => soma + emp.valorRateado, 0);
    diferenca = parseFloat((valorTotal - somaRateio).toFixed(2));
  } else {
    tR += "⚠️ Empresa 'Joinville' não encontrada para ajuste.\n";
  }
}

// 📋 Exibir resultado
tR += `### 💰 Rateio proporcional de R$ ${valorTotal.toLocaleString("pt-BR", {minimumFractionDigits: 2})}\n\n`;

rateio.forEach(emp => {
  tR += `- **${emp.nome}** (Funcionários: ${emp.funcionarios}) → R$ ${emp.valorRateado.toLocaleString("pt-BR", {minimumFractionDigits: 2})}\n`;
});

tR += `\n---\n`;
tR += `**Soma dos rateios:** R$ ${somaRateio.toLocaleString("pt-BR", {minimumFractionDigits: 2})} ✅ (valor conferido)\n`;
if (diferenca !== 0) {
  tR += `⚠️ Ainda há diferença de R$ ${diferenca.toFixed(2).replace(".", ",")}, verifique!`;
}
%>
