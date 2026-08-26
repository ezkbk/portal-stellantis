# Passagem de bastão — Portal Stellantis para a Cláudia

Objetivo: a **Cláudia passa a ser a responsável** por manter o Portal Stellantis (atualizar números,
criar usuários, mudanças de layout que o cliente pedir) **do notebook dela, sem depender do Israel** —
e o **Israel não perde o controle**.

Como isso funciona sem transferir a propriedade: o portal mora no repositório GitHub do Israel e sobe
sozinho no Netlify a cada push. A Cláudia entra como **colaboradora** (edita e publica), mas o **Israel
continua dono** do repositório e do Netlify. Todo push dela fica registrado no histórico — dá para auditar
e reverter. Ele pode remover o acesso dela a qualquer momento.

---

## Parte A — O que o ISRAEL faz (uma vez, ~15 min)

Precisa do **usuário/e-mail GitHub da Cláudia**. Se ela não tiver conta, ela cria primeiro (Parte B, passo 1).

1. **Adicionar a Cláudia como colaboradora do repositório:**
   - Ir em `https://github.com/ezkbk/portal-stellantis` → aba **Settings** → **Collaborators** →
     **Add people** → digitar o usuário/e-mail dela → permissão **Write** → convidar.
   - Ela recebe um e-mail e precisa **aceitar** o convite.

2. **Dar acesso a ela no Netlify** (para domínio, senha do site, redeploys):
   - No Netlify, abrir o time onde está o `portal-stellantis` → **Team settings** → **Members** →
     **Invite members** → e-mail dela → papel de membro. Ela aceita por e-mail.

3. **Passar os arquivos de referência** (já estão nesta pasta, vão junto no repositório):
   - `CLAUDE.md` — runbook técnico (o Claude dela lê sozinho).
   - `ACESSOS_Portal_Stellantis.md` — logins e senhas atuais.
   - Este arquivo.

4. **(Controle)** Manter para si:
   - A **propriedade** do repositório (ela é só colaboradora — dá para revogar em Settings → Collaborators).
   - A **propriedade** do time no Netlify.
   - Um **login global próprio** no portal (o `israel.kubiaki@enerbrax.com`) — não precisa apagar.

## Parte B — O que a CLÁUDIA faz (uma vez)

1. **Conta no GitHub** (se ainda não tiver): criar em `https://github.com` com o e-mail dela e mandar o
   usuário para o Israel (Parte A, passo 1). Aceitar o convite do repositório quando chegar.

2. **Instalar o Claude/Cowork** no notebook dela (a mesma ferramenta que o Israel usa).

3. **Instalar o GitHub Desktop** (`https://desktop.github.com`) e fazer login com a conta GitHub dela.

4. **Clonar o repositório:** GitHub Desktop → **File → Clone repository** → escolher
   `ezkbk/portal-stellantis` → salvar numa pasta do notebook dela (ex.: `Documentos\portal-stellantis`).

5. **Dar acesso da pasta ao Claude/Cowork:** abrir o Cowork e **selecionar a pasta clonada**. Pronto —
   o Claude dela vai ler o `CLAUDE.md` e já saber manter o portal.

## Parte C — O dia a dia da Cláudia

1. **Antes de começar:** no GitHub Desktop, clicar **Fetch/Pull** para pegar a versão mais recente
   (caso o Israel tenha mexido).
2. Pedir a mudança ao Claude (atualizar mês, criar usuário, ajuste de layout…).
3. O Claude edita o `index.html` **e valida** (sintaxe + carga + testes — ver `CLAUDE.md` seção 7).
4. No GitHub Desktop: escrever um resumo curto → **Commit to main** → **Push origin**.
5. Esperar 1–2 min e conferir em `https://portal-stellantis.netlify.app/`.

## Parte D — Como o Israel mantém o controle

- **Dono do repositório** = Israel. Colaborador ≠ dono; acesso pode ser revogado quando quiser.
- **Dono do Netlify** = Israel.
- **Histórico do Git** = trilha de auditoria: todo commit da Cláudia aparece com data, autor e o que mudou;
  qualquer alteração pode ser revertida (Right-click no commit → Revert).
- **Backup**: a pasta continua no OneDrive do Israel também.
- Se um dia quiser trancar mais: dá para exigir revisão (branch protection) antes de publicar — hoje está
  no fluxo simples (push direto), que é o mais rápido para quem é a responsável.

## Resumo em uma linha

Israel **duplica o acesso** (colaboradora no GitHub + membro no Netlify) e **mantém a propriedade**;
Cláudia trabalha no notebook dela pelo Claude + GitHub Desktop; cada mudança fica registrada e reversível.
