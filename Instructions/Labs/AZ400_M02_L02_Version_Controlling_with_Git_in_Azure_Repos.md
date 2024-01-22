---
lab:
  title: Controle de versão com o Git no Azure Repos
  module: 'Module 02: Work with Azure Repos and GitHub'
---

# Controle de versão com o Git no Azure Repos

## Manual de laboratório do aluno

## Requisitos do laboratório

- Este laboratório requer o **Microsoft Edge** ou um [navegador com suporte do Azure DevOps.](https://docs.microsoft.com/azure/devops/server/compatibility)

- **Configurar uma organização do Azure DevOps:** se você ainda não tiver uma organização Azure DevOps que possa usar para este laboratório, crie uma seguindo as instruções disponíveis em [Criar uma organização ou coleção de projetos](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization).

- Se você não tiver o Git 2.29.2 ou posterior instalado, inicie um navegador da Web, navegue até a [página de download do Git para Windows](https://gitforwindows.org/), baixe-o e instale-o.
- Se você ainda não tiver o Visual Studio Code instalado, navegue até a [página de download do Visual Studio Code](https://code.visualstudio.com/) na janela do navegador da Web, baixe-o e instale-o.
- Se você ainda não tiver a extensão C# do Visual Studio instalada, navegue até a [página de instalação da extensão C#](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp) na janela do navegador da Web e instale-a.

## Visão geral do laboratório

O Azure DevOps dá suporte a dois tipos de controle de versão, Git e TFVC (Controle de Versão do Team Foundation). Observe a visão geral rápida dos dois sistemas de controle de versão:

- **TFVC (Controle de Versão do Team Foundation):** o TFVC é um sistema de controle de versão centralizado. Normalmente, os membros da equipe têm somente uma versão de cada arquivo nos computadores de desenvolvimento. Os dados históricos são mantidos somente no servidor. As branches são baseadas em caminho e criadas no servidor.

- **Git:** o Git é um sistema de controle de versão distribuído. Os repositórios Git podem ser posicionados localmente (no computador de um desenvolvedor). Cada desenvolvedor tem uma cópia do repositório de origem em seu computador de desenvolvimento. Os desenvolvedores podem confirmar cada conjunto de alterações em seu computador de desenvolvimento e executar operações de controle de versão, como histórico e comparação, sem uma conexão de rede.

O Git é o provedor de controle de versão padrão para novos projetos. Você deve usar o Git para controle de versão em seus projetos, a menos que precise de recursos centralizados de controle de versão no TFVC.

Neste laboratório, você aprenderá a estabelecer um repositório Git local, que pode ser sincronizado facilmente com um repositório Git centralizado no Azure DevOps. Além disso, você aprenderá sobre o suporte de branch e mesclagem do Git. Você usará o Visual Studio Code, mas os mesmos processos se aplicam ao uso de qualquer cliente compatível com Git.

## Objetivos

Após concluir este laboratório, você poderá:

- Clone um repositório.
- Salve o trabalho com commits.
- Examine o histórico de alterações.
- Trabalhe com branches usando o Visual Studio Code.

## Tempo estimado: 60 minutos

## Instruções

### Exercício 0: configurar os pré-requisitos do laboratório

Neste exercício, você configurará os pré-requisitos para o laboratório, que consistem em um novo projeto do Azure DevOps com um repositório baseado no [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb).

#### Tarefa 1: (pular se feita) criar e configurar o projeto de equipe

Nesta tarefa, você criará um projeto **eShopOnWeb** do Azure DevOps para ser usado por vários laboratórios.

1. No computador do laboratório, em uma janela do navegador, abra sua organização do Azure DevOps. Clique em **Novo projeto**. Dê ao seu projeto o nome **eShopOnWeb** e escolha **Scrum** na lista suspensa **Processo de item de trabalho**. Clique em **Criar**.

    ![Criar Projeto](images/create-project.png)

#### Tarefa 2: (pular se feita) importar repositório do Git eShopOnWeb

Nesta tarefa, você importará o repositório eShopOnWeb do Git que será usado por vários laboratórios.

1. No computador do laboratório, em uma janela do navegador, abra sua organização do Azure DevOps e o projeto **eShopOnWeb** criado anteriormente. Clique em **Repos>Arquivos** , **Importar**. Na janela **Importar um repositório do Git**, cole a URL https://github.com/MicrosoftLearning/eShopOnWeb.git e clique em **Importar**:

    ![Importar repositório](images/import-repo.png)

2. O repositório está organizado da seguinte forma:
    - A pasta **.ado** contém os pipelines YAML do Azure DevOps
    - O contêiner da pasta **.devcontainer** está configurado para o desenvolvimento usando contêineres (localmente no VS Code ou no GitHub Codespaces)
    - A pasta **.azure** contém a infraestrutura Bicep&ARM como modelos de código usados em alguns cenários de laboratório.
    - A pasta **.github** contém definições de YAML do fluxo de trabalho do GitHub.
    - A pasta **src** contém o site do .NET 6 usado nos cenários do laboratório.

#### Tarefa 3: configurar o Git e o Visual Studio Code

Nesta tarefa, você instalará e configurará o Git e o Visual Studio Code, incluindo a configuração do auxiliar de credenciais do Git para armazenar com segurança as credenciais do Git usadas para se comunicar com o Azure DevOps. Se você já tiver implementado esses pré-requisitos, poderá prosseguir diretamente para a próxima tarefa.

1. No computador de laboratório, abra o **Visual Studio Code**.
2. Na interface do Visual Studio Code, no menu principal, selecione**Terminal \| Novo terminal** para abrir o painel **TERMINAL**.
3. Verifique se o Terminal atual está executando o **PowerShell** verificando se a lista suspensa no canto superior direito do painel **TERMINAL** está mostrando **1: powershell**

    > **Observação**: para alterar o shell do Terminal atual para **PowerShell**, clique na lista suspensa no canto superior direito do painel **TERMINAL** e clique em **Selecionar Shell Padrão**. Na parte superior da janela Visual Studio Code, selecione seu shell de terminal preferido, o **Windows PowerShell**, e clique no sinal de adição no lado direito da lista suspensa para abrir um novo terminal com o shell padrão selecionado.

4. No painel **TERMINAL**, execute o seguinte comando abaixo para configurar o auxiliar de credenciais.

    ```git
    git config --global credential.helper wincred
    ```

5. No painel **TERMINAL**, execute os seguintes comandos para configurar um nome de usuário e email para confirmações do Git (substitua os espaços reservados entre chaves pelo seu nome de usuário e email de preferência, eliminando os símbolos < e >):

    ```git
    git config --global user.name "<John Doe>"
    git config --global user.email <johndoe@example.com>
    ```

### Exercício 1: clonar um repositório existente

Neste exercício, você usará o Visual Studio Code para clonar o repositório do Git provisionado como parte do exercício anterior.

#### Tarefa 1: clonar um repositório existente

Nesta tarefa, você passará pelo processo de clonagem de um repositório do Git usando o Visual Studio Code.

1. Alterne para o navegador da Web que exibe sua organização do Azure DevOps com o projeto **eShopOnWeb** gerado no exercício anterior.
2. No painel de navegação vertical do portal do Azure DevOps, selecione o ícone **Repos**.

3. No canto superior direito do painel do repositório **eShopOnWeb**, clique em **Clonar**.

    ![Clonar repositório do Git](images/clone-repo.png)

    > **Observação**: fazer uma cópia local de um repositório do Git é chamado de *clonagem*. Todas as ferramentas de desenvolvimento mainstream oferecem suporte a isso e conseguirão se conectar ao Azure Repos para extrair a fonte mais recente com a qual trabalhar.

4. No painel **Clonar repositório**, com a opção Linha de comando **HTTPS** selecionada, clique no botão **Copiar para a área de transferência** ao lado da URL do clone do repositório.

    > **Observação**: você pode usar essa URL com qualquer ferramenta compatível com o Git para obter uma cópia da base de código.

5. Feche o painel **Clonar Repositório**.
6. Alterne para o **Visual Studio Code** em execução no computador do laboratório.
7. Clique no cabeçalho do menu **Exibir** e, no menu suspenso, clique em **Paleta de Comandos**.

    > **Observação**: a Paleta de Comandos fornece uma maneira fácil e conveniente de acessar uma ampla variedade de tarefas, incluindo aquelas implementadas como extensões de terceiros. Você também pode usar o atalho de teclado **Ctrl+Shift+P** ou **F1** para abri-la.

8. No prompt da Paleta de Comandos, execute o comando **Git: Clone**.

    ![Paleta de comandos do VS Code](images/vscode-command.png)

    > **Observação**: para ver todos os comandos relevantes, você pode começar digitando **Git**.

9. Na caixa de texto **Fornecer URL do repositório ou escolher uma fonte de repositório**, cole a URL do clone de repositório copiada anteriormente nesta tarefa e pressione a tecla **Enter**.
10. Na caixa de diálogo **Selecionar pasta**, navegue até a unidade C:, crie uma nova pasta chamada **Git**, selecione-a e clique em **Selecionar local do repositório**.
11. Quando solicitado, entre em sua conta do Azure DevOps.
12. Depois que o processo de clonagem for concluído, uma vez solicitado, no Visual Studio Code, clique em **Abrir** para abrir o repositório clonado.

    > **Observação**: você pode ignorar avisos que talvez receba sobre problemas com o carregamento do projeto. A solução pode não estar no estado adequado para uma compilação, mas vamos nos concentrar em trabalhar com o Git, então a compilação do projeto não é necessária.

### Exercício 2: salvar trabalho com confirmações

Neste exercício, você percorrerá vários cenários que envolvem o uso do Visual Studio Code para preparar e confirmar alterações.

Quando você fizer alterações em seus arquivos, o Git registrará as alterações no repositório local. Você pode selecionar as alterações que deseja confirmar preparando-as. As confirmações são sempre feitas em seu repositório Git local, para que você não precise se preocupar se a confirmação está perfeita ou pronta para compartilhar com outras pessoas. Você pode fazer mais confirmações à medida que continua trabalhando e enviar as alterações para outras pessoas quando elas estiverem prontas para serem compartilhadas.

As confirmações do Git consistem no seguinte:

- No(s) arquivo(s) alterado(s) na confirmação. O Git mantém o conteúdo de todas as alterações de arquivo em seu repositório nas confirmações. Isso o mantém rápido e permite a fusão inteligente.
- Uma referência à confirmação pai. O Git gerencia seu histórico de código usando essas referências.
- Uma mensagem descrevendo uma confirmação. Você dá essa mensagem ao Git quando cria a confirmação. É uma boa ideia manter essa mensagem descritiva, mas direta.

#### Tarefa 1: confirmar alterações

Nesta tarefa, você usará o Visual Studio Code para confirmar alterações.

1. Na janela Visual Studio Code, na parte superior da barra de ferramentas vertical, selecione a guia **EXPLORER**, navegue até o arquivo **/eShopOnWeb/src/Web/Program.cs** e selecione-o. Isso exibirá automaticamente seu conteúdo no painel de detalhes.
2. Adicione o comentário na primeira linha:

    ```csharp
    // My first change
    ```

    > **Observação**: não importa qual seja o comentário, já que o objetivo é apenas fazer uma mudança.

3. Pressione **Ctrl+S** para salvar as alterações.
4. Na janela Visual Studio Code, selecione a guia **CONTROLE DO CÓDIGO-FONTE** para verificar se o Git reconheceu a alteração mais recente no arquivo que reside no clone local do repositório do Git.
5. Com a guia **CONTROLE DO CÓDIGO-FONTE** selecionada, na parte superior do painel, na caixa de texto, digite **Minha confirmação** como a mensagem de confirmação e pressione **Ctrl+Enter** para confirmá-la localmente.

    ![Primeira confirmação](images/first-commit.png)

6. Se for perguntado se pretende preparar automaticamente as alterações e confirmá-las diretamente, clique em **Sempre**.

    > **Observação**: discutiremos sobre o **preparo** mais tarde no laboratório.

7. No canto inferior esquerdo da janela Visual Studio Code, à direita da etiqueta **main**, observe o ícone Sincronizar alterações** de um círculo com duas setas verticais apontando para direções opostas e o **número **1** ao lado da seta apontando para cima. Clique no ícone e, se solicitado se deseja continuar, clique em **OK** para efetuar push e pull de confirmações de e para **origem/main**.

#### Tarefa 2: revisar confirmações

Nesta tarefa, você usará o portal do Azure DevOps para revisar confirmações.

1. Alterne para a janela do navegador da Web que está exibindo a interface do Azure DevOps.
2. No painel de navegação vertical do portal do Azure DevOps, na seção **Repos**, selecione **Confirmações**.
3. Verifique se a confirmação aparece na parte superior da lista.

    ![Confirmações de repositório do ADO](images/ado-commit.png)

#### Tarefa 3: preparar alterações

Nesta tarefa, você explorará o uso de preparo de alterações usando o Visual Studio Code. Preparar as alterações permite que você adicione seletivamente determinados arquivos a uma confirmação enquanto passa as alterações feitas em outros arquivos.

1. Volte para a janela do **Visual Studio Code**.
2. Atualize a classe **Program.cs** alterando o primeiro comentário com o seguinte e salvando o arquivo.

    ```csharp
        //My second change
    ```

3. Na janela do Visual Studio Code, volte para a guia **EXPLORER**, navegue até o arquivo **/eShopOnWeb/src/Web/Constants.cs** e selecione-o. Isso exibirá automaticamente seu conteúdo no painel de detalhes.
4. Adicione ao arquivo **Constants.cs** um comentário na primeira linha e salve o arquivo.

    ```csharp
    // My third change
    ```

5. Na janela do Visual Studio Code, alterne para a guia **CONTROLE DO CÓDIGO-FONTE**, passe o ponteiro do mouse sobre a entrada **Program.cs** e clique no sinal de adição no lado direito dessa entrada.

    > **Observação**: isso prepara a alteração apenas para o arquivo **Program.cs**, preparando-o para confirmação sem **Constants.cs**.

6. Com a guia **CONTROLE DO CÓDIGO-FONTE** selecionada, na parte superior do painel, na caixa de texto, digite **Comentários adicionados** como a mensagem de confirmação.

    ![Alterações preparadas](images/staged-changes.png)

7. Na parte superior da guia **CONTROLE DO CÓDIGO-FONTE**, clique no símbolo de reticências. No menu suspenso, selecione **Confirmar** e, no menu em cascata, selecione **Confirmar preparadas**.
8. No canto inferior esquerdo da janela do Visual Studio Code, clique no botão **Sincronizar Alterações** para sincronizar as alterações confirmadas com o servidor e, se solicitado se deseja continuar, clique em **OK** para efetuar push e pull para e de **origem/main**.

    > **Observação**: como apenas a alteração preparada foi confirmada, a outra alteração ainda está pendente para ser sincronizada.

### Exercício 3: revisar o histórico

Neste exercício, você usará o portal do Azure DevOps para revisar o histórico de confirmações.

O Git usa as informações de referência pai armazenadas em cada confirmação para gerenciar um histórico completo do seu desenvolvimento. Você pode facilmente revisar esse histórico de confirmação para descobrir quando as alterações de arquivo foram feitas e determinar as diferenças entre as versões do seu código usando o terminal ou de uma das muitas extensões do Visual Studio Code disponíveis. Você também pode revisar alterações usando o portal do Azure DevOps.

O uso do recurso **Branches e Mesclagens** pelo Git funciona por meio de solicitações pull, portanto, o histórico de confirmação do seu desenvolvimento não necessariamente forma uma linha reta e cronológica. Ao usar o histórico para comparar versões, pense em termos de alterações de arquivo entre duas confirmações em vez de alterações de arquivo entre dois pontos no tempo. Uma alteração recente em um arquivo no branch main pode ter vindo de uma confirmação criada há duas semanas em um branch de recurso que foi mesclado ontem.

#### Tarefa 1: comparar arquivos

Nesta tarefa, você verá o histórico de confirmações usando o portal do Azure DevOps.

1. Com a guia **CONTROLE DO CÓDIGO-FONTE** da janela do Visual Studio Code aberta, selecione **Constants.cs** representando a versão não preparada do arquivo.

    ![Comparação de arquivos](images/file-comparison.png)

    > **Observação**: uma exibição de comparação é aberta para permitir que você localize facilmente as alterações feitas. Neste caso, é apenas um comentário.

2. Alterne para a janela do navegador da Web que exibe o painel **Confirmações** do portal do **Azure DevOps** para revisar os branches e mesclagens da fonte. Eles fornecem uma maneira conveniente de visualizar quando e como as alterações foram feitas na fonte.
3. Role para baixo até a entrada **Minha confirmação** (enviada por push antes) e passe o ponteiro do mouse sobre ela para revelar o símbolo de reticências no lado direito.
4. Clique nas reticências, no menu suspenso, selecione **Procurar arquivos** e revise os resultados.

    ![Confirmar navegação](images/commit-browse.png)

    > **Observação**: esta exibição representa o estado da origem correspondente à confirmação, permitindo que você revise e baixe todos os arquivos de origem.

### Exercício 4: trabalhar com branches

Neste exercício, você percorrerá cenários que envolvem o gerenciamento de branches usando o Visual Studio Code e o portal do Azure DevOps.

Você pode gerenciar em seu repositório Git do Azure DevOps na exibição **Branches** do **Azure Repos** no portal do Azure DevOps. Você também pode personalizar o modo de exibição para acompanhar os branches mais importantes para que você possa ficar por dentro das alterações feitas por sua equipe.

A confirmação de alterações em um branch não afetará outros branches, e você poderá compartilhar branches com outras pessoas sem precisar mesclar as alterações no projeto principal. Você também pode criar novos branches para isolar as alterações de um recurso ou uma correção de bug do branch main e de outro trabalho. Como os branches são leves, alternar entre eles é rápido e fácil. O Git não cria várias cópias de seu código-fonte ao trabalhar com branches. Ele usa as informações de histórico armazenadas nas confirmações para recriar os arquivos em um branch quando você começa a trabalhar nele. Seu fluxo de trabalho do Git deve criar e usar os branches para gerenciar recursos e correções de bugs. O restante do fluxo de trabalho do Git, como o compartilhamento de código e a revisão de código com solicitações de pull, funciona completamente por meio de branches. Isolar o trabalho em branches simplifica a alteração daquilo em que você está trabalhando ao alterar o branch atual.

#### Tarefa 1: criar um novo branch no seu repositório local

Nesta tarefa, você criará um branch usando o Visual Studio Code.

1. Alterne para o **Visual Studio Code** em execução no computador do laboratório.
2. Com a guia **CONTROLE DO CÓDIGO-FONTE** selecionada, no canto inferior esquerdo da janela do Visual Studio Code, clique em **main**.
3. Na janela pop-up, selecione **+ Criar novo branch de...**.

    ![Criar branch](images/create-branch.png)

4. Na caixa de texto **Nome do branch**, digite **dev** para especificar o novo branch e pressione **Enter**.
5. Na caixa de texto **Selecionar uma ref para criar o branch "dev" de**, selecione **main** como o branch de referência.

    > **Observação**: neste ponto, você muda automaticamente para o branch **dev** .

#### Tarefa 2: excluir um branch

Nesta tarefa, você usará o Visual Studio Code para trabalhar com um branch criado na tarefa anterior.

O Git controla em qual branch você está trabalhando e garante que, quando você faz check-out de algum branch, seus arquivos corresponderão à confirmação mais recente no branch. Os branches permitem que você trabalhe com várias versões do código-fonte no mesmo repositório Git local ao mesmo tempo. Você pode usar o Visual Studio Code para publicar, fazer check-out e excluir branches.

1. Na janela do **Visual Studio Code**, com a guia **CONTROLE DO CÓDIGO-FONTE** selecionada, no canto inferior esquerdo da janela do Visual Studio Code, clique no ícone **Publicar alterações** (diretamente à direita da etiqueta **dev** que representao branch recém-criado).
2. Alterne para a janela do navegador da Web que exibe o painel **Confirmações** do portal do **Azure DevOps** e selecione **Branches**.
3. Na guia **Meus** do painel **Branches**, verifique se a lista de branches inclui **dev**.
4. Passe o ponteiro do mouse sobre a entrada do branch **dev** para revelar o símbolo de reticências no lado direito.
5. Clique nas reticências, no menu pop-up, selecione **Excluir branch** e, quando for solicitada a confirmação, clique em **Excluir**.

    ![Excluir branch](images/delete-branch.png)

6. Volte para a janela do **Visual Studio Code** e, com a guia **CONTROLE DO CÓDIGO-FONTE** selecionada, no canto inferior esquerdo da janela do Visual Studio Code, clique na entrada **dev**. Isso exibirá os branches existentes na parte superior da janela do Visual Studio Code.
7. Verifique se agora há dois branches **dev** listados.

    > **Observação**: o branch local (**dev**) é listado porque sua existência não é afetada pela exclusão do branch no repositório remoto. O servidor (**origin/dev**) está listado porque não foi removido.

8. Na lista de branches, selecione o branch **main** para fazer check-out.
9. Pressione **Ctrl+Shift+P** para abrir a **Paleta de comandos**.
10. No prompt da ** Paleta de comandos**, comece a digitar **Git: Delete** e selecione **Git: Delete Branch** quando ficar visível.
11. Selecione a entrada **dev** na lista de branches a serem excluídos.
12. No canto inferior esquerdo da janela do Visual Studio Code, clique na entrada **main** novamente. Isso exibirá os branches existentes na parte superior da janela do Visual Studio Code.
13. Verifique se o branch **dev** local não aparece mais na lista, mas a **origin/dev** remoto ainda está lá.
14. Pressione **Ctrl+Shift+P** para abrir a **Paleta de comandos**.
15. No prompt da **Paleta de comandos**, comece a digitar **Git: Fetch** e selecione **Git: Fetch (Prune)** quando ficar visível.

    > **Observação**: este comando atualizará os branches de origem no instantâneo local e excluirá os que não estiverem mais lá.

    > **Observação**: você pode conferir exatamente o que essas tarefas estão fazendo selecionando a janela **Saída** na parte inferior direita inferior da janela do Visual Studio Code. Se você não vir os logs do Git no console de saída, certifique-se de selecionar **Git** como a origem.

16. No canto inferior esquerdo da janela do Visual Studio Code, clique na entrada **main** novamente.
17. Verifique se o branch **origin/dev** não aparece mais na lista de branches.

#### Tarefa 3: restaurar um branch

Nesta tarefa, você usará o portal do Azure DevOps para restaurar o branch excluído na tarefa anterior.

1. Vá para o navegador da Web que exibe a guia **Meus** do painel **Branches** no portal do Azure DevOps.
2. Na guia **Meus** do painel **Branches**, selecione a guia **Todos**.
3. Na guia **Todos** do painel **Branches**, na caixa de texto **Pesquisar nome do branch**, digite **dev**.
4. Revise a seção **Branches excluídos** que contém a entrada que representa o branch recém-excluído.
5. Na seção **Branches excluídos**, passe o ponteiro do mouse sobre a entrada do branch **dev** para revelar o símbolo de reticências no lado direito.
6. Clique nas reticências, no menu pop-up e selecione **Restaurar branch**.

    ![restaurar branch](images/restore-branch.png)

    > **Observação**: você pode usar essa funcionalidade para restaurar um branch excluído, desde que saiba seu nome exato.

#### Tarefa 4: políticas de branch

Nesta tarefa, você usará o portal do Azure DevOps para adicionar políticas ao branch main e permitir apenas alterações usando Solicitações de Pull que estejam em conformidade com as políticas definidas. Você deseja garantir que as alterações em um branch sejam revisadas antes de serem mescladas.

Para simplificar, trabalharemos diretamente no editor de repositório do navegador da Web (trabalhando diretamente na origem), em vez de usar o clone local no VS Code (recomendado para cenários reais).

1. Alterne para o navegador da Web que exibe a guia **Meus** do painel **Branches** no portal do Azure DevOps.
2. Na guia **Meus** do painel **Branches**, passe o ponteiro do mouse sobre a entrada do branch **main** para revelar o símbolo de reticências no lado direito.
3. Clique nas reticências e, no menu pop-up e selecione **Políticas de branch**.

    ![Políticas de branch](images/branch-policies.png)

4. Na guia **main** das configurações do repositório, ative a opção **Exigir número mínimo de revisores**. Adicione **1** revisor e marque a caixa **Permitir que os solicitantes aprovem suas próprias alterações** (já que você é o único usuário no projeto para o laboratório)
5. Na guia **main** das configurações do repositório, ative a opção **Verificar itens de trabalho vinculados** e deixe-a com a opção **Obrigatório**.

    ![Configurações de política](images/policy-settings.png)

#### Tarefa 5: testar a política de branch

Nesta tarefa, você usará o portal do Azure DevOps para testar a política e criar sua primeira solicitação de pull.

1. No painel de navegação vertical do portal do Azure DevOps, em **Repos>Arquivos**, verifique se o branch **main** está selecionado (lista suspensa acima do conteúdo mostrado).
2. Para garantir que as políticas estejam funcionando, tente fazer uma alteração e confirmá-la no branch **main**, navegue até o arquivo **/eShopOnWeb/src/Web/Program.cs** e selecione-o. Isso exibirá automaticamente seu conteúdo no painel de detalhes.
3. Adicione o comentário na primeira linha:

    ```csharp
    // Testing main branch policy
    ```

4. Clique em **Confirmar > Confirmar**. Você verá um aviso: as alterações no branch main só podem ser feitas usando uma solicitação de pull.

    ![Confirmação de política negada](images/policy-denied.png)

5. Clique em **Cancelar** para ignorar a confirmação.

#### Tarefa 6: trabalhar com solicitações de pull

Nesta tarefa, você usará o portal do Azure DevOps para criar uma Solicitação de pull, usando o branch **dev** para mesclar uma alteração no branch **main** protegido. Um item de trabalho do Azure DevOps deve ser vinculado às alterações para poder rastrear o trabalho pendente com a atividade de código.

1. No painel de navegação vertical do portal do Azure DevOps, na seção **Boards**, selecione **Itens de trabalho**.
2. Clique em **+ Novo Item de Trabalho > Item da Lista de Pendências do Produto**. No campo de título, escreva **Testando minha primeira PR** e clique em **Salvar**.
3. Agora volte para o painel de navegação vertical do portal do Azure DevOps, no **Repos>Arquivos**, verifique se o branch **dev** está selecionado.
4. Navegue até o arquivo **/eShopOnWeb/src/Web/Program.cs** e faça a seguinte alteração na primeira linha:

    ```csharp
    // Testing my first PR
    ```

5. Clique em **Confirmar > Confirmar** (deixe a mensagem de confirmação padrão). Desta vez, a confirmação funciona, o branch **dev** não tem políticas.
6. Uma mensagem será exibida, propondo a criação de uma Solicitação de pull (já que o branch **dev** agora está à frente nas alterações, em comparação com o **main**). Clique em **Criar uma Solicitação de pull**.

    ![Criar uma Solicitação de Pull](images/create-pr.png)

7. Na guia **Nova solicitação de pull**, deixe os padrões e clique em **Criar**.
8. A solicitação de pull mostrará alguns requisitos com falha/pendentes, com base nas políticas aplicadas ao nosso branch **main** de destino.
    - As alterações propostas devem ter um item de trabalho vinculado
    - Pelo menos um usuário deve revisar e aprovar as alterações.

9. Nas opções do lado direito, clique no botão **+** ao lado de **Itens de trabalho**. Vincule o item de trabalho criado anteriormente à Solicitação de Pull clicando nele. Você verá um dos requisitos mudará de status.

    ![Vincular itens de trabalho](images/link-wit.png)

10. Em seguida, abra a guia **Arquivos** para revisar as alterações propostas. Em uma Solicitação de pull mais completa, você seria capaz de revisar arquivos um a um (marcados como revisados) e abrir comentários para linhas que podem não estar claras (passar o mouse sobre o número da linha dá a opção de postar um comentário).
11. Volte para a guia **Visão geral** e, no canto superior direito, clique em **Aprovar**. Todos os requisitos mudarão para verde. Agora você pode clicar em **Concluir**.
12. Na guia **Concluir Solicitação de pull**, várias opções serão dadas antes de concluir a mesclagem:
    - **Tipo de mesclagem**: quatro tipos de mesclagem são oferecidos, você pode revisá-los [aqui](https://learn.microsoft.com/azure/devops/repos/git/complete-pull-requests?view=azure-devops&tabs=browser#complete-a-pull-request) ou observar as animações dadas. Escolha **Mesclar (sem avançar)**.
    - **Opções pós-conclusão**:
        - Marque **Concluir item de trabalho associado...**. O PBI associado passará para o estado **Concluído**.

13. Clique em **Concluir mesclagem**

#### Tarefa 7: aplicar marcas

A equipe de produto decidiu que a versão atual do site deve ser lançada como v1.1.0-beta.

1. No painel de navegação vertical do portal do Azure DevOps, na seção **Repos**, selecione **Maarcas**.
2. No painel **Marcas** , clique em **Nova**.
3. No painel **Criar uma marca**, na caixa de texto **Nome**, digite **v1.1.0-beta**. Na lista suspensa **Baseado em**, deixe a entrada **main** selecionada. Na caixa de texto **Descrição**, digite **Versão beta v1.1.0** e clique em **Criar**.

    > **Observação**: agora você marcou o repositório nesta versão (a confirmação mais recente é vinculada à marca). Você pode marcar confirmações por vários motivos, e o Azure DevOps oferece a flexibilidade de editá-las e excluí-las, bem como gerenciar suas permissões.

### Exercício 5: Remover as políticas de branch

Ao percorrer os diferentes laboratórios de curso na ordem em que são apresentados, a política de branch configurada durante esse laboratório bloqueará os exercícios em laboratórios futuros. Portanto, queremos que você remova as políticas de branch configuradas.

1. Na exibição do Projeto **EShopOnWeb** do Azure DevOps, navegue até **Repos** e selecione **Branches**. Selecione a guia **Minerar** do painel **Branches**.
2. Na guia **Minerar** do painel **Branches**, passe o ponteiro do mouse sobre a entrada do branch **principal** para revelar o símbolo de reticências (o “...“) no lado direito.
3. Clique nas reticências e, no menu pop-up, selecione **Políticas de branch**.

    ![Configurações de política](images/policy-settings.png)

4. Na guia **principal** das configurações do repositório, desabilite a opção para **Exigir número mínimo de revisores**.
5. Na guia **principal** das configurações do repositório, desabilite a opção para **Verificar se há itens de trabalho vinculados**.

    ![Políticas de branch](images/branch-policies.png)

6. Agora você desabilitou/removeu as políticas de branch do branch principal.
    

## Revisão

Neste laboratório, você usou o portal do Azure DevOps para gerenciar branches e repositórios.
