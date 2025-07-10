<%*
let Paste = await tp.system.clipboard()
let Estados = await tp.system.prompt("Estados")

// Process clipboard data
let data = Paste;

// Normaliza as quebras de linha e remove espaços em branco extras
data = data.replace(/\r\n/g, '\n').replace(/\r/g, '\n').trim();

// Remove os prefixos "Cód: " e "Local de atuação: " do texto
data = data.replace(/Cód: /g, '').replace(/Local de atuação: /g, '');

// Divide o texto em linhas e remove linhas vazias
let lines = data.split('\n').filter(line => line.trim() !== '');

// Inicializa um array vazio para armazenar as vagas
let vagas = [];

// Inicializa uma string vazia para armazenar o texto das vagas
let vagasText = "";

// Inicializa um objeto vazio para armazenar as vagas agrupadas
let groupedVagas = {};

// Função para remover acentos de uma string
function removeAccents(str) {
    return str.normalize("NFD").replace(/[\u0300-\u036f]/g, "");
}

// Nova função para capitalizar títulos corretamente
function capitalizeTitulo(texto) {
    const palavrasMinusculas = ['da', 'de', 'do', 'das', 'dos', 'na', 'no', 'nas', 'nos', 'em', 'para', 'a ', 'o ', 'e'];

    return texto.toLowerCase().split(' ').map((palavra, index) => {
        if (index === 0 || !palavrasMinusculas.includes(palavra)) {
            return palavra.charAt(0).toUpperCase() + palavra.slice(1);
        }
        return palavra;
    }).join(' ');
}

// Função para colocar siglas em caixa alta
function corrigirSiglas(txt) {
    const siglas = {
        'cnc': 'CNC',
        ' i': ' I',
        ' ii': ' II',
        ' iii': ' III',
        'movel': 'Móvel',
        'sac': 'SAC',
        'dp': 'DP',
        'eta': 'ETA',
        'pcp': 'PCP',
        'rh': 'RH',
        'dp': 'DP',
        'mig': 'MIG',
        ' a ': ' a ',
        ' A ': ' a ',
        ' e ': ' e ',
        ' E ': ' e '

    };

    let resultado = txt;
    for (let sigla in siglas) {
        const regex = new RegExp("\\b" + sigla + "\\b", "gi");
        resultado = resultado.replace(regex, siglas[sigla]);
    }
    return resultado;
}



