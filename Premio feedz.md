<%*

// ---------------------------------------------------------------------------------

//                          PROCESSADOR AUTOMÁTICO DE PRÊMIOS (DIRETO)

// ---------------------------------------------------------------------------------

// Autor: Seu Assistente de IA & Gustavo

// Versão: 6.3 (Otimizado: Inelegibilidade baseada na leitura de relatórios 'Premiados' anteriores)

//

// Descrição:

// O script foi otimizado para determinar a inelegibilidade de forma mais direta,

// lendo os relatórios "Premiados_..." dos 2 meses anteriores.

//

// Funcionalidades:

// - Lógica de inelegibilidade otimizada para ler relatórios passados.

// - Limite de Ranking: A seleção de vencedores é feita a partir do Top 10.

// - Totalmente autocontido, sem dependências externas.

//

// Pré-requisitos:

// - Nenhum.

// ---------------------------------------------------------------------------------

  

// --- INÍCIO DA CONFIGURAÇÃO ---

const NOME_COLUNA_NOME = "Colaborador";

const NOME_COLUNA_PAPEL = "Papel";

const NOME_COLUNA_MOEDAS = "Moedas";

const PAPEL_COLABORADOR = "Colaborador";

const PAPEIS_GESTAO = ["Administrador", "Gestor"];

const QTD_VENCEDORES_COLABORADOR = 3;

const QTD_VENCEDORES_GESTAO = 1;

const LIMITE_RANKING = 10;

  

// --- FIM DA CONFIGURAÇÃO ---

  

/**

 * Lê e converte um arquivo com formato CSV (.md ou .csv) para um array de objetos.

 */

async function lerDadosCsv(fileObject) {

    console.log(`[lerDadosCsv] Lendo: ${fileObject.name}`);

    try {

        const content = await app.vault.read(fileObject);

        const lines = content.trim().split(/\r?\n/);

        if (lines.length < 2) return [];

        const headers = lines[0].split(';').map(h => h.trim());

        return lines.slice(1).map(line => {

            const values = line.split(';').map(v => v.trim());

            const obj = {};

            headers.forEach((h, i) => {

                const val = values[i];

                obj[h] = !isNaN(parseFloat(val)) && isFinite(val) ? parseFloat(val) : val;

            });

            return obj;

        });

    } catch (e) {

        console.error(`Erro ao ler CSV: ${fileObject.name}`, e);

        return [];

    }

}

  

/**

 * Extrai e converte uma data 'DD-MM-YYYY' do nome do arquivo para um objeto Date.

 */

function parseDataDoNome(filename) {

    const match = filename.match(/_(\d{2})-(\d{2})-(\d{4})\./i);

    if (!match) return null;

    const date = new Date(match[3], match[2] - 1, match[1]);

    return isNaN(date.getTime()) ? null : date;

}

  

/**

 * Formata um objeto Date para uma string 'DD-MM-YYYY'.

 */

function formatarData(date) {

    const dia = String(date.getDate()).padStart(2, '0');

    const mes = String(date.getMonth() + 1).padStart(2, '0');

    const ano = date.getFullYear();

    return `${dia}-${mes}-${ano}`;

}

  

/**

 * NOVO: Lê um relatório 'Premiados_...' e extrai os nomes dos vencedores.

 * @param {TFile} fileObject - O arquivo de relatório a ser lido.

 * @returns {Promise<Set<string>>} - Um Set com os nomes dos vencedores.

 */

async function getNomesVencedoresDeRelatorio(fileObject) {

    const vencedores = new Set();

    try {

        const content = await app.vault.read(fileObject);

        const regex = /- \*\*(.*?)\*\*/g; // Padrão para encontrar "- **Nome do Vencedor**"

        let match;

        while ((match = regex.exec(content)) !== null) {

            vencedores.add(match[1].trim());

        }

    } catch(e) {

        console.error(`Erro ao ler relatório de vencedores: ${fileObject.name}`, e);

    }

    return vencedores;

}

  

/**

 * Função principal que orquestra todo o processo.

 */

