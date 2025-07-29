---
lab:
  title: Implantar contêineres do Docker em aplicativos Web do Serviço de Aplicativo do Azure
  module: 'Module 02: Implement CI with Azure Pipelines and GitHub Actions'
---

# Implantar contêineres do Docker em aplicativos Web do Serviço de Aplicativo do Azure

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

## Tempo estimado: 20 minutos

## Instruções

### Exercício 0: (pular se já foi feito) Configurar os pré-requisitos do laboratório

Neste exercício, você configurará os pré-requisitos para o laboratório, que consistem em um novo projeto do Azure DevOps com um repositório baseado no [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb).

#### Tarefa 1: (pular se feita) criar e configurar o projeto de equipe

Nesta tarefa, você criará um projeto **eShopOnWeb** do Azure DevOps para ser usado por vários laboratórios.

1. No computador do laboratório, em uma janela do navegador, abra sua organização do Azure DevOps. Clique em **Novo projeto**. Dê ao seu projeto o nome **eShopOnWeb** e escolha **Scrum** na lista suspensa **Processo de item de trabalho**. Clique em **Criar**.

#### Tarefa 2: (pular se feita) importar repositório do Git eShopOnWeb

Nesta tarefa, você importará o repositório eShopOnWeb do Git que será usado por vários laboratórios.

1. No computador do laboratório, em uma janela do navegador, abra sua organização do Azure DevOps e o projeto **eShopOnWeb** criado anteriormente. Clique em **Repos > Arquivos**, **Importar**. Na janela **Importar um repositório do Git**, cole a URL <https://github.com/MicrosoftLearning/eShopOnWeb.git> e clique em **Importar**:

1. O repositório está organizado da seguinte forma:
    - A pasta **.ado** contém os pipelines YAML do Azure DevOps.
    - O contêiner da pasta **.devcontainer** está configurado para o desenvolvimento usando contêineres (localmente no VS Code ou no GitHub Codespaces).
    - A pasta **infra** contém a infraestrutura Bicep e ARM como modelos de código usados em alguns cenários de laboratório.
    - A pasta **.github** contém definições de fluxo de trabalho YAML do GitHub.
    - A pasta **src** contém o site do .NET 8 usado em cenários de laboratório.

#### Tarefa 3: (pular se feita) definir o branch main como branch padrão

1. Vá para **Repos > Branches**.
1. Passe o mouse sobre o branch **main** e clique nas reticências à direita da coluna.
1. Clique em **Definir como branch padrão**.

### Exercício 1: Importar e executar o pipeline de CI

Neste exercício, você configurará a conexão de serviço com sua Assinatura do Azure e, em seguida, importará e executará o pipeline de CI.

#### Tarefa 1:  Importar e executar o pipeline de CI

1. Acesse **Pipelines > Pipelines**
1. Clique no botão **Novo pipeline** (ou **Criar pipeline** se você não tiver outros pipelines criados anteriormente)
1. Selecione **Repositórios Git do Azure (YAML)**
1. Selecione o repositório **eShopOnWeb**
1. Selecione **Arquivo YAML do Azure Pipelines existente**
1. Selecione o branch **principal** e o arquivo **/.ado/eshoponweb-ci-docker.yml** e clique em **Continuar**
1. Na definição de pipeline YAML, personalize:
   - **YOUR-SUBSCRIPTION-ID** com o seu ID de assinatura do Azure.
   - Substitua o **resourceGroup** pelo nome do grupo de recursos usado durante a criação da conexão de serviço, por exemplo, **AZ400-RG1**.

