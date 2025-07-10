<%*

const folderPath = "Vagas"; // 📂 Pasta onde os JSONs estão salvos
// 🔍 Busca todos os arquivos da pasta "Vagas"

const files = app.vault.getFiles().filter(f => f.path.startsWith(folderPath) && f.name.endsWith(".json"));
// 📅 Ordena pelos mais recentes

files.sort((a, b) => b.name.localeCompare(a.name));
const latestFile = files[0]; // Pega o mais recente
if (!latestFile) {

  tR += "❌ Nenhum arquivo de vagas encontrado na pasta `Vagas/`. Certifique-se de que há um arquivo JSON de vagas na pasta especificada.\n";

  return;

}
let rhBrasilData = [];

try {

  // 📂 Lê o JSON mais recente do arquivo local

  const content = await app.vault.read(latestFile);

  rhBrasilData = JSON.parse(content);
  // ✅ Adicionando verificação para garantir que rhBrasilData é um array e não está vazio

  if (!Array.isArray(rhBrasilData) || rhBrasilData.length === 0) {

    tR += "❌ O arquivo de vagas encontrado está vazio ou não contém os dados esperados no formato de array. Verifique o conteúdo do seu `vagas.json`.\n";

    return; // Aborta a execução se os dados não forem válidos

  }
} catch (error) {

  tR += `❌ Erro ao ler ou processar o arquivo JSON de vagas: ${error.message}. Verifique a formatação do JSON.\n`;

  return; // Aborta a execução se houver erro no JSON

}
// REMOVIDO: Suggesters para Estados, Cidade, Unidade e Jobcat, e prompt para QuantasVagas

// Agora, todas as vagas serão exibidas sem filtros iniciais.
// 📌 Funções utilitárias

function removeAccents(str) {

  return str.normalize("NFD").replace(/[\u0300-\u036f]/g, "");

}
function capitalizeTitulo(texto) {

  const palavrasMinusculas = ['da', 'de', 'do', 'das', 'dos', 'na', 'no', 'nas', 'nos', 'em', 'para', 'a ', 'o ', 'e'];

  return texto.toLowerCase().split(' ').map((palavra, index) =>

    (index === 0 || !palavrasMinusculas.includes(palavra)) ?

    palavra.charAt(0).toUpperCase() + palavra.slice(1) : palavra

  ).join(' ');

}
function corrigirSiglas(txt) {

  const siglas = {

    'cnc': 'CNC', ' i': ' I', ' ii': ' II', ' iii': ' III', 'movel': 'Móvel',

    'sac': 'SAC', 'dp': 'DP', 'eta': 'ETA', 'pcp': 'PCP', 'rh': 'RH', 'mig': 'MIG',

    ' a ': ' a ', ' A ': ' a ', ' e ': ' e ', ' E ': ' e '

  };

  let resultado = txt;

  for (let sigla in siglas) {

    const regex = new RegExp("\\b" + sigla + "\\b", "gi");

    resultado = resultado.replace(regex, siglas[sigla]);

  }

  return resultado;

}
// Função de Levenshtein para comparação de strings

function levenshtein(a, b) {

  const matrix = [];

  const lenA = a.length;

  const lenB = b.length;
  for (let i = 0; i <= lenB; i++) matrix[i] = [i];

  for (let j = 0; j <= lenA; j++) matrix[0][j] = j;
  for (let i = 1; i <= lenB; i++) {

    for (let j = 1; j <= lenA; j++) {

      if (b.charAt(i - 1) === a.charAt(j - 1)) {

        matrix[i][j] = matrix[i - 1][j - 1];

      } else {

        matrix[i][j] = Math.min(

          matrix[i - 1][j - 1] + 1,

          matrix[i][j - 1] + 1,

          matrix[i - 1][j] + 1

        );

      }

    }

  }

  return matrix[lenB][lenA];

}
// Cache global para otimização

const categoriaCache = new Map();
// ✅ Função de categorização aprimorada

