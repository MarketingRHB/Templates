<%*

try {

    // 1. LER O TEXTO DA ÁREA DE TRANSFERÊNCIA

    const conteudoCsv = await tp.system.clipboard();

    if (!conteudoCsv) {

        throw new Error("A área de transferência está vazia.");

    }

  

    // 2. "TRADUZIR" O CSV PARA OBJETOS QUE O JAVASCRIPT ENTENDE

    const linhas = conteudoCsv.trim().split('\n');

    const cabecalho = linhas[0].split(';').map(h => h.trim().replace(/"/g, ''));

  

    const registrosBrutos = linhas.slice(1).map(linha => {

        // Lida com ponto e vírgula dentro de campos com aspas (ex: biografia)

        const valores = linha.split(';').reduce((acc, val) => {

            if (acc.inQuote) {

                acc.parts[acc.parts.length - 1] += ';' + val;

                if (val.endsWith('"')) {

                    acc.inQuote = false;

                }

            } else {

                acc.parts.push(val);

                if (val.startsWith('"') && !val.endsWith('"')) {

                    acc.inQuote = true;

                }

            }

            return acc;

        }, { parts: [], inQuote: false }).parts;

  

        const registro = {};

        cabecalho.forEach((chave, i) => {

            registro[chave] = (valores[i] || "").trim().replace(/^"|"$/g, '');

        });

        return registro;

    });

  

    // --- FUNÇÕES AUXILIARES ---

  

    function garantirCampos(registro) {

        const camposPadrao = {

            "name": "", "birthday": "", "Email": "", "relationship": "", "CPF": "",

            "Cargo": "", "Departamento": "", "Papel": "", "Gestor Direto": "", "Gênero": "",

            "Data Admissão": "", "Último Acesso": "", "Situação": "Ativo",

            "Gestor Email": "", "notes": "", "phone": "Não informado", "address": "Não informado"

        };

        return { ...camposPadrao, ...registro };

    }

  

    function gerarConteudoYAML(registro) {

        const yaml = `---\n${[

            `ID: ${registro.ID || ''}`,

            `name: "${registro.name || ''}"`,

            `birthday: ${registro.birthday || ''}`,

            `Email: ${registro.Email || ''}`,

            `relationship: "${registro.relationship || ''}"`,

            `CPF: ${registro.CPF || ''}`,

            `Cargo: "${registro.Cargo || ''}"`,

            `Departamento: "${registro.Departamento || ''}"`,

            `Papel: "${registro.Papel || ''}"`,

            `Gestor Direto: "${registro["Gestor Direto"] || ''}"`,

            `Gênero: ${registro.Gênero || ''}`,

            `Data Admissão: ${registro["Data Admissão"] || ''}`,

            `Último Acesso: ${registro["Último Acesso"] || ''}`,

            `Situação: ${registro.Situação || ''}`,

            `Gestor Email: ${registro["Gestor Email"] || ''}`,

            // Escapa aspas simples dentro das notas para não quebrar o YAML

            `notes: '${(registro.notes || '').replace(/'/g, "''")}'`,

            `phone: ${registro.phone || 'Não informado'}`,

            `address: "${registro.address || 'Não informado'}"`

        ].join('\n')}\n---\n`;

  

        const botoes = [

            "\n`BUTTON[Editar-Perfil]`", "```meta-bind-button", "label: Editar Perfil",

            "hidden: true", 'id: "Editar-Perfil"', "style: primary", "actions:",

            "  - type: open", `    link: https://app.feedz.com.br/empresa/colaboradores/${registro.ID}`, "```",

            "", "`BUTTON[Ver-Perfil]`", "```meta-bind-button", "label: Ver Perfil",

            "hidden: true", 'id: "Ver-Perfil"', "style: primary", "actions:",

            "  - type: open", `    link: https://app.feedz.com.br/colaboradores/${registro.ID}/perfil`, "```"

        ].join('\n');

  

        return yaml + botoes;

    }

  

    function formatarCPF(raw) {

        if (!raw) return "";

        const digits = raw.replace(/\D/g, '').padStart(11, '0');

        if (digits === '00000000000') return "";

        return `${digits.slice(0, 3)}.${digits.slice(3, 6)}.${digits.slice(6, 9)}-${digits.slice(9)}`;

    }

  

    function formatarData(raw) {

        if (!raw || !raw.includes('/')) return raw;

        const partes = raw.split(/[\/\s:]+/);

        if (partes.length < 3 || isNaN(parseInt(partes[2]))) return raw;

        return `${partes[2]}-${partes[1].padStart(2, '0')}-${partes[0].padStart(2, '0')}`;

    }

  

    // --- FUNÇÃO PRINCIPAL DE PROCESSAMENTO ---

  

    async function processarRegistro(bruto) {

        const registro = {

            ID: bruto["ID"],

            name: bruto["Nome completo"] || bruto["Nome"],

            birthday: formatarData(bruto["Data de Nascimento"]),

            Email: bruto["Email"],

            relationship: bruto["Unidade"],

            CPF: formatarCPF(bruto["CPF"]),

            Cargo: bruto["Cargo"],

            Departamento: bruto["Departamento"],

            Papel: bruto["Papel"],

            "Gestor Direto": bruto["Gestor Direto"],

            Gênero: bruto["Gênero"],

            "Data Admissão": formatarData(bruto["Data Admissão"]),

            "Último Acesso": formatarData(bruto["Último Acesso"]),

            Situação: bruto["Situação"],

            "Gestor Email": bruto["Gestor Direto - E-mail"],

            notes: bruto["Biografia"]

        };

  

        if (!registro.name || !registro.CPF) {

            console.warn("Registro ignorado por falta de Nome ou CPF:", bruto);

            return;

        }

  

        const nomeArquivo = `${registro.name.replace(/[\/\\?%*:|"<>]/g, '-')}.md`;

        const pastaDestino = registro.Situação.toLowerCase() === "ativo" ? "Colaboradores" : "Ex-Colaboradores";

        const novoCaminho = `${pastaDestino}/${nomeArquivo}`;

  

        // Procura por arquivos com o mesmo CPF em ambas as pastas

        const todosArquivos = app.vault.getMarkdownFiles();

        let sourceFile = null;

        for (const file of todosArquivos) {

            if (file.path.startsWith("Colaboradores/") || file.path.startsWith("Ex-Colaboradores/")) {

                const content = await app.vault.cachedRead(file);

                const cpfMatch = content.match(/CPF:\s*(.*)/);

                if (cpfMatch && cpfMatch[1].trim() === registro.CPF) {

                    sourceFile = file;

                    break;

                }

            }

        }

  

        // Caso 1: Encontrou um arquivo com o mesmo CPF. Vamos atualizar e/ou mover.

        if (sourceFile) {

            const registroCompleto = garantirCampos(registro);

            const conteudoFinal = gerarConteudoYAML(registroCompleto);

  

            // Precisa mover o arquivo? (mudou de status)

            if (sourceFile.parent.name !== pastaDestino) {

                // Verifica se já existe um arquivo com o mesmo nome no destino (um homônimo)

                const collidingFile = app.vault.getAbstractFileByPath(novoCaminho);

                if (collidingFile) {

                    new Notice(`Conflito: ${novoCaminho} já existe. Removendo duplicata.`);

                    await app.vault.delete(collidingFile);

                }

                // Move o arquivo para a pasta correta

                await app.fileManager.renameFile(sourceFile, novoCaminho);

                const movedFile = app.vault.getAbstractFileByPath(novoCaminho);

                await app.vault.modify(movedFile, conteudoFinal);

                new Notice(`Movido e Atualizado: ${registro.name}`);

            } else {

                // Não precisa mover, só atualizar o conteúdo.

                await app.vault.modify(sourceFile, conteudoFinal);

                new Notice(`Atualizado: ${registro.name}`);

            }

        }

        // Caso 2: Não achou arquivo pelo CPF. Vamos criar um novo.

        else {

            let caminhoFinal = novoCaminho;

            // Verifica se já existe um arquivo com esse NOME (homônimo com CPF diferente)

            let contador = 1;

            while (app.vault.getAbstractFileByPath(caminhoFinal)) {

                const novoNomeArquivo = `${registro.name.replace(/[\/\\?%*:|"<>]/g, '-')} (${contador}).md`;

                caminhoFinal = `${pastaDestino}/${novoNomeArquivo}`;

                contador++;

                new Notice(`Homônimo encontrado. Tentando: ${caminhoFinal}`);

            }

  

            const registroCompleto = garantirCampos(registro);

            await app.vault.create(caminhoFinal, gerarConteudoYAML(registroCompleto));

            new Notice(`Criado: ${caminhoFinal.split('/').pop()}`);

        }

    }

  

    // --- EXECUÇÃO ---

    new Notice(`Iniciando processo para ${registrosBrutos.length} registros...`);

    for (const registro of registrosBrutos) {

        await processarRegistro(registro);

    }

    new Notice("Processo concluído!");

  

} catch (error) {

    console.error("Erro no script Templater:", error);

    new Notice("Erro no script! Verifique o console para detalhes (Ctrl+Shift+I).");

}

%>