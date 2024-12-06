---
lab:
  title: Como integrar o Azure Key Vault ao Azure DevOps
  module: 'Module 04: Implement a secure continuous deployment using Azure Pipelines'
---

# Como integrar o Azure Key Vault ao Azure DevOps

## Requisitos do laboratório

- Este laboratório requer o **Microsoft Edge** ou um [navegador com suporte do Azure DevOps.](https://learn.microsoft.com/azure/devops/server/compatibility)

- **Configurar uma organização do Azure DevOps:** se você ainda não tiver uma organização Azure DevOps que possa usar para este laboratório, crie uma seguindo as instruções disponíveis em [Criar uma organização ou coleção de projetos](https://learn.microsoft.com/azure/devops/organizations/accounts/create-organization).
- Identifique uma assinatura existente do Azure ou crie uma.

## Visão geral do laboratório

O Azure Key Vault fornece armazenamento seguro e gerenciamento de dados confidenciais, como chaves, senhas e certificados. O Azure Key Vault inclui suporte para módulos de segurança de hardware e uma variedade de algoritmos de criptografia e comprimentos de chave. O uso do Azure Key Vault pode minimizar a possibilidade de divulgar dados confidenciais por meio do código-fonte, o que é um erro comum cometido pelos desenvolvedores. O acesso ao Azure Key Vault requer autenticação e autorização adequadas, dando suporte a permissões refinadas para seu conteúdo.

Neste laboratório, você verá como integrar o Azure Key Vault a um Pipeline do Azure usando as seguintes etapas:

- Criar um Azure Key Vault para armazenar uma senha do ACR como um segredo.
- Fornecer acesso a segredos no Azure Key Vault.
- Configure permissões para ler o segredo.
- Configurar o pipeline para recuperar a senha do Azure Key Vault e passá-la para tarefas posteriores.

## Objetivos

Após concluir este laboratório, você poderá:

- Criar um Azure Key Vault.
- Recuperar um segredo do Azure Key Vault em um pipeline do Azure DevOps.
- Usar o segredo em uma tarefa subsequente no pipeline.
- Implantar uma imagem de contêiner na ACI (Instância de Contêiner do Azure) usando o segredo.

## Tempo estimado: 40 minutos

## Instruções

### Exercício 0: (pular se já foi feito) Configurar os pré-requisitos do laboratório

Neste exercício, você configurará os pré-requisitos para o laboratório, que consistem em um novo projeto do Azure DevOps com um repositório baseado no [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb).

#### Tarefa 1: (pular se feita) criar e configurar o projeto de equipe

Nesta tarefa, você criará um projeto **eShopOnWeb** do Azure DevOps para ser usado por vários laboratórios.

1. No computador do laboratório, em uma janela do navegador, abra sua organização do Azure DevOps. Clique em **Novo projeto**. Dê ao seu projeto o nome **eShopOnWeb** e deixe os outros campos com padrões. Clique em **Criar**.

    ![Captura de tela do painel criar um novo projeto.](images/create-project.png)

#### Tarefa 2: (pular se feita) importar repositório do Git eShopOnWeb

Nesta tarefa, você importará o repositório eShopOnWeb do Git que será usado por vários laboratórios.

1. No computador do laboratório, em uma janela do navegador, abra sua organização do Azure DevOps e o projeto **eShopOnWeb** criado anteriormente. Clique em **Repos > Arquivos**, **Importar**. Na janela **Importar um repositório do Git**, cole a URL <https://github.com/MicrosoftLearning/eShopOnWeb.git> e clique em **Importar**:

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

### Exercício 1: configurar o pipeline de CI para criar um contêiner eShopOnWeb

Neste exercício, você criará um pipeline de CI que cria e envia as imagens de contêiner do eShopOnWeb para um ACR (Registro de Contêiner do Azure). O pipeline usará o Docker Compose para criar as imagens e efetuar push delas no ACR.

#### Tarefa 1: Configurar e executar o pipeline de CI

Nesta tarefa, você importará, modificará e executará uma definição de pipeline YAML de CI existente. Será criado um novo Registro de Contêiner do Azure (ACR) e criará/publicará as imagens de contêiner eShopOnWeb.

1. No computador do laboratório, inicie um navegador da Web e navegue até o projeto **eShopOnWeb** do Azure DevOps. Vá para **Pipelines > Pipelines** e clique em **Criar pipeline** (ou **Novo pipeline**).

1. Na janela **Onde está seu código?**, selecione **Azure Repos Git (YAML)** e selecione o repositório **eShopOnWeb**.

1. Na seção **Configurar**, escolha o **Arquivo YAML existente do Azure Pipelines**. Selecione branch: **principal**, forneça o seguinte caminho **/.ado/eshoponweb-ci-dockercompose.yml** e clique em **Continuar**.

    ![Captura de tela do pipeline do YAML existente.](images/select-ci-container-compose.png)

1. Na definição de pipeline YAML, personalize o nome do grupo de recursos substituindo **NOME** em **AZ400-EWebShop-NOME** por um valor exclusivo e substitua **ID-DA-ASSINATURA** pela sua subscriptionId do Azure.

1. Clique em **Salvar e Executar** e aguarde até que o pipeline seja executado com êxito.

    > **Importante**: se você vir a mensagem "Este pipeline precisa de permissão para acessar recursos antes que esta execução possa continuar para o Docker Compose para ACI", clique em Exibir, Permitir e Permitir novamente. Isso é necessário para permitir que o pipeline crie o recurso.

    > **Observação**: o build poderá levar alguns minutos para ser concluído. A definição de build consiste nas seguintes tarefas:
    - **AzureResourceManagerTemplateDeployment** usa **bicep** para implantar um Registro de Contêiner do Azure.
    - A tarefa **PowerShell** obtém a saída do bicep (servidor de logon do acr) e cria a variável de pipeline.
    - A tarefa **DockerCompose** cria e envia as imagens de contêiner do eShopOnWeb para o Registro de Contêiner do Azure.

1. Seu pipeline terá um nome com base no nome do projeto. Vamos **renomear** o pipeline para melhor identificá-lo. Vá até **Pipelines > Pipelines** e clique no pipeline criado recentemente. Clique nas reticências e na opção **Renomear/Remover**. Nomeie-o **`eshoponweb-ci-dockercompose`** e clique em **Salvar**.

1. Quando a execução for concluída, no Portal do Azure, abra o Grupo de Recursos definido anteriormente e você encontrará um Registro de Contêiner do Azure (ACR) com as imagens de contêiner criadas **eshoppublicapi** e **eshopwebmvc**. Você só usará **eshopwebmvc** na fase de implantação.

    ![Captura de tela de imagens de contêiner no ACR.](images/azure-container-registry.png)

1. Clique em **Chaves de Acesso**, habilite o **usuário Administrador**, caso ainda não tenha feito isso, e copie o valor da **senha**. Ela será usada na tarefa a seguir, pois a manteremos como um segredo no Azure Key Vault.

    ![Captura de tela do local da senha do ACR.](images/acr-password.png)

#### Tarefa 2: criar um Azure Key Vault

Nesta tarefa, você criará um Azure Key Vault usando o portal do Azure.

Para esse cenário de laboratório, teremos uma ACI (Instância de Contêiner) do Azure que extrai e executa uma imagem de contêiner armazenada no Registro de Contêiner do Azure (ACR). Pretendemos armazenar a senha do ACR como um segredo no Key Vault.

1. No portal do Azure, na caixa de texto **Pesquisar recursos, serviços e documentos**, digite **`Key vault`** e pressione a tecla **Enter**.
1. Selecione a folha **Cofres de chaves**, clique em **Criar > Cofre de chaves**.
1. Na guia **Noções básicas** da folha **Criar um Key Vault**, especifique as seguintes configurações e clique em **Avançar**:

    | Configuração | Valor |
    | --- | --- |
    | Assinatura | o nome da assinatura do Azure que você está usando neste laboratório |
    | Grupo de recursos | o nome de um novo grupo de recursos **AZ400-EWebShop-NAME** |
    | Nome do cofre de chaves | qualquer nome válido exclusivo, como **ewebshop-kv-NAME (substitua NAME** ) |
    | Região | uma região do Azure próxima do local do seu ambiente de laboratório |
    | Tipo de preço | **Standard** |
    | Dias de retenção dos cofres excluídos | **7** |
    | Proteção contra limpeza | **Desabilitar proteção contra limpeza** |

1. Na guia **Configuração de acesso** da folha **Criar um Key Vault**, selecione **Política de acesso do Vault** e, na seção **Políticas de acesso**, clique em **+ Criar** para configurar uma nova política.

    > **Observação**: você precisa proteger o acesso aos cofres de chaves permitindo apenas aplicativos e usuários autorizados. Para acessar os dados no cofre, você precisará fornecer permissões de leitura (Obter/Listar) para a entidade de serviço criada anteriormente que será usada para autenticação no pipeline.

    1. Na folha **Permissão**, abaixo de **Permissões de segredo**, marque as permissões **Obter** e **Listar** . Clique em **Avançar**.
    2. Na folha **Entidade**, procure a **Entidade de Serviço criada anteriormente** usando a ID ou o Nome fornecido, e selecione-a na lista. Clique em **Avançar**, **Avançar**, **Criar** (política de acesso).
    3. Na folha **Revisar + criar**, clique em **Criar**.

1. De volta à folha **Criar um Key Vault**, clique em **Revisar + Criar > Criar**

    > **Observação**: aguarde até que o Azure Key Vault seja provisionado. Isso deverá levar menos de 1 minuto.

1. Na folha **A implantação foi concluída**, clique em **Ir para o recurso**.
1. Na folha Azure Key Vault (ewebshop-kv-NAME), no menu vertical no lado esquerdo da folha, na seção **Objetos**, clique em **Segredos**.
1. Na folha **Segredos**, clique em **Gerar/Importar**.
1. Na folha **Criar um segredo**, especifique as seguintes configurações e clique em **Criar** (deixe as demais com seus valores padrão):

    | Configuração | Valor |
    | --- | --- |
    | Opções de upload | **Manual** |
    | Nome | **acr-secret** |
    | Valor | Senha de acesso do ACR copiada na tarefa anterior |

#### Tarefa 3: criar um grupo de variáveis conectado ao Azure Key Vault

Nesta tarefa, você criará um Grupo de Variáveis no Azure DevOps que recuperará o segredo de senha do ACR pelo Key Vault usando a Conexão de Serviço criada anteriormente.

1. No computador do laboratório, inicie um navegador da Web e navegue até o projeto **eShopOnWeb** do Azure DevOps.

1. No painel de navegação vertical do portal do Azure DevOps, selecione **Pipelines > Biblioteca**. Clique em **+ Grupo de Variáveis**.

1. Na folha **Novo grupo de variáveis**, especifique as seguintes configurações:

    | Configuração | Valor |
    | --- | --- |
    | Nome do grupo de variáveis | **eshopweb-vg** |
    | Vincular segredos a partir de um Azure Key Vault | **enable** |
    | Assinatura do Azure | **Conexão de serviço do Azure disponível > Ass do Azure** |
    | Nome do cofre de chaves | O nome do cofre de chaves|

1. Em **Variáveis**, clique em **+ Adicionar** e selecione o segredo **acr-secret**. Clique em **OK**.
1. Clique em **Salvar**.

    ![Captura de tela da criação do grupo de variáveis.](images/vg-create.png)

#### Tarefa 4: configurar o pipeline de CD para implantar o contêiner na ACI (Instância de Contêiner) do Azure

Nesta tarefa, você importará, personalizará e executará um pipeline de CD para implantar a imagem de contêiner criada antes em uma Instância de Contêiner do Azure.

1. No computador do laboratório, inicie um navegador da Web e navegue até o projeto **eShopOnWeb** do Azure DevOps. Vá para **Pipelines > Pipelines** e clique em **Novo Pipeline**.

1. Na janela **Onde está seu código?**, selecione **Git do Azure Repos (YAML)** e selecione o repositório **eShopOnWeb**.

1. Na seção **Configurar**, escolha o **Arquivo YAML existente do Azure Pipelines**. Selecione branch: **principal**, forneça o seguinte caminho **/.ado/eshoponweb-cd-aci.yml** e clique em **Continuar**.

1. Na definição de pipeline YAML, personalize:

    - **YOUR-SUBSCRIPTION-ID** com sua ID de assinatura do Azure.
    - **az400eshop-NAME** substitua NAME para torná-lo globalmente único.
    - **YOUR-ACR.azurecr.io** e **ACR-USERNAME** com seu servidor de login do ACR (ambos precisam do nome do ACR, pode ser revisado no ACR > Chaves de acesso).
    - **AZ400-EWebShop-NAME** pelo nome do grupo de recursos definido antes no laboratório.

1. Clique em **Salvar e Executar**.
1. Abra o pipeline e aguarde a execução.

    > **Importante**: se você vir a mensagem "Este pipeline precisa de permissão para acessar recursos antes que esta execução possa continuar para o Docker Compose para ACI", clique em Exibir, Permitir e Permitir novamente. Isso é necessário para permitir que o pipeline crie o recurso.

    > **Observação**: a implantação pode levar alguns minutos para ser concluída. A definição de CD consiste nas seguintes tarefas:
    - **Recursos**: é preparado para acionar automaticamente com base na conclusão do pipeline de CI. Também faz o download do repositório para o arquivo bicep.
    - **Variáveis (para o estágio Implantar)** se conecta ao grupo de variáveis para consumir o segredo **acr-secret** do Azure Key Vault.
    - **AzureResourceManagerTemplateDeployment** implanta a Instância de Contêiner do Azure (ACI) usando o modelo bicep e fornece os parâmetros de logon do ACR para permitir que a ACI baixe a imagem de contêiner criada anteriormente do Registro de Contêiner do Azure (ACR).

1. Seu pipeline terá um nome com base no nome do projeto. Vamos **renomear** o pipeline para melhor identificá-lo. Vá até **Pipelines > Pipelines** e clique no pipeline criado recentemente. Clique nas reticências e na opção **Renomear/Remover**. Nomeie-o **eshoponweb-cd-aci** e clique em **Salvar**.

   > [!IMPORTANT]
   > Lembre-se de excluir os recursos criados no portal do Azure para evitar cobranças desnecessárias.

## Revisão

Neste laboratório, você integrou o Azure Key Vault a um pipeline do Azure DevOps usando as seguintes etapas:

- Criou um Azure Key Vault para armazenar uma senha do ACR como um segredo.
- Forneceu acesso aos segredos no Azure Key Vault.
- Configurou permissões para ler o segredo.
- Configurou o pipeline para recuperar a senha pelo Azure Key Vault e passá-la para tarefas posteriores.
- Implantou uma imagem de contêiner na ACI (Instância de Contêiner do Azure) usando o segredo.
- Criou um Grupo de Variáveis conectado ao Azure Key Vault.