1. Revise a definição do pipeline. A definição de CS consiste nas seguintes tarefas:
    - **Recursos**: baixa os arquivos do repositório que serão usados nas tarefas a seguir.
    - **AzureResourceManagerTemplateDeployment**: implanta o Registro de Contêiner do Azure usando o modelo bicep.
    - **PowerShell**: recupere o valor do **Servidor de Logon do ACR** na saída da tarefa anterior e crie um novo parâmetro **acrLoginServer**
    - [**Docker**](https://learn.microsoft.com/azure/devops/pipelines/tasks/reference/docker-v0?view=azure-pipelines)**– Compilar**: compile a imagem do Docker e crie duas tags (BuildID mais recente e atual)
    - **Docker – Push**: efetuar push da imagem do Docker para o Registro de Contêiner do Azure

1. Clique em **Salvar e Executar**.

1. Abra a execução do pipeline. Se você vir a mensagem "Este pipeline precisa de permissão para acessar um recurso antes que essa execução possa continuar a Criar", clique em **Exibir**, **Permitir** e **Permitir** novamente. Isso permitirá que o pipeline acesse a assinatura do Azure.

    > **Observação**: a implantação pode levar alguns minutos para ser concluída.

1. Seu pipeline terá um nome com base no nome do projeto. Vamos **renomear** o pipeline para melhor identificá-lo. Vá até **Pipelines > Pipelines** e clique no pipeline criado recentemente. Clique nas reticências e na opção **Renomear/mover**. Nomeie-o **eshoponweb-ci-docker** e clique em **Salvar**.

1. Navegue até o [**Portal do Azure**](https://portal.azure.com), procure o Registro de Contêiner do Azure no Grupo de Recursos criado recentemente (o nome é **AZ400-RG1**). No lado esquerdo, clique em **Repositórios** em **Serviços** e certifique-se de que o repositório **eshoponweb/web** foi criado. Ao clicar no link do repositório, você verá duas tags (uma delas é **mais recente**), estas são as imagens enviadas. Se você não vir isso, verifique o status do seu pipeline.

### Exercício 2: importar e executar o pipeline de CD

Neste exercício, você configurará a conexão de serviço com sua Assinatura do Azure e, em seguida, importará e executará o pipeline de CD.

#### Tarefa 1: Importar e executar o pipeline de CD

Nesta tarefa, você importará e executará o pipeline de CD.

1. Acesse **Pipelines > Pipelines**
1. Clique no botão **Novo pipeline**
1. Selecione **Git do Azure Repos** (YAML)
1. Selecione o repositório **eShopOnWeb**
1. Selecione **Arquivo YAML do Azure Pipelines Atual**
1. Selecione o branch **principal** e o arquivo **/.ado/eshoponweb-cd-webapp-docker.yml** e clique em **Continuar**
1. Na definição de pipeline YAML, personalize:
   - **YOUR-SUBSCRIPTION-ID** com o seu ID de assinatura do Azure.
   - Substitua o **resourceGroup** pelo nome do grupo de recursos usado durante a criação da conexão de serviço, por exemplo, **AZ400-RG1**.
   - Substitua **local** pela região do Azure onde os recursos serão implantados.

1. Revise a definição do pipeline. A definição de CD consiste nas seguintes tarefas:
    - **Recursos**: baixa os arquivos do repositório que serão usados nas tarefas a seguir.
    - **AzureResourceManagerTemplateDeployment**: implanta o Serviço de Aplicativo do Azure usando o modelo bicep.
    - **AzureResourceManagerTemplateDeployment**: adicionar atribuição de função usando o Bicep

1. Clique em **Salvar e Executar**.

1. Abra a execução do pipeline. Se você vir a mensagem "Este pipeline precisa de permissão para acessar um recurso antes que essa execução possa continuar a Implentar", clique em **Exibir**, **Permitir** e **Permitir** novamente. Isso permitirá que o pipeline acesse a assinatura do Azure.

    > **Importante**: Se você não autorizar o pipeline ao configurar, encontrará erros de permissão durante a execução. As mensagens de erro comuns incluem "Este pipeline precisa de permissão para acessar um recurso" ou "Falha na execução do pipeline devido a permissões insuficientes". Para resolver isso, navegue até a execução de pipeline, clique em **Exibir** ao lado da solicitação de permissão e clique em **Permitir** para conceder o acesso necessário à sua assinatura e recursos do Azure.

    > **Observação**: a implantação pode levar alguns minutos para ser concluída.

    > [!IMPORTANT]
    > Se você receber a mensagem de erro "TF402455: Não é permitido envios por push para este branch; você deve usar uma solicitação de pull para atualizar este branch.", você precisa desmarcar a regra de proteção de branch "Exigir um número mínimo de revisores" habilitada nos laboratórios anteriores.

1. Seu pipeline terá um nome com base no nome do projeto. Vamos **renomear** o pipeline para melhor identificá-lo. Vá para **Pipelines > Pipelines** e passe o mouse sobre o pipeline criado recentemente. Clique nas reticências e na opção **Renomear/mover**. Nomeie-o **eshoponweb-cd-webapp-docker** e clique em **Salvar**.

    > **Observação 1**: O uso do modelo **/infra/webapp-docker.bicep** cria um plano do serviço de aplicativo, um aplicativo Web com a identidade gerenciada atribuída pelo sistema habilitada e referencia a imagem do Docker enviada anteriormente: **${acr.properties.loginServer}/eshoponweb/web:latest**.

    > **Observação 2**: O uso do modelo **/infra/webapp-to-acr-roleassignment.bicep** cria uma atribuição de função para o aplicativo Web com a função AcrPull, a fim de conseguir recuperar a imagem do Docker. Isso poderia ser feito no primeiro modelo, mas como a atribuição de função pode levar algum tempo para se propagar, é uma boa ideia fazer as duas tarefas separadamente.

#### Tarefa 2: Testar a solução

1. No Portal do Azure, navegue até o Grupo de Recursos criado recentemente, agora você verá três recursos (Serviço de Aplicativo, Plano do Serviço de Aplicativo e Registro de Contêiner).

1. Navegue até o Serviço de Aplicativo e clique em **Procurar**, isso o levará ao site.

1. Verifique se o aplicativo eShopOnWeb está sendo executado com êxito. Uma vez confirmado, você concluiu o laboratório com êxito.

> [!IMPORTANT]
> Lembre-se de excluir os recursos criados no portal do Azure para evitar cobranças desnecessárias.

## Revisão

Neste laboratório, você aprendeu a usar um pipeline de CI/CD do Azure DevOps para criar uma imagem personalizada do Docker, efetuar push dela no Registro de Contêiner do Azure e implantá-la como um contêiner no Serviço de Aplicativo do Azure.