// Async function to fetch and process data
async function fetchData() {
    try {
        const url = 'https://gist.githubusercontent.com/GustavoNeneve/8d09e31038e856be2e5f08effe57bff0/raw/f3c6e21f565e504d19ae6adaebb95dbb3db4cbb6/estados-cidades.json';
        const response = await fetch(url);
        const data = await response.json();

        // Mapeia os dados para um formato mais conveniente
        let estados = data.estados.map(estado => ({
            sigla: estado.sigla,
            nome: estado.nome,
            cidades: estado.cidades
        }));

        // Itera sobre as linhas, agrupando informações de cada vaga
        for (let i = 0; i < lines.length; i += 4) {
            if (i + 3 < lines.length) {
                let localSemAcento = removeAccents(lines[i + 3].trim().slice(0, -3)).toLowerCase();
                //fazer um let que remova cacacteres especiais com ' " ! @ # $ - % * ( ) _ + = § ; : / º ª °\|
                let localSemCaracteresEspeciais = localSemAcento.replace(/[\'\"!@#$%\*\(\)_\+=§;:\/ºª°\\|]/g, '');
                let cidadeCorreta = lines[i + 3].trim().slice(0, -3);

                // Verifica se a cidade está nos dados do JSON
                estados.forEach(estado => {
                    estado.cidades.forEach(cidade => {
                        if (removeAccents(cidade).toLowerCase().replace(/[\'\"!@#$%\*\(\)_\+=§;:\/ºª°\\|]/g, '') === localSemCaracteresEspeciais) {
                            cidadeCorreta = cidade; // Atualiza o nome da cidade com a versão correta, preservando acentos
                        }     });
                });

                vagas.push({
                    codigo: lines[i].trim(), // Código da vaga
                    vaga: corrigir(lines[i + 1].trim()), // Descrição da vaga com correção ortográfica
                    local: cidadeCorreta, // Local de atuação com acentos corrigidos
                    sigla: lines[i + 3].trim().slice(-2) // Sigla do estado (últimos 2 caracteres)
                });
            }
        }

        // Ordena as vagas pelo local e, em seguida, pela descrição da vaga
        vagas.sort((a, b) => a.local.localeCompare(b.local) || a.vaga.localeCompare(b.vaga));

        // Agrupa as vagas por local
        groupedVagas = vagas.reduce((acc, vaga) => {
            if (!acc[vaga.local]) {
                acc[vaga.local] = [];
            }
            acc[vaga.local].push(vaga);
            return acc;
        }, {});

        // Preenche Estados com os nomes dos estados relacionados às siglas das vagas
        if (!Estados) {
            let siglas = new Set(vagas.map(vaga => vaga.sigla));
            Estados = estados.filter(estado => siglas.has(estado.sigla)).map(estado => estado.nome).join(', ');
        }

        const jobCategories = {
            'PRODUÇÃO': [
                'producao', 'moldes', 'maquinas', 'manufatura', 'operador', 'montador',
                'soldador', 'coquilhadeira', 'macheiro', 'usinagem', 'serralheiro',
                'ferramenteiro', 'fundicao', 'encarregado de producao', 'padeiro',
                'mig', 'confeiteiro', 'cozinha', 'pedreiro', 'eletricista predial',
                'carpinteiro', 'pintor', 'guindaste', 'mestre de obras', 'topografia',
                'gesseiro', 'funileiro', 'chef', 'encanador', 'betoneira', 'linha de montagem','producao alimenticia', 'qualidade alimentar', 'cozinheiro industrial','caldereiro', 'esmerilhador', 'preparador', 'processista', 'modelista','materia prima', 'auxiliar operacional', 'coletor', 'preparador jr','auxiliar embarcacao', 'op', 'maquina', 'extrusora', 'marceneiro',
                'tapeceiro', 'lubrificador', 'industrial', 'costureira', 'extrusora',
                'maq', 'fibra de vidro', 'acabamento', 'lustrador', 'pintor', 'montador',
                'soldador', 'caldeireiro', 'serralheiro', 'operador', 'mecânico',
                'ajustador', 'auxiliar de acabamento', 'estofador', 'cnc',
                'preparador','maquina', 'lustração'  // Added new production roles
            ],
            'LOGÍSTICA': [
                'expedicao', 'carga', 'descarga', 'motorista', 'caminhao', 'estoquista',
                'mercadorias', 'embalador', 'separador', 'logistica', 'supply chain',
                'transportes', 'distribuicao', 'armazenagem', 'suprimentos', 'armazem',
                'deposito', 'almoxarifado', 'almoxarife', 'estoque', 'conferente',
                'empilhadeira', 'armazenista', 'recebimento', 'frota', 'vistoriador',
                'expedidor' // Added new logistics role
            ],
            'ADMINISTRATIVA': [
                'departamento pessoal', 'recrutamento', 'selecao', 'recursos humanos',
                'rh', 'dp', 'administrativo', 'recepcionista', 'financeiro', 'contabil',
                'fiscal', 'vendedor', 'compras', 'marketing', 'e-commerce', 'promotor',
                'supervisor', 'atendente', 'telemarketing', 'comercial', 'instrutor',
                'professor', 'educacao', 'sac', 'compliance', 'consultor', 'contador',
                'tesouraria', 'credito', 'cobranca', 'controladoria', 'investimentos',
                'advogado', 'juridico', 'copywriter', 'designer', 'social media',
                'hotelaria', 'turismo', 'pedagogico', 'biblioteca', 'administracao', 
                'administrativa', 'admin','faturamento', 'televendas', 'comercio exterior', 
                'custos', 'importacao', 'pcp', 'remuneracao','assistente de vendas', 
                'comprador', 'Secretaria', 'analista de custos', 'auxiliar de vendas', 'comprador' // Added new administrative roles
            ],
            'MANUTENÇÃO': [
                'mecanico', 'eletricista', 'servicos gerais', 'porteiro', 'manutencao',
                'borracharia', 'mecanica automotiva', 'auxiliar de limpeza', 'auxiliar de operacoes','encarregado de oficina','oficina' // Added new maintenance role
            ],
            'TÉCNICA': [
                'seguranca do trabalho', 'engenheiro', 'tecnico', 'desenvolvedor',
                'sistemas', 'arquiteto de solucoes', 'dados', 'automacao', 'projetista',
                'suporte', 'quimico', 'qualidade', 'inspecao', 'sap', 'cloud', 'redes',
                'medico', 'enfermeiro', 'fisioterapeuta', 'nutricionista', 'psicologo',
                'laboratorio', 'ambiental', 'devops', 'machine learning', 'blockchain',
                'biomedicina', 'radiologia', 'farmaceutico', 'fonoaudiologo',
                'terapeuta', 'dentista', 'biologo', 'geologo', 'oceanografo', 'edificacoes',
                'desenhista', 'gerente de engenharia', 'gerente industrial', 'laboratorista',
                'especialista', 'técnico', 'mecânico', 'eletricista', 'automação', 'manutenção','instalador', 'programador', 'desenhista', 'projetista', 'auxiliar de ilustração' // Added new technical role
            ],
            'VAGAS PARA QUEM ESTA COMEÇANDO': [
                'estagio', 'estagiario', 'aprendiz', 'trainee', 'jovem aprendiz'
            ],
            'SERVIÇOS': [
                'garçom', 'oficina', 'padaria', 'açougue', 'restaurante', 'hotel', 'pizzaria',
                'churrascaria', 'lanchonete', 'cafeteria', 'cozinha', 'bar', 'balconista', 'copeira', 'copeiro', 'jardineiro', 'auxiliar de acougue' // Added new service roles
            ]
        };    

        function categorizeJob(jobTitle) {
            const normalizedTitle = jobTitle.toLowerCase()
                .normalize("NFD")
                .replace(/[\u0300-\u036f]/g, "");
            
            for (const [category, keywords] of Object.entries(jobCategories)) {
                if (keywords.some(keyword => {
                    const normalizedKeyword = keyword.toLowerCase()
                        .normalize("NFD")
                        .replace(/[\u0300-\u036f]/g, "");
                    return normalizedTitle.includes(normalizedKeyword);
                })) {
                    return category;
                }
            }
            return 'OUTROS';
        }

        // Group jobs by category and location
        let categoryGroups = {};
        for (let local in groupedVagas) {
            groupedVagas[local].forEach(vaga => {
                const category = categorizeJob(vaga.vaga);
                if (!categoryGroups[category]) {
                    categoryGroups[category] = {};
                }
                if (!categoryGroups[category][local]) {
                    categoryGroups[category][local] = [];
                }
                categoryGroups[category][local].push(vaga);
            });
        }

        // Format output by category
        let wikiLinks = "";
        Object.entries(categoryGroups).sort().forEach(([category, locations]) => {
            vagasText += `\n## ${category}\n\n`;
            Object.entries(locations).sort().forEach(([local, vagas]) => {
                vagasText += `📍 ${local}/${vagas[0].sigla}\n`;
                vagas.forEach(vaga => {
                    let cidadeSemSigla = local.replace(/\/[A-Z]{2}$/, '');
                    vagasText += `* ${vaga.vaga}: ${vaga.codigo}\n`;
                    wikiLinks += `[[${vaga.vaga}]] [[${cidadeSemSigla}]]\n`;
                });
                vagasText += '\n';
            });
        });

        return { vagasText, wikiLinks };

    } catch (error) {
        console.error('Erro ao buscar os dados:', error);
        return { vagasText: '', wikiLinks: '' };
    }
}

// Função para corrigir erros ortográficos comuns em títulos de vagas
function corrigir(txt) {
    if (txt) {
        const correcoes = {
            'logistica': 'Logística',
            'movimentacao': 'Movimentação',
            'producao': 'Produção',
            'mecanico': 'Mecânico',
            'tecnico': 'Técnico',
            'seguranca': 'Segurança',
            'expedicao': 'Expedição',
            'estagiario': 'Estagiário',
            'grafico': 'Gráfico',
            'veiculos': 'Veículos',
            'garcom': 'Garçom',
            'Estagio': 'Estágio',
            'acougueiro': 'Açougueiro',
            'eletromecanico': 'Eletromecânico',
            'eletrotecnico': 'Eletrotécnico',
            'contabil': 'Contábil',
            'metalmecanico': 'Metal-mecânico',
            'decoracao': 'Decoração',
            'lustracao': 'Ilustração',
            'manutencao': 'Manutenção',
            'servicos': 'Serviços',
            'operacoes': 'Operações',
            'negocios': 'Negócios',
            'supervisao': 'Supervisão',
            'coordenacao': 'Coordenação',
            'gerencia': 'Gerência',
            'relatorios': 'Relatórios',
            'instalacao': 'Instalação',
            'reparacao': 'Reparação',
            'atendimento': 'Atendimento',
            'comercial': 'Comercial',
            'financas': 'Finanças',
            'logistica reversa': 'Logística Reversa',
            'suporte tecnico': 'Suporte Técnico',
            'producao industrial': 'Produção Industrial',
            'tec ': 'Técnico ',
            'Cabecote': 'Cabeçote',
            'importacao': 'Importação',
            'metalicas': 'Metálicas',
            'fundicao': 'Fundição',
            'processista': 'Processista',
            'injecao': 'Injeção',
            'embarcacao': 'Embarcação',
            'farmaceutico': 'Farmacêutico',
            'senior': 'Sênior',
            'remuneracao': 'Remuneração',
            'mecanica': 'Mecânica',
            'caldereiro': 'Caldeireiro',
            'armazem': 'Armazém',
            'ajudande': 'Ajudante',
            'farmacia': 'Farmácia',
            'aux': 'Auxiliar',
            'maq': 'Maquina',
            'op': 'Operador', 
            'materia': 'Matéria',
            'extrusao': 'Extrusão',
            'processista': 'Processista',
            'producao': 'Produção',
            'coquilhadeira': 'Coquilhadeira',
            'fundicao': 'Fundição',
            'alimenticia': 'Alimentícia',
            'caldereiro': 'Caldeireiro',
            'materia': 'Matéria',
            'jr': 'Júnior',
            'embarcacao': 'Embarcação',
            'expedicao': 'Expedição',
            'caminhao': 'Caminhão',
            'estoquista': 'Estoquista',
            'logistica': 'Logística',
            'supplychain': 'Supply Chain',
            'distribuicao': 'Distribuição',
            'armazem': 'Armazém',
            'selecao': 'Seleção',
            'contabil': 'Contábil',
            'educacao': 'Educação',
            'compliance': 'Compliance',
            'cobranca': 'Cobrança',
            'juridico': 'Jurídico',
            'copywriter': 'Copywriter',
            'pedagogico': 'Pedagógico',
            'administracao': 'Administração',
            'admin': 'Admin',
            'faturamento': 'Faturamento',
            'importacao': 'Importação',
            'remuneracao': 'Remuneração',
            'mecanico': 'Mecânico',
            'servicos': 'Serviços',
            'manutencao': 'Manutenção',
            'mecanica': 'Mecânica',
            'operacoes': 'Operações',
            'seguranca': 'Segurança',
            'tecnico': 'Técnico',
            'solucoes': 'Soluções',
            'automacao': 'Automação',
            'quimico': 'Químico',
            'inspecao': 'Inspeção',
            'psicologo': 'Psicólogo',
            'laboratorio': 'Laboratório',
            'devops': 'DevOps',
            'machinelearning': 'Machine Learning',
            'blockchain': 'Blockchain',
            'farmaceutico': 'Farmacêutico',
            'fonoaudiologo': 'Fonoaudiólogo',
            'biologo': 'Biólogo',
            'geologo': 'Geólogo',
            'oceanografo': 'Oceanógrafo',
            'edificacoes': 'Edificações',
            'estagiario': 'Estagiário' ,
            'secretaria': 'Secretária',
            'Acougue':'Açougue'   };

        let resultado = txt;
        for (let erro in correcoes) {
            const regex = new RegExp("\\b" + erro + "\\b", "gi");
            resultado = resultado.replace(regex, correcoes[erro]);
        }
        resultado = capitalizeTitulo(resultado);
        resultado = corrigirSiglas(resultado);
        return resultado;
    }
    return txt;
}
// Process data and store result
let { vagasText: processedVagasText, wikiLinks } = await fetchData()

// Update file operations with correct path
if (tp.file.title) {
    const newPath = "/Requisição das Unidades/" + tp.file.title;
    try {
        if (!(await tp.file.exists(newPath))) {
            await tp.file.move(newPath);
        }
    } catch (error) {
        console.error('Error moving file:', error);
    }
}

// File renaming with proper path
let newFileName = "/Requisição das Unidades/Requisição " + tp.date.now("DD-MM-YYYY-HHmm");
try {
    if (!(await tp.file.exists(newFileName))) {
        await tp.file.rename(newFileName);
    }
} catch (error) {
    console.error('Error renaming file:', error);
}
-%>

📢 Vagas para <% Estados -%> por meio da RHBrasil!

Candidate-se, pesquisando o código, através do portal do candidato da RHBrasil: http://www.rhbrasil.com.br/portaldocandidato

<% processedVagasText -%>

💼 Para muitas outras vagas, acesse o Portal do Candidato da RHBrasil: http://www.rhbrasil.com.br/portaldocandidato

𝐁𝐨𝐚 𝐬𝐨𝐫𝐭𝐞!
𝐂𝐨𝐧𝐡𝐞𝐜𝐞 𝐚𝐥𝐠𝐮é𝐦 𝐪𝐮𝐞 𝐩𝐨𝐝𝐞 𝐬𝐞 𝐢𝐧𝐭𝐞𝐫𝐞𝐬𝐬𝐚𝐫? 𝐂𝐨𝐦𝐩𝐚𝐫𝐭𝐢𝐥𝐡𝐞 𝐞𝐬𝐭𝐞 𝐩𝐨𝐬𝐭!

RHBrasil, empresa líder em recrutamento e seleção de pessoas no país.

#RHBrasil #RH #Emprego #VagadeEmprego #OportunidadeDeEmprego

<%*
// Cria um array com locais únicos e adiciona hashtags ao conteúdo do arquivo
let uniqueLocations = [...new Set(Object.keys(groupedVagas))];
tR += " " + uniqueLocations.map(local => `#${local.replace(/\s+/g, '')}${groupedVagas[local][0].sigla}`).join(' ');
%>

<% wikiLinks -%>