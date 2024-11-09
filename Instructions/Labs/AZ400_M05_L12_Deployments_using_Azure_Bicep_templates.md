---
lab:
  title: Implantações usando modelos do Azure Bicep
  module: 'Module 05: Manage infrastructure as code using Azure and DSC'
---

# Implantações usando modelos do Azure Bicep

## Requisitos do laboratório

- Este laboratório requer o **Microsoft Edge** ou um [navegador com suporte do Azure DevOps.](https://docs.microsoft.com/azure/devops/server/compatibility)

- **Configurar uma organização do Azure DevOps:** se você ainda não tiver uma organização Azure DevOps que possa usar para este laboratório, crie uma seguindo as instruções disponíveis em [Criar uma organização ou coleção de projetos](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization).

- Identifique uma assinatura existente do Azure ou crie uma.

- Verifique se você tem uma conta Microsoft ou uma conta do Microsoft Entra com a função Proprietário na assinatura do Azure e a função Administrador Global no locatário do Microsoft Entra associado à assinatura do Azure. Para obter detalhes, veja [Listar designações de função do Azure usando o portal do Azure](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-list-portal) e [Exibir e designar funções de administrador no Azure Active Directory](https://docs.microsoft.com/azure/active-directory/roles/manage-roles-portal).

## Visão geral do laboratório

Neste laboratório, você criará um modelo do Azure Bicep e o modularizará usando o conceito Módulos do Azure Bicep. Em seguida, você modificará o modelo de implantação main para usar o módulo e, por fim, implantará todos os recursos no Azure.

## Objetivos

Após concluir este laboratório, você poderá:

- Entender a estrutura de um modelo do Azure Bicep.
- Criar um módulo do Bicep reutilizável.
- Modificar o modelo principal para usar o módulo
- Implantar todos os recursos no Azure usando pipelines YAML do Azure.

## Tempo estimado: 45 minutos

## Instruções

### Exercício 0: (pular se já foi feito) Configurar os pré-requisitos do laboratório

Neste exercício, você configurará os pré-requisitos para o laboratório, que consistem em um novo projeto do Azure DevOps com um repositório baseado no [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb).

#### Tarefa 1: (pular se feita) criar e configurar o projeto de equipe

Nesta tarefa, você criará um projeto **eShopOnWeb** do Azure DevOps para ser usado por vários laboratórios.

1. No computador do laboratório, em uma janela do navegador, abra sua organização do Azure DevOps. Clique em **Novo projeto**. Dê ao seu projeto o nome **eShopOnWeb** e deixe os outros campos com padrões. Clique em **Criar**.

    ![Captura de tela do painel criar um novo projeto.](images/create-project.png)

#### Tarefa 2: (pular se feita) importar repositório do Git eShopOnWeb

Nesta tarefa, você importará o repositório eShopOnWeb do Git que será usado por vários laboratórios.

1. No computador do laboratório, em uma janela do navegador, abra sua organização do Azure DevOps e o projeto **eShopOnWeb** criado anteriormente. Clique em **Repos > Arquivos**, **Importar um repositório**. Selecione **Importar**. Na janela **Importar um repositório do Git**, cole a seguinte URL <https://github.com/MicrosoftLearning/eShopOnWeb.git> e clique em **Importar**:

    ![Captura de tela do painel importar repositório.](images/import-repo.png)

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

### Exercício 1: compreender um modelo do Azure Bicep e simplificá-lo usando um módulo reutilizável

Neste laboratório, você analisará um modelo do Azure Bicep e o simplificará usando um módulo reutilizável.

#### Tarefa 1: criar modelo do Azure Bicep

Nesta tarefa, você usará o Visual Studio Code para criar um modelo do Azure Bicep

1. Na guia do navegador com seu projeto do Azure DevOps aberto, navegue até **Repositórios** e **Arquivos**. Abra a pasta `infra` e localize o arquivo `simple-windows-vm.bicep`.

   ![Captura de tela do caminho do arquivo simple-windows-vm.bicep.](./images/m06/browsebicepfile.png)

1. Revise o modelo para entender melhor a estrutura. Existem alguns parâmetros com tipos, valores padrão e validação, algumas variáveis e alguns recursos com esses tipos:

   - Microsoft.Storage/storageAccounts
   - Microsoft.Network/publicIPAddresses
   - Microsoft.Network/virtualNetworks
   - Microsoft.Network/networkInterfaces
   - Microsoft.Compute/virtualMachines

1. Preste atenção em como as definições de recursos são simples e na capacidade de referenciar implicitamente nomes simbólicos em vez de explícitos `dependsOn` no modelo.

#### Tarefa 2: criar um módulo Bicep reutilizável para recursos de armazenamento

Nesta tarefa, você criará um módulo de modelo de armazenamento **storage.bicep** que criará apenas uma conta de armazenamento e será importado pelo modelo principal. O módulo de modelo de armazenamento precisa passar um valor de volta para o modelo main, **main.bicep**, e esse valor será definido no elemento outputs do módulo de modelo de armazenamento.

1. Primeiro, precisamos remover o recurso de armazenamento do nosso modelo main. No canto superior direito da janela do navegador, clique no botão **Editar**:

   ![Captura de tela do botão editar pipeline.](./images/m06/edit.png)

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

1. Confirme o arquivo, mas ainda iremos usá-lo.

   ![Captura de tela do botão de commit de arquivo](./images/m06/commit.png)

1. Em seguida, passe o mouse sobre a pasta `Infra` e clique no ícone de reticências, depois selecione **Novo** e **Arquivo**. Para o nome, insira **`storage.bicep`** e clique em **Criar**.

   ![Captura de tela do menu novo arquivo.](./images/m06/newfile.png)

1. Agora, copie o seguinte snippet de código para o arquivo e confirme suas alterações:

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

#### Tarefa 3: modificar o modelo main para usar o módulo de modelo

Nesta tarefa, você modificará o modelo main para fazer referência ao módulo de modelo criado na tarefa anterior.

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

1. Também precisamos modificar a referência ao URI de blob da conta de armazenamento em nosso recurso de máquina virtual para usar a saída do módulo. Localize o recurso de máquina virtual e substitua a seção diagnosticsProfile pelo seguinte:

   ```bicep
   diagnosticsProfile: {
     bootDiagnostics: {
       enabled: true
       storageUri: storageModule.outputs.storageURI
     }
   }
   ```

1. Revise os detalhes a seguir no modelo main:

   - Um módulo no modelo main é usado para vincular a outro modelo.
   - O módulo tem um nome simbólico chamado `storageModule`. Esse nome é usado para configurar alguma dependência.
   - Você só pode usar o modo de implantação **Incremental** quando usar módulos de modelo.
   - Um caminho relativo é usado para o módulo de modelo.
   - Use parâmetros para passar valores do modelo main para os módulos de modelo.

1. Confirme o modelo.

### Exercício 2: implantar os modelos no Azure usando pipelines YAML

Neste laboratório, você criará um pipeline YAML do Azure DevOps para implantar seu modelo no ambiente do Azure.

#### Tarefa 1: Implantar recursos no Azure por pipelines YAML

1. Navegue de volta ao painel **Pipelines** no hub **Pipelines**.
1. Na janela **Criar seu primeiro pipeline**, clique em **Criar pipeline**.

    > **Observação**: usaremos o assistente para criar uma nova definição de pipeline do YAML com base em nosso projeto.

1. No painel **Onde está seu código?**, clique na opção **Git do Azure Repos (YAML).**
1. No painel **Selecionar um repositório**, clique em **eShopOnWeb**.
1. No painel **Configurar seu pipeline**, role para baixo e selecione **Arquivo YAML existente do Azure Pipelines**.
1. Na folha **Selecionar um arquivo YAML existente** , especifique os seguintes parâmetros:
   - Ramificação: **principal**
   - Caminho: **.ado/eshoponweb-cd-windows-cm.yml**
1. Clique em **Continuar** para salvar essas configurações.
1. Na seção de variáveis, escolha um nome para seu grupo de recursos, defina o local desejado e substitua o valor da conexão de serviço por uma de suas conexões de serviço existentes criadas anteriormente.
1. Clique no botão **Salvar e executar** no canto superior direito e, quando a caixa de diálogo de confirmação aparecer, clique em **Salvar e executar** novamente.

   ![Captura de tela do botão salvar e executar.](./images/m06/saveandrun.png)

1. Aguarde até que a implantação seja concluída e veja os resultados.
   ![Captura de tela da implantação bem-sucedida de recurso no Azure usando pipelines YAML.](./images/m06/deploy.png)

   > [!IMPORTANT]
   > Lembre-se de excluir os recursos criados no portal do Azure para evitar cobranças desnecessárias.

## Revisão

Neste laboratório, você aprendeu a criar um modelo do Azure Bicep, modularizá-lo usando um módulo de modelo, modificar o modelo de implantação main para usar o módulo e as dependências atualizadas e, por fim, a implantar os modelos no Azure usando pipelines YAML.
