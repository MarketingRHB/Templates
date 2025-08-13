<%*
try {
  let clipboard = await tp.system.clipboard();
  let data = clipboard;

  function extractData(regex, fallback = "") {
    const match = data.match(regex);
    return match ? match[1].trim() : fallback;
  }

  function capitalizeWords(text) {
    return text.toLowerCase().replace(/\b\w/g, c => c.toUpperCase());
  }

  function formatCPF(raw) {
    const digits = raw.replace(/\D/g, "");
    if (digits.length !== 11) return raw;
    return `${digits.slice(0,3)}.${digits.slice(3,6)}.${digits.slice(6,9)}-${digits.slice(9,11)}`;
  }

  function formatDateToISO(date) {
    const [day, month, year] = date.split("/");
    return `${year}-${month}-${day}`;
  }

  let nome = capitalizeWords(extractData(/Nome - (.+?)$/m));
  let setor = capitalizeWords(extractData(/setor (.+?) da unidade/m));
  let unidade = capitalizeWords(extractData(/Unidade - (.+?) -/m));
  let unidade2 = capitalizeWords(extractData(/RHBRASIL - (.+?)$/m));
  let email = extractData(/E-mail - (.+?)$/m);
  let cpf = formatCPF(extractData(/CPF - (.+?)$/m));
  let dataDeNascimento = extractData(/Data de Nascimento - (.+?)$/m);
  let dataDeAdmisao = extractData(/Admissão - (.+?)$/m);
  let cargo = capitalizeWords(extractData(/Cargo - (.+?)$/m));
  let gestor = capitalizeWords(extractData(/Gestor - (.+?)$/m));

  function normalizarSituacao(texto) {
    const t = (texto || "").trim().toLowerCase();
    if (t.startsWith("ativo")) return "Ativo";
    if (["desligado", "desativado", "inativo"].some(x => t.startsWith(x))) {
      return "Inativo";
    }
    return "Ativo";
  }

  let id = extractData(/ID - (.+?)$/m);
  let situacao = normalizarSituacao(extractData(/Situa(?:ção|cao) - (.+?)$/m, "Ativo"));

  let birthday = formatDateToISO(dataDeNascimento);
  let admissao = formatDateToISO(dataDeAdmisao);

  const pastaAtivo = "Colaboradores";
  const pastaInativo = "Ex-Colaboradores";
  const nomeArquivo = `${nome.replace(/[\\/\\?%*:|"<>]/g, '-')}.md`;
  const destino = `${situacao.toLowerCase() === 'ativo' ? pastaAtivo : pastaInativo}/${nomeArquivo}`;

  async function encontrarArquivo() {
    const arquivos = app.vault.getMarkdownFiles();
    const cpfDigits = cpf.replace(/\D/g, '');
    for (const file of arquivos) {
      if (file.path.startsWith(pastaAtivo + '/') || file.path.startsWith(pastaInativo + '/')) {
        const conteudo = await app.vault.cachedRead(file);
        const cpfMatch = conteudo.match(/cpf:\s*(.*)/i);
        const nomeMatch = conteudo.match(/name:\s*(.*)/i);
        const cpfFile = cpfMatch ? cpfMatch[1].replace(/\D/g, '') : '';
        const nomeFile = nomeMatch ? nomeMatch[1].trim().toLowerCase() : '';
        if (cpfFile === cpfDigits || nomeFile === nome.toLowerCase()) {
          return file;
        }
      }
    }
    return null;
  }

  function gerarBotoes(idValue) {
    if (!idValue) return '';
    return [
      'BUTTON[Editar Perfil]',
      '```meta-bind-button',
      'label: Editar Perfil',
      'icon: ""',
      'style: primary',
      'class: ""',
      'cssStyle: ""',
      'backgroundImage: ""',
      'tooltip: Tooltip',
      'id: EditarPerfil',
      'hidden: false',
      'actions:',
      '  - type: open',
      `    link: https://app.feedz.com.br/empresa/colaboradores/${idValue}`,
      '    newTab: true',
      '```',
      '',
      'BUTTON[Ver Perfil]',
      '```meta-bind-button',
      'label: Ver Perfil',
      'icon: ""',
      'style: secondary',
      'class: ""',
      'cssStyle: ""',
      'backgroundImage: ""',
      'tooltip: Tooltip',
      'id: VerPerfil',
      'hidden: false',
      'actions:',
      '  - type: open',
      `    link: https://app.feedz.com.br/colaboradores/${idValue}/perfil`,
      '    newTab: true',
      '```'
    ].join('\n');
  }

  const yaml = [
    '---',
    `name: ${nome}`,
    `birthday: ${birthday}`,
    `cpf: ${cpf}`,
    `email: ${email}`,
    `data_de_admissao: ${admissao}`,
    `cargo: ${cargo}`,
    `departamento: ${setor}`,
    `unidade: ${unidade} - ${unidade2}`,
    `gestor: ${gestor}`,
    `situacao: ${situacao}`,
    'papel: Colaborador',
    'genero:',
    'ultimo_acesso:',
    '---',
    gerarBotoes(id)
  ].join('\n');

  const existente = await encontrarArquivo();
  if (existente) {
    if (existente.path !== destino) {
      await app.fileManager.renameFile(existente, destino);
    }
    await app.vault.modify(existente, yaml);
    new Notice(`Atualizado: ${nome}`);
  } else {
    await app.vault.create(destino, yaml);
    new Notice(`Criado: ${nome}`);
  }

} catch (err) {
  console.error('Erro no template FeedzAdicionar:', err);
  new Notice('Erro ao executar template. Veja o console.');
}
%>
