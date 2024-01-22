---
lab:
  title: Integrar o Azure Key Vault ao Azure DevOps
  module: 'Module 05: Implement a secure continuous deployment using Azure Pipelines'
---

# Integrar o Azure Key Vault ao Azure DevOps

## Manual de laboratório do aluno

## Requisitos do laboratório

- Este laboratório requer o **Microsoft Edge** ou um [navegador com suporte do Azure DevOps.](https://learn.microsoft.com/azure/devops/server/compatibility)

- **Configurar uma organização do Azure DevOps:** se você ainda não tiver uma organização Azure DevOps que possa usar para este laboratório, crie uma seguindo as instruções disponíveis em [Criar uma organização ou coleção de projetos](https://learn.microsoft.com/azure/devops/organizations/accounts/create-organization).

- Identifique uma assinatura existente do Azure ou crie uma.

## Visão geral do laboratório

O Azure Key Vault fornece armazenamento seguro e gerenciamento de dados confidenciais, como chaves, senhas e certificados. O Azure Key Vault inclui suporte para módulos de segurança de hardware e uma variedade de algoritmos de criptografia e comprimentos de chave. O uso do Azure Key Vault pode minimizar a possibilidade de divulgar dados confidenciais por meio do código-fonte, o que é um erro comum cometido pelos desenvolvedores. O acesso ao Azure Key Vault requer autenticação e autorização adequadas, dando suporte a permissões refinadas para seu conteúdo.

Neste laboratório, você verá como integrar o Azure Key Vault a um Pipeline do Azure usando as seguintes etapas:

- Criar um Azure Key Vault para armazenar uma senha do ACR como um segredo.
- Criar uma entidade de serviço do Azure para fornecer acesso aos segredos do Azure Key Vault.
- Configurar permissões para permitir que a entidade de serviço leia o segredo.
- Configurar o pipeline para recuperar a senha do Azure Key Vault e passá-la para tarefas posteriores.

## Objetivos

Após concluir este laboratório, você poderá:

- Crie uma entidade de serviço do Microsoft Entra.
- Crie um Azure Key Vault.

## Tempo estimado: 40 minutos

## Instruções

### Exercício 0: configurar os pré-requisitos do laboratório

Neste exercício, você configurará os pré-requisitos para o laboratório, que consistem em um novo projeto do Azure DevOps com um repositório baseado no [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb).

#### Tarefa 1: (pular se feita) criar e configurar o projeto de equipe

Nesta tarefa, você criará um projeto **eShopOnWeb** do Azure DevOps para ser usado por vários laboratórios.

1. No computador do laboratório, em uma janela do navegador, abra sua organização do Azure DevOps. Clique em **Novo projeto**. Dê ao seu projeto o nome **eShopOnWeb** e deixe os outros campos com padrões. Clique em **Criar**.

    ![Criar Projeto](images/create-project.png)

#### Tarefa 2: (pular se feita) importar repositório do Git eShopOnWeb

Nesta tarefa, você importará o repositório eShopOnWeb do Git que será usado por vários laboratórios.

1. No computador do laboratório, em uma janela do navegador, abra sua organização do Azure DevOps e o projeto **eShopOnWeb** criado anteriormente. Clique em **Repos>Arquivos** , **Importar**. Na janela **Importar um repositório do Git**, cole a URL https://github.com/MicrosoftLearning/eShopOnWeb.git e clique em **Importar**:

    ![Importar repositório](images/import-repo.png)

2. O repositório está organizado da seguinte forma:
    - A pasta **.ado** contém os pipelines YAML do Azure DevOps.
    - O contêiner da pasta **.devcontainer** está configurado para o desenvolvimento usando contêineres (localmente no VS Code ou no GitHub Codespaces).
    - A pasta **.azure** contém a infraestrutura Bicep&ARM como modelos de código usados em alguns cenários de laboratório.
    - A pasta **.github** contém definições de YAML do fluxo de trabalho do GitHub.
    - A pasta **src** contém o site do .NET 6 usado nos cenários do laboratório.

### Exercício 1: configurar o pipeline de CI para criar um contêiner eShopOnWeb

Configurar o pipeline YAML de CI para:

- Criar um Registro de Contêiner do Azure para manter as images do contêiner
- Usar o Docker Compose para criar e enviar por push as  imagens de contêiner **eshoppublicapi** e **eshopwebmvc**. Somente o contêiner **eshopwebmvc** será implantado.

#### Tarefa 1: (pular se feita) criar uma entidade de serviço

Nesta tarefa, você criará uma entidade de serviço usando a CLI do Azure, que permitirá ao Azure DevOps:

- Implantar recursos na assinatura do Azure
- Ter acesso de leitura sobre os segredos do Key Vault criados posteriormente.

> **Observação**: se você já tiver uma entidade de serviço, poderá prosseguir diretamente para a próxima tarefa.

Você precisará de uma entidade de serviço para implantar recursos do Azure a partir do Azure Pipelines. Como vamos recuperar segredos em um pipeline, precisaremos conceder permissão ao serviço quando criarmos o Azure Key Vault.

Uma entidade de serviço é criada automaticamente pelo Azure Pipelines quando você se conecta a uma assinatura do Azure de dentro de uma definição de pipeline ou quando cria uma nova Conexão de Serviço na página de configurações do projeto (opção automática). Você também pode criar manualmente a entidade de serviço a partir do portal ou usando a CLI do Azure e reutilizá-la em projetos.

1. No computador do laboratório, inicie um navegador da Web, navegue até o [**Portal do Azure**](https://portal.azure.com) e entre com a conta de usuário que tem a função Proprietário na assinatura do Azure que você usará neste laboratório e tem a função de Administrador global no locatário do Microsoft Entra associado a essa assinatura.
2. No portal do Azure, clique no ícone do **Cloud Shell**, localizado diretamente à direita da caixa de texto de pesquisa na parte superior da página.
3. Se for solicitado que você selecione **Bash** ou **PowerShell**, selecione **Bash**.

   >**Observação**: se esta for a primeira vez que você está iniciando o **Cloud Shell** e você receber a mensagem **Você não tem nenhum armazenamento montado**, selecione a assinatura que você está usando no laboratório e selecione **Criar armazenamento**.

4. No prompt **Bash**, no painel **Cloud Shell**, execute os seguintes comandos para recuperar os valores da ID de assinatura do Azure e dos atributos de nome de assinatura:

    ```bash
    az account show --query id --output tsv
    az account show --query name --output tsv
    ```

    > **Observação**: copie ambos os valores para um arquivo de texto. Você precisará deles adiante neste laboratório.

5. No prompt **Bash**, no painel **Cloud Shell**, execute o seguinte comando para criar uma entidade de serviço (substitua **myServicePrincipalName** por qualquer cadeia de caracteres exclusiva que consista em letras e dígitos) e **mySubscriptionID** pela sua subscriptionId do Azure:

    ```bash
    az ad sp create-for-rbac --name myServicePrincipalName \
                         --role contributor \
                         --scopes /subscriptions/mySubscriptionID
    ```

    > **Observação**: o comando irá gerar uma saída JSON. Copie a saída para um arquivo de texto. Você precisará dela posteriormente neste laboratório.

6. Em seguida, no computador do laboratório, inicie um navegador da Web, navegue até o projeto ** eShopOnWeb** do Azure DevOps. Clique em **Configurações do Projeto>Conexões de Serviço (em Pipelines)** e **Nova Conexão de Serviço**.

    ![Nova conexão de serviço](images/new-service-connection.png)

    > **Observação**: Se não houver conexões de serviço criadas anteriormente na página, o botão de criação da conexão de serviço estará localizado no centro da página e terá o rótulo **Criar conexão de serviço**

7. Na folha **Nova conexão de serviço**, escolha **Azure Resource Manager** e **Avançar** (talvez seja necessário rolar para baixo).

8. Em seguida, escolha a **Entidade de serviço (manual)** e clique em **Avançar**.

9. Preencha os campos vazios usando as informações coletadas durante as etapas anteriores:
    - ID e nome da assinatura.
    - ID da entidade de serviço (appId), chave da entidade de serviço (senha) e ID do locatário (locatário).
    - Em **Nome da conexão de serviço**, digite **azure subs**. Esse nome será referenciado em pipelines YAML quando precisar de uma Conexão de Serviço do Azure DevOps para se comunicar com sua assinatura do Azure.

    ![Conexão de serviço do Azure](images/azure-service-connection.png)

10. Clique em **Verificar e Salvar**.

#### Tarefa 2: configurar e executar o pipeline de CI

Nesta tarefa, você importará, modificará e executará uma definição de pipeline YAML de CI existente. Será criado um novo Registro de Contêiner do Azure (ACR) e criará/publicará as imagens de contêiner eShopOnWeb.

1. No computador do laboratório, inicie um navegador da Web e navegue até o projeto **eShopOnWeb** do Azure DevOps. Vá para **Pipelines>Pipelines** e clique em **Criar Pipeline** (ou **Novo pipeline**).

2. Na janela **Onde está seu código?**, selecione **Git do Azure Repos (YAML)** e selecione o repositório **eShopOnWeb**.

3. Na seção **Configurar**, escolha o **Arquivo YAML existente do Azure Pipelines**. Forneça o seguinte caminho **/.ado/eshoponweb-ci-dockercompose.yml** e clique em **Continuar**.

    ![Selecionar Pipeline](images/select-ci-container-compose.png)

4. Na definição de pipeline do YAML, personalize o nome do seu Grupo de Recursos substituindo **NAME** em **AZ400-EWebShop-NAME** e substitua **YOUR-SUBSCRIPTION-ID** pelo seu subscriptionId do Azure.

5. Clique em **Salvar e Executar** e aguarde até que o pipeline seja executado com êxito.

    > **Observação**: o build poderá levar alguns minutos para ser concluído. A definição de build consiste nas seguintes tarefas:
    - **AzureResourceManagerTemplateDeployment** usa **bicep** para implantar um Registro de Contêiner do Azure.
    - A tarefa **PowerShell** obtém a saída do bicep (servidor de logon do acr) e cria a variável de pipeline.
    - A tarefa **DockerCompose** cria e envia as imagens de contêiner do eShopOnWeb para o Registro de Contêiner do Azure.

6. Seu pipeline terá um nome com base no nome do projeto. Vamos **renomear** o pipeline para melhor identificá-lo. Vá até **Pipelines>Pipelines** e clique no pipeline criado recentemente. Clique nas reticências e na opção **Renomear/Remover**. Nomeie-o **eshoponweb-ci-dockercompose** e clique em **Salvar**.

7. Quando a execução for concluída, no Portal do Azure, abra o Grupo de Recursos definido anteriormente e você encontrará um Registro de Contêiner do Azure (ACR) com as imagens de contêiner criadas **eshoppublicapi** e **eshopwebmvc**. Você só usará **eshopwebmvc** na fase de implantação.

    ![Imagens de contêiner no ACR](images/azure-container-registry.png)

8. Clique em **Chaves de Acesso** e copie o valor da **senha**, ele será usado na tarefa a seguir, pois o manteremos como um segredo no Azure Key Vault.

    ![Senha do ACR](images/acr-password.png)

#### Tarefa 2: criar um Azure Key Vault

Nesta tarefa, você criará um Azure Key Vault usando o portal do Azure.

Para esse cenário de laboratório, teremos uma ACI (Instância de Contêiner) do Azure que extrai e executa uma imagem de contêiner armazenada no Registro de Contêiner do Azure (ACR). Pretendemos armazenar a senha do ACR como um segredo no Key Vault.

1. No portal do Azure, na caixa de texto **Pesquisar recursos, serviços e documentos**, digite **Key Vault** e pressione a tecla **Enter**.
2. Selecione a folha **Key Vault**, clique em **Criar>Key Vault**.
3. Na guia **Noções básicas** da folha **Criar um Key Vault**, especifique as seguintes configurações e clique em **Avançar**:

    | Configuração | Valor |
    | --- | --- |
    | Assinatura | o nome da assinatura do Azure que você está usando neste laboratório |
    | Grupo de recursos | o nome de um novo grupo de recursos **AZ400-EWebShop-NAME** |
    | Nome do cofre de chaves | qualquer nome válido exclusivo, como **ewebshop-kv-NAME (substitua NAME** ) |
    | Região | uma região do Azure próxima do local do seu ambiente de laboratório |
    | Tipo de preço | **Standard** |
    | Dias de retenção dos cofres excluídos | **7** |
    | Proteção contra limpeza | **Desabilitar proteção contra limpeza** |

4. Na guia **Configuração de acesso** da folha **Criar um Key Vault**, selecione **Política de acesso do Vault** e, na seção **Políticas de acesso**, clique em **+ Criar** para configurar uma nova política.

    > **Observação**: você precisa proteger o acesso aos cofres de chaves permitindo apenas aplicativos e usuários autorizados. Para acessar os dados no cofre, você precisará fornecer permissões de leitura (Obter/Listar) para a entidade de serviço criada anteriormente que será usada para autenticação no pipeline. 

    1. Na folha **Permissão**, abaixo de **Permissões de segredo**, marque as permissões **Obter** e **Listar** . Clique em **Avançar**.
    2. Na folha **Entidade**, procure a **Entidade de Serviço criada anteriormente** usando a ID ou o Nome fornecido, e selecione-a na lista. Clique em **Avançar**, **Avançar**, **Criar** (política de acesso).
    3. Na folha **Revisar + criar**, clique em **Criar**.

5. De volta à folha **Criar um Key Vault**, clique em **Revisar + Criar > Criar**

    > **Observação**: aguarde até que o Azure Key Vault seja provisionado. Isso deverá levar menos de 1 minuto.

6. Na folha **A implantação foi concluída**, clique em **Ir para o recurso**.
7. Na folha Azure Key Vault (ewebshop-kv-NAME), no menu vertical no lado esquerdo da folha, na seção **Objetos**, clique em **Segredos**.
8. Na folha **Segredos**, clique em **Gerar/Importar**.
9. Na folha **Criar um segredo**, especifique as seguintes configurações e clique em **Criar** (deixe as demais com seus valores padrão):

    | Configuração | Valor |
    | --- | --- |
    | Opções de upload | **Manual** |
    | Nome | **acr-secret** |
    | Valor | Senha de acesso do ACR copiada na tarefa anterior |

#### Tarefa 3: criar um grupo de variáveis conectado ao Azure Key Vault

Nesta tarefa, você criará um Grupo de Variáveis no Azure DevOps que recuperará o segredo de senha do ACR pelo Key Vault usando a Conexão de Serviço (entidade de serviço).

1. No computador do laboratório, inicie um navegador da Web e navegue até o projeto **eShopOnWeb** do Azure DevOps.

2. No painel de navegação vertical do portal do Azure DevOps, selecione **Pipelines>Biblioteca**. Clique em **+ Grupo de Variáveis**.

3. Na folha **Novo grupo de variáveis**, especifique as seguintes configurações:

    | Configuração | Valor |
    | --- | --- |
    | Nome do grupo de variáveis | **eshopweb-vg** |
    | Vincular segredos a partir de um Azure Key Vault | **enable** |
    | Assinatura do Azure | **Conexão de serviço do Azure disponível > Ass do Azure** |
    | Nome do cofre de chaves | O nome do cofre de chaves|

4. Em **Variáveis**, clique em **+ Adicionar** e selecione o segredo **acr-secret**. Clique em **OK**.
5. Clique em **Salvar**.

    ![Criar grupo de variáveis](images/vg-create.png)

#### Tarefa 4: configurar o pipeline de CD para implantar o contêiner na ACI (Instância de Contêiner) do Azure

Nesta tarefa, você importará, personalizará e executará um pipeline de CD para implantar a imagem de contêiner criada antes em uma Instância de Contêiner do Azure.

1. No computador do laboratório, inicie um navegador da Web e navegue até o projeto **eShopOnWeb** do Azure DevOps. Vá para **Pipelines>Pipelines** e clique em **Novo Pipeline**.

2. Na janela **Onde está seu código?**, selecione **Git do Azure Repos (YAML)** e selecione o repositório **eShopOnWeb**.

3. Na seção **Configurar**, escolha o **Arquivo YAML existente do Azure Pipelines**. Forneça o seguinte caminho **/.ado/eshoponweb-cd-aci.yml** e clique em **Continuar**.

4. Na definição de pipeline YAML, personalize:

    - **YOUR-SUBSCRIPTION-ID** com sua ID de assinatura do Azure.
    - **az400eshop-NAME** substitua NAMEpara torná-lo globalmente único.
    - **YOUR-ACR.azurecr.io** e **ACR-USERNAME** com seu servidor de login do ACR (ambos precisam do nome do ACR, pode ser encontrado em ACR>Chaves de Acesso).
    - **AZ400-EWebShop-NAME** pelo nome do grupo de recursos definido antes no laboratório.

5. Clique em **Salvar e Executar**.
6. Abra o pipeline e aguarde a execução.

    > **Importante**: se você vir a mensagem "Este pipeline precisa de permissão para acessar recursos antes que esta execução possa continuar para o Docker Compose para ACI", clique em Exibir, Permitir e Permitir novamente. Isso é necessário para permitir que o pipeline crie o recurso.

    > **Observação**: a implantação pode levar alguns minutos para ser concluída. A definição de CD consiste nas seguintes tarefas:
    - **Recursos**: é preparado para acionar automaticamente com base na conclusão do pipeline de CI. Também faz o download do repositório para o arquivo bicep.
    - **Variáveis (para o estágio Implantar)** se conecta ao grupo de variáveis para consumir o segredo **acr-secret** do Azure Key Vault.
    - **AzureResourceManagerTemplateDeployment** implanta a Instância de Contêiner do Azure (ACI) usando o modelo bicep e fornece os parâmetros de logon do ACR para permitir que a ACI baixe a imagem de contêiner criada anteriormente do Registro de Contêiner do Azure (ACR).

7. Seu pipeline terá um nome com base no nome do projeto. Vamos **renomear** o pipeline para melhor identificá-lo. Vá até **Pipelines>Pipelines** e clique no pipeline criado recentemente. Clique nas reticências e na opção **Renomear/Remover**. Nomeie-o **eshoponweb-cd-aci** e clique em **Salvar**.

### Exercício 2: remover os recursos do laboratório do Azure

Neste exercício, você removerá os recursos do Azure provisionados neste laboratório para eliminar cobranças inesperadas.

>**Observação**: lembre-se de remover todos os recursos do Azure que acabam de ser criados e que você não usa mais. Remover recursos não utilizados garante que você não veja encargos inesperados.

#### Tarefa 1: remover os recursos do laboratório do Azure

Nesta tarefa, você usará o Azure Cloud Shell para remover os recursos do Azure provisionados neste laboratório para eliminar cobranças desnecessárias.

1. No portal do Azure, abra o Grupo de recursos criado e clique em **Excluir grupo de recursos**.

## Revisão

Neste laboratório, você integrou o Azure Key Vault a um pipeline do Azure DevOps usando as seguintes etapas:

- Criou uma entidade de serviço do Azure para fornecer acesso a um segredo do Azure Key Vault e autenticar a implantação no Azure a partir do Azure DevOps.
- Executou dois pipelines YAML importados de um repositório do Git.
- Configurou um pipeline para recuperar a senha pelo Azure Key Vault usando um Grupo de Variáveis e usá-la em tarefas subsequentes.
