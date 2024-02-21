---
lab:
  title: Configurar e executar testes
  module: 'Module 05: Implement a secure continuous deployment using Azure Pipelines'
---

# Configurar e executar testes

## Manual de laboratório do aluno

## Requisitos do laboratório

- Este laboratório requer o **Microsoft Edge** ou um [navegador com suporte do Azure DevOps.](https://docs.microsoft.com/azure/devops/server/compatibility)

- **Configurar uma organização do Azure DevOps:** se você ainda não tiver uma organização Azure DevOps que possa usar para este laboratório, crie uma seguindo as instruções disponíveis em [Criar uma organização ou coleção de projetos](https://learn.microsoft.com/dotnet/architecture/modern-web-apps-azure/test-asp-net-core-mvc-apps).

## Visão geral do laboratório

Um software de qualquer complexidade pode falhar de maneiras inesperadas em resposta a alterações. Portanto, depois de fazer alterações, é necessário realizar um teste em todos os aplicativos, exceto os mais triviais (ou menos críticos). O teste manual é a maneira mais lenta, menos confiável e mais cara de testar um software.

Há muitos tipos de testes automatizados para aplicativos de software. O teste mais simples de nível mais baixo é o teste de unidade. Em um nível ligeiramente superior, há testes de integração e testes funcionais. Outros tipos de teste, como os de interface do usuário, de carga, de estresse e smoke tests, estão além do escopo deste laboratório.

*Se você quiser saber mais sobre os diferentes tipos de teste, recomendamos a leitura deste artigo: [Testar aplicativos ASP.NET Core MVC](https://learn.microsoft.com/dotnet/architecture/modern-web-apps-azure/test-asp-net-core-mvc-apps).*

## Objetivos

Depois de concluir este laboratório, você poderá configurar um pipeline de CI para um aplicativo .Net que inclui:

- Testes de Unidade
- Testes de integração
- Testes funcionais

## Tempo estimado: 60 minutos

## Instruções

### Exercício 0: configurar os pré-requisitos do laboratório

Neste exercício, você configurará os pré-requisitos para o laboratório, que consistem em um novo projeto do Azure DevOps com um repositório baseado no [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb).

#### Tarefa 1: (pular se feita) criar e configurar o projeto de equipe

Nesta tarefa, você criará um projeto **eShopOnWeb** do Azure DevOps para ser usado por vários laboratórios.

1. No computador do laboratório, em uma janela do navegador, abra sua organização do Azure DevOps. Clique em **Novo projeto**. Dê ao seu projeto o nome **eShopOnWeb** e deixe os outros campos com padrões. Clique em **Criar**.

#### Tarefa 2: (ignorar se concluído) importar repositório Git do eShopOnWeb

Nesta tarefa, você importará o repositório Git do eShopOnWeb que será usado por vários laboratórios.

1. No computador do laboratório, em uma janela do navegador, abra sua organização do Azure DevOps e o projeto **eShopOnWeb** criado anteriormente. Clique em **Repos>Arquivos**, **Importar um repositório**. Selecione **Importar**. Na janela **Importar um repositório do Git**, cole a seguinte URL <https://github.com/MicrosoftLearning/eShopOnWeb.git> e clique em **Importar**:

1. O repositório está organizado da seguinte forma:
    - A pasta **.ado** contém os pipelines YAML do Azure DevOps.
    - O contêiner da pasta **.devcontainer** está configurado para o desenvolvimento usando contêineres (localmente no VS Code ou no GitHub Codespaces).
    - A pasta **infra** contém a infraestrutura Bicep e ARM como modelos de código usados em alguns cenários de laboratório.
    - A pasta **.github** contém definições de fluxo de trabalho YAML do GitHub.
    - A pasta **src** contém o site .NET usado nos cenários do laboratório.

### Exercício 1: configurar testes no pipeline de CI

Neste exercício, você configurará testes no pipeline de CI.

#### Tarefa 1: (ignorar se concluído) importar a definição de compilação YAML para CI

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
1. Clique no botão **Salvar** (não em **Salvar e executar**) para salvar a definição de pipeline.

#### Tarefa 2: adicionar testes ao pipeline de CI

Nesta tarefa, você adicionará os testes funcionais e de integração ao pipeline de CI.

Você pode notar que a tarefa Testes de Unidade já faz parte do pipeline.

- Os **testes de unidade** testam uma única parte da lógica do aplicativo. Ainda podemos descrevê-lo listando algumas das coisas que ele não é. Um teste de unidade não testa como o código funciona com as dependências ou a infraestrutura – é para isso que servem os testes de integração.

1. Agora você precisa adicionar a tarefa Testes de Integração após a tarefa Testes de Unidade:

    ```YAML
    - task: DotNetCoreCLI@2
      displayName: Integration Tests
      inputs:
        command: 'test'
        projects: 'tests/IntegrationTests/*.csproj'
    ```

    > Os **testes de integração** testam como seu código funciona com dependências ou infraestrutura. Embora seja uma boa ideia encapsular o código que interage com a infraestrutura, como bancos de dados e sistemas de arquivos, você ainda terá uma parte do código e, provavelmente, desejará testá-lo. Além disso, você deve verificar se as camadas do código interagem conforme esperado quando as dependências do aplicativo são totalmente resolvidas. Essa funcionalidade é de responsabilidade dos testes de integração.

1. Em seguida, você precisa adicionar a tarefa Testes de funcionais após a tarefa Testes de Unidade:

    ```YAML
    - task: DotNetCoreCLI@2
      displayName: Functional Tests
      inputs:
        command: 'test'
        projects: 'tests/FunctionalTests/*.csproj'
    ```

    > Os **Testes funcionais** são escritos da perspectiva do usuário e verificam a correção do sistema com base em seus requisitos. Diferente dos testes de integração que são escritos da perspectiva do desenvolvedor, para verificar se alguns componentes do sistema funcionam corretamente juntos.

1. Clique em **Salvar**, no painel **Salvar**, clique em **Salva**r novamente para confirmar as mudanças diretamente no branch principal.

#### Tarefa 4: verificar o resumo dos testes

1. Clique em **Executar** e, na guia **Executar pipeline**, clique em **Executar** novamente.

1. Aguarde até que o pipeline seja iniciado e até que ele conclua o estágio de compilação com êxito.

1. Depois de concluído, a guia **Teste** será exibida como parte da execução de pipeline. Clique nela para conferir o resumo. É semelhante ao mostrado abaixo:

    ![Resumo dos testes](images/AZ400_M05_L09_Tests_Summary.png)

1. Para obter mais detalhes, na parte inferior da página, a tabela mostra uma lista dos diferentes testes de execução.

    >**Observação**: se a tabela estiver vazia, você precisará redefinir os filtros para ter todos os detalhes sobre a execução dos testes.

    ![Tabela de testes](images/AZ400_M05_L09_Tests_Table.png)

## Revisão

Neste laboratório, você aprendeu a configurar e executar diferentes tipos de teste usando Azure Pipelines e .Net.
