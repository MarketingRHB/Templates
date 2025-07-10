<%*
let rhBrasilData = [];

try {
    const rhBrasilUrl = 'https://www.rhbrasil.com.br/uploads/portal_candidato/vagas-publicas.json';
    const rhBrasilResponse = await fetch(rhBrasilUrl);

    if (!rhBrasilResponse.ok) {
        throw new Error(`Erro ao buscar dados: ${rhBrasilResponse.status} ${rhBrasilResponse.statusText}`);
    }

    rhBrasilData = await rhBrasilResponse.json();
} catch (e) {
    tR += "**Erro ao buscar as vagas da RHBrasil.** Verifique sua conexão com a internet ou tente novamente mais tarde.\n";
    console.error("Erro de fetch:", e.message);
    return; // Abort further execution if fetch fails
}

// Extract unique states
const estadosUnicos = [...new Set(rhBrasilData.flatMap(unidade =>
    unidade.LOCAIS_ATUACAO.map(local => local.LOCAL_ATUACAO.split('/')[1].trim())
))];
let Estados = await tp.system.suggester(estadosUnicos, estadosUnicos);

// Extract unique cities
const cidadesUnicas = [...new Set(rhBrasilData.flatMap(unidade =>
    unidade.LOCAIS_ATUACAO.map(local => local.LOCAL_ATUACAO.split('/')[0].trim())
))];
let Cidade = await tp.system.suggester(cidadesUnicas, cidadesUnicas);

// Extract unique units
const unidadesUnicas = [...new Set(rhBrasilData.map(unidade => unidade.UNIDADE_RHBRASIL))];
let Unidade = await tp.system.suggester(unidadesUnicas, unidadesUnicas);

let Jobcat = await tp.system.suggester(
    ["Indiferente", "PRODUÇÃO", "LOGÍSTICA", "ADMINISTRATIVA", "MANUTENÇÃO", "TÉCNICA", 
     "VAGAS PARA QUEM ESTA COMEÇANDO", "SERVIÇOS"],
    ["Indiferente", "PRODUÇÃO", "LOGÍSTICA", "ADMINISTRATIVA", "MANUTENÇÃO", "TÉCNICA", 
     "VAGAS PARA QUEM ESTA COMEÇANDO", "SERVIÇOS"]);
let QuantasVagas = await tp.system.prompt("Quantas Vagas? (deixe em branco para todas)");

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

// Ensure filteredVagas is defined and accessible
let filteredVagas = []; // Initialize filteredVagas

