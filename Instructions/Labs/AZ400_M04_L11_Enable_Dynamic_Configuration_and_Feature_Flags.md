---
lab:
  title: Habilitar sinalizadores de recursos e a configuração dinâmica
  module: 'Module 04: Implement a secure continuous deployment using Azure Pipelines'
---

# Habilitar sinalizadores de recursos e a configuração dinâmica

## Requisitos do laboratório

- Este laboratório requer o **Microsoft Edge** ou um [navegador com suporte do Azure DevOps.](https://learn.microsoft.com/azure/devops/server/compatibility)

- **Configurar uma organização do Azure DevOps:** se você ainda não tiver uma organização Azure DevOps que possa usar para este laboratório, crie uma seguindo as instruções disponíveis em [Criar uma organização ou coleção de projetos](https://learn.microsoft.com/azure/devops/organizations/accounts/create-organization?view=azure-devops).

- Identifique uma assinatura existente do Azure ou crie uma.

- Verifique se você tem uma conta Microsoft ou uma conta do Microsoft Entra com a função de Colaborador ou Proprietário na assinatura do Azure. Para obter detalhes, veja [Listar designações de função do Azure usando o portal do Azure](https://learn.microsoft.com/azure/role-based-access-control/role-assignments-list-portal) e [Exibir e designar funções de administrador no Azure Active Directory](https://learn.microsoft.com/azure/active-directory/roles/manage-roles-portal).

## Visão geral do laboratório

A [Configuração de aplicativos do Azure](https://learn.microsoft.com/azure/azure-app-configuration/overview) fornece um serviço para gerenciar as configurações de aplicativo e os sinalizadores de recursos de maneira centralizada. Os programas modernos, especialmente aqueles em execução em uma nuvem, costumam ter muitos componentes distribuídos. A distribuição das definições de configuração entre esses componentes pode levar a erros difíceis de serem resolvidos durante uma implantação de aplicativo. Use a Configuração de aplicativos para armazenar todas as configurações do aplicativo e proteger os acessos em um só lugar.

## Objetivos

Após concluir este laboratório, você poderá:

- Habilitar a configuração dinâmica.
- Gerenciar sinalizadores de recursos.

## Tempo estimado: 45 minutos

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

### Exercício 1: (pular se feito) importar e executar pipelines de CI/CD

Neste exercício, você importará pipelines de CI/CD para criar e implantar o aplicativo eShopOnWeb. O pipeline de CI já está preparado para construir o aplicativo e executar testes. O pipeline de CD implantará o aplicativo em um aplicativo Web do Azure.

#### Tarefa 1: importar e executar o pipeline de CI

Vamos começar importando o pipeline de CI chamado [eshoponweb-ci.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-ci.yml).

1. Acesse **Pipelines > Pipelines**.
1. Clique no botão **Criar pipeline (se não houver pipelines)** ou no botão **Novo pipeline** (se já houver pipelines criados).
1. Selecione **Git do Azure Repos** (Yaml).
1. Selecione o repositório **eShopOnWeb**.
1. Selecione **Arquivo YAML do Azure Pipelines existente**.
1. Selecione o branch **principal** e o arquivo **/.ado/eshoponweb-ci.yml** e clique em **Continuar**.
1. Clique no botão **Executar** para executar o pipeline.
1. Seu pipeline terá um nome com base no nome do projeto. Vamos **renomear** o pipeline para melhor identificá-lo. Vá até **Pipelines > Pipelines** e clique no pipeline criado recentemente. Clique nas reticências e na opção **Renomear/Remover**. Nomeie-o **eshoponweb-ci** e clique em **Salvar**.

#### Tarefa 2: importar e executar o pipeline de CD

Vamos importar o pipeline de CD chamado [eshoponweb-cd-webapp-code.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-cd-webapp-code.yml).

1. Acesse **Pipelines > Pipelines**.
1. Clique no botão **Novo pipeline**.
1. Selecione **Git do Azure Repos (Yaml)**.
1. Selecione o repositório **eShopOnWeb**.
1. Selecione **Arquivo YAML do Azure Pipelines existente**.
1. Selecione o **branch principal** e o arquivo **/.ado/eshoponweb-cd-webapp-code.yml** e clique em **Continuar**.
1. Na definição de pipeline YAML, personalize:
   - **YOUR-SUBSCRIPTION-ID** com sua ID de assinatura do Azure.
   - **az400eshop-NAME** substitua NAMEpara torná-lo globalmente único.
   - **AZ400-EWebShop-NAME** pelo nome do grupo de recursos definido antes no laboratório.

1. Clique em **Salvar e Executar** e aguarde até que o pipeline seja executado com êxito.

    > **Observação**: Você deve clicar em **Salvar e Executar** duas vezes. Se a janela Trabalhos exibir uma mensagem de permissão necessária, selecione **Implantar** na janela Trabalhos, selecione **Exibir** e, em seguida, **Permitir** duas vezes para concluir a execução do pipeline.

    > **Observação**: a implantação pode levar alguns minutos para ser concluída.

    A definição de CD consiste nas seguintes tarefas:
    - **Recursos**: está preparado para acionar automaticamente com base na conclusão do pipeline de CI. Ele também baixa o repositório para o arquivo do Bicep.
    - **AzureResourceManagerTemplateDeployment**: implanta o Aplicativo Web do Azure usando o modelo do Bicep.

1. Seu pipeline assumirá um nome com base no nome do projeto. Vamos **renomear** o pipeline para melhor identificá-lo. Vá até **Pipelines > Pipelines** e clique no pipeline criado recentemente. Clique nas reticências e na opção **Renomear/Remover**. Nomeio como **eshoponweb-cd-webapp-code** e clique em **Salvar**.

### Exercício 2: gerenciar a Configuração de Aplicativos do Azure.

Neste exercício, você criará o recurso Configuração de Aplicativos no Azure, habilitará a identidade gerenciada e testará a solução completa.

> **Observação**: este exercício não requer nenhuma habilidade em codificação. O código do site já implementa as funcionalidades da Configuração de Aplicativos do Azure.

Se você quiser saber como implementar isso em seu aplicativo, dê uma olhada nestes tutoriais: [Usar configuração dinâmica em um aplicativo ASP.NET Core](https://learn.microsoft.com/azure/azure-app-configuration/enable-dynamic-configuration-aspnet-core) e [Gerenciar sinalizadores de recurso na Configuração de Aplicativos do Azure](https://learn.microsoft.com/azure/azure-app-configuration/manage-feature-flags).

#### Tarefa 1: criar o recurso de Configuração de Aplicativos

1. No Portal do Azure, procure o serviço **Configuração de Aplicativos**
1. Clique em **Criar configuração de aplicativo** e selecione:
    - Sua assinatura do Azure.
    - O Grupo de Recursos criado anteriormente (chamado **AZ400-EWebShop-NAME).**
    - O local.
    - Um nome exclusivo como **appcs-NAME-REGION** , por exemplo.
    - Selecione o tipo de preço **Gratuito**.
1. Clique em **Examinar + criar** e depois **Criar**.
1. Depois de criar o serviço Configuração de Aplicativos, vá para **Visão geral** e copie/salve o valor do **Ponto de extremidade**.

#### Tarefa 2: habilitar identidade gerenciada

1. Vá para o aplicativo Web implantado usando o pipeline (chamado **az400-webapp-NAME).**
1. Na seção **Configurações**, clique em **Identidade**, alterne o status para **Ativado** na seção **Sistema Atribuído**, clique em **salvar > sim** e aguarde alguns segundos para que a operação seja concluída.
1. Volte para o serviço Configuração de Aplicativos e clique em **Controle de acesso** e em **Adicionar atribuição de função**.
1. Na seção **Função**, selecione **Leitor de Dados da Configuração de Aplicativos**.
1. Na seção **Membros**, verifique **Gerenciar Identidade** e clique em **+ Selecionar membros**. No campo **Identidade Gerenciada**, selecione **Serviço de Aplicativo (1)**, selecione seu serviço de aplicativo e clique em **Selecionar**.
1. Clique em **Examinar e atribuir** duas vezes para concluir a atribuição de função.

#### Tarefa 3: configurar o aplicativo Web

Para se certificar de que seu site está acessando Configuração de Aplicativos, você precisa atualizar sua configuração.

1. Volte para seu aplicativo Web.
1. Na seção **Configurações**, clique em **Variáveis de Ambiente**.
1. Adicionar duas novas configurações de aplicativo:
    - Primeira configuração do aplicativo
        - **Name:** UseAppConfig
        - **Valor:** true
    - Segunda configuração do aplicativo
        - **Name:** AppConfigEndpoint
        - **Valor:***o valor que você salvou/copiou anteriormente do Ponto de Extremidade de Configuração de Aplicativos. Deve ser algo como <https://appcs-NAME-REGION.azconfig.io>*

1. Clique em **Aplicar** e **Confirmar** e, então, aguarde a atualização das configurações.
1. Vá até **Visão geral** e clique em **Procurar**
1. Nesta etapa, você não verá alterações no site, já que a Configuração de Aplicativos não contém dados. Isso é o que você fará nas próximas tarefas.

#### Tarefa 4: testar o gerenciamento de configuração

1. Em seu site, selecione **Visual Studio** na lista suspensa **Marca** e clique no botão de seta (**>**).
1. Você verá uma mensagem dizendo *"NÃO HÁ RESULTADOS QUE CORRESPONDAM À SUA PESQUISA".* O objetivo deste Laboratório é ser capaz de atualizar esse valor sem atualizar o código do site ou reimplantá-lo.
1. Para tentar isso, volte para Configuração de Aplicativos.
1. Na seção **Operações**, selecione **Gerenciador de Configurações**.
1. Clique em **Criar > Valor-chave** e adicione:
    - **Chave:** eShopWeb:Settings:NoResultsMessage
    - **Valor:***digite sua mensagem personalizada*
1. Clique em **Aplicar** e, em seguida, volte ao seu site e atualize a página.
1. Você verá sua nova mensagem em vez do valor padrão antigo.

#### Tarefa 5: testar o sinalizador de recursos

Vamos continuar testando o Gerenciador de recursos.

1. Para tentar isso, volte para Configuração de Aplicativos.
1. Na seção **Operações**, selecione **Gerenciador de recursos**.
1. Clique em **Criar** e adicione:
    - **Ativar sinalizador de recurso:** Marcado
    - **Nome do sinalizador de recursos:** SalesWeekend
1. Clique em **Aplicar** e, em seguida, volte ao seu site e atualize a página.
1. Você verá uma imagem com o texto "TODAS AS CAMISETAS À VENDA NESTE FIM DE SEMANA".
1. Você pode desativar esse recurso na Configuração de Aplicativos e, em seguida, verá que a imagem desaparece.

   > [!IMPORTANT]
   > Lembre-se de excluir os recursos criados no portal do Azure para evitar cobranças desnecessárias. Certifique-se de desabilitar o pipeline **eshoponweb-cd-webapp-code** ou ele recriará um grupo de recursos excluído e os recursos associados após a próxima execução do **eshoponweb-ci**.

## Revisão

Neste laboratório, você aprendeu a habilitar dinamicamente a configuração e gerenciar sinalizadores de recursos.
