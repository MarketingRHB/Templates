<%*
try {
    const conteudo = await tp.system.clipboard();

    if (!conteudo) {
        throw new Error("A área de transferência está vazia.");
    }

    const yamlSemDelimitadores = conteudo.trim().replace(/^---\s*\n?|---\s*$/g, '');
    const registrosBrutos = await tp.user.parseYaml(yamlSemDelimitadores);

    function garantirCampos(registro) {
        const campos = {
            "name": "",
            "birthday": "",
            "Email": "",
            "relationship": "",
            "CPF": "",
            "Cargo": "",
            "Departamento": "",
            "Papel": "",
            "Gestor Direto": "",
            "Gênero": "",
            "Data Admissão": "",
            "Último Acesso": "",
            "Situação": "Ativo",
            "Gestor Email": "",
            "notes": "",
            "phone": "Tem que ser adicionado",
            "address": "Tem que ser adicionado com aspas"
        };
        return { ...campos, ...registro };
    }

    function gerarConteudoYAML(registro) {
        const yaml = `---\n${[
            `ID: ${registro.ID}`,
            `name: ${registro.name}`,
            `birthday: ${registro.birthday}`,
            `Email: ${registro.Email}`,
            `relationship: ${registro.relationship}`,
            `CPF: ${registro.CPF}`,
            `Cargo: ${registro.Cargo}`,
            `Departamento: ${registro.Departamento}`,
            `Papel: ${registro.Papel}`,
            `Gestor Direto: ${registro["Gestor Direto"]}`,
            `Gênero: ${registro.Gênero}`,
            `Data Admissão: ${registro["Data Admissão"]}`,
            `Último Acesso: ${registro["Último Acesso"]}`,
            `Situação: ${registro.Situação}`,
            `Gestor Email: ${registro["Gestor Email"]}`,
            `notes: '${registro.notes.replace(/'/g, "\\'")}'`,
            `phone: ${registro.phone}`,
            `address: "${registro.address.replace(/"/g, '')}"`
        ].join('\n')}\n---\n`;

        const botoes = [
            "`BUTTON[Editar-Perfil]`",
            "```meta-bind-button",
            "label: Editar Perfil",
            "hidden: true",
            'id: "Editar-Perfil"',
            "style: primary",
            "actions:",
            "  - type: open",
            `    link: https://app.feedz.com.br/empresa/colaboradores/${registro.ID}`,
            "```",
            "",
            "`BUTTON[Ver-Perfil]`",
            "```meta-bind-button",
            "label: Ver Perfil",
            "hidden: true",
            'id: "Ver-Perfil"',
            "style: primary",
            "actions:",
            "  - type: open",
            `    link: https://app.feedz.com.br/colaboradores/${registro.ID}/perfil`,
            "```"
        ].join('\n');

        return yaml + botoes;
    }

    function formatarCPF(raw) {
        const digits = raw.replace(/\D/g, '');
        if (digits.length !== 11) return raw;
        return `${digits.slice(0, 3)}.${digits.slice(3, 6)}.${digits.slice(6, 9)}-${digits.slice(9)}`;
    }

    function formatarData(raw) {
        if (!raw || !raw.includes('/')) return raw;
        const partes = raw.split(/[\/\s:]+/);
        if (partes.length < 3) return raw;
        return `${partes[2]}-${partes[1].padStart(2, '0')}-${partes[0].padStart(2, '0')}`;
    }

    async function encontrarArquivoExistente(nomeArquivo) {
        const pastas = ["Colaboradores", "Ex-Colaboradores"];
        for (const pasta of pastas) {
            const caminho = `${pasta}/${nomeArquivo}`;
            const existe = await app.vault.adapter.exists(caminho);
            if (existe) {
                return caminho;
            }
        }
        return null;
    }

    async function processarRegistro(bruto) {
        const registro = {
            name: bruto["Nome"] || "",
            birthday: formatarData(bruto["Data de Nascimento"]),
            Email: bruto["Email"] || "",
            relationship: bruto["Unidade"] || "",
            CPF: formatarCPF(bruto["CPF"] || ""),
            Cargo: bruto["Cargo"] || "",
            Departamento: bruto["Departamento"] || "",
            Papel: bruto["Papel"] || "",
            "Gestor Direto": bruto["Gestor Direto"] || "",
            Gênero: bruto["Gênero"] || "",
            "Data Admissão": formatarData(bruto["Data Admissão"]),
            "Último Acesso": formatarData(bruto["Último Acesso"]),
            Situação: bruto["Situação"] || "Ativo",
            "Gestor Email": bruto["Gestor Direto - E-mail"] || "",
            notes: bruto["Biografia"] || "",
            ID: bruto["ID"] || ""
        };

        if (!registro.name || !registro.CPF) {
            console.warn("Registro ignorado por falta de dados essenciais:", registro);
            return;
        }

        const nomeArquivo = `${registro.name}.md`;
        const situacao = registro.Situação.toLowerCase();
        const pastaAtual = situacao === "ativo" ? "Colaboradores" : "Ex-Colaboradores";
        const novoCaminho = `${pastaAtual}/${nomeArquivo}`;

        const caminhoExistente = await encontrarArquivoExistente(nomeArquivo);
        const conteudoExistente = caminhoExistente
            ? await app.vault.adapter.read(caminhoExistente)
            : null;

        if (conteudoExistente) {
            const cpfExistente = conteudoExistente.match(/CPF: (.*)/)?.[1]?.trim() || '';

            if (cpfExistente === registro.CPF) {
                const yamlExistente = conteudoExistente.match(/---\n([\s\S]*?)\n---/)?.[1] || "";
                const registroExistente = await tp.user.parseYaml(yamlExistente);
                const dadosMesclados = { ...registroExistente, ...registro };

                const registroCompleto = garantirCampos(dadosMesclados);
                const conteudoFinal = gerarConteudoYAML(registroCompleto);

                if (!caminhoExistente.startsWith(pastaAtual)) {
                    await app.vault.adapter.remove(caminhoExistente);
                    console.log(`Movido de ${caminhoExistente} para ${novoCaminho}`);
                }

                await app.vault.adapter.write(novoCaminho, conteudoFinal);
                console.log(`Atualizado: ${novoCaminho}`);
                return;
            } else {
                let contador = 1;
                let novoNomeArquivo = `${registro.name} (${contador}).md`;
                let novoCaminhoTentativa = `${pastaAtual}/${novoNomeArquivo}`;

                while (await app.vault.adapter.exists(novoCaminhoTentativa)) {
                    contador++;
                    novoNomeArquivo = `${registro.name} (${contador}).md`;
                    novoCaminhoTentativa = `${pastaAtual}/${novoNomeArquivo}`;
                }

                const registroCompleto = garantirCampos(registro);
                await app.vault.adapter.write(novoCaminhoTentativa, gerarConteudoYAML(registroCompleto));
                console.log(`Criado novo arquivo para CPF diferente: ${novoCaminhoTentativa}`);
                return;
            }
        } else {
            const registroCompleto = garantirCampos(registro);
            await app.vault.adapter.write(novoCaminho, gerarConteudoYAML(registroCompleto));
            console.log(`Novo arquivo criado: ${novoCaminho}`);
        }
    }

    for (const registro of registrosBrutos) {
        await processarRegistro(registro);
    }

} catch (error) {
    console.error("Erro:", error.message);
}
%>
