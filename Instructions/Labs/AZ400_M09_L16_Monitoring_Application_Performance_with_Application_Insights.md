---
lab:
  title: Monitorar o desempenho do aplicativo com o Teste de Carga do Azure
  module: 'Module 09: Implement continuous feedback'
---

# Monitorar o desempenho do aplicativo com o Application Insights e o Teste de Carga do Azure

## Manual de laboratório do aluno

## Requisitos do laboratório

- Este laboratório requer o **Microsoft Edge** ou um [navegador compatível com o Azure DevOps](https://docs.microsoft.com/azure/devops/server/compatibility).

- **Configurar uma organização do Azure DevOps:** se você ainda não tiver uma organização Azure DevOps que possa usar para este laboratório, crie uma seguindo as instruções disponíveis em [Criar uma organização ou coleção de projetos](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization).

- Identifique uma assinatura existente do Azure ou crie uma.

- Verifique se você tem uma conta Microsoft ou uma conta do Microsoft Entra com a função Proprietário na assinatura do Azure e a função Administrador Global no locatário do Microsoft Entra associado à assinatura do Azure. Para obter detalhes, veja [Listar atribuições de função do Azure usando o portal do Azure](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-list-portal) e [Exibir e atribuir funções de administrador no Azure Active Directory](https://docs.microsoft.com/azure/active-directory/roles/manage-roles-portal#view-my-roles).

## Visão geral do laboratório

O **Teste de Carga do Azure** é um serviço de teste de carga totalmente gerenciado que permite gerar cargas de alta escala. O serviço simula o tráfego para seus aplicativos, independentemente de onde estão hospedados. Desenvolvedores, testadores e engenheiros de QA (garantia de qualidade) podem usá-lo para otimizar o desempenho, a escalabilidade ou a capacidade do aplicativo.
Crie rapidamente um teste de carga para seu aplicativo Web usando uma URL e sem conhecimento prévio das ferramentas de teste. O Teste de Carga do Azure abstrai a complexidade e a infraestrutura para executar seu teste de carga em escala.
Para cenários mais avançados de teste de carga, você pode criar um teste de carga reutilizando um script de teste do Apache JMeter existente, uma ferramenta de desempenho e carga de código aberto popular. Por exemplo, o plano de teste pode consistir em várias solicitações de aplicativo, você deseja chamar pontos de extremidade que não sejam HTTP ou está usando dados e parâmetros de entrada para que o teste fique mais dinâmico.

Neste laboratório, você aprenderá sobre como usar o Teste de Carga do Azure para simular o teste de desempenho em um aplicativo Web em execução ao vivo com diferentes cenários de carga. Por fim, você aprenderá a integrar o Teste de Carga do Azure em seus pipelines de CI/CD. 

## Objetivos

Após concluir este Laboratório, você poderá:

- Implantar aplicativos Web do Azure App Service.
- Compor e executar um pipeline de CI/CD baseado em YAML.
- Implante o Teste de carga do Azure.
- Investigar o desempenho do aplicativo Web do Azure usando o Teste de Carga do Azure.
- Integre o Teste de Carga do Azure em seus pipelines de CI/CD.

## Tempo estimado: 60 minutos

## Instruções

### Exercício 0: configurar os pré-requisitos do laboratório

Neste exercício, você configurará os pré-requisitos para o laboratório, que consistem em um novo projeto do Azure DevOps com um repositório baseado no [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb).

#### Tarefa 1: (ignorar se concluído) Criar e configurar o projeto de equipe

Nesta tarefa, você criará um projeto de **eShopOnWeb** Azure DevOps para ser usado por vários laboratórios.

1. No computador do laboratório, em uma janela do navegador, abra sua organização do Azure DevOps. Clique em **Novo projeto**. Dê ao seu projeto o nome **eShopOnWeb** e escolha **Scrum** na lista suspensa Processo de **item** de trabalho. Clique em **Criar**.

    ![Criar Projeto](images/create-project.png)

#### Tarefa 2: (ignorar se concluído) Importar repositório Git do eShopOnWeb

Nesta tarefa, você criará um projeto de eShopOnWeb Git para ser usado por vários laboratórios.

1. No computador do laboratório, em uma janela do navegador, abra sua organização do Azure DevOps e o projeto **eShopOnWeb** anteriormente criado. Clique em **Repos>Files** , **Importar**. Na janela**Importar um repositório Git**, cole a seguinte URL https://github.com/MicrosoftLearning/eShopOnWeb.git e clique em **Importar**:

    ![Importar repositório](images/import-repo.png)

2. O repositório está organizado da seguinte forma:
    - A pasta **.ado** contém os pipelines YAML do Azure DevOps
    - O contêiner da pasta **.devcontainer** está configurado para o desenvolvimento usando contêineres (localmente no VS Code ou no GitHub Codespaces)
    - A pasta **.azure** contém a infraestrutura Bicep&ARM como modelos de código usados em alguns cenários do laboratório.
    - Definições de fluxo de trabalho do YAML GitHub do contêiner de pasta **.github**.
    - A pasta **src** contém o site .NET 6 usado nos cenários do laboratório.

#### Tarefa 2: criar recursos do Azure

Nesta tarefa, você criará um aplicativo Web do Azure usando o shell de nuvem no portal do Azure.

1. No computador do laboratório, abra um navegador. Navegue até o [**Portal do Azure**](https://portal.azure.com), entre com a conta de usuário que tem a função Proprietário na assinatura do Azure que você usará neste laboratório e tem a função de Administrador Global no locatário do Microsoft Entra associado a essa assinatura.
2. No portal do Azure, na barra de ferramentas, selecione o ícone **Cloud Shell** localizado à direita da caixa de pesquisa.
3. Se for solicitado que você selecione **Bash** ou **PowerShell**, selecione **Bash**.
    >**Observação**: se esta for a primeira vez que você está iniciando o **Cloud Shell ** e você receber a mensagem **Você não tem nenhum armazenamento montado**, selecione a assinatura que você está usando no laboratório e selecione **Criar armazenamento**.

4. No ** prompt Bash**, no painel **Cloud Shell**, execute o seguinte comando para criar um grupo de recursos (substitua o `<region>` espaço reservado pelo nome da região do Azure mais próxima de você, como 'eastus').

    ```bash
    RESOURCEGROUPNAME='az400m09l16-RG'
    LOCATION='<region>'
    az group create --name $RESOURCEGROUPNAME --location $LOCATION
    ```

5. Execute o plano de serviços do Aplicativo do Windows executando o seguinte comando:

    ```bash
    SERVICEPLANNAME='az400l16-sp'
    az appservice plan create --resource-group $RESOURCEGROUPNAME \
        --name $SERVICEPLANNAME --sku B3 
    ```

6. Crie um aplicativo Web com um nome exclusivo.

    ```bash
    WEBAPPNAME=partsunlimited$RANDOM$RANDOM
    az webapp create --resource-group $RESOURCEGROUPNAME --plan $SERVICEPLANNAME --name $WEBAPPNAME 
    ```

    > **Observação**: registre o nome do aplicativo Web. Você precisará delas mais adiante neste laboratório.

### Exercício 1: Configurar pipelines de CI/CD como código com YAML no Azure DevOps.

Neste exercício, você configurará pipelines de CI/CD como código com YAML no Azure DevOps.

#### Tarefa 1: Adicionar uma definição de compilação e implantação do YAML

Nesta tarefa, você adicionará uma definição de compilação YAML ao projeto existente.

1. Navegue de volta para o painel **Pipelines** no **hub Pipelines**.
2. Na janela **Criar seu primeiro pipeline**, clique em **Criar pipeline**.

    > **Observação**: usaremos o assistente para criar uma nova definição de Pipeline YAML com base em nosso projeto.

3. No painel **Onde está seu código?**, clique na opção **Azure Repos Git (YAML)**.
4. No painel **Selecionar um repositório**, clique em **eShopOnWeb**.
5. Na página **Configurar seu pipeline**, selecione **Pipeline inicial**.
6. **Selecione** todas as linhas do Pipeline inicial e exclua-as.
7. **Copie** o pipeline de modelo completo abaixo, sabendo que você precisará fazer modificações **de parâmetro antes de salvar** as alterações:

```
#Template Pipeline for CI/CD 
# trigger:
# - main

resources:
  repositories:
    - repository: self
      trigger: none

stages:
- stage: Build
  displayName: Build .Net Core Solution
  jobs:
  - job: Build
    pool:
      vmImage: ubuntu-latest
    steps:
    - task: DotNetCoreCLI@2
      displayName: Restore
      inputs:
        command: 'restore'
        projects: '**/*.sln'
        feedsToUse: 'select'

    - task: DotNetCoreCLI@2
      displayName: Build
      inputs:
        command: 'build'
        projects: '**/*.sln'
    
    - task: DotNetCoreCLI@2
      displayName: Publish
      inputs:
        command: 'publish'
        publishWebProjects: true
        arguments: '-o $(Build.ArtifactStagingDirectory)'
    
    - task: PublishBuildArtifacts@1
      displayName: Publish Artifacts ADO - Website
      inputs:
        pathToPublish: '$(Build.ArtifactStagingDirectory)'
        artifactName: Website
    
- stage: Deploy
  displayName: Deploy to an Azure Web App
  jobs:
  - job: Deploy
    pool:
      vmImage: 'windows-2019'
    steps:
    - task: DownloadBuildArtifacts@1
      inputs:
        buildType: 'current'
        downloadType: 'single'
        artifactName: 'Website'
        downloadPath: '$(Build.ArtifactStagingDirectory)'

```
4. Defina o cursor em uma nova linha no final da definição YAML (linha 69).

    > **Observação**: este será o local onde novas tarefas serão adicionadas.

5. Na lista de tarefas no lado direito do painel de código, procure e selecione a tarefa **Implantar do Azure App Service **.
6. No painel de **implantação do Azure App Service**, especifique as seguintes configurações e clique em **Adicionar**:

    - na lista suspensa **Assinatura do Azure**, selecione a assinatura do Azure na qual você implantou os recursos do Azure anteriormente no laboratório, clique em **Autorizar** e, quando solicitado, autentique-se usando a mesma conta de usuário usada durante a implantação de recursos do Azure.
    - na lista suspensa **Nome do Serviço de Aplicativo**, selecione o nome do aplicativo Web implantado anteriormente no laboratório.
    - na caixa de texto **Pacote ou pasta** , **atualize** o Valor Padrão para `$(Build.ArtifactStagingDirectory)/**/Web.zip`.
7. Confirme as configurações no painel Assistente clicando no botão **Adicionar** .

    > **Observação**: isso adicionará automaticamente a tarefa de implantação à definição de pipeline YAML.

8. O snippet de código adicionado ao editor deve ser semelhante ao abaixo, refletindo seu nome para os parâmetros azureSubscription e WebappName:

> **Observação**: o **parâmetro packageForLinux** é enganoso no contexto deste laboratório, mas é válido para Windows ou Linux.

    ```yaml
        - task: AzureRmWebAppDeployment@4
          inputs:
            ConnectionType: 'AzureRM'
            azureSubscription: 'AZURE SUBSCRIPTION HERE (b999999abc-1234-987a-a1e0-27fb2ea7f9f4)'
            appType: 'webApp'
            WebAppName: 'eshoponWebYAML369825031'
            packageForLinux: '$(Build.ArtifactStagingDirectory)/**/Web.zip'
    ```
9. Clique em **Salvar**, no painel **Salvar**, clique em **Salvar** novamente para confirmar a alteração diretamente no branch principal.

    > **Observação**: como nosso CI-YAML original não foi configurado para acionar automaticamente uma nova compilação, temos que iniciar esta manualmente.

10. No menu esquerdo do Azure DevOps, navegue até a guia **Pipelines** e selecione **Pipelines** novamente.
11. Abra o **EShopOnWeb_MultiStageYAML** Pipeline e clique em **Executar Pipeline**.
12. Confirme a opção **Executar** no painel exibido.
13. Observe que os 2 estágios diferentes, **Criar solução principal do .Net** e **Implantar no aplicativo Web do Azure** aparecem.
14. Aguarde até que o pipeline seja iniciado e aguarde até que ele conclua o Build Stage com êxito.
15. Quando o Estágio de Implantação quiser iniciar, você será solicitado com **Permissões Necessárias**, bem como uma barra laranja dizendo:

    ```text
    This pipeline needs permission to access a resource before this run can continue to Deploy to an Azure Web App
    ```

16. Clique em **Exibição**.
17. No **painel Aguardando revisão** , clique em **Permitir**.
18. Valide a mensagem na janela **pop-up** Permitir e confirme clicando em **Permitir**.
19. Isso inicia o Estágio de implantação. Aguarde a conclusão.

#### Tarefa 2: Revisar o site implantado

1. Volte para a janela do navegador da Web que exibe o portal do Azure e navegue até a folha que exibe as propriedades do aplicativo Web do Azure.
2. Na folha Aplicativo Web do Azure, clique em **Visão geral** e, na folha Visão geral, clique em **Procurar** para abrir seu site em uma nova guia do navegador da Web.
3. Verifique se o site implantado é carregado conforme o esperado na nova guia do navegador, mostrando o site de comércio eletrônico EShopOnWeb.

### Exercício 2: Implantar e configurar o Teste de Carga do Azure

Neste exercício, você implantará um Recurso de Teste de Carga do Azure no Azure e configurará diferentes cenários de Teste de Carga para seu Azure App Service em execução ao vivo.

#### 1: Implante o Teste de carga do Azure

A tarefa, implanta um novo recurso de teste de carga do Azure em sua assinatura do Azure.

1. Navegue do Azure Portal (https://portal.azure.com), navegar para **Criar um recurso no portal do Azure**.
2. No campo de pesquisa 'Serviços de Pesquisa e marketplace', insira **Teste de Carga** do Azure.
3. Selecione **Teste de carga do Azure** (publicado pela Microsoft), a partir dos resultados de pesquisa.
4. Na Página de Teste de Carga do Azure, clique em **Criar** para iniciar o processo de implantação.
5. Na página 'Criar um recurso de teste de carga', forneça os detalhes necessários para a implantação do recurso:
- **Assinatura**: selecione sua assinatura do Azure
- **Grupo de Recursos**: selecione o Grupo de Recursos usado para implantar o Serviço de Aplicativo Web no exercício anterior
- **Nome**: EShopOnWebLoadTesting
- **Região**: selecione uma região próxima à sua

    > **Observação**: o serviço de Teste de Carga do Azure não está disponível em todas as regiões do Azure.

6. Revise as configurações e clique em **Criar**, para validar suas configurações.
7. Clique em **Criar** para confirmar e implantar o recurso de Teste de Carga do Azure.
8. Você alternará para a página 'A implantação está em andamento'. Aguarde alguns minutos, até que a implantação seja concluída com sucesso.
9. Clique em **Ir para Recurso** na página de progresso da implantação para navegar até o **recurso de Teste de Carga do Azure EShopOnWebLoadTesting**. 

    > **Observação**: se você fechou a folha ou fechou o Portal do Azure durante a implantação do Recurso de Teste de Carga do Azure, poderá encontrá-lo novamente no campo Pesquisa no Portal do Azure ou na lista de recursos Recursos/Recentes. 

#### 2: crie o Teste de carga do Azure

Nesta tarefa, você criará diferentes testes de Teste de Carga do Azure, usando diferentes definições de configuração de carga. 

10. Na folha Recurso de Teste de Carga do Azure **EShopOnWebLoadTesting**, observe a **Introdução a um teste rápido** e clique no **botão Teste Rápido**.
11. Conclua os seguintes parâmetros e configurações para criar um teste de carga:
- **URL de teste**: insira a URL do Azure App Service implantado no exercício anterior (EShopOnWeb... azureWebsites.net), **incluindo https://**
- **Especificar carga**: usuários virtuais
- **Número de usuários virtuais**: 50
- **Duração do teste (segundos):** 120
- **Tempo de ramp-up (segundos):** 0
12. Confirme a configuração do teste, clicando em **Executar teste**.
13. O teste será executado a cada 2 minutos. 
14. Com o teste em execução, navegue de volta para a página Recurso de Teste de Carga do Azure **EShopOnWebLoadTesting** e navegue até **Testes**, selecione **Testes** e veja um teste **Get_eshoponWeb...**
15. No menu superior, clique em **Criar**, **Criar um teste rápido**, para criar um 2º teste de carga.
16. Conclua os seguintes parâmetros e configurações para criar outro teste de carga:
- **URL de teste**: insira a URL do Azure App Service implantado no exercício anterior (EShopOnWeb... azureWebsites.net), **incluindo https://**
- **Especificar carga**: solicitações por segundo (RPS)
- **RPS (solicitações por segundo)**: 100
- **Tempo de resposta (milissegundos)**: 500
- **Duração do teste (segundos)**: 120
- **Tempo de ramp-up (segundos):** 0
17. Confirme a configuração do teste, clicando em **Executar teste**.
18. O teste será executado a cada 2 minutos.

#### Tarefa 3: valide os resultados do Teste de carga do Azure

Nesta tarefa, você validará o resultado de um TestRun de Teste de Carga do Azure. 

Com ambos os testes rápidos concluídos, vamos fazer algumas alterações neles e validar os resultados.

19. Na folha Recurso **EShopOnWebLoadTesting** , navegue até **Testes** e selecione o primeiro Get_eshoponWebyaml... teste. Clique em **Editar** a partir da parte de cima do menu.
20. Aqui, o portal permite que você altere o **Nome do Teste** do padrão gerado para um mais descritivo. Ele também permite que você ainda faça alterações em qualquer um dos parâmetros definidos anteriormente.
21. Na folha **Editar teste**, navegue até a guia **Plano de teste**. 
22. É aqui que você pode gerenciar o arquivo de script de teste de carga do **Apache JMeter**, que é o que o Teste de Carga do Azure está usando como uma estrutura. Observe o arquivo **quick_test.jmx**. Selecione esse arquivo para **abri-lo** na máquina virtual do laboratório. Na janela pop-up, selecione **Visual Studio Code** como o editor para abrir o arquivo.
23. Observe a estrutura de linguagem XML do arquivo.

    > Observação: para obter informações adicionais e entender a sintaxe mais avançada do Apache JMeter, verifique o seguinte [link Teste de Carga do Azure - Jmeter](https://learn.microsoft.com/en-us/azure/load-testing/how-to-create-and-run-load-test-with-jmeter-script) .

24. De volta à **visualização Testes**, mostrando ambos os testes, selecione um deles, para abrir uma visualização mais detalhada, **clicando** em um dos testes. Isso redireciona você para a página de teste mais detalhada. A partir daqui, você pode validar os detalhes das execuções reais, selecionando o **TestRun_mm/dd/yy-hh:hh** na lista resultante.
25. Na página **TestRun detalhada**, identifique o resultado real da simulação do Teste de Carga do Azure. Alguns dos valores são:
- Total de solicitações/cargas
- Duração
- Tempo de Resposta (mostra o resultado em segundos, refletindo o tempo de resposta do percentil 90 - isso significa que, para 90% das solicitações, o tempo de resposta estará dentro dos resultados fornecidos)
- Taxa de transferência em solicitações por segundo
26. Mais abaixo, vários desses valores são representados usando as exibições do painel de gráficos de linhas.
27. Reserve alguns minutos para **comparar os resultados** dos testes simulados entre si e **identificar o impacto** de mais usuários no desempenho do Serviço de Aplicativo.

### Exercício 2: Automatizar um teste de carga com CI/CD em pipelines do Azure DevOps

Comece a automatizar os testes de carga no Teste de Carga do Azure adicionando-o a um pipeline de CI/CD. Depois de executar um teste de carga no portal do Azure, exporte os arquivos de configuração e configure um pipeline de CI/CD no Azure Pipelines (existe uma capacidade semelhante para GitHub Actions).

Depois de concluir este exercício, você terá um fluxo de trabalho de CI/CD configurado para executar um teste de carga com o Teste de Carga do Azure.

#### Tarefa 1: Identificar detalhes da conexão do serviço ADO

Nesta tarefa, você concederá as permissões necessárias à entidade de serviço da Conexão de Serviço do Azure DevOps.

1. No **portal do Azure DevOps**(, navegue até o https://dev.azure.com)projeto de **eShopOnWeb**.
2. Selecione **Configurações do projeto** no canto inferior esquerdo.
3. Na seção **Pipelines**, selecione **Conexões de serviço**.
4. Observe a Conexão de Serviço, com o nome de sua Assinatura do Azure que você usou para implantar os Recursos do Azure no início do exercício de laboratório.
5. **Selecione Conexões de serviço**. Na guia **Visão geral**, navegue até **Detalhes e selecione ****Gerenciar entidade de serviço** .
6. Isso redireciona você para o Portal do Azure, de onde ele abre os detalhes da **entidade de serviço** para o objeto de identidade.
7. Copie o valor **Nome de exibição** (formatado como Name_of_ADO_Organization_EShopOnWeb_-b86d9ae1-7552-4b75-a1e0-27fb2ea7f9f4) de lado, pois você precisará disso nas próximas etapas.

#### Tarefa 2: conceder permissões para a entidade de serviço

O Teste de Carga do Azure usa o Azure RBAC para conceder permissões e executar atividades específicas no recurso de teste de carga. Para executar um teste de carga a partir do pipeline de CI/CD, conceda a função **Colaborador de Teste de Carga** à entidade de serviço.

1. No **portal do Azure**, acesse seu recurso de **Teste de Carga do Azure**.
2. Selecione **Controle de acesso (IAM)** >Adicionar atribuição de função.
3. Na guia **Função**, selecione **Colaborador de Teste de Carga** na lista de funções de trabalho.
4. Na **guia Membros**, selecione **Selecionar membros** e, em seguida, use o **nome de exibição** que você copiou anteriormente para pesquisar a entidade de serviço.
5. Selecione a **entidade de serviço** e, em seguida, selecione **Selecionar**.
6. Na **guia Examinar + atribuir**, selecione **Examinar + atribuir** para adicionar a atribuição de função.

Agora você pode usar a conexão de serviço na definição de fluxo de trabalho do Azure Pipelines para acessar o recurso de teste de carga do Azure.

#### Tarefa 3: Exportar arquivos de entrada de teste de carga e Importar para o controle de código-fonte do Azure DevOps

Para executar um teste de carga com o Teste de Carga do Azure em um fluxo de trabalho de CI/CD, é necessário adicionar as configurações do teste de carga e todos os arquivos de entrada no repositório de controle do código-fonte. Se você tiver um teste de carga existente, poderá baixar as definições de configuração e todos os arquivos de entrada no portal do Azure.

Execute as etapas a seguir para baixar os arquivos de entrada de um teste de carga existente no portal do Azure:

1. No **portal do Azure**, acesse seu recurso de **Teste de Carga do Azure**.
2. No painel esquerdo, selecione **Testes** para exibir a lista de testes de carga e selecione o **seu teste**.
3. Selecione as reticências (**...**) ao lado da execução de teste com a qual está trabalhando e, em seguida, selecione **Baixar arquivo de entrada**.
4. O navegador baixa uma pasta compactada que contém os arquivos de entrada do teste de carga.
5. Use qualquer ferramenta zip para extrair os arquivos de entrada. A pasta contém os seguintes arquivos:

- *config.yaml*: o arquivo de configuração YAML do teste de carga. Faça referência a esse arquivo na definição de fluxo de trabalho de CI/CD.
- *quick_test.jmx*: o script de teste do Jeter

6. Confirme todos os arquivos de entrada extraídos no repositório de controle do código-fonte. Para fazer isso, navegue até o **Portal do Azure DevOps**(https://dev.azure.com) e navegue até o Projeto **EShopOnWeb** DevOps. 
7. Selecione **Repositório**. Na estrutura de pastas do código-fonte, observe a **subpasta testes** . Observe as reticências (...) e selecione **Nova pasta > Arquivo**.
8. especifique **jmeter** como nome da pasta e **placeholder.txt** para o nome do arquivo (Observação: uma pasta não pode ser criada como vazia)
9. Clique em **Confirmar** para confirmar a criação do arquivo de espaço reservado e da pasta jmeter.
10. Na **estrutura de pastas**, navegue até a nova ** subpasta jmeter** criada. Clique nas **reticências (...)** e selecione **Carregar arquivo(s).**
11. Usando a **opção Procurar** , navegue até o local do arquivo zip extraído e selecione **config.yaml** e **quick_test.jmx**.
12. Clique em **Confirmar** para confirmar o carregamento do arquivo no controle do código-fonte.

#### Tarefa 4: Atualizar o arquivo de definição YAML do fluxo de trabalho CI/CD

Nesta tarefa, você importará a extensão do Teste de Carga do Azure - Azure DevOps Marketplace, bem como atualizará o pipeline de CI/CD existente com a tarefa AzureLoadTest.

1. Para criar e executar um teste de carga, a definição de fluxo de trabalho do Azure Pipelines usa a extensão **Tarefa de Teste de Carga do Azure** do Azure DevOps Marketplace. Abra a [extensão de tarefa do Teste de Carga do Azure](https://marketplace.visualstudio.com/items?itemName=AzloadTest.AzloadTesting) no Azure DevOps Marketplace e selecione **Obter gratuitamente**.
2. Selecione sua organização do Azure DevOps e escolha **Instalar** a extensão.
3. No Portal e Projeto do Azure DevOps, navegue até **Pipelines** e selecione o pipeline criado no início deste exercício. Clique em **Editar**.
4. No script YAML, navegue até a **linha 56** e pressione ENTER/RETURN, para adicionar uma nova linha vazia. (isso é logo antes do Estágio de implantação do arquivo YAML).
5. Na linha 57, selecione o Assistente de Tarefas à direita e procure **Teste de Carga do Azure**.
6. Conclua o painel gráfico com as configurações corretas do seu cenário:
- Assinatura do Azure: selecione a assinatura do Azure do seu grupo de recursos.
- Arquivo de teste de carga: '$(Build.SourcesDirectory)/tests/jmeter/config.yaml' 
- Grupo de Recursos de Teste de Carga: o Grupo de Recursos que contém seus Recursos de Teste de Carga do Azure
- Nome do recurso de teste de carga: ESHopOnWebLoadTesting
- Nome de execução de teste de carga: ado_run
- Descrição da execução do teste de carga: teste de carga do ADO
7. Confirme a injeção dos parâmetros como um trecho de YAML clicando em **Adicionar**
8. Se o recuo do trecho YAML estiver dando erros (linhas vermelhas), corrija-os adicionando 2 espaços ou tabulação para posicionar o trecho corretamente.  
9. O trecho de exemplo abaixo mostra como deve ser a aparência do código YAML
```
     - task: AzureLoadTest@1
      inputs:
        azureSubscription: 'AZURE DEMO SUBSCRIPTION(b86d9ae1-1234-4b75-a8e7-27fb2ea7f9f4)'
        loadTestConfigFile: '$(Build.SourcesDirectory)/tests/jmeter/config.yaml'
        resourceGroup: 'az400m05l11-RG'
        loadTestResource: 'EShopOnWebLoadTesting'
        loadTestRunName: 'ado_run'
        loadTestRunDescription: 'load testing from ADO'
```
10. abaixo do trecho YAML inserido, adicione uma nova linha vazia pressionando ENTER/RETURN. 
11. abaixo dessa linha vazia, adicione um snippet para a tarefa Publicar, mostrando os resultados da tarefa de teste de Carga do Azure durante a execução de pipeline:

```
    - publish: $(System.DefaultWorkingDirectory)/loadTest
      artifact: loadTestResults
```
12.  Se o recuo do trecho YAML estiver dando erros (linhas vermelhas), corrija-os adicionando 2 espaços ou tabulação para posicionar o trecho corretamente.  
13. Com ambos os trechos adicionados ao pipeline de CI/CD, **salve** as alterações. 
14. Depois de salvo, clique em **Executar** para acionar o pipeline.
15. Confirme o branch (principal) e clique no botão **Executar** para iniciar a execução de pipeline.
16. Na página de status do pipeline, clique no estágio **Build** para abrir os detalhes completos do log das diferentes tarefas no pipeline.
17. Aguarde até que o pipeline inicie o Estágio de Criação e chegue à tarefa **AzureLoadTest** no fluxo do pipeline. 
18. Enquanto a tarefa está em execução, navegue até o **Teste de Carga do Azure** no Portal do Azure e veja como o pipeline cria um novo RunTest, chamado **adoloadtest1**. Você pode selecioná-lo para mostrar os valores de resultado do trabalho TestRun.
19. Navegue de volta para o modo de exibição Execução de Pipeline de CI/CD do Azure DevOps, onde a **tarefa AzureLoadTest** foi concluída com êxito. A partir da saída de log detalhado, os valores resultantes do teste de carga também serão visíveis:

```
Task         : Azure Load Testing
Description  : Automate performance regression testing with Azure Load Testing
Version      : 1.2.30
Author       : Microsoft Corporation
Help         : https://docs.microsoft.com/azure/load-testing/tutorial-cicd-azure-pipelines#azure-load-testing-task
==============================================================================
Test '0d295119-12d0-482d-94be-a7b84787c004' already exists
Uploaded test plan for the test
Creating and running a testRun for the test
View the load test run in progress at: https://portal.azure.com/#blade/Microsoft_Azure_CloudNativeTesting/NewReport//resourceId/%2fsubscriptions%4b75-a1e0-27fb2ea7f9f4%2fresourcegroups%2faz400m05l11-rg%2fproviders%2fmicrosoft.loadtestservice%2floadtests%2feshoponwebloadtesting/testId/0d295119-12d0-787c004/testRunId/161046f1-d2d3-46f7-9d2b-c8a09478ce4c
TestRun completed

-------------------Summary ---------------
TestRun start time: Mon Jul 24 2023 21:46:26 GMT+0000 (Coordinated Universal Time)
TestRun end time: Mon Jul 24 2023 21:51:50 GMT+0000 (Coordinated Universal Time)
Virtual Users: 50
TestStatus: DONE

------------------Client-side metrics------------

Homepage
response time        : avg=1359ms min=59ms med=539ms max=16629ms p(90)=3127ms p(95)=5478ms p(99)=13878ms
requests per sec     : avg=37
total requests       : 4500
total errors         : 0
total error rate     : 0
Finishing: AzureLoadTest

```
20. Agora você executou um teste de carga automatizado como parte de uma execução de pipeline. Na última tarefa, você especificará condições para falha, ou seja, não permitiremos que nosso Estágio de implantação seja iniciado, se o desempenho do aplicativo Web estiver abaixo de um determinado limite. 

#### Tarefa 5: adicionar critérios de falha/êxito ao pipeline de teste de carga

Nesta tarefa, você usará critérios de falha de teste de carga para ser alertado (ter um pipeline com falha executado como resultado) quando o aplicativo não atender aos seus requisitos de qualidade.

1. No Azure DevOps, navegue até o Projeto EShopOnWeb e abra **Repos**.
2. Em Repos, navegue até a subpasta **/tests/jmeter** criada e usada anteriormente.
3. Abra o arquivo de teste de carga*config.yaml**. Clique em Editar **** para permitir a edição do arquivo.
4. Ao final do arquivo, adicione o seguinte código:

```
failureCriteria:
  - avg(response_time_ms) > 300
  - percentage(error) > 50
```
5. Salve as alterações no config.yaml clicando em **Confirmar** e Confirmar mais uma vez.
6. Navegue de volta para **Pipelines** e execute o pipeline **EShopOnWeb** novamente. Após alguns minutos, ele concluirá a execução com um status de **falha** para a tarefa **AzureLoadTest**. 
7. Abra o modo de exibição de log detalhado para o pipeline e valide os detalhes do **AzureLoadtest**. Um exemplo de saída é mostrado abaixo:

```
Creating and running a testRun for the test
View the load test run in progress at: https://portal.azure.com/#blade/Microsoft_Azure_CloudNativeTesting/NewReport//resourceId/%2fsubscriptions%2fb86d9ae1-7552-47fb2ea7f9f4%2fresourcegroups%2faz400m05l11-rg%2fproviders%2fmicrosoft.loadtestservice%2floadtests%2feshoponwebloadtesting/testId/0d295119-12d0-a7b84787c004/testRunId/f4bec76a-8b49-44ee-a388-12af34f0d4ec
TestRun completed

-------------------Summary ---------------
TestRun start time: Mon Jul 24 2023 23:00:31 GMT+0000 (Coordinated Universal Time)
TestRun end time: Mon Jul 24 2023 23:06:02 GMT+0000 (Coordinated Universal Time)
Virtual Users: 50
TestStatus: DONE

-------------------Test Criteria ---------------
Results          :1 Pass 1 Fail

Criteria                     :Actual Value        Result
avg(response_time_ms) > 300                       1355.29               FAILED
percentage(error) > 50                                                  PASSED


------------------Client-side metrics------------

Homepage
response time        : avg=1355ms min=58ms med=666ms max=16524ms p(90)=2472ms p(95)=5819ms p(99)=13657ms
requests per sec     : avg=37
total requests       : 4531
total errors         : 0
total error rate     : 0
##[error]TestResult: FAILED
Finishing: AzureLoadTest
```

8. Observe como a última linha da saída do teste de carga diz **##[error]TestResult: FAILED**; como definimos um FailCriteria** com um tempo médio de resposta de > de 300 ou uma porcentagem de erro de > de 20, agora vendo um **tempo médio de resposta superior a 300, a tarefa será sinalizada como falha. 

    > Observação: imagine que, em um cenário real, você validaria o desempenho do seu Serviço de Aplicativo e, se o desempenho estiver abaixo de um determinado treshold - normalmente significando que há mais carga no Aplicativo Web, você poderá disparar uma nova implantação para um Azure App Service adicional. Como não podemos controlar o tempo de resposta para ambientes de laboratório do Azure, decidimos reverter a lógica para garantir a falha.

9.  O status FAILED da tarefa de pipeline, na verdade, reflete um ÊXITO da validação de critérios de requisito do Teste de Carga do Azure.

### Exercício 3: Remover os recursos do laboratório do Azure

Neste exercício, você removerá os recursos do Azure provisionados neste laboratório para eliminar cobranças inesperadas.

>**Observação**: lembre-se de remover todos os recursos do Azure que acabam de ser criados e que você não usa mais. Remover recursos não utilizados garante que você não veja encargos inesperados.

#### Tarefa 1: remover os recursos do Azure Lab

Neste exercício, você usará o Azure Cloud Shell para remover os recursos do Azure provisionados neste laboratório para eliminar cobranças inesperadas.

1. No portal do Azure, abra a sessão **Bash** no painel **Cloud Shell**.
2. Exclua todos os grupos de recursos criados em todos os laboratórios deste módulo executando o seguinte comando:

    ```sh
    az group list --query "[?starts_with(name,'az400m16l01')].name" --output tsv
    ```

3. Exclua todos os grupos de recursos criados em todos os laboratórios deste módulo executando o seguinte comando:

    ```sh
    az group list --query "[?starts_with(name,'az400m16l01')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**Observação**: o comando é executado de modo assíncrono (conforme determinado pelo parâmetro --nowait), portanto, embora você possa executar outro comando da CLI do Azure imediatamente depois na mesma sessão Bash, levará alguns minutos antes de o grupo de recursos ser removido.

## Revisão

Neste exercício, você implantou um aplicativo Web no Azure App Service usando o Azure Pipelines, bem como implantando um Recurso de Teste de Carga do Azure com TestRuns. Em seguida, você integrou o arquivo config.yaml de teste de carga do Jmeter ao controle do código-fonte do Azure DevOps Repos e estendeu seu pipeline de CI/CD com o Teste de Carga do Azure. No último exercício, você aprendeu a definir os critérios de sucesso do LoadTest.
