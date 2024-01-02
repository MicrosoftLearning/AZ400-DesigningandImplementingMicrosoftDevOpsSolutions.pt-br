---
lab:
  title: Habilitar sinalizadores de recursos e a configuração dinâmica
  module: 'Module 05: Implement a secure continuous deployment using Azure Pipelines'
---

# Habilitar sinalizadores de recursos e a configuração dinâmica

## Manual de laboratório do aluno

## Requisitos do laboratório

- Este laboratório requer o **Microsoft Edge** ou um [navegador compatível com o Azure DevOps.](https://learn.microsoft.com/azure/devops/server/compatibility)

- **Configurar uma organização do Azure DevOps:** se você ainda não tiver uma organização Azure DevOps que possa usar para este laboratório, crie uma seguindo as instruções disponíveis em [Criar uma organização ou coleção de projetos](https://learn.microsoft.com/azure/devops/organizations/accounts/create-organization?view=azure-devops).

- Identifique uma assinatura existente do Azure ou crie uma.

- Verifique se você tem uma conta Microsoft ou uma conta do Microsoft Entra com a função de Colaborador ou Proprietário na assinatura do Azure. Para obter detalhes, veja [Listar designações de função do Azure usando o portal do Azure](https://learn.microsoft.com/azure/role-based-access-control/role-assignments-list-portal) e [Exibir e designar funções de administrador no Azure Active Directory](https://learn.microsoft.com/azure/active-directory/roles/manage-roles-portal).

## Visão geral do laboratório

A [Configuração de aplicativos do Azure](https://learn.microsoft.com/azure/azure-app-configuration/overview) fornece um serviço para gerenciar as configurações de aplicativo e os sinalizadores de recursos de maneira centralizada. Os programas modernos, especialmente aqueles em execução em uma nuvem, costumam ter muitos componentes distribuídos. A distribuição das definições de configuração entre esses componentes pode levar a erros difíceis de serem resolvidos durante uma implantação de aplicativo. Use a Configuração de aplicativos para armazenar todas as configurações do aplicativo e proteger os acessos em um só lugar.

## Objetivos

Após concluir este laboratório, você poderá:

- Habilitar a configuração dinâmica.
- Gerenciar sinalizadores de recursos.

## Tempo estimado: 60 minutos

## Instruções

### Exercício 0: configurar os pré-requisitos do laboratório

Neste exercício, você configurará os pré-requisitos para o laboratório, que consistem em um novo projeto do Azure DevOps com um repositório baseado no [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb).

#### Tarefa 1: (ignorar se concluído) criar e configurar o projeto de equipe

Nesta tarefa, você criará um projeto do Azure DevOps no **eShopOnWeb** para ser usado por vários laboratórios.

1. No computador do laboratório, em uma janela do navegador, abra sua organização do Azure DevOps. Clique em **Novo projeto**. Dê ao seu projeto o nome **eShopOnWeb** e escolha **Scrum** na lista suspensa **Processo de item de trabalho**. Clique em **Criar**.

#### Tarefa 2: (ignorar se concluído) importar repositório Git do eShopOnWeb

Nesta tarefa, você importará o repositório Git do eShopOnWeb que será usado por vários laboratórios.

1. No computador do laboratório, em uma janela do navegador, abra sua organização do Azure DevOps e o projeto criado anteriormente do **eShopOnWeb**. Clique em **Repos>Files**, **Importar**. Na janela, **Importar um repositório Git**, cole a seguinte URL https://github.com/MicrosoftLearning/eShopOnWeb.git e clique em **Importar**:

2. O repositório está organizado da seguinte forma:
    - A pasta **.ado** contém os pipelines YAML do Azure DevOps.
    - O contêiner da pasta **.devcontainer** está configurado para o desenvolvimento usando contêineres (localmente no VS Code ou no GitHub Codespaces).
    - A pasta **.azure** contém a infraestrutura Bicep&ARM como modelos de código usados em alguns cenários do laboratório.
    - A pasta **.github** contém definições de fluxo de trabalho YAML do GitHub.
    - A pasta **src** contém o site .NET 6 usado nos cenários do laboratório.

#### Tarefa 3: (ignorar se concluído) definir branch principal como branch padrão

1. Vá para **Repos > Branches**.
2. Passe o mouse sobre o branch **principal** e clique nas reticências à direita da coluna.
3. Clique em **Definir como branch padrão**.

### Exercício 1: (ignorar se concluído) importar e executar pipelines de CI/CD

Neste exercício, você importará e executará o pipeline de CI, configurará a conexão de serviço com sua assinatura do Azure e, em seguida, importará e executará o pipeline de CD.

#### Tarefa 1: importar e executar o pipeline de CI

Vamos começar importando o pipeline de CI chamado [eshoponWeb-ci.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-ci.yml).

1. Vá para **Pipelines>Pipelines**.
2. Clique no botão **Criar pipeline** (se não houver pipelines) ou botão **Novo pipeline** (se já houver pipelines criados).
3. Selecione **Git do Azure Repos (YAML)**.
4. Selecione o repositório **eShopOnWeb**.
5. Selecione **Arquivo YAML do Azure Pipelines existente**.
6. Selecione o arquivo **/.ado/eshoponWeb-ci.yml** e clique em **Continuar**.
7. Clique no botão **Executar** para executar o pipeline.
8. O pipeline terá um nome com base no nome do projeto. Vamos **renomeá-lo** para identificar melhor o pipeline. Vá para **Pipelines>Pipelines** e clique no pipeline criado recentemente. Clique nas reticências e na opção **Renomear/Remover**. Nomeie-o como **eshoponWeb-ci** e clique em **Salvar**.

#### Tarefa 2: gerenciar a conexão de serviço

Você pode criar uma conexão do Azure Pipelines com serviços externos e remotos para executar tarefas em um trabalho.

Nesta tarefa, você criará uma entidade de serviço usando a CLI do Azure, que permitirá ao Azure DevOps:

- Implantar recursos na assinatura do Azure
- Implantar o aplicativo eShopOnWeb

> **Observação**: se você já tiver uma entidade de serviço, poderá prosseguir diretamente para a próxima tarefa.

Você precisará de uma entidade de serviço para implantar recursos do Azure a partir do Azure Pipelines.

Uma entidade de serviço é criada automaticamente pelo Azure Pipelines, quando você se conecta a uma assinatura do Azure de dentro de uma definição de pipeline ou quando cria uma nova Conexão de serviço na página de configurações do projeto (opção automática). Você também pode criar manualmente a entidade de serviço a partir do portal ou usando a CLI do Azure e reutilizá-la em projetos.

1. No computador do laboratório, inicie um navegador da Web, navegue até o [**Portal do Azure**](https://portal.azure.com) e entre com a conta de usuário que tem a função Proprietário na assinatura do Azure que você usará neste laboratório e tem a função de Administrador global no locatário do Microsoft Entra associado a essa assinatura.
2. No portal do Azure, clique no ícone do **Cloud Shell**, localizado à direita da caixa de texto de pesquisa na parte superior da página.
3. Se for solicitado que você selecione **Bash** ou **PowerShell**, selecione **Bash**.

   >**Observação**: se esta for a primeira vez que você está iniciando o **Cloud Shell** e você receber a mensagem **Você não tem nenhum armazenamento montado**, selecione a assinatura que você está usando no laboratório e selecione **Criar armazenamento**.

4. No prompt **Bash**, no painel **Cloud Shell**, execute os seguintes comandos para recuperar os valores do atributo de ID de assinatura do Azure:

    ```sh
    subscriptionName=$(az account show --query name --output tsv)
    subscriptionId=$(az account show --query id --output tsv)
    echo $subscriptionName
    echo $subscriptionId
    ```

    > **Observação**: copie ambos os valores para um arquivo de texto. Você precisará deles mais adiante neste laboratório.

5. No prompt **Bash**, no painel **Cloud Shell**, execute os seguintes comandos para criar uma entidade de serviço:

    ```sh
    az ad sp create-for-rbac --name sp-az400-azdo --role contributor --scopes /subscriptions/$subscriptionId
    ```

    > **Observação**: o comando gerará uma saída JSON. Copie a saída em um arquivo de texto. Você precisará dela mais adiante neste laboratório.

6. Em seguida, no computador do laboratório, inicie um navegador da Web, navegue até o projeto **eShopOnWeb** do Azure DevOps. Clique em **Configurações do Projeto > Conexões de Serviço (em Pipelines)** e **Nova conexão de serviço**.

7. Na tela **Nova conexão de serviço**, escolha **Azure Resource Manager** e **Avançar** (pode ser necessário rolar para baixo).

8. Escolha **Entidade de serviço (manual)** e clique em **Avançar**.

9. Preencha os campos vazios usando as informações coletadas durante as etapas anteriores:
    - ID e nome da assinatura
    - ID da entidade de serviço (ou clientId), chave (ou senha) e TenantId.
    - Em **Nome da conexão de serviço**, digite **azure subs**. Esse nome será referenciado em pipelines YAML quando precisar de uma conexão de serviço do Azure DevOps para se comunicar com a assinatura do Azure.

10. Clique em **Verificar e Salvar**.

#### Tarefa 3: importar e executar o pipeline de CD

Vamos importar o pipeline de CD chamado [eshoponWeb-cd-Webapp-code.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-cd-webapp-code.yml).

1. Vá para **Pipelines>Pipelines**.
2. Clique no botão **Novo pipeline**.
3. Selecione **Git do Azure Repos (YAML)**.
4. Selecione o repositório **eShopOnWeb**.
5. Selecione **Arquivo YAML do Azure Pipelines existente**.
6. Selecione o arquivo **/.ado/eshoponWeb-cd-Webapp-code.yml** e depois clique em **Continuar**.
7. Na definição de pipeline do YAML, personalize:
   - **YOUR-SUBSCRIPTION-ID** pelo ID da assinatura do Azure.
   - **az400eshop-NAME** substitua NAME para torná-lo globalmente exclusivo.
   - **AZ400-EWebShop-NAME** pelo nome do grupo de recursos definido antes no laboratório.

8. Clique em **Salvar e Executar** e aguarde até que o pipeline seja executado com êxito.

    > **Observação**: a implantação pode levar alguns minutos para ser concluída.

    A definição de CD é composta pelas seguintes tarefas:
    - **Recursos**: preparado para acionar automaticamente com base na conclusão do pipeline de CI. Ele também baixa o repositório para o arquivo bicep.
    - **AzureResourceManagerTemplateDeployment**: implanta o aplicativo Web do Azure usando o modelo bicep.

9. O pipeline terá um nome com base no nome do projeto. Vamos **renomeá-lo** para identificar melhor o pipeline. Vá para **Pipelines>Pipelines** e clique no pipeline criado recentemente. Clique nas reticências e na opção **Renomear/Remover**. Nomeie-o como **eshoponWeb-cd-Webapp-code** e clique em **Salvar**.

### Exercício 2: gerenciar a configuração de aplicativos do Azure

Neste exercício, você criará o recurso configuração de aplicativos no Azure, habilitará a identidade gerenciada e testará a solução completa.

> **Observação**: este exercício não requer nenhuma habilidade de codificação. O código do site já implementa as funcionalidades de configuração de aplicativos do Azure.

Se você quiser saber como implementar isso no aplicativo, dê uma olhada nestes tutoriais: [Usar a configuração dinâmica em um aplicativo ASP.NET Core](https://learn.microsoft.com/azure/azure-app-configuration/enable-dynamic-configuration-aspnet-core) e [Gerenciar sinalizadores de recurso na configuração de aplicativos do Azure](https://learn.microsoft.com/azure/azure-app-configuration/manage-feature-flags).

#### Tarefa 1: criar o recurso de configuração de aplicativos

1. No Portal do Azure, procure o serviço de **Configuração de aplicativos**
2. Clique em **Criar configuração de aplicativos** e depois selecione:
    - A assinatura do Azure.
    - O Grupo de recursos criado anteriormente (ele deve ser chamado **AZ400-EWebShop-NAME**).
    - O local.
    - Um nome exclusivo como **appcs-NAME-REGION**, por exemplo.
    - Selecione o tipo de preço **gratuito**.
3. Clique em **Revisar + criar** e depois **Criar**.
4. Depois de criar o serviço configuração de aplicativos, vá para **Visão geral** e copie/salve o valor do **Ponto de extremidade**.

#### Tarefa 2: habilitar identidade gerenciada

1. Vá para o aplicativo Web implantado usando o pipeline (ele deve ser chamado **az400-Webapp-NAME)**.
2. Na seção **Configurações**, clique em **Identidade** e alterne o status para **Ativado** na seção **Designado pelo sistema**, clique em **salvar > sim** e aguarde alguns segundos para que a operação seja concluída.
3. Volte para o serviço configuração de aplicativos e clique em **Controle de acesso** e depois em **Adicionar atribuição de função**.
4. Na seção **Função**, selecione **Leitor de dados da configuração de aplicativos**.
5. Na seção **Membros**, marque **Gerenciar identidade** e selecione a identidade gerenciada do aplicativo Web (eles devem ter o mesmo nome).
6. Clique em **Revisar e designar**.

#### Tarefa 3: configurar o aplicativo Web

Para se certificar de que seu site está acessando a configuração de aplicativos, você precisa atualizar sua configuração.

1. Vá para seu aplicativo Web.
2. Na seção **Configurações**, clique em **Configuração**.
3. Adicionar duas novas configurações de aplicativo:
    - Configuração do primeiro aplicativo
        - **Nome:** UseAppConfig
        - **Valor: true**
    - Configuração do segundo aplicativo
        - **Nome:** AppConfigEndpoint
        - **Valor:** *o valor que você salvou/copiou anteriormente do ponto de extremidade da configuração de aplicativos. Deve parecer com https://appcs-NAME-REGION.azconfig.io*

4. Clique em **Ok** e em **Salvar** e aguarde até que as configurações sejam atualizadas.
5. Vá para **Visão geral** e clique em **Procurar**
6. Nesta etapa, você não verá alterações no site, já que a configuração de aplicativos não contém dados. Isso é o que você fará nas próximas tarefas.

#### Tarefa 4: testar o gerenciamento de configuração

1. No site, selecione **Visual Studio** na lista suspensa **Marca** e clique no botão de seta (**>**).
2. Você verá uma mensagem dizendo *"NÃO HÁ RESULTADOS CORRESPONDENTES À SUA PESQUISA"*. O objetivo deste laboratório é conseguir atualizar esse valor sem atualizar o código do site ou reimplantá-lo.
3. Para tentar isso, volte para Configuração de aplicativos.
4. Na seção **Operações**, selecione **Gerenciador de configurações**.
5. Clique em **Criar > Valor-chave** e adicione:
    - **Chave:** eShopWeb:Settings:NoResultsMessage
    - **Valor:** *digite sua mensagem personalizada*
6. Clique em **Aplicar** e, em seguida, volte ao seu site e atualize a página.
7. Você deve ver sua nova mensagem em vez do valor padrão antigo.

Parabéns! Nesta tarefa, você testou o **Gerenciador de configurações** na configuração de aplicativos do Azure.

#### Tarefa 5: testar o sinalizador de recursos

Vamos continuar a testar o Gerenciador de recursos.

1. Para tentar isso, volte para Configuração do aplicativo.
2. Na seção **Operações**, selecione **Gerenciador de recursos**.
3. Clique em **Criar** e depois adicione:
    - **Habilitar sinalizador de recursos:** marcado
    - **Nome do sinalizador de recurso:** SalesWeekend
4. Clique em **Aplicar** e, em seguida, volte ao seu site e atualize a página.
5. Você deve ver uma imagem com o texto "TODAS AS CAMISETAS À VENDA NESTE FIM DE SEMANA".
6. Você pode desabilitar esse recurso na Configuração de aplicativos e, em seguida, verá que a imagem desaparece.

Parabéns! Nesta tarefa, você testou o **Gerenciador de recursos** na configuração de aplicativos do Azure.

### Exercício 3: Remover os recursos do laboratório do Azure

Neste exercício, você removerá os recursos do Azure provisionados neste laboratório para eliminar cobranças inesperadas.

>**Observação**: lembre-se de remover todos os recursos do Azure que acabam de ser criados e que você não usa mais. Remover recursos não utilizados garante que você não veja encargos inesperados.

#### Tarefa 1: Remover os recursos do laboratório do Azure

Nesta tarefa, você usará o Azure Cloud Shell para remover os recursos do Azure provisionados neste laboratório para eliminar cobranças desnecessárias.

1. No portal do Azure, abra a sessão de shell **Bash** no painel **Cloud Shell**.
2. Liste todos os grupos de recursos criados em todos os laboratórios deste módulo executando o seguinte comando:

    ```sh
    az group list --query "[?starts_with(name,'AZ400-EWebShop-')].name" --output tsv
    ```

3. Exclua todos os grupos de recursos criados em todos os laboratórios deste módulo executando o seguinte comando:

    ```sh
    az group list --query "[?starts_with(name,'AZ400-EWebShop-')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**Observação**: o comando é executado de modo assíncrono (conforme determinado pelo parâmetro --nowait), portanto, embora você possa executar outro comando da CLI do Azure imediatamente depois na mesma sessão Bash, levará alguns minutos antes de o grupo de recursos ser removido.

## Revisão

Neste laboratório, você aprendeu a habilitar dinamicamente a configuração e gerenciar sinalizadores de recursos.