async function executarProcessamento() {

    new Notice("Iniciando processador de prêmios...", 3000);

    console.log("🚀 INICIANDO PROCESSADOR (v6.3) 🚀");

  

    const todosOsArquivos = app.vault.getFiles();

    const sourceFiles = todosOsArquivos.filter(f =>

        f.name.toLowerCase().startsWith('historicomoedas') && (f.extension === 'md' || f.extension === 'csv')

    );

    const premiadosFiles = todosOsArquivos.filter(f =>

        f.name.toLowerCase().startsWith('premiados_') && f.extension === 'md'

    );

  

    console.log(`🔎 Encontrados ${sourceFiles.length} arquivos de 'HistoricoMoedas'.`);

    console.log(`📄 Encontrados ${premiadosFiles.length} relatórios 'Premiados' existentes.`);

  

    const datasProcessadas = new Set(

        premiadosFiles.map(f => {

            const date = parseDataDoNome(f.name);

            return date ? formatarData(date) : null;

        }).filter(Boolean)

    );

  

    const toProcess = sourceFiles.filter(f => {

        const date = parseDataDoNome(f.name);

        return date && !datasProcessadas.has(formatarData(date));

    });

  

    if (toProcess.length === 0) {

        new Notice("Nenhum relatório novo para processar.", 5000);

        return;

    }

  

    toProcess.sort((a, b) => {

        const da = parseDataDoNome(a.name);

        const db = parseDataDoNome(b.name);

        return (da?.getTime() || 0) - (db?.getTime() || 0);

    });

  

    let count = 0;

    for (const sf of toProcess) {

        const date = parseDataDoNome(sf.name);

        if (!date) continue;

  

        console.log(`\n--- Processando: ${sf.name} ---`);

        new Notice(`Processando: ${sf.name}`, 3000);

  

        // OTIMIZAÇÃO: Lógica de inelegibilidade agora lê relatórios 'Premiados_...'

        const dataMes1 = new Date(date.getFullYear(), date.getMonth() - 1, 1);

        const prev1 = `${String(dataMes1.getMonth() + 1).padStart(2, '0')}-${dataMes1.getFullYear()}`;

        const dataMes2 = new Date(date.getFullYear(), date.getMonth() - 2, 1);

        const prev2 = `${String(dataMes2.getMonth() + 1).padStart(2, '0')}-${dataMes2.getFullYear()}`;

        console.log(`Lendo relatórios anteriores para inelegibilidade dos meses: ${prev1} e ${prev2}`);

        const inelegiveis = new Set();

        for (const relatorio of premiadosFiles) {

             const relatorioDate = parseDataDoNome(relatorio.name);

             if (relatorioDate) {

                 const relatorioKey = `${String(relatorioDate.getMonth() + 1).padStart(2, '0')}-${relatorioDate.getFullYear()}`;

                 if (relatorioKey === prev1 || relatorioKey === prev2) {

                     console.log(`[Inelegibilidade] Analisando vencedores de ${relatorio.name}`);

                     const vencedoresAnteriores = await getNomesVencedoresDeRelatorio(relatorio);

                     vencedoresAnteriores.forEach(v => inelegiveis.add(v));

                 }

             }

        }

        console.log(`Lista de inelegíveis (vencedores dos 2 meses anteriores):`, Array.from(inelegiveis));

  

        const current = await lerDadosCsv(sf);

        if (current.length === 0) continue;

  

        const colab = current.filter(r => r[NOME_COLUNA_PAPEL] === PAPEL_COLABORADOR)

                             .sort((a, b) => (b[NOME_COLUNA_MOEDAS] || 0) - (a[NOME_COLUNA_MOEDAS] || 0))

                             .slice(0, LIMITE_RANKING);

        const gest = current.filter(r => PAPEIS_GESTAO.includes(r[NOME_COLUNA_PAPEL]))

                            .sort((a, b) => (b[NOME_COLUNA_MOEDAS] || 0) - (a[NOME_COLUNA_MOEDAS] || 0))

                            .slice(0, LIMITE_RANKING);

        console.log(`Aplicando limite de ranking (Top ${LIMITE_RANKING}): ${colab.length} colaboradores e ${gest.length} gestores pré-selecionados.`);

  

        const pick = (list, q) => list.filter(r => !inelegiveis.has(r[NOME_COLUNA_NOME])).slice(0, q);

        const winCol = pick(colab, QTD_VENCEDORES_COLABORADOR);

        const winGest = pick(gest, QTD_VENCEDORES_GESTAO);

  

        const mesAnoFormatado = `${date.toLocaleString('pt-BR', { month: 'long' })} de ${date.getFullYear()}`;

        let md = `## 🏆 Vencedores do Mês - ${mesAnoFormatado}\n\n`;

        md += `Análise baseada no arquivo: \`${sf.name}\`\n\n### 🏅 Colaboradores\n`;

        md += winCol.length ? winCol.map(r => `- **${r[NOME_COLUNA_NOME]}** (${r[NOME_COLUNA_MOEDAS]} moedas)`).join('\n')

                             : '- *Não foram encontrados colaboradores elegíveis.*';

        md += '\n\n### 👑 Gestão / Administração\n';

        md += winGest.length ? winGest.map(r => `- **${r[NOME_COLUNA_NOME]}** (${r[NOME_COLUNA_MOEDAS]} moedas)`).join('\n')

                              : '- *Não foi encontrado um gestor/administrador elegível.*';

  

        const dataString = formatarData(date);

        const novoNomeArquivo = `Premiados_${dataString}.md`;

        const outPath = `${sf.parent.path}/${novoNomeArquivo}`;

  

        console.log(`📝 Criando arquivo de relatório em: ${outPath}`);

        await app.vault.create(outPath, md);

        count++;

    }

    new Notice(`Concluído: ${count} relatórios gerados.`, 5000);

    console.log(`\n🎉 Processo concluído! ${count} relatório(s) gerado(s).`);

}

  

// Inicia a execução

try {

    await executarProcessamento();

} catch (error) {

    console.error("❌ Erro inesperado na execução principal do script:", error);

    new Notice("Ocorreu um erro crítico no script. Verifique o console.", 10000);

}

%>