async function categorizeJobFromFolder(tituloOriginal) {

  if (categoriaCache.has(tituloOriginal)) {

    return categoriaCache.get(tituloOriginal);

  }
  const normTitulo = removeAccents(tituloOriginal.toLowerCase().trim());

  console.log(`\n🔍 Analisando vaga: "${tituloOriginal}" (norm: "${normTitulo}")`);
  const cargoFiles = app.vault.getFiles().filter(f =>

    f.path.startsWith("Cargos/") && f.name.endsWith(".md")

  );
  if (cargoFiles.length === 0) {

    console.warn("❌ Nenhum arquivo de categoria encontrado na pasta Cargos/");

    return "OUTROS";

  }
  let melhoresMatches = [];

  const palavrasTitulo = normTitulo.split(/\s+/);
  for (const file of cargoFiles) {

    const categoria = file.name.replace('.md', '').toUpperCase();

    let content = await app.vault.read(file);

    // Limpa e prepara o conteúdo

    content = content

      .replace(/\/\/.*$/mg, '')

      .replace(/<!--[\s\S]*?-->/g, '')

      .replace(/[\n\r]/g, ',')

      .trim();
    const palavrasChave = content

      .split(',')

      .map(p => removeAccents(p.trim().toLowerCase()))

      .filter(p => p.length > 2);
    // Sistema de pontuação

    let pontuacao = 0;

    // Match exato tem pontuação máxima

    if (palavrasChave.includes(normTitulo)) {

      categoriaCache.set(tituloOriginal, categoria);

      return categoria;

    }
    // Pontuação por palavra exata do título

    for (const palavra of palavrasTitulo) {

      if (palavrasChave.includes(palavra)) {

        pontuacao += 10;

      }

    }
    // Pontuação por palavras-chave contidas no título

    for (const palavraChave of palavrasChave) {

      if (palavraChave.length > 3 && normTitulo.includes(palavraChave)) {

        pontuacao += palavraChave.length; // Palavras maiores têm mais peso

      }

    }
    if (pontuacao > 0) {

      melhoresMatches.push({

        categoria,

        pontuacao,

        palavrasEncontradas: palavrasChave

          .filter(p => normTitulo.includes(p))

          .join(', ')

      });

    }

  }
  // Ordena por pontuação e pega o melhor match

  if (melhoresMatches.length > 0) {

    melhoresMatches.sort((a, b) => b.pontuacao - a.pontuacao);

    const melhorMatch = melhoresMatches[0];

    console.log(`✅ Melhor match: ${melhorMatch.categoria} (${melhorMatch.pontuacao} pontos)`);

    console.log(`📝 Palavras encontradas: ${melhorMatch.palavrasEncontradas}`);

    categoriaCache.set(tituloOriginal, melhorMatch.categoria);

    return melhorMatch.categoria;

  }
  console.log(`❌ Nenhum match encontrado para: "${tituloOriginal}"`);

  categoriaCache.set(tituloOriginal, "OUTROS");

  return "OUTROS";

}
const dicionarioPath = "Dicionário/Correcoes.md";

let mapaDeCorrecao = null;
async function carregarDicionario() {

  if (mapaDeCorrecao !== null) return;

  mapaDeCorrecao = new Map();

  const file = app.vault.getAbstractFileByPath(dicionarioPath);

  if (!file) {

    console.warn(`⚠️ Arquivo de dicionário não encontrado em: ${dicionarioPath}`);

    return;

  }
  const content = await app.vault.read(file);

  content.split(/\r?\n/).forEach(linha => {

    linha = linha.trim();

    if (!linha || linha.startsWith("//") || linha.startsWith("#")) return;

    const partes = linha.split(/=>|:/);

    if (partes.length === 2) {

      const de = removeAccents(partes[0].trim().toLowerCase());

      const para = partes[1].trim();

      mapaDeCorrecao.set(de, para);

    }

  });

}
// Nova função de correção com Levenshtein

async function corrigir(textoOriginal) {

  if (!textoOriginal || typeof textoOriginal !== 'string') {

    console.warn('⚠️ Texto inválido:', textoOriginal);

    return "";

  }
  await carregarDicionario();

  const textoNorm = removeAccents(textoOriginal.toLowerCase().trim());

  // Primeiro tenta match exato

  if (mapaDeCorrecao.has(textoNorm)) {

    return mapaDeCorrecao.get(textoNorm);

  }
  // Se não encontrou, usa Levenshtein

  let melhorMatch = null;

  let menorDistancia = Infinity;
  for (const [errado, certo] of mapaDeCorrecao.entries()) {

    const distancia = levenshtein(textoNorm, removeAccents(errado.toLowerCase()));

    if (distancia < menorDistancia) {

      menorDistancia = distancia;

      melhorMatch = certo;

    }

  }
  // Só aceita correções com boa similaridade

  const threshold = Math.max(3, textoNorm.length * 0.4);

  if (menorDistancia <= threshold) {

    console.log(`🧠 Match por Levenshtein: "${textoOriginal}" → "${melhorMatch}" (d=${menorDistancia})`);

    return melhorMatch;

  }
  // Fallback: capitalização básica

  console.log(`⚠️ Sem match próximo para: "${textoOriginal}"`);

  return capitalizeTitulo(textoOriginal);

}
// 📊 Processar vagas

