---
lab:
  title: Configurar e executar testes funcionais
  module: 'Module 05: Implement a secure continuous deployment using Azure Pipelines'
---

# Configurar e executar testes funcionais

## Manual de laboratório do aluno

## Requisitos do laboratório

- Este laboratório requer o **Microsoft Edge** ou um [navegador compatível com o Azure DevOps.](https://docs.microsoft.com/azure/devops/server/compatibility?view=azure-devops)

- **Configurar uma organização do Azure DevOps:** se você ainda não tiver uma organização Azure DevOps que possa usar para este laboratório, crie uma seguindo as instruções disponíveis em [Criar uma organização ou coleção de projetos](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization).

- Identifique uma assinatura existente do Azure ou crie uma.

- Verifique se você tem uma conta Microsoft ou do Azure AD com a função Proprietário na assinatura do Azure e a função Administrador Global no locatário do Azure AD associado à assinatura do Azure. Para obter detalhes, veja [Listar designações de função do Azure usando o portal do Azure](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-list-portal) e [Exibir e designar funções de administrador no Azure Active Directory](https://docs.microsoft.com/azure/active-directory/roles/manage-roles-portal).

## Visão geral do laboratório

[Selenium](http://www.seleniumhq.org/): é uma estrutura de teste de código aberto portátil para aplicativos Web. Ele pode operar em quase todos os sistemas operacionais. Ele dá suporte a todos os navegadores modernos e várias linguagens, incluindo .NET (C#) e Java.

Este laboratório ensinará você a executar casos de teste do Selenium em um aplicativo Web do C# como parte do pipeline de lançamento do Azure DevOps.

## Objetivos

Após concluir este laboratório, você poderá:

- Configurar um agente auto-hospedado do Azure DevOps.
- Configurar o pipeline de lançamento.
- Disparar a compilação e o lançamento.
- Executar testes no Chrome e no Firefox.

## Tempo estimado: 60 minutos

## Instruções

### Exercício 0: configurar os pré-requisitos do laboratório

Neste exercício, você configurará os pré-requisitos para o laboratório, que incluem o projeto de equipe pré-configurado da Parts Unlimited com base em um modelo do Gerador de demonstração do Azure DevOps e recursos do Azure.

#### Tarefa 1: configurar o projeto de equipe

Nesta tarefa, você usará o Gerador de demonstração do Azure DevOps para gerar um novo projeto com base no modelo **Selenium**.

1. No computador do laboratório, inicie um navegador da Web e navegue até o [Gerador de demonstração do Azure DevOps](https://azuredevopsdemogenerator.azurewebsites.net). Este site de utilitário automatizará o processo de criação de um novo projeto do Azure DevOps na conta que é pré-preenchido com conteúdo (itens de trabalho, repositórios, etc.) necessário para o laboratório.

    > **Observação**: para obter mais informações no site, consulte [O que é o Gerador de demonstração dos serviços do Azure DevOps?](https://docs.microsoft.com/azure/devops/demo-gen).

2. Clique em **Entrar** e entre usando sua conta Microsoft associada a uma assinatura do Azure DevOps.
3. Se necessário, na página **Gerador de demonstração do Azure DevOps**, clique em **Aceitar** para aceitar as solicitações de permissão para acessar a assinatura do Azure DevOps.
4. Na página **Criar novo projeto**, na caixa de texto **Nome do novo projeto**, digite **Configurar e executar testes funcionais**, na lista suspensa **Selecionar organização**, selecione a organização do Azure DevOps e depois clique em **Escolher modelo**.
5. Na lista de modelos, na barra de ferramentas, clique em **Laboratórios DevOps**, selecione o modelo **Selenium** e clique em **Selecionar modelo**.
6. De volta à página **Criar novo projeto**, clique em **Criar projeto**

    > **Observação**: aguarde o processo ser concluído. Isso deverá levar cerca de dois minutos. Caso o processo falhe, navegue até a organização de DevOps, exclua o projeto e tente novamente.

7. Na página **Criar novo projeto**, clique em **Navegar até o projeto**.

#### Tarefa 2: criar recursos do Azure

Nesta tarefa, você provisionará uma VM do Azure executando o Windows Server 2016 junto com o SQL Express 2017, o Chrome e o Firefox.

1. Clique aqui no link **[Implantar no Azure](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2Falmvm%2Fmaster%2Flabs%2Fvstsextend%2Fselenium%2Farmtemplate%2Fazuredeploy.json)**. Isso redirecionará você automaticamente para a folha **Implantação personalizada** no portal do Azure.
2. Se solicitado, entre com a conta de usuário que tem a função Proprietário na assinatura do Azure que você usará neste laboratório e tem a função de Administrador global no locatário do Azure AD associado a essa assinatura.
3. Na folha **Implantação personalizada**, selecione **Editar modelo**.
4. Na folha **Editar modelo**, localize a linha `"https://raw.githubusercontent.com/microsoft/azuredevopslabs/master/labs/vstsextend/selenium/armtemplate/chrome_firefox_VSTSagent_IIS.ps1"`, substitua-a por `"https://raw.githubusercontent.com/MicrosoftLearning/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/master/Allfiles/Labs/11b/chrome_firefox_VSTSagent_IIS.ps1"` e clique em **Salvar**.
5. De volta à folha **Implantação personalizada**, especifique as seguintes configurações:

    | Configuração | Valor |
    | --- | --- |
    | Assinatura | o nome da assinatura do Azure que você usará neste laboratório |
    | Grupo de recursos | o nome de um novo grupo de recursos **az400m11l02-RG** |
    | Região | o nome da região do Azure na qual você deseja implantar os recursos do Azure neste laboratório |
    | Nome da Máquina Virtual | **az40011bvm** |

6. Clique em **Revisar + criar** e em **Criar**.

    > **Observação**: aguarde o processo ser concluído. Isso deverá levar cerca de 15 minutos.

### Exercício 1: implementar testes do Selenium usando um agente auto-hospedado do Azure DevOps

Neste exercício, você implementará testes do Selenium usando um agente auto-hospedado do Azure DevOps.

#### Tarefa 1: configurar um agente auto-hospedado do Azure DevOps

Nesta tarefa, você configurará um agente auto-hospedado usando a VM implantada no exercício anterior. O Selenium requer que o agente seja executado no modo interativo para executar os testes de interface do usuário.

1. Na janela do navegador da Web exibindo o portal do Azure, pesquise e selecione **Máquinas virtuais** e na folha **Máquinas virtuais**, selecione**az40011bvm**.
2. Na folha **az40011bvm**, selecione **Conectar**, no menu suspenso selecione **RDP**, na guia **RDP** da folha **az40011bvm \| Conectar**, selecione **Baixar arquivo RDP** e abra o arquivo baixado.
3. Quando solicitado , entre com as seguintes credenciais:

    | Configuração | Valor |
    | --- | --- |
    | Nome do Usuário | **vmadmin** |
    | Senha | **P2ssw0rd@123** |

4. Na sessão da Área de trabalho remota para **az40011bvm**, abra uma janela do navegador da Web Chrome, navegue até**<https://dev.azure.com>** e entre na organização do Azure DevOps.
5. No canto inferior esquerdo do portal do **Azure DevOps**, clique em **Configurações da organização**.
6. No menu vertical no lado esquerdo da página, na seção **Pipelines**, clique em **Pools de agentes**.
7. No painel **Pools de agentes**, clique em**Padrão**.
8. No painel **Padrão** , clique em **Novo agente**.
9. No painel **Obter o agente**, garanta que a guia **Windows** e a seção **x64** estão selecionadas e depois clique em **Download**.
10. Inicie o Explorador de arquivos, crie um diretório **C:\\AzAgent** e extraia o conteúdo do arquivo zip do agente baixado que reside na pasta **Downloads** para esse diretório.
11. Na sessão de Área de trabalho remota para **az40011bvm**, clique com o botão direito do mouse no menu ** Iniciar** e clique em **Prompt de comando (Admin)**.
12. Na janela **Administrador: prompt de comando**, execute o seguinte para iniciar a instalação dos binários do agente:

    ```cmd
    cd C:\AzAgent
    Config.cmd
    ```

13. Na janela **Administrator: prompt de comando**, quando for solicitado **Inserir URL do servidor**, digite **https://dev.azure.com/\<your-DevOps-organization-name\>**, em que **\<your-DevOps-organization-name\>** representa o nome da organização do Azure DevOps e pressione a tecla **Enter**.
14. Na janela **Administrador: prompt de comando**, quando solicitado **Inserir tipo de autenticação (pressione Enter para PAT)**, pressione a **tecla Enter**.
15. Na janela **Administrator: prompt de comando** quando for solicitado **Inserir token de acesso pessoal**, alterne para o portal do Azure DevOps, feche o painel **Obter o agente**, no canto superior direito da página do Azure DevOps. Clique no ícone **Configurações do usuário**, no menu suspenso, clique em **Tokens de acesso pessoal**, no painel **Tokens de acesso pessoal** e clique em **+ Novo token**.
16. No painel **Criar um novo token de acesso pessoal**, especifique as configurações a seguir e clique em **Criar** (deixe todos os outros com valores padrão):

    | Configuração | Valor |
    | --- | --- |
    | Nome | **Laboratório Configurar e executar testes funcionais** |
    | Escopos | **Definido como personalizado** |
    | Escopos | Clique **Mostrar todos os escopos** (na parte inferior da janela) |
    | Escopos | **Pools de agentes** -  **Ler e gerenciar** |

17. No painel **Sucess**, copie o valor do token de acesso pessoal para a área de transferência.

    > **Observação**: certifique-se de copiar o token. Você não poderá recuperá-lo depois de fechar este painel.

18. No painel **Success** , clique em **Fechar**.
19. Volte para a janela **Administrador: prompt de comando**, cole o conteúdo da área de transferência e pressione a **tecla Enter**.
20. Na janela **Administrador: prompt de comando**, quando solicitado **Inserir pool de agentes (pressione Enter para padrão)**, pressione a **tecla Enter**.
21. Na janela **Administrador: prompt de comando**, quando solicitado **Inserir nome do agente (pressione Enter para az40011bvm)**, pressione a **tecla Enter**.
22. Na janela **Administrador: prompt de comando**, quando solicitado **Inserir pasta de trabalho (pressione Enter para _work)**, pressione a **tecla Enter**.
23. Na janela **Administrador: prompt de comando**, quando solicitado **Inserir agente de execução como serviço (S/N) (pressione Enter para N)**, pressione a **tecla Enter**.
24. Na janela **Administrador: prompt de comando**, quando solicitado **Inserir configuração de login automático e agente de execução na inicialização (S/N) (pressione Enter para N)**, pressione a **tecla Enter**.
25. Depois que o agente estiver registrado, na janela **Administrador: prompt de comando**, digite **run.cmd** e pressione **Enter** para iniciar o agente.

    > **Observação**: você também precisa instalar o Dac Framework que é usado pelo aplicativo que você implantará posteriormente no laboratório.

26. Na sessão da área de trabalho remota para **az40011bvm**, inicie outra instância do navegador da Web, navegue até a [página de download do Microsoft SQL Server Data-Tier Application Framework (18.2)](https://www.microsoft.com/download/details.aspx?id=58207&WT.mc_id=rss_alldownloads_extensions) e clique em **Download**.
27. Em **Escolher o download desejado**, marque a caixa de seleção **EN\x64\DacFramework.msi** e clique em **Avançar**. Isso acionará o download automático do arquivo **DacFramework.msi**.
28. Depois que o download do arquivo **DacFramework.msi** for concluído, use-o para executar a instalação do Microsoft SQL Server Data-Tier Application Framework com as configurações padrão.

#### Tarefa 2: configurar um pipeline de lançamento

Nesta tarefa, você configurará um pipeline de lançamento.

> **Observação**: a VM do Azure tem o agente configurado para implantar os aplicativos e executar casos de teste do Selenium. A definição de lançamento usa **[Fases](https://docs.microsoft.com/vsts/build-release/concepts/process/phases)** para implantar em servidores de destino.

1. Na sessão de Área de trabalho remota para **az40011bvm**, na janela do navegador que exibe o portal do **Azure DevOps**, clique no símbolo do **Azure DevOps** no canto superior esquerdo.
2. No painel que exibe os projetos da organização, clique no bloco que representa o projeto **Configurar e executar testes funcionais**.
3. No painel **Configurar e executar testes funcionais**, no painel de navegação vertical, selecione **Pipelines**, na seção **Pipelines**, clique em **Lançamentos** e, em seguida, no painel **Selenium**, clique em **Editar**.
4. No painel **Todos os pipelines > Selenium**, clique no cabeçalho da guia **Tarefas** e, no menu suspenso, clique em **Dev**.
5. Na lista de tarefas do estágio **Dev**, revise as fases de implantação **Implantação de IIS**, **Implantação de SQL** e **execução de teste Selenium**.

   - **Fase implantação do IIS**: nesta fase, nós implantamos a aplicativo na VM usando as tarefas a seguir:

     - **Gerenciar aplicativo Web IIS**: esta tarefa é executada na máquina de destino onde registramos o agente. Ela cria um *site* e um *Pool de aplicativos* localmente com o nome**PartsUnlimited** em execução sob a porta **82** , [**http://localhost:82**](http://localhost:82)
     - **Implantação do aplicativo Web IIS**: esta tarefa implanta o aplicativo ao servidor IIS usando a **Implantação da Web**.

   - **Fase de implantação do banco de dados**: nesta fase, nós usamos a tarefa [**Implantação do banco de dados SQL Server**](https://github.com/Microsoft/vsts-tasks/blob/master/Tasks/SqlDacpacDeploymentOnMachineGroup/README.md) para implantar o arquivo [**dacpac**](https://docs.microsoft.com/sql/relational-databases/data-tier-applications/data-tier-applications) ao servidor de BD.

   - **Execução de testes Selenium**: execução de **testes de IU** como parte do processo de lançamento nos permite detectar mudanças inesperadas. A configuração de testes automatizados baseados em navegador impulsiona a qualidade no aplicativo, sem precisar fazer isso manualmente. Nesta fase, executaremos testes Selenium no aplicativo Web implantado. As tarefas seguintes descrevem o uso do Selenium para testar os sites no pipeline de lançamento.

     - **Instalador da plataforma de testes do Visual Studio**: a tarefa [Instalador da plataforma de testes do Visual Studio](https://docs.microsoft.com/azure/devops/pipelines/tasks/tool/vstest-platform-tool-installer?view=vsts) vai adquirir a plataforma de testes da Microsoft do nuget.org ou de um feed especificado e a adicionará ao cache de ferramentas. Ela cumpre com os requisitos de **vstest** de forma que as tarefas de teste seguintes do Visual Studio em uma compilação ou pipeline de lançamento possam ser executadas sem precisar de uma instalação completa do Visual Studio na máquina do agente.
     - **Executar testes de UI do Selenium**: esta [tarefa](https://github.com/Microsoft/azure-pipelines-tasks/blob/master/Tasks/VsTestV2/README.md) usa **vstest.console.exe** para executar os casos de teste do Selenium nas máquinas do agente.

6. No painel **Todos os pipelines > Selenium**, clique na fase **Implantação do IIS** e, no painel de **Trabalho do agente**, verifique se o pool de agentes **Padrão** está selecionado.
7. Repita a etapa anterior para as fases **Implantação de SQL** e **execução de testes Selenium**. Se necessário, clique em **Salvar** para salvar as mudanças.

#### Tarefa 3: acionar compilação e lançamento

Nesta tarefa, acionaremos a **Compilação** para compilar scripts Selenium C# junto com o aplicativo Web. Os binários resultantes são copiados para o agente auto-hospedado e os scripts Selenium são executados como parte do **lançamento** automatizado.

1. Na sessão da Área de trabalho remota para **az40011bvm**, na janela do navegador que exibe o portal do **Azure DevOps**, no painel de navegação vertical, na seção **Pipelines**, clique em **Pipelines** e depois no painel **Pipelines**, clique em **Selenium**.
2. No painel **Selenium**, clique em **Executar pipeline** e em **Executar pipeline**, clique em **Executar**.

    > **Observação**: essa compilação publicará os artefatos de teste no Azure DevOps, que será usado no lançamento.

    > **Observação**: depois que a compilação for bem-sucedida, o lançamento será acionado.

3. No painel de execuções do pipeline, na seção **Trabalhos**, clique em **Fase 1** e monitore o progresso da compilação até sua conclusão.
4. Na janela do navegador que exibe o portal do **Azure DevOps**, no painel de navegação vertical, na seção **Pipelines**, clique em **Lançamentos**, clique na entrada que representa o lançamento e no painel **Selenium > Release-1**, clique em **Dev**.
5. No painel **Selenium > Release-1 > Dev**, monitore a implantação correspondente.
6. Quando a fase de **execução do teste Selenium** for iniciada, monitore os testes do navegador da Web.
7. Quando a versão for concluída, no painel **Selenium > Release-1 > Dev**, clique na guia **Testes** para analisar os resultados do teste. Selecione os filtros necessários na lista suspensa na seção **Resultados** para exibir os testes e os status deles.

### Exercício 2: remover os recursos do Azure Lab

Neste exercício, você removerá os recursos do Azure provisionados neste laboratório para eliminar cobranças inesperadas.

>**Observação**: lembre-se de remover todos os recursos do Azure que acabam de ser criados e que você não usa mais. Remover recursos não utilizados garante que você não veja encargos inesperados.

#### Tarefa 1: Remover os recursos do laboratório do Azure

Nesta tarefa, você usará o Azure Cloud Shell para remover os recursos do Azure provisionados neste laboratório para eliminar cobranças desnecessárias.

1. No portal do Azure, abra a sessão de shell **Bash** no painel **Cloud Shell**.
2. Liste todos os grupos de recursos criados em todos os laboratórios deste módulo executando o seguinte comando:

    ```sh
    az group list --query "[?starts_with(name,'az400m11l02-RG')].name" --output tsv
    ```

3. Exclua todos os grupos de recursos criados em todos os laboratórios deste módulo executando o seguinte comando:

    ```sh
    az group list --query "[?starts_with(name,'az400m11l02-RG')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**Observação**: o comando é executado de modo assíncrono (conforme determinado pelo parâmetro --nowait), portanto, embora você possa executar outro comando da CLI do Azure imediatamente depois na mesma sessão Bash, levará alguns minutos antes de o grupo de recursos ser removido.

## Revisão

Neste laboratório, você aprendeu a executar casos de teste do Selenium em um aplicativo Web do C# como parte do pipeline de lançamento do Azure DevOps.
