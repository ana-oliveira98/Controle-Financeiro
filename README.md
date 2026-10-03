# Controle Financeiro

Sistema web de controle financeiro doméstico: receitas mês a mês, despesas fixas, variáveis e de cartão, regra 50/30/20, categorias, gráficos, cobranças compartilhadas entre usuários e exportação para Excel.

É um site estático (HTML + CSS + JavaScript). Login e dados ficam no **Firebase** (projeto `domestic-budget-27ab7`).

## Arquivos

| Arquivo | Para que serve |
|---|---|
| `index.html` | O sistema inteiro (página, lógica e conexão com o Firebase) |
| `styles.css` | Estilos gerados pelo Tailwind a partir do `index.html` |
| `firestore.rules` | Regras de segurança do banco de dados (publicar no Firebase Console) |
| `tailwind.config.js`, `src/input.css` | Configuração para gerar o `styles.css` novamente |
| `manifest.webmanifest`, `icons/` | Configuração e ícones do app instalável (tela de início do celular) |

## Instalar como app no celular

- **iPhone/iPad:** abra o site no Safari → botão **Compartilhar** → **Adicionar à Tela de Início**.
- **Android:** abra no Chrome → menu **⋮** → **Instalar app** (ou "Adicionar à tela inicial").

O app abre em tela cheia, com ícone próprio. Ele se atualiza sozinho quando uma nova versão é publicada no GitHub; se não aparecer, feche o app e abra de novo.
No iPhone, o app instalado **não compartilha o login com o Safari**: na primeira vez, é preciso entrar com e-mail e senha.

## Colocar no ar (GitHub Pages)

1. Repositório: https://github.com/ana-oliveira98/Controle-Financeiro
2. Em **Add file → Upload files**, envie `index.html`, `styles.css`, `firestore.rules`, `tailwind.config.js`, `README.md`, `.gitignore` e a pasta `src`. Clique em **Commit changes**.
3. Em **Settings → Pages**, em *Branch*, escolha `main` e a pasta `/ (root)`. Salve.
4. Em alguns minutos o site estará em https://ana-oliveira98.github.io/Controle-Financeiro/.

## Configuração obrigatória no Firebase

