---
lab:
  title: Enabling Continuous Integration with Azure Pipelines
  module: 'Module 03: Implement CI with Azure Pipelines and GitHub Actions'
---

# Enabling Continuous Integration with Azure Pipelines

## Manual de laboratório do aluno

## Requisitos do laboratório

- Este laboratório requer o **Microsoft Edge** ou um [navegador com suporte do Azure DevOps.](https://docs.microsoft.com/azure/devops/server/compatibility)

- **Configurar uma organização do Azure DevOps:** se você ainda não tiver uma organização Azure DevOps que possa usar para este laboratório, crie uma seguindo as instruções disponíveis em [Criar uma organização ou coleção de projetos](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization).

## Visão geral do laboratório

Neste laboratório, você aprenderá a definir pipelines de build no Azure DevOps usando YAML.
Os pipelines serão usados em dois cenários:

- Como parte do processo de validação de solicitação de pull.
- Como parte da implementação da integração contínua.

## Objetivos

Após concluir este laboratório, você poderá:

- Incluir a validação de build como parte de uma solicitação de pull.
- Configure o pipeline de CI como código com YAML.

## Tempo estimado: 45 minutos

## Instruções

### Exercício 0: configurar os pré-requisitos do laboratório

Neste exercício, você configurará os pré-requisitos para o laboratório, que consistem em um novo projeto do Azure DevOps com um repositório baseado no [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb).

#### Tarefa 1: (pular se feita) criar e configurar o projeto de equipe

Nesta tarefa, você criará um projeto **eShopOnWeb** do Azure DevOps para ser usado por vários laboratórios.

1. No computador do laboratório, em uma janela do navegador, abra sua organização do Azure DevOps. Clique em **Novo projeto**. Dê ao seu projeto o nome **eShopOnWeb** e deixe os outros campos com padrões. Clique em **Criar**.

#### Tarefa 2: (pular se feita) importar repositório do Git eShopOnWeb

Nesta tarefa, você importará o repositório eShopOnWeb do Git que será usado por vários laboratórios.

1. No computador do laboratório, em uma janela do navegador, abra sua organização do Azure DevOps e o projeto **eShopOnWeb** criado anteriormente. Clique em **Repos>Arquivos**, **Importar um repositório**. Selecione **Importar**. Na janela **Importar um repositório do Git**, cole a seguinte URL https://github.com/MicrosoftLearning/eShopOnWeb.git e clique em **Importar**:

1. O repositório está organizado da seguinte forma:
    - A pasta **.ado** contém os pipelines YAML do Azure DevOps.
    - O contêiner da pasta **.devcontainer** está configurado para o desenvolvimento usando contêineres (localmente no VS Code ou no GitHub Codespaces).
    - A pasta **infra** contém a infraestrutura Bicep e ARM como modelos de código usados em alguns cenários de laboratório.
    - A pasta **.github** contém definições de fluxo de trabalho YAML do GitHub.
    - A pasta **src** contém o site do .NET usado nos cenários do laboratório.

### Exercício 1: Incluir a validação de build como parte de uma solicitação de pull

Neste exercício, você incluirá a validação de compilação para validar uma solicitação de pull.

#### Tarefa 1: Importar a definição de build do YAML

Nesta tarefa, você importará a definição de build YAML que será usada como uma Política de Ramificação para validar as solicitações pull.

Vamos começar importando o pipeline de build chamado [eshoponWeb-ci-pr.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-ci-pr.yml).

1. Vá para **Pipelines>Pipelines**
1. Clique no botão **Criar Pipeline** ou **Novo Pipeline**
1. Selecione **Repositórios Git do Azure (YAML)**
1. Selecione o repositório **eShopOnWeb**
1. Selecione **Arquivo YAML do Azure Pipelines Atual**
1. Selecione o arquivo **/.ado/eshoponWeb-ci-pr.yml** e clique em **Continuar**

    A definição de build é composta pelas seguintes tarefas:
    - **DotNet Restore**: com a Restauração de Pacotes NuGet, você pode instalar todas as dependências de seu projeto sem precisar armazená-las no controle do código-fonte.
    - **DotNet Build**: compila um projeto e todas as suas dependências.
    - **DotNet Test**: driver de teste do .NET usado para executar testes de unidade.
    - **DotNet Publish**: publica o aplicativo e suas dependências em uma pasta para implantação em um sistema de hospedagem. Nesse caso, é o **Build.ArtifactStagingDirectory**.

1. Clique no botão **Salvar** para salvar a definição do pipeline.
1. Seu pipeline terá um nome com base no nome do projeto. Vamos **renomear** o pipeline para melhor identificá-lo. Vá até **Pipelines>Pipelines** e clique no pipeline criado recentemente. Clique nas reticências e na opção **Renomear/Mover**. Nomeio-o como **eshoponWeb-ci-pr** e clique em **Salvar**.

#### Tarefa 2: Políticas de branch

Nesta tarefa, você adicionará políticas à branch principal e só permitirá alterações usando Solicitações Pull que estejam em conformidade com as políticas definidas. Você deseja garantir que as alterações em um branch sejam revisadas antes de serem mescladas.

1. Vá para a seção **Repositório>Branches** .
1. Na guia **Mine** do painel de **Branches**, passe o ponteiro do mouse sobre a entrada do branch **main** para revelar o símbolo de reticências no lado direito.
1. Clique nas reticências e, no menu pop-up, selecione **Políticas de branch**.
1. Na guia **principal** das configurações do repositório, ative a opção **Exigir número mínimo de revisores**. Adicione **1** revisor e marque a caixa **Permitir que os solicitantes aprovem suas próprias alterações** (já que você é o único usuário em seu projeto para o laboratório)
1. Na guia **principal** das configurações do repositório, na seção **Validação de compilação**, clique em **+** (Adicionar nova política de compilação) e na lista **Compilar pipeline**, selecione **eshoponWeb-ci-pr** e clique em **Salvar**.

#### Tarefa 3: Trabalhar com solicitações de pull

Nesta tarefa, você usará o portal do Azure DevOps para criar uma solicitação de pull, usando uma novo branch para mesclar uma alteração no branch **principal** protegida.

1. Navegue até a seção **Repositórios** na navegação do eShopOnWeb e clique em **Branches**.
1. Crie uma novo branch chamada **Feature01** com base no branch** principal**.
1. Clique em **Feature01** e navegue até o arquivo **/eShopOnWeb/src/Web/Program.cs** como parte do branch **Feature01**
1. Clique no botão **Editar** no canto superior direito.
1. Faça a seguinte alteração na primeira linha:

    ```csharp
    // Testing my PR
    ```

1. Clique em **Confirmar > Confirmar** (deixar mensagem de confirmação padrão).
1. Uma mensagem será exibida, propondo a criação de uma solicitação de pull (já que suo branch **Feature01** agora está à frente em alterações, em comparação com a **principal**). Clique em **Criar solicitação de pull**.
1. Na guia **Nova solicitação de pull**, deixe os padrões e clique em **Criar**.
1. A solicitação de pull mostrará alguns requisitos pendentes, com base nas políticas aplicadas à branch principal** de destino**.
    - Pelo menos 1 usuário deve revisar e aprovar as alterações.
    - Validação de compilação, você verá que a compilação **eshoponWeb-ci-pr** foi acionada automaticamente

1. Depois que todas as validações forem bem-sucedidas, no canto superior direito clique em **Aprovar**. Agora, na lista suspensa Definir preenchimento** automático, **você pode clicar em **Concluir**.
1. Na guia **Concluir solicitação de pull**, clique em **Concluir mesclagem**

### Exercício 2: Configurar o pipeline de CI como código com YAML.

Neste exercício você configurará o pipeline de CI como código com YAML.

#### Tarefa 1: Importar a definição de build do YAML para CI

Nesta tarefa, você adicionará a definição de build YAML que será usada para implementar a Integração Contínua.

Vamos começar importando o pipeline de CI chamado [eshoponWeb-ci.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-ci.yml).

1. Vá para **Pipelines>Pipelines**.
1. Clique no botão **Novo pipeline**.
1. Selecione **Git do Azure Repos (YAML)**.
1. Selecione o repositório **eShopOnWeb**.
1. Selecione **Arquivo YAML existente do Azure Pipelines**.
1. Selecione o arquivo **/.ado/eshoponWeb-ci.yml** e clique em **Continuar**.

    A definição de CI é composta pelas seguintes tarefas:
    - **DotNet Restore**: com a Restauração de Pacotes NuGet, você pode instalar todas as dependências de seu projeto sem precisar armazená-las no controle do código-fonte.
    - **DotNet Build**: compila um projeto e todas as suas dependências.
    - **DotNet Test**: driver de teste do .NET usado para executar testes de unidade.
    - **DotNet Publish**: publica o aplicativo e suas dependências em uma pasta para implantação em um sistema de hospedagem. Nesse caso, é o **Build.ArtifactStagingDirectory**.
    - **Publicar artefato - Site**: publique o artefato do aplicativo (criado na etapa anterior) e disponibilize-o como um artefato de pipeline.
    - **Publicar artefato - Bicep**: publique o artefato de infraestrutura (arquivo Bicep) e disponibilize-o como um artefato do pipeline.

#### Tarefa 2: Habilitar a integração contínua

A definição de pipeline de build padrão não habilita a Integração Contínua.

1. Clique no botão **Editar** no canto superior direito.
1. Agora, você precisa substituir as linhas **# trigger:** e **# - main** pelo seguinte código:

    ```YAML
    trigger:
      branches:
        include:
        - main
      paths:
        include:
        - src/web/*
    ```

    Isso disparará automaticamente o pipeline de build se qualquer alteração for feita no branch principal e no código do aplicativo Web (a pasta src/Web).

    Como você habilitou as Políticas de branch, você precisa passar por uma solicitação de pull para atualizar seu código.

1. Clique no botão **Salvar** (não em **Salvar e executar**) para salvar a definição de pipeline.
1. Selecione **Criar um novo branch para este commit**.
1. Mantenha o nome do branch padrão e **Iniciar uma solicitação de pull** verificada.
1. Clique em **Salvar**.
1. Seu pipeline terá um nome com base no nome do projeto. Vamos **renomear** o pipeline para melhor identificá-lo. Vá até **Pipelines>Pipelines** e clique no pipeline criado recentemente. Clique nas reticências e na opção **Renomear/Mover**. Nomeie-o **eshoponweb-ci** e clique em **Salvar**.
1. Vá para **Repositórios > Pull requests**.
1. Clique na no pull request **"Atualizar o eshoponweb-ci.yml para o Azure Pipelines"**.
1. Depois que todas as validações forem bem-sucedidas, no canto superior direito clique em **Aprovar**. Agora você pode clicar em **Concluir**.
1. Na guia **Concluir solicitação de pull**, clique em **Concluir mesclagem**

#### Tarefa 3: Testar o pipeline de CI

Nesta tarefa, você criará uma solicitação de pull usando uma novo branch para mesclar uma alteração no branch **principal** protegida e acionar automaticamente o pipeline de CI.

1. Navegue até a seção **Repos**.
1. Crie um novo branch chamado **Feature02** com base no **branch principal**.
1. Clique no novo branch **Feature02**.
1. Navegue até o arquivo **/eShopOnWeb/src/Web/Program.cs** e clique em **Editar** no canto superior direito.
1. Remova a primeira linha:

    ```csharp
    // Testing my PR
    ```

1. Clique em **Confirmar > Confirmar** (deixar mensagem de confirmação padrão).
1. Uma mensagem será exibida, propondo a criação de uma solicitação de pull (já que suo branch do **Feature02** agora está à frente em alterações, em comparação com a **principal**).
1. Clique em **Criar solicitação de Pull**.
1. Na guia **Nova solicitação de pull**, deixe os padrões e clique em **Criar**.
1. A solicitação de pull mostrará alguns requisitos pendentes, com base nas políticas aplicadas à branch principal** de destino**.
1. Depois que todas as validações forem bem-sucedidas, no canto superior direito clique em **Aprovar**. Agora, na lista suspensa Definir preenchimento** automático, **você pode clicar em **Concluir**.
1. Na guia **Concluir solicitação de pull**, clique em **Concluir mesclagem**
1. Volte para **Pipelines>Pipelines**, você notará que a compilação **eshoponWeb-ci** foi acionada automaticamente depois que o código foi mesclado.
1. Clique na **compilação eshoponWeb-ci** e selecione a última execução.
1. Após sua execução bem-sucedida, clique em **Related > Published** para verificar os artefatos publicados:
    - Bíceps: o artefato de infraestrutura.
    - Site: o artefato do aplicativo.

## Revisão

Neste laboratório, você habilitou a validação de solicitação de pull usando uma definição de build e configurou o pipeline de CI como código com YAML no Azure DevOps.
