---
lab:
  title: Implantações usando modelos do Azure Bicep
  module: 'Module 06: Manage infrastructure as code using Azure and DSC'
---

# Implantações usando modelos do Azure Bicep

## Manual de laboratório do aluno

## Requisitos do laboratório

- Este laboratório requer o **Microsoft Edge** ou um [navegador compatível com o Azure DevOps.](https://docs.microsoft.com/azure/devops/server/compatibility)

- **Configurar uma organização do Azure DevOps:** se você ainda não tiver uma organização Azure DevOps que possa usar para este laboratório, crie uma seguindo as instruções disponíveis em [Criar uma organização ou coleção de projetos](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization).

- Identifique uma assinatura existente do Azure ou crie uma.

- Verifique se você tem uma conta Microsoft ou uma conta do Microsoft Entra com a função Proprietário na assinatura do Azure e a função Administrador Global no locatário do Microsoft Entra associado à assinatura do Azure. Para obter detalhes, veja [Listar designações de função do Azure usando o portal do Azure](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-list-portal) e [Exibir e designar funções de administrador no Azure Active Directory](https://docs.microsoft.com/azure/active-directory/roles/manage-roles-portal).

## Visão geral do laboratório

Neste laboratório, você criará um modelo do Azure Bicep e o modulará usando o conceito Módulos do Azure Bicep. Em seguida, você modificará o modelo de implantação principal para usar o módulo e, por fim, implantar todos os recursos no Azure.

## Objetivos

Após concluir este laboratório, você poderá:

- Entender a estrutura de um modelo do Azure Bicep.
- Criar um módulo Bicep reutilizável.
- Modificar o modelo principal para usar o módulo
- Implantar todos os recursos no Azure usando pipelines de YAML do Azure.

## Tempo estimado: 45 minutos

## Instruções

### Exercício 0: configurar os pré-requisitos do laboratório

Neste exercício, você configurará os pré-requisitos para o laboratório, que consistem em um novo projeto do Azure DevOps com um repositório baseado no [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb).

#### Tarefa 1: (ignorar se concluído) criar e configurar o projeto de equipe

Nesta tarefa, você criará um projeto do Azure DevOps no **eShopOnWeb** para ser usado por vários laboratórios.

1. No computador do laboratório, em uma janela do navegador, abra sua organização do Azure DevOps. Clique em **Novo projeto**. Dê ao seu projeto o nome **eShopOnWeb** e deixe os outros campos com padrões. Clique em **Criar**.

    ![Criar Projeto](images/create-project.png)

#### Tarefa 2: (ignorar se concluído) importar repositório Git do eShopOnWeb

Nesta tarefa, você importará o repositório Git do eShopOnWeb que será usado por vários laboratórios.

1. No computador do laboratório, em uma janela do navegador, abra sua organização do Azure DevOps e o projeto criado anteriormente do **eShopOnWeb**. Clique em **Repos>Files**, **Importar um repositório**. Selecione **Importar**. Na janela, **Importar um repositório Git**, cole a seguinte URL https://github.com/MicrosoftLearning/eShopOnWeb.git e clique em **Importar**:

    ![Importar repositório](images/import-repo.png)

1. O repositório está organizado da seguinte forma:
    - A pasta **.ado** contém os pipelines YAML do Azure DevOps.
    - O contêiner da pasta **.devcontainer** está configurado para o desenvolvimento usando contêineres (localmente no VS Code ou no GitHub Codespaces).
    - A pasta **.azure** contém a infraestrutura Bicep&ARM como modelos de código usados em alguns cenários do laboratório.
    - A pasta **.github** contém definições de fluxo de trabalho YAML do GitHub.
    - A pasta **src** contém o site .NET 6 usado nos cenários do laboratório.

### Exercício 1: compreender um modelo do Azure Bicep e simplificá-lo usando um módulo reutilizável

Neste laboratório, você revisará e simplificará um modelo do Azure Bicep usando um módulo reutilizável.

#### Tarefa 1: criar modelos do Azure Bicep

Nesta unidade, você usará o Visual Studio Code para criar um modelo do Azure Bicep

1. Na guia do navegador onde você tem o projeto do Azure DevOps aberto, navegue até **Repos** e **Arquivos**. Localize e abra a pasta `.azure\bicep` e localize o arquivo `simple-windows-vm.bicep`.

   ![Arquivo Simple-windows-vm.bicep](./images/m06/browsebicepfile.png)

1. Revise o modelo para entender melhor sua estrutura. Existem alguns parâmetros com tipos, valores padrão e validação, algumas variáveis e alguns recursos com esses tipos:

   - Microsoft.Storage/storageAccounts
   - Microsoft.Network/publicIPAddresses
   - Microsoft.Network/virtualNetworks
   - Microsoft.Network/networkInterfaces
   - Microsoft.Compute/virtualMachines

1. Preste atenção em como as definições de recursos são simples e na capacidade de referenciar implicitamente nomes simbólicos em vez de `dependsOn` explícitos em todo o modelo.

#### Tarefa 2: criar um módulo Bicep para recursos de armazenamento

Nesta tarefa, você criará um módulo de modelo de armazenamento **storage.bicep** que criará apenas uma conta de armazenamento e será importado pelo modelo principal. O módulo de modelo de armazenamento precisa transmitir um valor de volta para o modelo principal, **main.bicep** e esse valor será definido no elemento de saída do módulo de modelo de armazenamento.

1. Primeiro, precisamos remover o recurso de armazenamento do nosso modelo principal. No canto superior direito da janela do navegador, clique no botão **Editar**:

   ![Botão Editar](./images/m06/edit.png)

1. Agora exclua o recurso de armazenamento:

   ```bicep
   resource storageAccount 'Microsoft.Storage/storageAccounts@2022-05-01' = {
     name: storageAccountName
     location: location
     sku: {
       name: 'Standard_LRS'
     }
     kind: 'Storage'
   }
   ```

1. Confirme o arquivo, no entanto, ainda não terminamos.

   ![Confirmar o arquivo](./images/m06/commit.png)

1. Em seguida, passe o mouse sobre a pasta bicep e clique no ícone de reticências e selecione **Novo** e **Arquivo**. Insira **storage.bicep** como o nome e clique em **Criar**.

   ![Menu Novo arquivo](./images/m06/newfile.png)

1. Agora, copie o seguinte snippet de código para o arquivo e confirme as mudanças:

   ```bicep
   @description('Location for all resources.')
   param location string = resourceGroup().location

   @description('Name for the storage account.')
   param storageAccountName string

   resource storageAccount 'Microsoft.Storage/storageAccounts@2022-05-01' = {
     name: storageAccountName
     location: location
     sku: {
       name: 'Standard_LRS'
     }
     kind: 'Storage'
   }

   output storageURI string = storageAccount.properties.primaryEndpoints.blob
   ```

#### Tarefa 3: modificar o modelo principal para usar o módulo do modelo

Nesta tarefa, você modificará o modelo principal para fazer referência ao módulo de modelo criado na tarefa anterior.

1. Navegue de volta para o arquivo `simple-windows-vm.bicep` e clique no botão **Editar** mais uma vez.

1. Em seguida, adicione o seguinte código após as variáveis:

   ```bicep
   module storageModule './storage.bicep' = {
     name: 'linkedTemplate'
     params: {
       location: location
       storageAccountName: storageAccountName
     }
   }
   ```

1. Também precisamos modificar a referência ao URI de blob da conta de armazenamento no nosso recurso de máquina virtual para usar a saída do módulo. Localize o recurso de máquina virtual e substitua a seção diagnosticsProfile pelo seguinte:

   ```bicep
   diagnosticsProfile: {
     bootDiagnostics: {
       enabled: true
       storageUri: storageModule.outputs.storageURI
     }
   }
   ```

1. Os detalhes a seguir são solicitados no modelo principal:

   - Um módulo no modelo principal é usado para vincular a outro modelo.
   - O módulo tem um nome simbólico chamado `storageModule`. Esse nome é usado para configurar dependências.
   - Você só pode usar o modo de implantação **Incremental** ao usar módulos de modelos.
   - Um caminho relativo é usado para o módulo de modelo.
   - Use parâmetros para transmitir valores do modelo principal para os módulos de modelo.

1. Confirme o modelo.

### Exercício 2: implantar os modelos no Azure usando pipelines YAML

Neste laboratório, você criará uma conexão de serviço e a usará em um pipeline YAML do Azure DevOps para implantar o modelo ao ambiente do Azure.

#### Tarefa 1: (ignorar se concluído) criar uma conexão de serviço para implantação

Nesta tarefa, você criará uma entidade de serviço usando a CLI do Azure, que permitirá ao Azure DevOps:

- Implantar recursos na assinatura do Azure.
- Ter acesso de leitura sobre os segredos do Cofre de chaves criados por último.

> **Observação**: se você já tiver uma entidade de serviço, poderá prosseguir diretamente para a próxima tarefa.

Você precisará de uma entidade de serviço para implantar recursos do Azure a partir do Azure Pipelines. Como vamos recuperar segredos em um pipeline, precisaremos conceder permissão ao serviço quando criarmos o Azure Key Vault.

Uma entidade de serviço é criada automaticamente pelo Azure Pipelines, quando você se conecta a uma assinatura do Azure de dentro de uma definição de pipeline ou quando cria uma nova Conexão de serviço na página de configurações do projeto (opção automática). Você também pode criar manualmente a entidade de serviço a partir do portal ou usando a CLI do Azure e reutilizá-la em projetos.

1. No computador do laboratório, inicie um navegador da Web, navegue até o [**Portal do Azure**](https://portal.azure.com) e entre com a conta de usuário que tem a função Proprietário na assinatura do Azure que você usará neste laboratório e tem a função de Administrador global no locatário do Microsoft Entra associado a essa assinatura.
1. No portal do Azure, clique no ícone do **Cloud Shell**, localizado à direita da caixa de texto de pesquisa na parte superior da página.
1. Se for solicitado que você selecione **Bash** ou **PowerShell**, selecione **Bash**.

   >**Observação**: se esta for a primeira vez que você está iniciando o **Cloud Shell** e você receber a mensagem **Você não tem nenhum armazenamento montado**, selecione a assinatura que você está usando no laboratório e selecione **Criar armazenamento**.

1. No prompt **Bash**, no painel **Cloud Shell**, execute os seguintes comandos para recuperar os valores da ID de assinatura do Azure e dos atributos de nome de assinatura:

    ```bash
    az account show --query id --output tsv
    az account show --query name --output tsv
    ```

    > **Observação**: copie ambos os valores para um arquivo de texto. Você precisará deles mais adiante neste laboratório.

1. No prompt **Bash**, no painel **Cloud Shell**, execute o seguinte comando para criar uma entidade de serviço (substitua **myServicePrincipalName** por uma cadeia de caracteres exclusiva que consista em letras e dígitos) e **mySubscriptionID** por sua subscriptionId do Azure:

    ```bash
    az ad sp create-for-rbac --name myServicePrincipalName \
                         --role contributor \
                         --scopes /subscriptions/mySubscriptionID
    ```

    > **Observação**: o comando gerará uma saída JSON. Copie a saída em um arquivo de texto. Você precisará dela mais adiante neste laboratório.

1. Em seguida, no computador do laboratório, inicie um navegador da Web, navegue até o projeto **eShopOnWeb** do Azure DevOps. Clique em **Configurações do Projeto > Conexões de Serviço (em Pipelines)** e **Nova conexão de serviço**.

    ![Nova conexão de serviço](images/new-service-connection.png)

1. Na tela **Nova conexão de serviço**, escolha **Azure Resource Manager** e **Avançar** (pode ser necessário rolar para baixo).

1. Escolha **Entidade de serviço (manual)** e clique em **Avançar**.

1. Preencha os campos vazios usando as informações coletadas durante as etapas anteriores:
    - ID e nome da assinatura.
    - ID da entidade de serviço (appId), chave da entidade de serviço (senha) e ID do locatário (locatário).
    - Em **Nome da conexão de serviço**, digite **azure subs**. Esse nome será referenciado em pipelines YAML quando precisar de uma conexão de serviço do Azure DevOps para se comunicar com a assinatura do Azure.

    ![Conexão de serviço do Azure](images/azure-service-connection.png)

1. Clique em **Verificar e Salvar**.

#### Tarefa 2: implantar recursos ao Azure com pipelines YAML
1. Navegue de volta para o painel **Pipelines** no hub **Pipelines**.
1. Na janela **Criar seu primeiro pipeline**, clique em **Criar pipeline**.

    > **Observação**: usaremos o assistente para criar uma nova definição de Pipeline YAML com base no nosso projeto.

1. Na página **Onde está seu código?**, selecione a opção **Git do Azure Repos (YAML)**.
1. No painel **Selecionar um repositório**, clique em **eShopOnWeb**.
1. Na tela **Configurar o pipeline**, role para baixo e selecione **Arquivo YAML existente do Azure Pipelines**.
1. Na folha **Selecionando um arquivo YAML existente**, especifique os seguintes parâmetros:
   - Ramificação: **principal**
   - Caminho: **.ado/eshoponWeb-cd-windows-cm.yml**
1. Para salvar as configurações, clique em **Continuar**.
1. Na seção de variáveis, escolha um nome para o grupo de recursos, defina o local desejado e substitua o valor da conexão de serviço por uma das conexões de serviço existentes criadas anteriormente.
1. Clique no botão **Salvar e executar** na ordem superior direita e, quando a caixa de diálogo de confirmação aparecer, clique em **Salvar e executar** novamente.

   ![Salvar e executar o pipeline YAML depois de fazer mudanças](./images/m06/saveandrun.png)

1. Aguarde a conclusão da implantação e analise os resultados.
   ![Implantação bem-sucedida de recursos no Azure usando pipelines YAML](./images/m06/deploy.png)

#### Tarefa 3: Remover os recursos do laboratório do Azure

Nesta tarefa, você usará o Azure Cloud Shell para remover os recursos do Azure provisionados neste laboratório para eliminar cobranças desnecessárias.

1. No portal do Azure, abra a sessão de shell **Bash** no painel **Cloud Shell**.
1. Exclua todos os grupos de recursos criados em todos os laboratórios deste módulo executando o seguinte comando (substitua o nome do grupo de recursos pelo que você escolheu):

   ```bash
   az group list --query "[?starts_with(name,'AZ400-EWebShop-NAME')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
   ```

   > **Observação**: o comando é executado de modo assíncrono (conforme determinado pelo parâmetro --nowait), portanto, embora você possa executar outro comando da CLI do Azure imediatamente depois na mesma sessão Bash, levará alguns minutos antes de o grupo de recursos ser removido.

## Revisão

Neste laboratório, você aprendeu como criar um modelo do Azure Bicep, modularizá-lo usando um módulo de modelo, modificar o modelo de implantação principal para usar o módulo e as dependências atualizadas e, finalmente, implantar os modelos no Azure usando pipelines YAML.