let vagas = [];

for (const { UNIDADE_RHBRASIL, LOCAIS_ATUACAO } of rhBrasilData) {

  for (const { LOCAL_ATUACAO, VAGAS } of LOCAIS_ATUACAO) {

    const [cidade, sigla] = LOCAL_ATUACAO.split('/');

    for (const { NOME_VAGA, CODIGO_VAGA } of VAGAS) {

      try {

        const vagaCorrigida = await corrigir(NOME_VAGA);

        console.log(`🔄 Processando: "${NOME_VAGA}" → "${vagaCorrigida}"`);

        vagas.push({

          codigo: CODIGO_VAGA,

          vaga: vagaCorrigida,

          local: cidade.trim(),

          sigla: sigla.trim(),

          unidade: UNIDADE_RHBRASIL,

        });

      } catch (erro) {

        console.error(`❌ Erro ao processar vaga ${NOME_VAGA}:`, erro);

      }

    }

  }

}
// 🎯 Aplicar filtros - AGORA EXIBE TODAS AS VAGAS SEM FILTRO INICIAL

let filteredVagas = vagas; // Removemos os filtros de estado, cidade, unidade e categoria inicial.
// Otimização: Classifica todas as vagas uma única vez

for (const vaga of filteredVagas) {

  vaga.category = await categorizeJobFromFolder(vaga.vaga);

}
// 📋 Organizar saída por categoria e local

let output = `📢 Todas as Vagas disponíveis na RHBrasil!\n`; // Mensagem inicial atualizada

let grouped = {};

let wikiLinks = "";
// Classificar e agrupar vagas

filteredVagas = await Promise.all(filteredVagas.map(async v => {

  const category = await categorizeJobFromFolder(v.vaga);

  return { ...v, category };

}));
filteredVagas.forEach(v => {

  const category = v.category;

  if (!grouped[category]) grouped[category] = {};

  const key = `${v.local}/${v.sigla}`;

  if (!grouped[category][key]) grouped[category][key] = [];

  grouped[category][key].push(v);

});
// Ordenação alfabética das vagas

Object.keys(grouped).sort().forEach(category => {

  output += `### ${category}\n`;

  Object.keys(grouped[category]).sort().forEach(local => {

    output += `📍 ${local}\n`;

    // Ordena as vagas dentro de cada local

    grouped[category][local].sort((a, b) => a.vaga.localeCompare(b.vaga)).forEach(v => {

      output += `* ${v.vaga}: ${v.codigo}\n`;

      wikiLinks += `[[${v.vaga}]] [[${v.local}]]\n`;

    });

    output += '\n';

  });

});
// Ensure vagasText is defined

let vagasText = output || "⚠️ Nenhuma vaga encontrada."; // Mensagem atualizada para caso não haja vagas

tR += vagasText;
%>
💼 Para muitas outras vagas, acesse o Portal do Candidato da RHBrasil: http://www.rhbrasil.com.br/portaldocandidato
𝐁𝐨𝐚 𝐬𝐨𝐫𝐭𝐞!
𝐂𝐨𝐧𝐡𝐞𝐜𝐞 𝐚𝐥𝐠𝐮é𝐦 𝐪𝐮𝐞 𝐩𝐨𝐝𝐞 𝐬𝐞 𝐢𝐧𝐭𝐞𝐫𝐞𝐬𝐬𝐚𝐫? 𝐂𝐨𝐦𝐩𝐚𝐫𝐭𝐢𝐥𝐡𝐞 𝐞𝐬𝐭𝐞 𝐩𝐨𝐬𝐭!
RHBrasil, empresa líder em recrutamento e seleção de pessoas no país.

#RHBrasil #RH #Emprego #VagadeEmprego #OportunidadeDeEmprego
<%*

// Geração correta das hashtags de localização

const uniqueLocations = new Set();

filteredVagas.forEach(vaga => {

    const formattedLocation = `${removeAccents(vaga.local).replace(/[^\w]/g, '')}${vaga.sigla}`;

    uniqueLocations.add(formattedLocation);

});
tR += " " + Array.from(uniqueLocations).map(location => `#${location}`).join(' ');

%>
<% wikiLinks %>

