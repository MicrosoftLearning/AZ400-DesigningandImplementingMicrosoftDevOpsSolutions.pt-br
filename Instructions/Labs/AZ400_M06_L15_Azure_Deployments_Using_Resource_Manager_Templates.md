---
lab:
  title: Implantações usando modelos do Azure Bicep
  module: 'Module 06: Manage infrastructure as code using Azure and DSC'
---

# Implantações usando modelos do Azure Bicep

# Manual de laboratório do aluno

## Requisitos do laboratório

- Este laboratório requer o **Microsoft Edge** ou um [navegador compatível com o Azure DevOps.](https://docs.microsoft.com/en-us/azure/devops/server/compatibility?view=azure-devops#web-portal-supported-browsers)

- **Configurar uma organização do Azure DevOps:** se você ainda não tiver uma organização Azure DevOps que possa usar para este laboratório, crie uma seguindo as instruções disponíveis em [Criar uma organização ou coleção de projetos](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/create-organization?view=azure-devops).

- Identifique uma assinatura existente do Azure ou crie uma.

- Verifique se você tem uma conta Microsoft ou do Azure AD com a função Proprietário na assinatura do Azure e a função Administrador Global no locatário do Azure AD associado à assinatura do Azure.

- [Visual Studio Code](https://code.visualstudio.com/). Ele será instalado como parte dos pré-requisitos deste laboratório.

## Visão geral do laboratório

Neste laboratório, você criará um modelo do Azure Bicep e o modulará usando o conceito Módulos do Azure Bicep. Em seguida, você modificará o modelo de implantação principal para usar o módulo e, por fim, implantar todos os recursos no Azure.

## Objetivos

Após concluir este laboratório, você poderá:

- Entenda e crie Modelos do Azure Bicep.
- Crie um módulo Bicep reutilizável para recursos de armazenamento.
- Carregar o modelo vinculado no Armazenamento de Blobs do Azure e gerar um token SAS.
- Modifique o modelo principal para usar o módulo.
- Modificar o modelo principal para atualizar dependências.
- Implante todos os recursos no Azure usando Modelos do Azure Bicep.

## Tempo estimado: 60 minutos

## Instruções

### Exercício 0: configurar os pré-requisitos do laboratório

Neste exercício, você configurará os pré-requisitos para o laboratório, que incluem o Visual Studio Code.

#### Tarefa 1: Instalar e configurar o Git e o Visual Studio Code

Nesta tarefa, você instalará o Visual Studio Code. Se você já tiver implantado esse pré-requisito, poderá prosseguir diretamente para a próxima tarefa.

1. Se você ainda não tiver o Visual Studio Code instalado, no computador do laboratório, inicie um navegador da Web e navegue até a [página de download do Visual Studio Code](https://code.visualstudio.com/), baixe-o e instale-o.

### Exercício 1: criar e implantar modelos do Azure Bicep

Neste laboratório, você criará um modelo do Azure Bicep e um módulo de modelo. Em seguida, você modificará o modelo de implantação principal para usar o módulo do modelo e atualizar as dependências e, por fim, implantará os modelos no Azure.

#### Tarefa 1: criar modelos do Azure Bicep

Nesta unidade, você usará o Visual Studio Code para criar um modelo do Azure Bicep

1. No computador do laboratório, inicie o Visual Studio Code, no Visual Studio Code, clique no menu de nível superior em **Arquivo**, no menu suspenso, selecione **Preferências**, no menu em cascata, selecione **Extensões**, na caixa de texto **Extensões de Pesquisa**, digite **Bicep**, selecione o publicado pela Microsoft e clique em **Instalar** para instalar o suporte ao idioma do Azure Bicep.
1. Em um navegador da Web, conecte-se ao **<https://github.com/Azure/azure-quickstart-templates/blob/master/quickstarts/microsoft.compute/vm-simple-windows/main.bicep>**. Clique na opção **Bruto** para o arquivo. Copie o conteúdo da janela do código e cole-o no editor do Visual Studio Code.

   > **Observação**: em vez de criar um modelo do zero, usaremos um dos [Azure QuickStart Templates](https://azure.microsoft.com/en-us/resources/templates/) chamado **Implantar uma VM de modelo simples do Windows**. Os modelos podem ser baixados do GitHub - [vm-simple-windows](https://github.com/Azure/azure-quickstart-templates/blob/master/quickstarts/microsoft.compute/vm-simple-windows).

1. No computador do laboratório, abra o Explorador de arquivos e crie a seguinte pasta local que servirá para armazenar modelos:

   - **C:\\templates**

1. Volte para a janela Visual Studio Code com nosso modelo main.bicep, clique no menu de nível superior **Arquivo**, no menu suspenso, clique em **Salvar como** e salve o modelo como **main.bicep** na pasta local recém-criada **C:\\templates**.
1. Revise o modelo para entender melhor sua estrutura. Há cinco tipos de recursos inclusos no modelo:

   - Microsoft.Storage/storageAccounts
   - Microsoft.Network/publicIPAddresses
   - Microsoft.Network/virtualNetworks
   - Microsoft.Network/networkInterfaces
   - Microsoft.Compute/virtualMachines

1. No Visual Studio Code, salve o arquivo novamente, mas desta vez escolha **C:\\templates** como o destino e **storage.bicep** como o nome do arquivo.

   > **Observação**: agora temos dois arquivos JSON idênticos: **C:\\templates\\main.bicep** e **C:\\templates\\storage.bicep**.

#### Tarefa 2: criar um módulo de modelo para recursos de armazenamento

Nesta tarefa, você modificará os modelos salvos na tarefa anterior, de modo que o módulo de modelo de armazenamento **storage.bicep** criará apenas uma conta de armazenamento, ela será importada pelo primeiro modelo. O módulo de modelo de armazenamento precisa transmitir um valor de volta para o modelo principal, **main.bicep** e esse valor será definido no elemento de saída do módulo de modelo de armazenamento.

1. No arquivo **storage.bicep** exibido na janela do Visual Studio Code, na **seção de recursos**, remova todos os elementos do recurso, exceto o recurso ** storageAccounts**. Isso deve resultar em uma seção de recursos com a seguinte aparência:

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

1. Em seguida, remova todas as definições de variáveis:

   ```bicep
   var storageAccountName = 'bootdiags${uniqueString(resourceGroup().id)}'
   var nicName = 'myVMNic'
   var addressPrefix = '10.0.0.0/16'
   var subnetName = 'Subnet'
   var subnetPrefix = '10.0.0.0/24'
   var virtualNetworkName = 'MyVNET'
   var networkSecurityGroupName = 'default-NSG'
   var securityProfileJson = {
     uefiSettings: {
       secureBootEnabled: true
       vTpmEnabled: true
     }
     securityType: securityType
   }
   var extensionName = 'GuestAttestation'
   var extensionPublisher = 'Microsoft.Azure.Security.WindowsAttestation'
   var extensionVersion = '1.0'
   var maaTenantName = 'GuestAttestation'
   var maaEndpoint = substring('emptyString', 0, 0)
   ```

1. Em seguida, remova todos os valores de parâmetro, exceto local e adicione o seguinte código de parâmetro, resultando no seguinte resultado:

   ```bicep
   @description('Location for all resources.')
   param location string = resourceGroup().location

   @description('Name for the storage account.')
   param storageAccountName string
   ```

1. Em seguida, no final do arquivo, remova a saída atual e adicione uma nova chamada valor de saída storageURI. Modifique a saída para que ela seja semelhante ao seguinte.

   ```bicep
   output storageURI string = storageAccount.properties.primaryEndpoints.blob
   ```

1. Salve o módulo de modelo storage.bicep. O modelo de armazenamento agora deve ter a seguinte aparência:

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

1. No Visual Studio Code, clique no menu de nível superior **Arquivo**, no menu suspenso, selecione **Abrir Arquivo**, na caixa de diálogo Abrir Arquivo, navegue até **C:\\templates\\main.bicep**, selecione-o e clique em **Abrir**.
1. No arquivo **main.bicep**, na seção de recursos, remova o elemento de recurso de armazenamento

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

1. Em seguida, adicione o seguinte código diretamente no mesmo local onde o elemento de recurso de armazenamento recém-excluído estava:

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
   - O módulo tem um nome simbólico chamado storageModule. Esse nome é usado para configurar dependências.
   - Você só pode usar o modo de implantação Incremental ao usar módulos de modelos.
   - Um caminho relativo é usado para o módulo de modelo.
   - Use parâmetros para transmitir valores do modelo principal para os módulos de modelo.

> **Observação**: com os modelos do ARM do Azure, você teria usado uma conta de armazenamento para carregar o modelo vinculado para facilitar o uso por outras pessoas. Com os módulos do Azure Bicep, você tem a opção de carregá-los no registro do módulo bicep do Azure, que tem opções de registro público e privado. Mais informações podem ser encontradas na [documentação de bicep do Azure](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/modules#file-in-registry).

1. Salve o modelo.

#### Tarefa 4: implantar recursos ao Azure usando módulos de modelo

> **Observação**: você pode implantar modelos de várias maneiras, como usando a CLI do Azure instalada localmente ou a partir do Azure Cloud Shell ou de um pipeline de CI/CD. Neste laboratório, use a CLI do Azure no Azure Cloud Shell.

> **Observação**: ao contrário dos modelos de ARM, você não pode usar o portal do Azure para implantar diretamente modelos do Bicep.

> **Observação**: para usar o Azure Cloud Shell, você carregará os arquivos main.bicep e storage.bicep no diretório base do Cloud Shell.

> **Observação**: atualmente, a CLI do Azure não dá suporte à implantação de arquivos Bicep remotos. Você pode criar os arquivos bicep para obter o JSON do modelo de ARM e, em seguida, carregá-los em uma conta de armazenamento e, em seguida, implantá-los remotamente.

1. No computador do laboratório, no navegador da Web que exibe o portal do Azure, clique no ícone do **Cloud Shell** para abrir o Cloud Shell.
   > **Observação**: se você tiver a sessão do PowerShell do início deste exercício ainda ativa, alterne para Bash (próxima etapa).
1. No painel do Cloud Shell, clique em **PowerShell**, no menu suspenso, clique em **Bash** e, quando solicitado, clique em **Confirmar**.
1. No painel Cloud Shell, clique no ícone **Carregar/baixar arquivos** e, no menu suspenso, clique em **Carregar**.
1. Na caixa de diálogo **Abrir**, navegue e selecione **C:\\templates\\main.bicep** e clique em **Abrir**.
1. Siga as mesmas etapas para carregar o arquivo **C:\\templates\\storage.bicep** também.
1. Em uma sessão **Bash** no painel do Cloud Shell, execute o seguinte para executar uma implantação usando um modelo recém-carregado:

   ```bash
   az deployment group what-if --name az400m06l15deployment --resource-group az400m06l15-RG --template-file main.bicep
   ```

1. Quando solicitado a fornecer o valor de “adminUsername”, digite **Student** e pressione a tecla **Enter**.
1. Quando solicitado a fornecer o valor de “adminUsername”, digite **Pa55w.rd1234** e pressione a tecla **Enter**. (A digitação da senha não será exibida)
1. Analise o resultado deste comando que valida sua implantação e vamos saber se há algum erro em seus modelos. Isso é muito valioso, especialmente ao implantar modelos com muitos recursos e em ambientes de nuvem comercialmente críticos.

1. Em uma sessão **Bash** no painel do Cloud Shell, execute o seguinte para executar uma implantação usando um modelo recém-carregado:

   ```bash
   LOCATION='<region>'
   ```
   > **Observação**: substitua o nome da região por uma região próxima à sua localização. Se você não souber quais locais estão disponíveis, execute o comando `az account list-locations -o table`.
  
   ```bash
   az group create --name az400m06l15-RG --location $LOCATION
   ```

   ```bash   
   az deployment group create --name az400m06l15deployment --resource-group az400m06l15-RG --template-file main.bicep
   ```

1. Quando solicitado a fornecer o valor de “adminUsername”, digite **Student** e pressione a tecla **Enter**.
1. Quando solicitado a fornecer o valor de “adminUsername”, digite **Pa55w.rd1234** e pressione a tecla **Enter**. (A digitação da senha não será exibida)

1. Se você receber erros ao executar o comando acima para implantar o modelo, tente o seguinte:

   - Se você tiver várias assinaturas do Azure, certifique-se de ter definido o contexto de assinatura para o correto onde o grupo de recursos está implantado.
   - Verifique se o modelo vinculado está acessível por meio do URI especificado.

> **Observação**: como próxima etapa, agora você pode modularizar as definições de recursos restantes no modelo de implantação principal, como as definições de recursos de rede e máquina virtual.

> **Observação**: se você não planeja usar os recursos implantados, exclua-os para evitar cobranças associadas. Você pode fazer isso simplesmente excluindo o grupo de recursos **az400m06l15-RG**.

### Exercício 2: remover os recursos do Azure Lab

Neste exercício, você removerá os recursos do Azure provisionados neste laboratório para eliminar cobranças inesperadas.

> **Observação**: lembre-se de remover todos os recursos do Azure que acabam de ser criados e que você não usa mais. Remover recursos não utilizados garante que você não veja encargos inesperados.

#### Tarefa 1: Remover os recursos do laboratório do Azure

Nesta tarefa, você usará o Azure Cloud Shell para remover os recursos do Azure provisionados neste laboratório para eliminar cobranças desnecessárias.

1. No portal do Azure, abra a sessão de shell **Bash** no painel **Cloud Shell**.
1. Liste todos os grupos de recursos criados em todos os laboratórios deste módulo executando o seguinte comando:

   ```bash
   az group list --query "[?starts_with(name,'az400m06l15-RG')].name" --output tsv
   ```

1. Exclua todos os grupos de recursos criados em todos os laboratórios deste módulo executando o seguinte comando:

   ```bash
   az group list --query "[?starts_with(name,'az400m06l15-RG')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
   ```

   > **Observação**: o comando é executado de modo assíncrono (conforme determinado pelo parâmetro --nowait), portanto, embora você possa executar outro comando da CLI do Azure imediatamente depois na mesma sessão Bash, levará alguns minutos antes de o grupo de recursos ser removido.

## Revisão

Neste laboratório, você aprendeu como criar um modelo do Azure Resource manager, modularizá-lo usando um modelo vinculado, modificar o modelo de implantação principal para chamar o modelo vinculado e as dependências atualizadas e, finalmente, implantar os modelos no Azure.
