<%*
// Prompt the user to select the type of request and filter criteria
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

%>