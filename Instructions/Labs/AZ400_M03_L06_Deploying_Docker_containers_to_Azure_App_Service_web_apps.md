---
lab:
  title: Implantar contêineres do Docker em aplicativos Web do Serviço de Aplicativo do Azure
  module: 'Module 03: Implement CI with Azure Pipelines and GitHub Actions'
---

# Implantar contêineres do Docker em aplicativos Web do Serviço de Aplicativo do Azure

## Manual de laboratório do aluno

## Requisitos do laboratório

- Este laboratório requer o **Microsoft Edge** ou um [navegador com suporte do Azure DevOps.](https://learn.microsoft.com/azure/devops/server/compatibility)

- **Configurar uma organização do Azure DevOps:** se você ainda não tiver uma organização Azure DevOps que possa usar para este laboratório, crie uma seguindo as instruções disponíveis em [Criar uma organização ou coleção de projetos](https://learn.microsoft.com/azure/devops/organizations/accounts/create-organization).

- Identifique uma assinatura existente do Azure ou crie uma.

- Verifique se você tem uma conta Microsoft ou uma conta do Microsoft Entra com a função de Colaborador ou Proprietário na assinatura do Azure. Para obter detalhes, veja [Listar designações de função do Azure usando o portal do Azure](https://learn.microsoft.com/azure/role-based-access-control/role-assignments-list-portal) e [Exibir e designar funções de administrador no Azure Active Directory](https://learn.microsoft.com/azure/active-directory/roles/manage-roles-portal).

## Visão geral do laboratório

Neste laboratório, você aprenderá a usar um pipeline de CI/CD do Azure DevOps para criar uma imagem personalizada do Docker, efetuar push dela no Registro de Contêiner do Azure e implantá-la como um contêiner no Azure App Service.

## Objetivos

Após concluir este laboratório, você poderá:

- Criar uma imagem personalizada do Docker usando um agente do Linux hospedado pela Microsoft.
- Enviar uma imagem por push para o Registro de Contêiner do Azure.
- Implantar uma imagem do Docker como um contêiner no Azure App Service usando o Azure DevOps.

## Tempo estimado: 30 minutos

## Instruções

### Exercício 0: configurar os pré-requisitos do laboratório

Neste exercício, você configurará os pré-requisitos para o laboratório, que consistem em um novo projeto do Azure DevOps com um repositório baseado no [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb).

#### Tarefa 1: (pular se feita) criar e configurar o projeto de equipe

Nesta tarefa, você criará um projeto **eShopOnWeb** do Azure DevOps para ser usado por vários laboratórios.

1. No computador do laboratório, em uma janela do navegador, abra sua organização do Azure DevOps. Clique em **Novo projeto**. Dê ao seu projeto o nome **eShopOnWeb** e escolha **Scrum** na lista suspensa **Processo de item de trabalho**. Clique em **Criar**.

#### Tarefa 2: (pular se feita) importar repositório do Git eShopOnWeb

Nesta tarefa, você importará o repositório eShopOnWeb do Git que será usado por vários laboratórios.

1. No computador do laboratório, em uma janela do navegador, abra sua organização do Azure DevOps e o projeto **eShopOnWeb** criado anteriormente. Clique em **Repos>Arquivos** , **Importar**. Na janela **Importar um repositório do Git**, cole a URL https://github.com/MicrosoftLearning/eShopOnWeb.git e clique em **Importar**:

2. O repositório está organizado da seguinte forma:
    - A pasta **.ado** contém os pipelines YAML do Azure DevOps.
    - O contêiner da pasta **.devcontainer** está configurado para o desenvolvimento usando contêineres (localmente no VS Code ou no GitHub Codespaces).
    - A pasta **.azure** contém a infraestrutura Bicep&ARM como modelos de código usados em alguns cenários de laboratório.
    - A pasta **.github** contém definições de YAML do fluxo de trabalho do GitHub.
    - A pasta **src** contém o site do .NET 6 usado nos cenários do laboratório.

#### Tarefa 3: (pular se feita) definir o branch main como branch padrão

1. Vá para **Repos>Branches**.
2. Passe o mouse sobre o branch **main** e clique nas reticências à direita da coluna.
3. Clique em **Definir como branch padrão**.

### Exercício 1: gerenciar a conexão de serviço

Neste exercício, você configurará a conexão de serviço com sua Assinatura do Azure e, em seguida, importará e executará o pipeline de CI.

#### Tarefa 1: (pular se feita) gerenciar a conexão de serviço

Você pode criar uma conexão do Azure Pipelines com serviços externos e remotos para executar tarefas em um trabalho.

Nesta tarefa, você criará uma entidade de serviço usando a CLI do Azure, que permitirá ao Azure DevOps:

- Implantar recursos na sua assinatura do Azure.
- Efetuar push da imagem do Docker para o Registro de Contêiner do Azure.
- Adicione uma atribuição de função para permitir que o Serviço de Aplicativo do Azure extraia a imagem do Docker do Registro de Contêiner do Azure.

> **Observação**: se você já tiver uma entidade de serviço, poderá prosseguir diretamente para a próxima tarefa.

Você precisará de uma entidade de serviço para implantar recursos do Azure a partir do Azure Pipelines.

Uma entidade de serviço é criada automaticamente pelo Azure Pipeline quando você se conecta a uma assinatura do Azure de dentro de uma definição de pipeline ou quando cria uma nova conexão de serviço na página de configurações do projeto (opção automática). Você também pode criar manualmente a entidade de serviço a partir do portal ou usando a CLI do Azure e reutilizá-la em projetos.

1. No computador do laboratório, inicie um navegador da Web, navegue até o [**Portal do Azure**](https://portal.azure.com) e entre com a conta de usuário que tem a função Proprietário na assinatura do Azure que você usará neste laboratório e tem a função de Administrador global no locatário do Microsoft Entra associado a essa assinatura.
2. No portal do Azure, clique no ícone do **Cloud Shell**, localizado diretamente à direita da caixa de texto de pesquisa na parte superior da página.
3. Se for solicitado que você selecione **Bash** ou **PowerShell**, selecione **Bash**.

   >**Observação**: se esta for a primeira vez que você está iniciando o **Cloud Shell** e você receber a mensagem **Você não tem nenhum armazenamento montado**, selecione a assinatura que você está usando no laboratório e selecione **Criar armazenamento**.

4. No prompt **Bash**, no **painel Cloud Shell**, execute os seguintes comandos para recuperar os valores do atributo ID de assinatura do Azure:

    ```bash
    subscriptionName=$(az account show --query name --output tsv)
    subscriptionId=$(az account show --query id --output tsv)
    echo $subscriptionName
    echo $subscriptionId
    ```

    > **Observação**: copie ambos os valores para um arquivo de texto. Você precisará deles adiante neste laboratório.

5. No prompt **Bash**, no painel **Cloud Shell**, execute o seguinte comando para criar uma entidade de serviço:

    ```bash
    az ad sp create-for-rbac --name sp-az400-azdo --role contributor --scopes /subscriptions/$subscriptionId
    ```

    > **Observação**: o comando irá gerar uma saída JSON. Copie a saída para um arquivo de texto. Você precisará dela posteriormente neste laboratório.

6. Em seguida, no computador do laboratório, inicie um navegador da Web, navegue até o projeto ** eShopOnWeb** do Azure DevOps. Clique em **Configurações do Projeto>Conexões de Serviço (em Pipelines)** e **Nova Conexão de Serviço**.
7. Na tela **Nova conexão de serviço**, escolha **Azure Resource Manager** e **Avançar** (talvez você precise rolar para baixo).
8. Escolha a **Entidade de serviço (manual)** e clique em **Avançar**.
9. Preencha os campos vazios usando as informações coletadas durante as etapas anteriores:
    - ID e nome da assinatura.
    - ID da entidade de serviço (appId), chave da entidade de serviço (senha) e ID do locatário (locatário).
    - Em **Nome da conexão de serviço**, digite **azure-connection**. Esse nome será referenciado em pipelines YAML quando precisar de uma Conexão de Serviço do Azure DevOps para se comunicar com sua assinatura do Azure.

10. Clique em **Verificar e Salvar**.

### Exercício 2: Importar e executar o pipeline de CI

Nexte exercício, você importará e executará o pipeline de CI.

#### Tarefa 1: importar e executar o pipeline de CI

1. Acesse **Pipelines>Pipelines**
2. Clique no botão **Novo pipeline**
3. Selecione **Git do Azure Repos** (YAML)
4. Selecione o repositório **eShopOnWeb**
5. Selecione **Arquivo YAML do Azure Pipelines existente**
6. Selecione o arquivo **/.ado/eshoponweb-ci-docker.yml** e clique em **Continuar**
7. Na definição de pipeline YAML, personalize:
   - **YOUR-SUBSCRIPTION-ID** com o seu ID de assinatura do Azure.
   - **rg-az400-container-NAME** com o nome do grupo de recursos que será criado pelo pipeline (também pode ser um grupo de recursos existente).

8. Clique em **Salvar e Executar** e aguarde até que o pipeline seja executado com êxito.

    > **Observação**: a implantação pode levar alguns minutos para ser concluída.

    A definição de CS consiste nas seguintes tarefas:
    - **Recursos**: baixa os arquivos do repositório que serão usados nas tarefas a seguir.
    - **AzureResourceManagerTemplateDeployment**: implanta o Registro de Contêiner do Azure usando o modelo bicep.
    - **PowerShell**: recupere o valor do **Servidor de Logon do ACR** na saída da tarefa anterior e crie um novo parâmetro **acrLoginServer**
    - [**Docker**](https://learn.microsoft.com/azure/devops/pipelines/tasks/reference/docker-v0?view=azure-pipelines)**– Compilar**: compile a imagem do Docker e crie duas tags (BuildID mais recente e atual)
    - **Docker – Push**: efetuar push da imagem do Docker para o Registro de Contêiner do Azure

9. Seu pipeline terá um nome com base no nome do projeto. Vamos **renomear** o pipeline para melhor identificá-lo. Vá até **Pipelines>Pipelines** e clique no pipeline criado recentemente. Clique nas reticências e na opção **Renomear/mover**. Nomeie-o **eshoponweb-ci-docker** e clique em **Salvar**.

10. Navegue até o [**Portal do Azure**](https://portal.azure.com), procure o Registro de Contêiner do Azure no Grupo de Recursos criado recentemente (o nome é **rg-az400-container-NAME**). No lado esquerdo, clique em **Repositórios** em **Serviços** e certifique-se de que o repositório **eshoponweb/web** foi criado. Ao clicar no link do repositório, você verá duas tags (uma delas é **mais recente**), estas são as imagens enviadas. Se você não vir isso, verifique o status do seu pipeline.

### Exercício 3: importar e executar o pipeline de CD

Neste exercício, você configurará a conexão de serviço com sua Assinatura do Azure e, em seguida, importará e executará o pipeline de CD.

#### Tarefa 1: adicionar uma nova atribuição de função

Nesta tarefa, você adicionará uma nova atribuição de função para permitir que o Serviço de Aplicativo do Azure extraia a imagem do Docker a partir do Registro de Contêiner do Azure.

1. Navegue para o [**Portal do Azure**](https://portal.azure.com).
2. No portal do Azure, clique no ícone do **Cloud Shell**, localizado diretamente à direita da caixa de texto de pesquisa na parte superior da página.
3. Se for solicitado que você selecione **Bash** ou **PowerShell**, selecione **Bash**.
4. No prompt **Bash**, no **painel Cloud Shell**, execute os seguintes comandos para recuperar os valores do atributo ID de assinatura do Azure:

    ```sh
    spId=$(az ad sp list --display-name sp-az400-azdo --query "[].id" --output tsv)
    echo $spId
    roleName=$(az role definition list --name "User Access Administrator" --query "[0].name" --output tsv)
    echo $roleName
    ```

5. Depois de obter a ID da entidade de serviço e o nome da função, vamos criar a atribuição de função executando este comando (substitua **rg-az400-container-NAME** pelo nome do seu grupo de recursos)

    ```sh
    az role assignment create --assignee $spId --role $roleName --scope /subscriptions/$subscriptionId/resourceGroups/**g-az400-container-NAME**
    ```

Agora você verá a saída JSON, que confirma o sucesso da execução do comando.

#### Tarefa 2: importar e executar o pipeline de CD

Nesta tarefa, você importará e executará o pipeline de CD.

1. Acesse **Pipelines>Pipelines**
2. Clique no botão **Novo pipeline**
3. Selecione **Git do Azure Repos** (YAML)
4. Selecione o repositório **eShopOnWeb**
5. Selecione **Arquivo YAML do Azure Pipelines existente**
6. Selecione o arquivo **/.ado/eshoponweb-cd-webapp-docker.yml** e clique em **Continuar**
7. Na definição de pipeline YAML, personalize:
   - **YOUR-SUBSCRIPTION-ID** com o seu ID de assinatura do Azure.
   - **rg-az400-container-NAME** com o nome do grupo de recursos definido antes no laboratório.

8. Clique em **Salvar e Executar** e aguarde até que o pipeline seja executado com êxito.

    > **Observação**: a implantação pode levar alguns minutos para ser concluída.
    
    > **Importante**: se você receber a mensagem de erro "TF402455: Não é permitido envios por push para este branch; você deve usar uma solicitação de pull para atualizar este branch.", você precisa desmarcar a regra de proteção de branch "Exigir um número mínimo de revisores" habilitada nos laboratórios anteriores.

    A definição de CD consiste nas seguintes tarefas:
    - **Recursos**: baixa os arquivos do repositório que serão usados nas tarefas a seguir.
    - **AzureResourceManagerTemplateDeployment**: implanta o Serviço de Aplicativo do Azure usando o modelo bicep.
    - **AzureResourceManagerTemplateDeployment**: adicionar atribuição de função usando o Bicep

9. Seu pipeline terá um nome com base no nome do projeto. Vamos **renomear** o pipeline para melhor identificá-lo. Vá para **Pipelines>Pipelines** e passe o mouse sobre o pipeline criado recentemente. Clique nas reticências e na opção **Renomear/mover**. Nomeie-o **eshoponweb-cd-webapp-docker** e clique em **Salvar**.

    > **Observação 1**: o uso do modelo **/.azure/bicep/webapp-docker.bicep** cria um plano do serviço de aplicativo, um aplicativo Web com identidade gerenciada atribuída ao sistema habilitada e faz referência à imagem do Docker enviada anteriormente: **${acr.properties.loginServer}/eshoponweb/web:latest**.

    > **Observação 2**: o uso do modelo **/.azure/bicep/webapp-to-acr-roleassignment.bicep** cria uma nova atribuição de função para o aplicativo Web com a função AcrPull para poder recuperar a imagem do Docker. Isso poderia ser feito no primeiro modelo, mas como a atribuição de função pode levar algum tempo para se propagar, é uma boa ideia fazer as duas tarefas separadamente.

#### Tarefa 3: testar a solução

1. No Portal do Azure, navegue até o Grupo de Recursos criado recentemente, agora você verá três recursos (Serviço de Aplicativo, Plano do Serviço de Aplicativo e Registro de Contêiner).

1. Navegue até o Serviço de Aplicativo e clique em **Procurar**, isso o levará ao site.

Parabéns! Neste exercício, você implantou um site usando uma imagem personalizada do Docker.

### Exercício 4: remover os recursos do laboratório do Azure

Neste exercício, você removerá os recursos do Azure provisionados neste laboratório para eliminar cobranças inesperadas.

>**Observação**: lembre-se de remover todos os recursos do Azure que acabam de ser criados e que você não usa mais. Remover recursos não utilizados garante que você não veja encargos inesperados.

#### Tarefa 1: remover os recursos do laboratório do Azure

Nesta tarefa, você usará o Azure Cloud Shell para remover os recursos do Azure provisionados neste laboratório para eliminar cobranças desnecessárias.

1. No portal do Azure, abra a sessão **Bash** no painel **Cloud Shell**.
2. Liste todos os grupos de recursos criados nos laboratórios deste módulo executando o seguinte comando:

    ```sh
    az group list --query "[?starts_with(name,'rg-az400-container-')].name" --output tsv
    ```

3. Exclua todos os grupos de recursos criados em todos os laboratórios deste módulo executando o seguinte comando:

    ```sh
    az group list --query "[?starts_with(name,'rg-az400-container-')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**Observação**: o comando é executado de modo assíncrono (conforme determinado pelo parâmetro --nowait), portanto, embora você possa executar outro comando da CLI do Azure imediatamente depois na mesma sessão Bash, levará alguns minutos antes de o grupo de recursos ser removido.

## Revisão

Neste laboratório, você aprenderá a usar um pipeline de CI/CD do Azure DevOps para criar uma imagem personalizada do Docker, efetuar push dela no Registro de Contêiner do Azure e implantá-la como um contêiner no Serviço de Aplicativo do Azure.