No [Firebase Console](https://console.firebase.google.com), projeto `domestic-budget-27ab7`:

1. **Authentication → Settings → Authorized domains**: adicione `ana-oliveira98.github.io`.
   Sem isso, o login não funciona no site publicado.
2. **Firestore Database → Regras**: cole o conteúdo de `firestore.rules` e clique em **Publicar**.
   Sempre que o arquivo mudar, publique de novo.

3. **Authentication → Templates → Redefinição de senha → ✏️ → Personalizar URL de ação**: informe
   `https://ana-oliveira98.github.io/Controle-Financeiro/` e salve.
   Assim, o link do e-mail abre a tela de troca de senha do próprio sistema, em português. O link só é usado quando a pessoa salva a senha nova, e se ele estiver vencido aparece a opção de pedir outro.
   Essa URL vale para todos os modelos de e-mail; o sistema também trata a confirmação e a recuperação de e-mail.

### Recomendado

- **Restringir a chave da API** no [Google Cloud Console](https://console.cloud.google.com/apis/credentials) (projeto `domestic-budget-27ab7`):
  abra a chave "Browser key", marque *Sites* e informe `https://ana-oliveira98.github.io/*` e `http://localhost/*`.
  A chave que aparece no `index.html` é pública por natureza no Firebase; quem protege os dados são as regras do Firestore. A restrição evita que a chave seja usada a partir de outros sites.
- **Authentication → Settings → User actions**: mantenha a *proteção contra enumeração de e-mail* ativada.
- **Modelos de e-mail** (Authentication → Templates): traduza o e-mail de redefinição de senha para português.

## Atualizar o sistema

- Alterou só lógica ou textos? Envie o novo `index.html` ao repositório.
- Adicionou ou mudou **classes do Tailwind** no `index.html`? Gere o `styles.css` de novo:
  1. Baixe o [Tailwind CLI para Windows (v3.4.17)](https://github.com/tailwindlabs/tailwindcss/releases/download/v3.4.17/tailwindcss-windows-x64.exe) e salve nesta pasta como `tailwindcss.exe`.
  2. No PowerShell, dentro desta pasta:
     ```
     .\tailwindcss.exe -c tailwind.config.js -i src/input.css -o styles.css --minify
     ```
  3. Envie o `index.html` e o `styles.css` atualizados.

Depois de publicar, peça para os usuários recarregarem com **Ctrl+F5**, para não ficarem com a versão antiga em cache.

## Onde ficam os dados (Firestore)

| Coleção | Conteúdo | Quem acessa |
|---|---|---|
| `paineis/{uid}` | Todos os lançamentos, receitas e contatos do usuário | Só o próprio usuário |
| `usuarios/{email}` | Registro para saber se um e-mail tem cadastro | Qualquer usuário logado pode consultar **um** e-mail; ninguém pode listar |
| `cobrancas/{id}` | Cobranças enviadas entre usuários | Quem cobrou e quem deve |
| `acessos/{email}` | E-mails liberados para usar o sistema | Só a administradora vê e altera; cada pessoa só confere o próprio e-mail |
| `config/categorias` | Categorias personalizadas | Todos com acesso leem; só a administradora altera |
| `dividas/{id}` | Avisos "devo a" enviados por quem deve | Quem deve (cria e altera) e quem vai receber (confirma o recebimento) |

## Funcionalidades principais

- **Lançamentos:** data completa (dia/mês). Fora do cartão, a data define o mês do lançamento; no cartão, é a data da compra e o lançamento fica na fatura do mês selecionado. Há opção de "Projeção" (previsto) e de "EU devo a outra pessoa". A divisão pode ser feita com várias pessoas, e todos os campos são editáveis, inclusive tipo de pagamento e número de parcelas ou meses.
- **Busca:** cada bloco da aba Lançamentos tem uma lupa que procura por descrição, categoria, pessoa, cartão, valor ou data.
- **Fatura em PDF:** o botão "Importar fatura (PDF)", no bloco do cartão, lê o PDF no próprio navegador (o arquivo não é enviado a nenhum servidor) e mostra a lista para escolher o que lançar. O formato varia por banco; se um banco não for reconhecido, é preciso ajustar a leitura.
- **Devo a:** se a pessoa a quem você deve tiver e-mail vinculado, ela recebe o aviso na aba Cobranças dela ("Te devem"), confirma quando receber e pode lançar o valor como receita (coleção `dividas`).
- **Recebi:** cada gasto de terceiros tem a caixa "Recebi". O que não for confirmado aparece nos meses seguintes em vermelho, como "Pendente de [mês]". O controle começa no mês em que esta versão foi aberta pela primeira vez; o que é de antes é considerado acertado.
- **Aba Pagamentos:** check das despesas fixas e das faturas dos cartões (valor total, incluindo terceiros), com vencimento, situação (em dia, vence em X dias ou atrasado) e a caixa "Pago". O dia de vencimento de cada cartão é informado na própria aba.
- **Encontro de contas:** a aba Cobranças mostra, por pessoa e no mês selecionado, quanto ela te deve e quanto você deve a ela. O detalhe aparece ao passar o mouse sobre o nome.
- **Categorias:** a administradora cria, edita e exclui categorias na aba Administração (coleção `config/categorias`), e elas valem para todos.

## Controle de acesso

- A administradora é **anapaularealengo@hotmail.com**, definida no `firestore.rules` (função `isAdmin`) e no `index.html` (`ADMIN_EMAIL`). Para trocar, altere os dois e publique as regras de novo.
- Na aba **Administração**, visível só para a administradora, ela libera ou remove os e-mails que podem usar o sistema.
- Quem não está na lista consegue criar o login, mas vê a tela "Acesso não liberado" e não acessa nenhum dado. O bloqueio é feito pelas regras do Firestore.
- Remover um acesso não apaga os dados da pessoa.

### Limites conhecidos

- Cada usuário tem os dados em **um único documento** no Firestore, que aceita até **1 MB** (cerca de 3 a 4 mil lançamentos). Para uso doméstico isso dá vários anos. Se um dia chegar perto do limite, será preciso separar os lançamentos por ano.
- Não há backup automático. Faça cópias de segurança manualmente, como explicado abaixo.

## Backup e restauração

- **Fazer backup:** clique em **Relatório / Backup → Baixar Excel → Todos os meses com lançamentos** e guarde o arquivo.
  Além das abas visíveis, a planilha leva uma aba oculta **"Backup"** com a cópia exata dos dados (séries, parcelas e contatos). Não apague nem edite essa aba.
- **Restaurar:** em **Relatório / Backup → Restaurar backup**, escolha a planilha. Aparece um resumo, e você escolhe:
  - **Substituir só os meses da planilha** (recomendado): os outros meses não são alterados.
  - **Substituir TODOS os dados da conta.**

  Por padrão, o sistema baixa antes uma cópia dos dados atuais, para ser possível desfazer.
- Cada pessoa restaura o backup **na própria conta**: é preciso estar logado nela.
- **Planilhas antigas** (geradas antes da aba "Backup") também podem ser importadas. Os dados são reconstruídos pelas abas Receitas, Despesas e Cobranças, e as séries recorrentes e parceladas são recriadas automaticamente. Itens com mesmo nome e mesmo valor no mesmo mês ficam separados e aparecem no aviso de revisão.
- **Alterações nas abas visíveis não são importadas** quando a planilha tem a aba "Backup", porque a restauração usa a cópia exata. Para corrigir um lançamento, edite pelo sistema depois de restaurar.
