<%*
const hoje = window.moment();
const doisDiasAtras = window.moment().subtract(2, 'days');

// Pergunta se quer ver o mês todo
const mesInteiro = await tp.system.suggester(
    ["Sim", "Não"],
    ["true", "false"],
    "Mostrar aniversariantes do mês todo?"
);

// === [1] Lista todos os arquivos da pasta "Colaboradores" ===
let todosArquivos = app.vault.getMarkdownFiles();
let colaboradores = todosArquivos.filter(f => f.path.startsWith("Colaboradores/"));

// === [2] Filtra os aniversariantes do mês atual e até 2 dias atrás ===
let aniversariantesRecentes = [];

for (let arquivo of colaboradores) {
    let fm = app.metadataCache.getFileCache(arquivo)?.frontmatter;
    if (!fm || !fm.birthday) continue;

    // birthday no formato YYYY-MM-DD
    let nascimento = window.moment(fm.birthday);
    
    // Ajusta para o ano atual para comparar com "hoje"
    let aniversarioEsteAno = window.moment(fm.birthday).year(hoje.year());

    // Verifica baseado na escolha do usuário
    if (mesInteiro === "true") {
        if (aniversarioEsteAno.month() === hoje.month()) {
            aniversariantesRecentes.push({
                nome: arquivo.basename,
                departamento: fm.Departamento || "Departamento não encontrado",
                genero: fm.Gênero || "Outro",
                data: aniversarioEsteAno,
                hoje: aniversarioEsteAno.isSame(hoje, 'day')
            });
        }
    } else {
        if (
            aniversarioEsteAno.month() === hoje.month() &&
            aniversarioEsteAno.isBetween(doisDiasAtras.clone().subtract(1, 'day'), hoje.clone().add(1, 'day'), null, '[)')
        ) {
            aniversariantesRecentes.push({
                nome: arquivo.basename,
                departamento: fm.Departamento || "Departamento não encontrado",
                genero: fm.Gênero || "Outro",
                data: aniversarioEsteAno,
                hoje: aniversarioEsteAno.isSame(hoje, 'day')
            });
        }
    }
}

// Ordena por dia do mês
aniversariantesRecentes.sort((a, b) => a.data.date() - b.data.date());

// === [3] Se não houver aniversariantes, avisa e encerra
if (aniversariantesRecentes.length === 0) {
    tR += "Nenhum aniversário recente encontrado 🧐";
} else {
    for (let pessoa of aniversariantesRecentes) {
        let artigo = pessoa.genero.toLowerCase() === 'feminino' ? 'da' : 'do';
        let textoData = pessoa.hoje
            ? `Hoje é o aniversário ${artigo}`
            : `Dia ${pessoa.data.format("DD/MM")} foi o aniversário ${artigo}`;
        
        tR += `Uhuuul! ${textoData} @${pessoa.nome} ${artigo} ${pessoa.departamento} 🎁🧧\n\n`;
        tR += `Parabéns e muitos anos de vida!!! 🥳🎉🎊\n\n`;
        tR += `@todos, vamos celebrar com ${artigo === 'da' ? 'ela' : 'ele'}!!! 🎂 🍰 🧁\n\n---\n\n`;
    }
}
%>
