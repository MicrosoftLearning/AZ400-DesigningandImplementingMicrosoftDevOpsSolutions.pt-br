---
lab:
  title: Monitore o desempenho do aplicativo com o Azure Load Testing
  module: 'Module 08: Implement continuous feedback'
---

# Monitore o desempenho do aplicativo com o Azure Load Testing

## Requisitos do laboratório

- Este laboratório requer o **Microsoft Edge** ou um [navegador com suporte do Azure DevOps.](https://docs.microsoft.com/azure/devops/server/compatibility)

- **Configurar uma organização do Azure DevOps:** se você ainda não tiver uma organização Azure DevOps que possa usar para este laboratório, crie uma seguindo as instruções disponíveis em [Criar uma organização ou coleção de projetos](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization).

- Identifique uma assinatura existente do Azure ou crie uma.

- Verifique se você tem uma conta Microsoft ou uma conta do Microsoft Entra com a função Proprietário na assinatura do Azure e a função Administrador Global no locatário do Microsoft Entra associado à assinatura do Azure. Para obter detalhes, veja [Listar designações de função do Azure usando o portal do Azure](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-list-portal) e [Exibir e designar funções de administrador no Azure Active Directory](https://docs.microsoft.com/azure/active-directory/roles/manage-roles-portal#view-my-roles).

## Visão geral do laboratório

O **Teste de Carga do Azure** é um serviço de teste de carga totalmente gerenciado que permite gerar cargas de alta escala. O serviço simula o tráfego para seus aplicativos, independentemente de onde estão hospedados. Desenvolvedores, testadores e engenheiros de QA (garantia de qualidade) podem usá-lo para otimizar o desempenho, a escalabilidade ou a capacidade do aplicativo.
Crie rapidamente um teste de carga para seu aplicativo Web usando uma URL e sem conhecimento prévio das ferramentas de teste. O Teste de Carga do Azure abstrai a complexidade e a infraestrutura para executar seu teste de carga em escala.
Para cenários mais avançados de teste de carga, você pode criar um teste de carga reutilizando um script de teste do Apache JMeter existente, uma ferramenta de desempenho e carga de código aberto popular. Por exemplo, o plano de teste pode consistir em várias solicitações de aplicativo, você deseja chamar pontos de extremidade que não sejam HTTP ou está usando dados e parâmetros de entrada para que o teste fique mais dinâmico.

Neste laboratório, você aprenderá sobre como usar o Teste de Carga do Azure para simular teste de desempenho em um aplicativo Web em execução em tempo real com diferentes cenários de carga. Por fim, você aprenderá a integrar o Teste de Carga do Azure em seus pipelines de CI/CD.

## Objetivos

Após concluir este laboratório, você poderá:

- Implantar aplicativos Web do Azure App Service.
- Compor e executar um pipeline de CI/CD baseado em YAML.
- Implantar o Teste de carga do Azure
- Investigar o desempenho do aplicativo Web do Azure usando o Teste de Carga do Azure.
- Integrar o Teste de Carga do Azure em seus pipelines de CI/CD.

## Tempo estimado: 60 minutos

## Instruções

### Exercício 0: configurar os pré-requisitos do laboratório

Neste exercício, você configurará os pré-requisitos para o laboratório.

#### Tarefa 1: (pular se feita) criar e configurar o projeto de equipe

Nesta tarefa, você criará um projeto **eShopOnWeb** do Azure DevOps para ser usado por vários laboratórios.

1. No computador do laboratório, em uma janela do navegador, abra sua organização do Azure DevOps. Clique em **Novo projeto**. Dê ao seu projeto o nome **eShopOnWeb** e escolha **Scrum** na lista suspensa **Processo de item de trabalho**. Clique em **Criar**.

   ![Captura de tela do painel criar um novo projeto.](images/create-project.png)

#### Tarefa 2: (pular se feita) importar repositório do Git eShopOnWeb

Nesta tarefa, você importará o repositório eShopOnWeb do Git que será usado por vários laboratórios.

1. No computador do laboratório, em uma janela do navegador, abra sua organização do Azure DevOps e o projeto **eShopOnWeb** criado anteriormente. Clique em **Repos > Arquivos**, **Importar**. Na janela **Importar um repositório Git**, cole a URL <https://github.com/MicrosoftLearning/eShopOnWeb.git> e clique em **Importar**:

   ![Captura de tela do painel importar repositório.](images/import-repo.png)

1. O repositório está organizado da seguinte forma:
   - A pasta **.ado** contém os pipelines YAML do Azure DevOps
   - O contêiner da pasta **.devcontainer** está configurado para o desenvolvimento usando contêineres (localmente no VS Code ou no GitHub Codespaces)
   - A pasta **infra** contém a infraestrutura Bicep e ARM como modelos de código usados em alguns cenários de laboratório.
   - A pasta **.github** contém definições de fluxo de trabalho YAML do GitHub.
   - A pasta **src** contém o site do .NET 8 usado em cenários de laboratório.

#### Tarefa 3: (pular se feita) definir o branch main como branch padrão

1. Vá para **Repos > Branches**.
1. Passe o mouse sobre o branch **main** e clique nas reticências à direita da coluna.
1. Clique em **Definir como branch padrão**.

#### Tarefa 4: Criar recursos do Azure

Nesta tarefa, você criará um aplicativo Web do Azure usando o Cloud Shell no portal do Azure.

1. No computador do laboratório, inicie um navegador da Web, navegue até o [**Portal do Azure**](https://portal.azure.com) e conecte-se.
1. No portal do Azure, na barra de ferramentas, clique no ícone do **Cloud Shell** localizado ao lado da caixa de pesquisa.
1. Se for solicitado que você selecione **Bash** ou **PowerShell**, selecione **Bash**.

   > **Observação**: se esta for a primeira vez que você está iniciando o **Cloud Shell** e você receber a mensagem **Você não tem nenhum armazenamento montado**, selecione a assinatura que você está usando no laboratório e selecione **Criar armazenamento**.

1. No prompt  **Bash**, no painel **Cloud Shell**, execute o seguinte comando para criar um grupo de recursos (substitua o espaço reservado `<region>` pelo nome da região do Azure mais próxima de você, como 'eastus').

   ```bash
   RESOURCEGROUPNAME='az400m08l14-RG'
   LOCATION='<region>'
   az group create --name $RESOURCEGROUPNAME --location $LOCATION
   ```

1. Execute o comando a seguir para criar um plano do Serviço de Aplicativo.

   ```bash
   SERVICEPLANNAME='az400l14-sp'
   az appservice plan create --resource-group $RESOURCEGROUPNAME \
       --name $SERVICEPLANNAME --sku B3
   ```

1. Crie um aplicativo Web com um nome exclusivo.

   ```bash
   WEBAPPNAME=az400eshoponweb$RANDOM$RANDOM
   az webapp create --resource-group $RESOURCEGROUPNAME --plan $SERVICEPLANNAME --name $WEBAPPNAME
   ```

   > **Observação**: registre o nome do aplicativo Web. Você precisará dela posteriormente neste laboratório.

### Exercício 1: configurar pipelines de CI/CD como código com YAML no Azure DevOps

Neste exercício, você vai configurar pipelines de CI/CD como código com YAML no Azure DevOps.

#### Tarefa 1: adicionar uma definição de compilação e implantação do YAML

Nesta tarefa, você adicionará uma definição de compilação do YAML ao projeto existente.

1. Navegue de volta ao painel **Pipelines** no hub **Pipelines**.
1. Clique em **Novo pipeline** (ou em Criar Pipeline se este for o primeiro que você criar).

   > **Observação**: usaremos o assistente para criar uma nova definição de pipeline do YAML com base em nosso projeto.

1. No painel **Onde está seu código?**, clique na opção **Git do Azure Repos (YAML).**
1. No painel **Selecionar um repositório**, clique em **eShopOnWeb**.
1. Na página **Configurar seu pipeline**, role para baixo e selecione **Pipeline inicial**.
1. **Selecione** todas as linhas do Pipeline inicial e exclua-as.
1. **Copie** o pipeline de modelo completo abaixo, sabendo que você precisará fazer modificações de parâmetro **antes de salvar** as alterações:

   ```yml
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
                 command: "restore"
                 projects: "**/*.sln"
                 feedsToUse: "select"

             - task: DotNetCoreCLI@2
               displayName: Build
               inputs:
                 command: "build"
                 projects: "**/*.sln"

             - task: DotNetCoreCLI@2
               displayName: Publish
               inputs:
                 command: "publish"
                 publishWebProjects: true
                 arguments: "-o $(Build.ArtifactStagingDirectory)"

             - task: PublishBuildArtifacts@1
               displayName: Publish Artifacts ADO - Website
               inputs:
                 pathToPublish: "$(Build.ArtifactStagingDirectory)"
                 artifactName: Website

     - stage: Deploy
       displayName: Deploy to an Azure Web App
       jobs:
         - job: Deploy
           pool:
             vmImage: "windows-2019"
           steps:
             - task: DownloadBuildArtifacts@1
               inputs:
                 buildType: "current"
                 downloadType: "single"
                 artifactName: "Website"
                 downloadPath: "$(Build.ArtifactStagingDirectory)"
   ```

1. Coloque o cursor em uma nova linha no final da definição do YAML. **Posicione o cursor no recuo do nível de tarefa anterior**.

   > **Observação**: este será o local onde novas tarefas serão adicionadas.

1. Clique em **Mostrar Assistente** do lado direito do portal. Na lista de tarefas, pesquise e selecione a tarefa **Implantação do Serviço de Aplicativo do Azure**.
1. No painel **Implantação do Serviço de Aplicativo do Azure**, especifique as seguintes configurações e clique em **Adicionar**:

   - Na lista suspensa **Assinatura do Azure**, selecione a conexão de serviço que você acabou de criar.
   - Valide que o **Tipo de Serviço de Aplicativo** aponta para o Aplicativo Web no Windows.
   - Na lista suspensa **Nome do Serviço de Aplicativo**, selecione o nome do aplicativo Web implantado anteriormente no laboratório (\*\*az400eshoponweb...).
   - na caixa de texto **Pacote ou pasta**, **atualize** o Valor Padrão para `$(Build.ArtifactStagingDirectory)/**/Web.zip`.
   - Expanda **Definições de aplicativo e configuração** e, na caixa de texto Configurações do aplicativo, adicione os seguintes pares de chave-valor: `-UseOnlyInMemoryDatabase true -ASPNETCORE_ENVIRONMENT Development`.

1. Confirme as configurações no painel Assistente clicando no botão **Adicionar** .

   > **Observação**: isso adicionará automaticamente a tarefa de implantação à definição de pipeline YAML.

1. O snippet de código adicionado ao editor deve ser semelhante ao abaixo, refletindo seu nome para os parâmetros azureSubscription e WebappName:

   ```yml
   - task: AzureRmWebAppDeployment@4
     inputs:
       ConnectionType: "AzureRM"
       azureSubscription: "SERVICE CONNECTION NAME"
       appType: "webApp"
       WebAppName: "az400eshoponWeb369825031"
       packageForLinux: "$(Build.ArtifactStagingDirectory)/**/Web.zip"
       AppSettings: "-UseOnlyInMemoryDatabase true -ASPNETCORE_ENVIRONMENT Development"
   ```

   > **Observação**: o parâmetro **packageForLinux** é enganoso no contexto deste laboratório, mas é válido para Windows ou Linux.

1. Antes de salvar as atualizações no arquivo yml, dê a ele um nome mais claro. Na parte superior da janela do editor yaml, ele mostra **EShopOnweb/azure-pipelines-#.yml**. (em que # é um número, normalmente 1, mas pode ser diferente em sua configuração.) Selecione **esse nome de arquivo** e renomeie-o para **m08l14-pipeline.yml**

1. Clique em **Salvar**. No painel **Salvar**, clique em **Salvar** novamente para confirmar a alteração diretamente no branch main.

   > **Observação**: como nosso CI-YAML original não foi configurado para acionar automaticamente uma nova compilação, temos que iniciar esta manualmente.

1. No menu Azure DevOps à esquerda, navegue até **Pipelines** e selecione **Pipelines** novamente. Em seguida, selecione **Todos** para abrir todas as definições de pipeline, não apenas as recentes.

   > **Observação**: se você manteve todos os pipelines anteriores de exercícios de laboratório anteriores, este novo pipeline pode ter reutilizado um nome de sequência padrão **eShopOnWeb (#)** para o pipeline, conforme mostrado na captura de tela abaixo. Selecione um pipeline (provavelmente aquele com o número de sequência mais alto, clique em Editar e validar aponta para o arquivo de código m08l14-pipeline.yml).

   ![Captura de tela do Azure Pipelines mostrando execuções do eShopOnWeb](images/m3/eshoponweb-m9l16-pipeline.png)

1. Confirme para executar esse pipeline clicando em **Executar** no painel exibido e confirme clicando em **Executar** mais uma vez.
1. Duas fases diferentes são exibidas, **Compilar solução .Net Core** e **Implantar no aplicativo Web do Azure**.
1. Aguarde a conclusão da execução de pipeline.

1. **Ignore** avisos exibidos durante o Estágio de Build. Aguarde até que ele conclua o Estágio de Build com êxito. (Você pode selecionar o estágio de build real para ver mais detalhes nos logs.)

1. Quando a Fase Implantar quiser iniciar, será solicitado as **Permissões Necessárias**, e aparecerá uma barra laranja dizendo:

   ```text
   This pipeline needs permission to access a resource before this run can continue to Deploy to an Azure Web App
   ```

1. Clique em **Exibição**.
1. No painel **Aguardando revisão**, clique em **Permitir**.
1. Valide a mensagem na janela **pop-up Permitir** e confirme clicando em **Permitir**.
1. Isso inicia a Fase Implantar. Aguarde a conclusão.

#### Tarefa 2: revisar o site implantado

1. Volte para a janela do navegador da Web que exibe o portal do Azure e navegue até a folha que exibe as propriedades do aplicativo Web do Azure.
1. Na folha Aplicativo Web do Azure, clique em **Visão geral** e, na folha Visão geral, clique em **Procurar** para abrir seu site em uma nova guia do navegador da Web.
1. Verifique se o site implantado é carregado conforme o esperado na nova guia do navegador, mostrando o site de comércio eletrônico eShopOnWeb.

### Exercício 2: implantar e configurar o Teste de Carga do Azure

Neste exercício, você implantará um Recurso de Teste de Carga do Azure no Azure e configurará diferentes cenários de Teste de Carga para seu Serviço de Aplicativo do Azure em execução em tempo real.

> **Importante**: o Teste de Carga do Azure é um **serviço pago**. Testes de carga geram custos. Certifique-se de limpar os recursos depois de concluir o laboratório para evitar incorrer em custos adicionais. Para cada "Recurso de Teste de Carga" que estiver ativo durante qualquer parte de um mês, será cobrada a taxa mensal e você terá acesso aos 50 VUH incluídos. Para mais informações, confira a [página de preços do Teste de Carga do Azure](https://azure.microsoft.com/pricing/details/load-testing).

#### Tarefa 1: implantar o Teste de Carga do Azure

Nesta tarefa, você implantará um novo recurso de Teste de Carga do Azure em sua assinatura do Azure.

1. No Portal do Azure (<https://portal.azure.com>), navegue até **Criar recurso do Azure**.
1. No campo de pesquisa "Serviços de Pesquisa e marketplace", insira **`Azure Load Testing`**.
1. Nos resultados da pesquisa, selecione **Teste de Carga do Azure** (publicado pela Microsoft).
1. Na página Teste de Carga do Azure, clique em **Criar** para iniciar o processo de implantação.
1. Na página "Criar um recurso de teste de carga", forneça os detalhes necessários para a implantação do recurso:

   - **Assinatura**: selecione sua assinatura do Azure
   - **Grupo de recursos**: selecione o Grupo de recursos usado para implantar o Serviço de Aplicativo Web no exercício anterior
   - **Nome**: `eShopOnWebLoadTesting`
   - **Região**: selecione uma região próxima à sua região

   > **Observação**: o serviço Teste de Carga do Azure não está disponível em todas as regiões do Azure.

1. Clique em **Revisar e criar** para validar suas configurações.
1. Clique em **Criar** para confirmar e implantar o recurso Teste de Carga do Azure.
1. Você alternará para a página "A implantação está em andamento". Espere alguns minutos para a conclusão da implantação.
1. Clique em **Ir para o recurso** na página de progresso da implantação para navegar até o recurso do Teste de Carga do Azure **eShopOnWebLoadTesting**.

   > **Observação**: se você fechou a folha ou fechou o portal do Azure durante a implantação do Recurso Teste de Carga do Azure, poderá encontrá-lo novamente no campo Pesquisa no Portal do Azure ou na lista de recursos Recursos/Recentes.

#### Tarefa 2: criar testes de Teste de Carga do Azure

Nesta tarefa, você criará diferentes testes de Teste de Carga do Azure, usando diferentes definições de configuração de carga.

1. De dentro da folha **eShopOnWebLoadTesting** do recurso do Teste de Carga do Azure, navegue até **Testes** em **Testes**. Clique na opção de menu **+Criar** e em **Criar um teste baseado em URL**.
1. Desmarque a caixa de seleção **Ativar configurações avançadas** para exibir as configurações avançadas.
1. Conclua os seguintes parâmetros e configurações para criar um teste de carga:

   - **URL de Teste**: Insira a URL do Serviço de Aplicativo do Azure que você implantou no exercício anterior (az400eshoponweb...azurewebsites.net), **incluindo https://**
   - **Especificar carga**: usuários virtuais
   - **Número de usuários virtuais**: 50
   - **Duração do Teste (minutos)**: 5
   - **Tempo de espera (minutos)**:  1

1. Confirme a configuração do teste clicando em **Examinar e Criar**(não faça nenhuma alteração nas outras guias). Clique em **Criar** mais uma vez.
1. Isso inicia os Testes de Carga, que serão executados. O teste será executado por 5 minutos.
1. Com o teste em execução, navegue de volta até a página **eShopOnWebLoadTesting** do recurso do Teste de Carga do Azure e navegue até **Testes**, selecione **Testes** e veja um teste **Get_eshoponweb...**
1. No menu superior, clique em **Criar**, **Criar um teste baseado em URL**, para criar um segundo teste de carga.
1. Conclua os seguintes parâmetros e configurações para criar outro teste de carga:

   - **URL de Teste**: Insira a URL do Serviço de Aplicativo do Azure que você implantou no exercício anterior (eShopOnWeb... azurewebsites.net), **incluindo https://**
   - **Especificar carga**: solicitações por segundo (RPS)
   - **Solicitações por segundo (RPS)**: 100
   - **Tempo de resposta (milissegundos)**: 500
   - **Duração do Teste (minutos)**: 5
   - **Tempo de espera (minutos)**:  1

1. Confirme a configuração do teste clicando mais uma vez em **Examinar + criar**e **Criar**.
1. O teste será executado por cerca de cinco minutos.

#### Tarefa 3: validar os resultados do Teste de Carga do Azure

Nesta tarefa, você validará o resultado de um TestRun de Teste de Carga do Azure.

Com ambos os testes rápidos concluídos, vamos fazer algumas alterações neles e validar os resultados.

1. Em **Teste de Carga do Azure**, navegue até **Testes**. Para abrir uma exibição mais detalhada, selecione qualquer uma das definições de teste **clicando** em um dos testes. Isso redireciona você para a página de teste mais detalhada. Nela, você pode validar os detalhes das execuções reais selecionando o **TestRun_mm/dd/yy-hh:hh** na lista resultante.
1. Na página **TestRun** detalhada, identifique o resultado real da simulação do Teste de Carga do Azure. Alguns dos valores são:

   - Carga/total de solicitações
   - Duração
   - Tempo de Resposta (mostra o resultado em segundos, refletindo o percentil 90 do tempo de resposta – isso significa que, para 90% das solicitações, o tempo de resposta estará dentro dos resultados fornecidos)
   - Taxa de transferência em solicitações por segundo

1. Mais abaixo, vários desses valores são representados usando visualizações de linhas e gráficos de painel.
1. Reserve alguns minutos para **comparar os resultados** de ambos os testes simulados entre si e **identificar o impacto** de mais usuários no desempenho do Serviço de Aplicativo.

### Exercício 3: Automatizar um Teste de Carga com CI/CD no Azure Pipelines

Comece a automatizar os testes de carga no Teste de Carga do Azure adicionando-o a um pipeline de CI/CD. Depois de executar um teste de carga no portal do Azure, exporte os arquivos de configuração e configure um pipeline de CI/CD no no Azure Pipelines (existe uma capacidade semelhante para o GitHub).

Depois de concluir este exercício, você terá um fluxo de trabalho de CI/CD configurado para executar um teste de carga com o Teste de Carga do Azure.

#### Tarefa 1: Identificar detalhes da conexão do serviço Azure DevOps

Nesta tarefa, você concederá as permissões necessárias à Conexão de Serviço do Azure DevOps.

1. No Portal do **Azure DevOps**(<https://aex.dev.azure.com>), navegue até oprojeto **eShopOnWeb**.
1. Selecione **Configurações do projeto** no canto inferior esquerdo.
1. Na seção **Pipelines**, selecione **Conexões de serviço**.
1. Observe a Conexão de Serviço, com o nome de sua Assinatura do Azure que você usou para implantar os Recursos do Azure no início do exercício de laboratório.
1. **Selecione a Conexão de serviço**. Na guia **Visão geral**, navegue até **Detalhes** e clique em **Gerenciar funções da conexão de serviço**.
1. O Portal do Azure será aberto, de onde ele abre os detalhes do grupo de recursos na folha de controle de acesso (IAM).

#### Tarefa 2: Conceder permissões ao recurso de Teste de Carga do Azure

O Teste de Carga do Azure usa o Azure RBAC para conceder permissões e executar atividades específicas no recurso de teste de carga. Para executar um teste de carga a partir do pipeline de CI/CD, conceda a função **Colaborador de Teste de Carga** à conexão de serviço do Azure DevOps.

1. Selecione **+ Adicionar** e **Adicionar atribuição de função**.
1. Na guia **Função**, selecione **Colaborador de Teste de Carga** na lista de funções de trabalho.
1. Na guia **Membros**, clique em + **Selecionar membros**, localize e selecione sua conta de usuário e clique em **Selecionar**.
1. Na **guia Examinar + atribuir**, selecione **Examinar + atribuir** para adicionar a atribuição de função.

Agora você pode usar a conexão de serviço na definição de fluxo de trabalho do Azure Pipelines para acessar o recurso de teste de carga do Azure.

#### Tarefa 3: Exportar arquivos de entrada de teste de carga e importar para o Azure Repos

Para executar um teste de carga com o Teste de Carga do Azure em um fluxo de trabalho de CI/CD, é necessário adicionar as configurações do teste de carga e todos os arquivos de entrada no repositório de controle do código-fonte. Se você tiver um teste de carga existente, poderá baixar as definições de configuração e todos os arquivos de entrada no portal do Azure.

Execute as etapas a seguir para baixar os arquivos de entrada de um teste de carga existente no portal do Azure:

1. No **portal do Azure**, acesse seu recurso **Teste de Carga do Azure**.
1. No painel esquerdo, selecione **Testes** para exibir a lista de testes de carga e selecione o **seu teste**.
1. Selecione as reticências (**...**) ao lado da execução de teste com a qual está trabalhando e, em seguida, selecione **Baixar arquivo de entrada**.
1. O navegador baixa uma pasta compactada que contém os arquivos de entrada do teste de carga.
1. Use qualquer ferramenta zip para extrair os arquivos de entrada. A pasta contém os seguintes arquivos:

   - _config.yaml_: o arquivo de configuração YAML de teste de carga. Faça referência a esse arquivo na definição de fluxo de trabalho de CI/CD.
   - _quick_test.jmx_: o script de teste do JMeter

1. Confirme todos os arquivos de entrada extraídos no repositório de controle do código-fonte. Para fazer isso, navegue até o **Portal do Azure DevOps**(<https://aex.dev.azure.com/>) e navegue até o projeto **eShopOnWeb** do DevOps.
1. Selecione **Repositório**. Na estrutura de pastas do código-fonte, observe a subpasta **testes**. Observe as reticências (...) e selecione **Novo > Pasta**.
1. especifique **jmeter** como nome da pasta e **placeholder.txt** para o nome do arquivo (Observação: uma pasta não pode ser criada como vazia)
1. Clique em **Confirmar** para confirmar a criação do arquivo de espaço reservado e da pasta jmeter.
1. Na **Estrutura de pastas**, navegue até a nova subpasta jmeter** criada**. Clique nas **reticências (...)** e selecione **Carregar arquivo(s).**
1. Usando a opção **Procurar**, navegue até o local do arquivo zip extraído e selecione **config.yaml** e **quick_test.jmx**.
1. Clique em **Confirmar** para confirmar o carregamento do arquivo no controle do código-fonte.

#### Tarefa 4: atualizar o arquivo de definição YAML do fluxo de trabalho CI/CD

1. Para criar e executar um teste de carga, a definição de fluxo de trabalho do Azure Pipelines usa a extensão **Tarefa de Teste de Carga do Azure** do Azure DevOps Marketplace. Abra a [extensão de tarefa do Teste de Carga do Azure](https://marketplace.visualstudio.com/items?itemName=AzloadTest.AzloadTesting) no Azure DevOps Marketplace e selecione **Obter gratuitamente**.
1. Selecione sua organização do Azure DevOps e escolha **Instalar** a extensão.
1. No Portal e Projeto do Azure DevOps, navegue até **Pipelines** e selecione o pipeline criado no início deste exercício. Clique em **Editar**.
1. No script YAML, navegue até a **linha 56** e pressione ENTER/RETURN para adicionar uma nova linha vazia. (isso é logo antes da Fase Implantar do arquivo YAML).
1. Na linha 57, selecione o Assistente de Tarefas à direita e procure **Teste de Carga do Azure**.
1. Conclua o painel gráfico com as configurações corretas do seu cenário:

   - Assinatura do Azure: selecione a assinatura que executa seus recursos do Azure
   - Arquivo de teste de carga: "$(Build.SourcesDirectory)/tests/jmeter/config.yaml"
   - Grupo de Recursos do Teste de Carga: o Grupo de Recursos que contém seus Recursos de Teste de Carga do Azure
   - Nome do recurso de teste de carga: `eShopOnWebLoadTesting`
   - Nome da execução do teste de carga: ado_run
   - Descrição da execução do teste de carga: teste de carga do ADO

1. Confirme a injeção dos parâmetros como um snippet de YAML clicando em **Adicionar**
1. Se o recuo do snippet YAML estiver com erros (sublinhado ondulado vermelho), corrija-o adicionando dois espaços ou uma tabulação para posicionar o snippet corretamente.
1. O snippet de exemplo abaixo mostra como o código YAML deve ser

   ```yml
        - task: AzureLoadTest@1
         inputs:
           azureSubscription: 'AZURE DEMO SUBSCRIPTION'
           loadTestConfigFile: '$(Build.SourcesDirectory)/tests/jmeter/config.yaml'
           resourceGroup: 'az400m08l14-RG'
           loadTestResource: 'eShopOnWebLoadTesting'
           loadTestRunName: 'ado_run'
           loadTestRunDescription: 'load testing from ADO'
   ```

1. abaixo do snippet YAML inserido, adicione uma nova linha vazia pressionando ENTER/RETURN.
1. abaixo dessa linha vazia, adicione um snippet para a tarefa Publicar, mostrando os resultados da tarefa de teste de Carga do Azure durante a execução do pipeline:

   ```yml
   - publish: $(System.DefaultWorkingDirectory)/loadTest
     artifact: loadTestResults
   ```

1. Se o recuo do snippet YAML estiver com erros (sublinhado ondulado vermelho), corrija-o adicionando dois espaços ou uma tabulação para posicionar o snippet corretamente.
1. Com ambos os trechos adicionados ao pipeline de CI/CD, **salve** as alterações.
1. Depois de salvo, clique em **Executar** para acionar o pipeline.
1. Confirme o branch (main) e clique no botão **Executar** para iniciar a execução do pipeline.
1. Na página de status do pipeline, clique na fase **Compilar** para abrir os detalhes do log das diferentes tarefas no pipeline.
1. Aguarde até que o pipeline inicie a Fase Compilar e chegue à tarefa **AzureLoadTest** no fluxo do pipeline.
1. Enquanto a tarefa estiver em execução, navegue até o **Teste de Carga do Azure** no Portal do Azure e veja como o pipeline cria um novo RunTest, chamado **adoloadtest1**. Você pode selecioná-lo para mostrar os valores de resultado do trabalho TestRun.
1. Navegue de volta para a exibição Execução de Pipeline de CI/CD do Azure DevOps, em que a **tarefa AzureLoadTest** foi concluída com êxito. Os valores resultantes do teste de carga também ficarão visíveis na saída de log detalhado.

   ```text
   Task         : Azure Load Testing
   Description  : Automate performance regression testing with Azure Load Testing
   Version      : 1.2.30
   Author       : Microsoft Corporation
   Help         : https://docs.microsoft.com/azure/load-testing/tutorial-cicd-azure-pipelines#azure-load-testing-task
   ==============================================================================
   Test '0d295119-12d0-482d-94be-a7b84787c004' already exists
   Uploaded test plan for the test
   Creating and running a testRun for the test
   View the load test run in progress at: https://portal.azure.com/#blade/Microsoft_Azure_CloudNativeTesting/NewReport//resourceId/%2fsubscriptions%4b75-a1e0-27fb2ea7f9f4%2fresourcegroups%2faz400m08l14-RG%2fproviders%2fmicrosoft.loadtestservice%2floadtests%2feshoponwebloadtesting/testId/0d295119-12d0-787c004/testRunId/161046f1-d2d3-46f7-9d2b-c8a09478ce4c
   TestRun completed

   -------------------Summary ---------------
   TestRun start time: Mon Jul 24 2023 21:46:26 GMT+0000 (Coordinated Universal Time)
   TestRun end time: Mon Jul 24 2023 21:51:50 GMT+0000 (Coordinated Universal Time)
   Virtual Users: 50
   TestStatus: DONE

   ------------------Client-side metrics------------

   Homepage
   response time         : avg=1359ms min=59ms med=539ms max=16629ms p(90)=3127ms p(95)=5478ms p(99)=13878ms
   requests per sec      : avg=37
   total requests        : 4500
   total errors          : 0
   total error rate      : 0
   Finishing: AzureLoadTest

   ```

1. Você executou um teste de carga automatizado como parte de uma execução de pipeline. Na última tarefa, você especificará condições para falha, ou seja, não permitiremos que a Fase de implantação seja iniciada se o desempenho do aplicativo Web estiver abaixo de um determinado limite.

#### Tarefa 5: adicionar critérios de falha/êxito ao pipeline de teste de carga

Nesta tarefa, você usará critérios de falha de teste de carga para receber alertas (execução de pipeline com falha como resultado) quando o aplicativo não atender aos seus requisitos de qualidade.

1. No Azure DevOps, navegue até o projeto eShopOnWeb e abra **Repos**.
1. Em Repos, navegue até a subpasta **/tests/jmeter** criada e usada anteriormente.
1. Abra o arquivo \*config.yaml* de Teste de Carga. Clique em **Editar\*\* para permitir a edição do arquivo.
1. Substitua `failureCriteria: []` pelo seguinte snippet de código:

   ```text
   failureCriteria:
     - avg(response_time_ms) > 300
     - percentage(error) > 50
   ```

1. Salve as alterações no config.yaml clicando em **Confirmar** e Confirmar mais uma vez.
1. Navegue de volta para **Pipelines** e execute o pipeline **eShopOnWeb** novamente. Após alguns minutos, ele concluirá a execução com um status **falhou** para a tarefa **AzureLoadTest**.
1. Abra a exibição de log detalhado para o pipeline e valide os detalhes do **AzureLoadtest**. Um exemplo de saída semelhante é mostrado abaixo:

   ```text
   Creating and running a testRun for the test
   View the load test run in progress at: https://portal.azure.com/#blade/Microsoft_Azure_CloudNativeTesting/NewReport//resourceId/%2fsubscriptions%2fb86d9ae1-7552-47fb2ea7f9f4%2fresourcegroups%2faz400m08l14-RG%2fproviders%2fmicrosoft.loadtestservice%2floadtests%2feshoponwebloadtesting/testId/0d295119-12d0-a7b84787c004/testRunId/f4bec76a-8b49-44ee-a388-12af34f0d4ec
   TestRun completed

   -------------------Summary ---------------
   TestRun start time: Mon Jul 24 2023 23:00:31 GMT+0000 (Coordinated Universal Time)
   TestRun end time: Mon Jul 24 2023 23:06:02 GMT+0000 (Coordinated Universal Time)
   Virtual Users: 50
   TestStatus: DONE

   -------------------Test Criteria ---------------
   Results           :1 Pass 1 Fail

   Criteria                  :Actual Value        Result
   avg(response_time_ms) > 300                       1355.29               FAILED
   percentage(error) > 50                                                  PASSED


   ------------------Client-side metrics------------

   Homepage
   response time         : avg=1355ms min=58ms med=666ms max=16524ms p(90)=2472ms p(95)=5819ms p(99)=13657ms
   requests per sec      : avg=37
   total requests        : 4531
   total errors          : 0
   total error rate      : 0
   ##[error]TestResult: FAILED
   Finishing: AzureLoadTest

   ```

1. Como a última linha da saída do teste de carga diz **##[error]TestResult: FAILED**; como definimos um **FailCriteria** com um tempo médio de resposta de > de 300 ou uma porcentagem de erro de > de 20, agora vendo um tempo médio de resposta superior a 300, a tarefa será sinalizada como falha.

   > **Observação**: imagine que, em um cenário da vida real, você validaria o desempenho do Serviço de Aplicativo e, se o desempenho estivesse abaixo de um determinado limite, o que normalmente significa que há mais carga no Aplicativo Web, você poderia disparar uma nova implantação para um Serviço de Aplicativo do Azure adicional. Como não podemos controlar o tempo de resposta para ambientes de laboratório do Azure, decidimos reverter a lógica para garantir a falha.

1. O status FAILED da tarefa de pipeline, na verdade, reflete um ÊXITO da validação dos critérios de requisito do Teste de Carga do Azure.

   > [!IMPORTANT]
   > Lembre-se de excluir os recursos criados no portal do Azure para evitar cobranças desnecessárias.

## Revisão

Neste exercício, você implantou um aplicativo Web no Serviço de Aplicativo do Azure usando o Azure Pipelines, bem como implantando um Recurso de Teste de Carga do Azure com TestRuns. Em seguida, você integrou o arquivo config.yaml do teste de carga JMeter ao controle do código-fonte do Azure Repos, estendendo o pipeline de CI/CD com o Teste de Carga do Azure. No último exercício, você aprendeu a definir os critérios de sucesso do LoadTest.