// Async function to fetch and process data
async function fetchData() {
    try {
        // Fetch data from both sources
        const estadosUrl = 'https://gist.githubusercontent.com/GustavoNeneve/8d09e31038e856be2e5f08effe57bff0/raw/f3c6e21f565e504d19ae6adaebb95dbb3db4cbb6/estados-cidades.json';
        const rhBrasilUrl = 'https://www.rhbrasil.com.br/uploads/portal_candidato/vagas-publicas.json';

        const [estadosResponse, rhBrasilResponse] = await Promise.all([
            fetch(estadosUrl).catch(err => { throw new Error("Erro ao buscar dados de estados.") }),
            fetch(rhBrasilUrl).catch(err => { throw new Error("Erro ao buscar dados de vagas públicas.") })
        ]);

        if (!estadosResponse.ok || !rhBrasilResponse.ok) {
            throw new Error("Erro ao carregar os dados. Verifique as URLs ou a conexão com a internet.");
        }

        const estadosData = await estadosResponse.json();
        const rhBrasilData = await rhBrasilResponse.json();

        // Normalize function
        const normalize = (str) => 
            removeAccents(str || '').toLowerCase().replace(/[^\w\s]/g, '').trim();

        // Map states for easy lookup
        const estados = estadosData.estados.map(estado => ({
            nome: estado.nome,
            sigla: estado.sigla,
            cidades: estado.cidades
        }));

        // Process RH Brasil dataset
        let vagas = [];
        rhBrasilData.forEach(({ UNIDADE_RHBRASIL, LOCAIS_ATUACAO }) => {
            LOCAIS_ATUACAO.forEach(({ LOCAL_ATUACAO, VAGAS }) => {
                const [cidade, sigla] = LOCAL_ATUACAO.split('/');
                VAGAS.forEach(({ NOME_VAGA, CODIGO_VAGA }) => {
                    vagas.push({
                        codigo: CODIGO_VAGA,
                        vaga: corrigirSiglas(capitalizeTitulo(NOME_VAGA)),
                        local: cidade.trim(),
                        sigla: sigla.trim(),
                        unidade: UNIDADE_RHBRASIL,
                        fonte: 'RHBRASIL'
                    });
                });
            });
        });

        // Collect user filters
        const estadoFilter = Estados ? Estados.trim().toUpperCase() : null;
        const cidadeFilter = Cidade ? Cidade.trim().toUpperCase() : null;
        const unidadeFilter = Unidade ? Unidade.trim().toUpperCase() : null;

        // Filter vacancies based on user input
        filteredVagas = vagas.filter(vaga => {
            const matchEstado = !estadoFilter || vaga.sigla === estadoFilter;
            const matchCidade = !cidadeFilter || vaga.local.toUpperCase() === cidadeFilter;
            const matchUnidade = !unidadeFilter || vaga.unidade.toUpperCase() === unidadeFilter;
            return matchEstado && matchCidade && matchUnidade;
        });

        // Filter by category if specified
        if (Jobcat !== 'Indiferente') {
            filteredVagas = filteredVagas.filter(vaga => categorizeJob(vaga.vaga) === Jobcat);
        }

        // Limit quantity if specified
        if (QuantasVagas) {
            filteredVagas = filteredVagas.slice(0, parseInt(QuantasVagas));
        }

        // Categorize jobs
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

        // Group and format output
        let vagasText = '';
        let wikiLinks = '';
        let groupedVagas = {}; // Ensure groupedVagas is defined

        groupedVagas = filteredVagas.reduce((acc, vaga) => {
            const category = categorizeJob(vaga.vaga);
            const localKey = `${vaga.local}/${vaga.sigla}`;
            acc[category] = acc[category] || {};
            acc[category][localKey] = acc[category][localKey] || [];
            acc[category][localKey].push(vaga);
            return acc;
        }, {});

        Object.keys(groupedVagas).sort().forEach(category => {
            vagasText += `\n## ${category}\n\n`;
            Object.keys(groupedVagas[category]).sort().forEach(local => {
                vagasText += `📍 ${local}\n`;
                groupedVagas[category][local].forEach(vaga => {
                    vagasText += `* ${vaga.vaga}: ${vaga.codigo}\n`;
                    wikiLinks += `[[${vaga.vaga}]] [[${vaga.local}]]\n`;
                });
                vagasText += '\n';
            });
        });

        // Ensure data is returned correctly
        return { vagasText, wikiLinks, groupedVagas };

    } catch (error) {
        console.error('Erro ao processar dados:', error.message);
        return { vagasText: 'Erro ao carregar vagas. Verifique sua conexão ou os dados fornecidos.', wikiLinks: '', groupedVagas: {} };
    }
}

const { vagasText, wikiLinks, groupedVagas } = await fetchData();

// Ensure output is written
if (!vagasText.trim() || vagasText.includes("Erro ao carregar vagas")) {
    console.error('Nenhuma vaga encontrada ou erro ao carregar os dados.');
    tR += "Nenhuma vaga encontrada ou erro ao carregar os dados.";
} else {
    tR += vagasText;
}

-%>

📢 Vagas para <% Estados -%> por meio da RHBrasil!

Candidate-se, pesquisando o código, através do portal do candidato da RHBrasil: http://www.rhbrasil.com.br/portaldocandidato

<% vagasText %>

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

<% wikiLinks -%>