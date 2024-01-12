---
lab:
  title: Controlling Deployments using Release Gates
  module: 'Module 04: Design and implement a release strategy'
---

# Controlling Deployments using Release Gates

# Manual de laboratório do aluno

## Requisitos do laboratório

- Este laboratório requer o **Microsoft Edge** ou um [navegador compatível com o Azure DevOps](https://docs.microsoft.com/en-us/azure/devops/server/compatibility?view=azure-devops#web-portal-supported-browsers).

- **Configurar uma organização do Azure DevOps:** se você ainda não tiver uma organização Azure DevOps que possa usar para este laboratório, crie uma seguindo as instruções disponíveis em [Criar uma organização ou coleção de projetos](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/create-organization?view=azure-devops).

- Identifique uma assinatura existente do Azure ou crie uma.

- Verifique se você tem uma conta Microsoft ou do Azure AD com a função Proprietário na assinatura do Azure e a função Administrador Global no locatário do Azure AD associado à assinatura do Azure. Para obter detalhes, veja [Listar atribuições de função do Azure usando o portal do Azure](https://docs.microsoft.com/en-us/azure/role-based-access-control/role-assignments-list-portal) e [Exibir e atribuir funções de administrador no Azure Active Directory](https://docs.microsoft.com/en-us/azure/active-directory/roles/manage-roles-portal#view-my-roles).

## Visão geral do laboratório

Este laboratório aborda a configuração dos portões de implantação e detalha como usá-los para controlar a execução do Azure Pipelines. Para ilustrar sua implementação, você configurará uma definição da versão com dois ambientes para um Aplicativo Web do Azure. Você será implantado no ambiente canário somente quando não houver bugs de bloqueio para o aplicativo e marcar o ambiente canário concluído somente quando não houver alertas ativos nos Insights do Aplicativo do Azure Monitor.

Um pipeline de lançamento especifica o processo de lançamento de ponta a ponta para que um aplicativo seja implantado em vários ambientes. As implantações em cada ambiente são totalmente automatizadas usando trabalhos e tarefas. O ideal é que você não queira que novas atualizações para os aplicativos sejam expostas simultaneamente a todos os usuários. É uma prática recomendada expor as atualizações em fases, ou seja, expor a um subconjunto de usuários, monitorar seu uso e expor a outros usuários com base na experiência do conjunto inicial de usuários.

As aprovações e os portões oferecem mais controle sobre o início e a conclusão das implantações em uma versão. Você pode aguardar que os usuários aprovem ou rejeitem implantações com aprovações manualmente. Usando portões de versão, você pode especificar critérios de integridade do aplicativo a serem atendidos antes que a versão seja promovida para o ambiente a seguir. Antes ou depois de qualquer implantação de ambiente, todos os portões especificados são avaliados automaticamente até que passem ou atinjam o período de tempo limite definido e falhem.

Os portões podem ser adicionados a um ambiente na definição da versão das condições de pré-implantação ou do painel de condições de pós-implantação. Várias portas podem ser adicionadas às condições de ambiente para garantir que todas as entradas sejam bem-sucedidas para a versão.

Por exemplo:

- Os portões de pré-implantação garantirá nenhum problema ativo no item de trabalho ou no sistema de gerenciamento de problemas antes de implantar um build em um ambiente.
- Os portões de pós-implantação garantirá nenhum incidente do sistema de monitoramento ou gerenciamento de incidentes do aplicativo depois de serem implantados antes de promover a versão para o ambiente a seguir.

Há quatro tipos de portões incluídos por padrão em cada conta.

- Invocar função do Azure: dispare a execução de uma função do Azure e garanta uma conclusão bem-sucedida.
- Consultar alertas do Azure Monitor: observe as regras de alerta do Azure Monitor configuradas para os alertas ativos.
- Invocar a API REST: faça uma chamada em uma API REST e continue se ela retornar uma resposta bem-sucedida.
- Consultar itens de trabalho: verifique se o número de itens de trabalho correspondentes retornados de uma consulta está dentro de um limite.

## Objetivos

Após concluir este laboratório, você poderá:

- Configurar pipelines de lançamento.
- Configurar os portões de versão.
- Testar os portões de versão.

## Tempo estimado: 75 minutos

## Instruções

### Exercício 0: configurar os pré-requisitos do laboratório

> **Observação**: se você já criou este projeto durante laboratórios anteriores, este exercício pode ser ignorado.

Neste exercício, você configurará os pré-requisitos para o laboratório, que consistem em um novo projeto do Azure DevOps com um repositório baseado no [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb). 

#### Tarefa 1: (ignorar se já tiver concluído) criar e configurar o projeto de equipe

Nesta tarefa, você criará um projeto do Azure DevOps com base no **eShopOnWeb** para ser usado por vários laboratórios.

1.  No computador do laboratório, em uma janela do navegador, abra sua organização do Azure DevOps. Clique em **Novo projeto**. Dê ao seu projeto o nome **eShopOnWeb** e deixe os outros campos com padrões. Clique em **Criar**.

    ![Criar Projeto](images/create-project.png)

#### Tarefa 2: (ignorar se já tiver concluído) importar repositório Git do eShopOnWeb

Nesta tarefa você importará o repositório Git do eShopOnWeb que será usado por vários laboratórios.

1.  No computador do laboratório, em uma janela do navegador, abra a organização do Azure DevOps e o projeto **eShopOnWeb** criado anteriormente. Clique em **Repos>Arquivos**, **Importar um Repositório**. Selecione **Importar**. Na janela **Importar um repositório Git**, cole a seguinte URL https://github.com/MicrosoftLearning/eShopOnWeb.git e clique em **Importar**:

    ![Importar repositório](images/import-repo.png)

1.  O repositório está organizado da seguinte forma:
    - A pasta **.ado** contém os pipelines YAML do Azure DevOps
    - O contêiner da pasta **.devcontainer** está configurado para o desenvolvimento usando contêineres (localmente no VS Code ou no GitHub Codespaces)
    - A pasta **.azure** contém a infraestrutura Bicep&ARM como modelos de código usados em alguns cenários do laboratório.
    - O contêiner da pasta **.github** contém as definições de fluxo de trabalho do GitHub YAML.
    - A pasta **src** contém o site .NET 6 usado nos cenários do laboratório.

#### Tarefa 3: (ignorar se já tiver concluído) configurar o pipeline de CI como código com YAML no Azure DevOps.

Nesta tarefa, você adicionará uma definição de build do YAML ao projeto.

1. Volte para o painel **Pipelines** no hub **Pipelines**.
1. Na janela **Criar seu primeiro pipeline **, clique em **Criar pipeline**.

    > **Observação**: usaremos o assistente para criar uma nova definição de pipeline YAML com base no projeto.

1. Na página **Onde está seu código?**, clique na opção **Git do Azure Repos (YAML)**.
1. No painel **Selecionar um repositório**, clique em **eShopOnWeb**.
1. No painel **Configurar o pipeline**, role para baixo e selecione **Arquivo YAML do Azure Pipelines existente**.
1. Na folha **Selecionar um Arquivo YAML existente**, especifique os seguintes parâmetros:
- Branch: **principal**
- Caminho: **.ado/eshoponWeb-ci.yml**
1. Clique em **Continuar** para salvar essas configurações.
1. Na tela **Revisar seu Pipeline YAML**, clique em **Executar** para iniciar o processo de pipeline de build.
1. Aguarde o pipeline de build ser concluído com sucesso. Ignore os avisos sobre o código-fonte, pois eles não são relevantes para este exercício de laboratório.

    > **Observação**: cada tarefa do arquivo YAML está disponível para revisão, incluindo avisos e erros.

### Exercício 2: criar os recursos do Azure necessários para o pipeline de lançamento

#### Tarefa 1: criar dois aplicativos Web do Azure

Nesta tarefa, você criará dois aplicativos Web do Azure representando os ambientes **Canário** e de **Produção**, nos quais implantará o aplicativo por meio do Azure Pipelines.

1. No computador do laboratório, inicie um navegador da Web, navegue até o [**Portal do Azure**](https://portal.azure.com) e entre com a conta de usuário que tem a função de proprietário na assinatura do Azure que você usará neste laboratório e tem a função de administrador global no locatário do Azure AD associado a essa assinatura.
1. No portal do Azure, clique no ícone do **Cloud Shell**, localizado à direita da caixa de texto de pesquisa na parte superior da página.
1. Se for solicitado que você selecione **Bash** ou **PowerShell**, selecione **Bash**.

    >**Observação**: se esta for a primeira vez que você está iniciando o **Cloud Shell** e você receber a mensagem **Você não tem nenhum armazenamento montado**, selecione a assinatura que você está usando no laboratório e selecione **Criar armazenamento**.

1. No prompt **Bash**, no painel do **Cloud Shell**, execute o seguinte comando para criar um grupo de recursos (substitua o espaço reservado variável `<region>` pelo nome da região do Azure que hospedará os dois aplicativos Web do Azure, por exemplo, "westeurope" ou "centralus" ou qualquer outra região disponível de sua escolha):

    > **Observação**: localizações possíveis podem ser encontradas executando o seguinte comando, use o **Nome** em `<region>` : `az account list-locations -o table`.

    ```bash
    REGION='centralus'
    RESOURCEGROUPNAME='az400m04l09-RG'
    az group create -n $RESOURCEGROUPNAME -l $REGION
    ```

1. Criar um plano de serviço de aplicativo

    ```bash
    SERVICEPLANNAME='az400m04l09-sp1'
    az appservice plan create -g $RESOURCEGROUPNAME -n $SERVICEPLANNAME --sku S1
    ```

1.  Crie dois aplicativos Web com nomes de aplicativos exclusivos.
 
     ```bash
     SUFFIX=$RANDOM$RANDOM
     az webapp create -g $RESOURCEGROUPNAME -p $SERVICEPLANNAME -n RGATES$SUFFIX-Canary
     az webapp create -g $RESOURCEGROUPNAME -p $SERVICEPLANNAME -n RGATES$SUFFIX-Prod
     ```

    > **Observação**: registre o nome do aplicativo Web como Canário. Você precisará dela mais adiante neste laboratório.

1. Aguarde até que o processo de provisionamento dos recursos dos serviços do aplicativo Web seja concluído e feche o painel do **Cloud Shell**.

#### Tarefa 2: conectar-se a um recurso do Application Insights

1. No portal do Azure, use a caixa de texto **Recursos de pesquisa, serviços e documentos** na parte superior da página para localizar o **Application Insights** e, na lista de resultados, selecione **Application Insights**.
1. Na folha **Application Insights**, selecione **+ Criar**.
1. Na folha **Application Insights**, na guia **Noções básicas**, especifique as seguintes configurações (deixe as outras com os valores padrão):

    | Configuração | Valor |
    | --- | --- |
    | Grupo de recursos | **az400m04l09-RG** |
    | Nome | o nome do aplicativo Web Canário que você registrou na tarefa anterior |
    | Região | a mesma região do Azure na qual você implantou os aplicativos Web na tarefa anterior |
    | Modo de Recurso | **Clássico** |

    > **Observação**: desconsidere a mensagem de depreciação. Isso é necessário para evitar falhas da tarefa Habilitar DevOps de Integração Contínua que você usará posteriormente neste laboratório.

1. Clique em **Revisar + criar** e em **Criar**.
1. Aguarde o processo de provisionamento ser concluído.
1. No portal do Azure, navegue até o grupo de recursos **az400m04l09-RG** criado na tarefa anterior.
1. Na lista de recursos, clique no aplicativo Web **Canário**.
1. Na página do aplicativo Web **canário** no menu vertical à esquerda, na seção **Configurações** , clique em **Application Insights**.
1. No folha do **Application Insights**, clique em **Ativar o Application Insights**.
1. Na seção **Alterar seu recurso**, clique na opção **Selecionar recurso existente**, na lista de recursos existentes, selecione o recurso recém-criado do Application Insights, clique em **Aplicar** e, quando solicitado para confirmação, clique em **Sim**.
1. Aguarde até que a alteração entre em vigor.

    > **Observação**: você criará alertas de monitor aqui que serão usados mais adiante neste laboratório.
1. Na mesma opção do menu **Configurações** /  do** Application Insights** no Aplicativo Web, selecione **Exibir Dados do Application Insights**. Essa ação redireciona você à folha do Application Insights no portal do Azure.
1.  Na folha de recursos do Application Insights, na seção **Monitoramento**, clique em **Alertas** e em **Criar > regra de alerta**.
1.  Na folha **Selecionar um sinal**, na caixa de texto **Pesquisar por nome de sinal**, digite **Solicitações**. Na lista de resultados, selecione **Solicitações com Falha**. 
1.  Na folha **Criar uma Regra de Alerta**, na seção **Condição**, deixe o **Limite** definido como **Estático** e valide as outras configurações padrão da seguinte maneira:
- Tipo de agregação: contagem
- Operador: maior que
- Unidade: Contagem
1. Na caixa de texto **Valor do limite**, digite **0** e clique em **Próximo: Ações**. Não faça alterações na folha de configurações de **Ações** e defina os seguintes parâmetros na seção **Detalhes**:

    | Configuração | Valor |
    | --- | --- |
    | Gravidade | **2- Aviso** |
    | Nome da regra de alerta | **RGATESCanary_FailedRequests** |
    | Opções avançadas: resolva alertas automaticamente | **desmarcado** |

    > **Observação**: as regras de alerta de métrica podem levar até 10 minutos para serem ativadas.

    > **Observação**: você pode criar várias regras de alerta em métricas diferentes, como disponibilidade maior que 99%, tempo de resposta do servidor menor que 5 segundos ou exceções de servidor maior que 0.

1. Confirme a criação da regra de alerta clicando em **Revisar+Criar** e confirme mais uma vez clicando em **Criar**. Aguarde até que a regra de alerta seja criada com êxito.

### Exercício 3: configurar o pipeline de lançamento.

Neste exercício, você configurará um pipeline de lançamento.

#### Tarefa 1: configurar tarefas de lançamento

Nesta tarefa, você configurará as tarefas de lançamento como parte do pipeline de lançamento.

1. No projeto **eShopOnWeb** no portal do Azure DevOps, no painel de navegação vertical, selecione **Pipelines** e, na seção **Pipelines**, clique em **Versão**.
1. Clique em **Novo pipeline**.
1. Na janela **Selecionar um modelo**, **escolha** **Implantação do Azure App Service** (Implantar seu aplicativo no Azure App Service. Escolha entre aplicativo Web no Windows, Linux, contêineres, aplicativos de funções ou WebJobs) na lista de modelos **Em destaque**.
1. Clique em **Aplicar**.
1. Na janela **Estágio**, atualize o nome do estágio "Estágio 1" padrão para **Canário**. Feche a janela pop-up usando o botão **X**. Agora você está no editor gráfico do pipeline de lançamento, que mostra o estágio Canário.
1. Passe o mouse sobre o estágio Canário e clique no botão **Clonar** para copiá-lo para um estágio adicional. Nomeie este estágio de **Produção**.

    > **Observação**: o pipeline agora contém dois estágios com o nome **Canário** e **Produção**.

1. Na guia **Pipeline**, selecione o retângulo **Adicionar um Artefato** e selecione o **eShopOnWeb** no campo **Fonte (pipeline de build)**. Clique em **Adicionar** para confirmar a seleção do artefato.
1. No retângulo **Artefatos**, observe o **Gatilho de Integração Contínua** (formato de raio). Clique nele para abrir as configurações do **gatilho de Implantação contínua**. Clique no gatilho de implantação contínua para alternar a opção para habilitá-lo. Deixe todas as outras configurações como padrão e feche o painel do ** gatilho de Implantação contínua **, clicando no **X** no canto superior direito.
1. No estágio de **Ambientes Canário**, clique na etiqueta **1 trabalho, 2 tarefas** e revise as tarefas desse estágio.

    > **Observação**: o ambiente Canário tem uma tarefa que, respectivamente, publica o pacote de artefatos no aplicativo Web do Azure.

1. No painel **Todos os pipelines > Novo pipeline de lançamento**, verifique se o estágio **Canário** está selecionado. Na lista suspensa** Assinatura do Azure**, selecione sua assinatura do Azure e clique em **Autorizar**. Se solicitado, faça autenticação com a conta de usuário com a função de proprietário na assinatura do Azure.
1. Confirme se o tipo de aplicativo está definido como "Aplicativo Web no Windows". Em seguida, na lista suspensa **Nome do Serviço de Aplicativo**, selecione o nome do aplicativo Web **Canário**.
1. Selecione a tarefa **Implantar Azure App Service**. No campo **Pacote ou Pasta**, atualize o valor padrão de "$(System.DefaultWorkingDirectory)/\*\*/\*.zip" para "$(System.DefaultWorkingDirectory)/\*\*/Web.zip"

    > Observe um ponto de exclamação ao lado da guia Tarefas. Isso é esperado, pois precisamos definir as configurações para o estágio de produção.

1. No painel **Todos os pipelines > Novo pipeline de lançamento**, navegue até a guia **Pipeline** e, desta vez, no estágio de **Produção**, clique na etiqueta **1 trabalho, 2 tarefas**. Semelhante ao estágio Canário anterior, conclua as configurações de pipeline. Na guia Tarefas / Processo de Implantação de Produção, na lista suspensa **Assinatura do Azure**, selecione a assinatura do Azure que você usou para o estágio de **Ambiente Canário**, exibida em **Conexões de Serviço do Azure Disponíveis**, pois já criamos a conexão de serviço antes ao autorizar o uso da assinatura.
1. Na lista suspensa **Nome do Serviço de Aplicativo**, selecione o nome do aplicativo Web **Prod**.
1. Selecione a tarefa **Implantar Azure App Service**. No campo **Pacote ou Pasta**, atualize o valor padrão de "$(System.DefaultWorkingDirectory)/\*\*/\*.zip" para "$(System.DefaultWorkingDirectory)/\*\*/Web.zip" 
1. No painel **Todos os pipelines > Novo pipeline de lançamento**, clique em **Salvar** e, na caixa de diálogo **Salvar**, clique em **OK**.

Agora você configurou com êxito o pipeline de lançamento.

1. Na janela do navegador que exibe o projeto ** eShopOnWeb**, no painel de navegação vertical, na seção **Pipelines**, clique em **Pipelines**.
1. No painel **Pipelines**, clique na entrada que representa o pipeline de build **eShopOnWeb** e, no painel **eShopOnWeb**, clique em **Executar Pipeline**.
1. No painel **Executar Pipeline**, aceite as configurações padrão e clique em **Executar** para disparar o pipeline. **Aguarde a conclusão do pipeline de build**.

    > **Observação**: depois que a build for bem-sucedida, a versão será acionada automaticamente e o aplicativo será implantado em ambos os ambientes. Valide as ações da versão, uma vez que o pipeline de build foi concluído com êxito.

1. No painel de navegação vertical, na seção **Pipelines**, clique em **Versões** e, no painel **eShopOnWeb**, clique na entrada que representa a versão mais recente.
1. Na folha **eShopOnWeb > Versão-1**, acompanhe o progresso da versão e verifique se a implantação em ambos os aplicativos Web foi concluída com êxito.
1. Alterne para a interface do portal do Azure, navegue até o grupo de recursos **az400m04l09-RG**, na lista de recursos, clique no aplicativo Web **Canário**, na folha do aplicativo Web, clique em **Procurar** e verifique se a página da Web (site de comércio eletrônico) é carregada com êxito em uma nova guia do navegador da Web.
1. Alterne para a interface do portal do Azure, desta vez indo até o grupo de recursos **az400m04l09-RG**, na lista de recursos, clique no aplicativo Web **Produção**, na folha do aplicativo Web, clique em **Procurar** e verifique se a página da Web é carregada com êxito em uma nova guia do navegador da Web.
1. Feche a guia do navegador com o site da Web do **EShopOnWeb**.

    > **Observação**: agora você tem o aplicativo com CI/CD configurado. No próximo exercício, configuraremos o portão de qualidade como parte de um pipeline de lançamento mais avançado.

### Exercício 4: configurar os portões de liberação

Neste exercício, você configurará os portões de qualidade no pipeline de lançamento.

#### Tarefa 1: configurar portões de pré-implantação para aprovações

Nesta tarefa, você configurará portões de pré-implantação.

1. Alterne para a janela do navegador da Web do portal do Azure DevOps e o projeto **eShopOnWeb**. No painel de navegação vertical, na seção **Pipelines**, clique em **Versões** e, no painel **Novo pipeline de lançamento**, clique em **Editar**.
1. No painel **Todos os pipelines > Novo pipeline de lançamento**, na borda esquerda do retângulo que representa o estágio de **Ambiente Canário**, clique na forma oval que representa as **Condições de pré-implantação**.
1. No painel **Condições de pré-implantação**, defina o controle deslizante **Aprovações de pré-implantação** como **Habilitado** e, na caixa de texto **Aprovadores**, digite e selecione o nome da sua conta do Azure DevOps.

    > **Observação**: em um cenário real, esse deve ser um alias de nome da equipe de DevOps em vez do seu nome.

1. **Salve** as configurações de pré-aprovação e feche a janela pop-up.
1. Clique em **Criar Versão** e confirme pressionando o botão **Criar** na janela pop-up.
1. Observe a mensagem de confirmação verde, que diz que "Versão-2" foi criada. Clique no link de "Versão-2" para navegar até os detalhes.
1. Observe que o estágio **Canário** está em um estado de **Aprovação Pendente**. Clique no botão **Aprovar**. Isso inicia o estágio Canário novamente.

#### Tarefa 2: configurar portões de pós-implantação para o Azure Monitor

Nesta tarefa, você habilitará o portão de pós-implantação para o ambiente Canário.

1.  No painel **Todos os pipelines > Novo pipeline de lançamento**, na borda direita do retângulo que representa o estágio de **Ambiente Canário**, clique na forma oval que representa as **Condições de pós-implantação**.
1.  No painel **Condições de pós-implantação**, defina o controle deslizante **Portões** como **Habilitado**, clique em **+ Adicionar** e, no menu pop-up, clique em **Consultar Alertas do Azure Monitor**.
1.  No painel **Condições de pós-implantação**, na seção **Consultar Alertas do Azure Monitor**, na lista suspensa **Assinatura do Azure**, selecione a entrada de **conexão de serviço** que representa a conexão com sua assinatura do Azure e, na lista suspensa **Grupo de recursos**, selecione a entrada **az400m04l09-RG**.
1. No painel **Condições de pós-implantação**, expanda a seção **Avançado** e configure as seguintes opções:

- Tipo de filtro: **nenhum**
- Gravidade: **Sev0, Sev1, Sev2, Sev3, Sev4**
- Intervalo de tempo: **Última hora**
- Estado de alerta: **Reconhecido, Novo**
- Condição do monitor: **Acionado**

1.  No painel **Condições de pós-implantação**, expanda as opções de **Avaliação** e configure as seguintes opções:

- Defina o valor de **Tempo entre a reavaliação dos portões** como **5 Minutos**.
- Defina o valor de **Tempo limite após falha nos portões** como **8 Minutos**.
- Selecione a opção **Em portais bem-sucedidos, solicitar aprovações**.

    > **Observação**: o intervalo de amostragem e o tempo limite funcionam juntos para que os portões chamem as funções em intervalos adequados e rejeitem a implantação se não forem bem-sucedidos durante o mesmo intervalo de amostragem dentro do período do tempo limite.

1. Feche o painel **Condições de pós-implantação**, clicando no **X** no canto superior direito.
1. De volta ao painel **Novo pipeline de lançamento**, clique em **Salvar** e, na caixa de diálogo **Salvar**, clique em **OK**.

### Exercício 5: testar os portões de liberação

Neste exercício, você testará os portões de liberação atualizando o aplicativo, o que acionará uma implantação.

#### Tarefa 1: atualizar e implantar o aplicativo após adicionar portões de liberação

Nesta tarefa, você primeiro gerará alguns alertas para o aplicativo Web Canário, seguido pelo acompanhamento do processo de liberação com os portões de liberação habilitados.

1. No portal do Azure, navegue até o recurso do **Aplicativo Web Canário** implantado anteriormente.
1. No painel Visão geral, observe o campo **URL** que exibe o Hiperlink do aplicativo Web. Clique nesse link. Você será redirecionado para o aplicativo Web EShopOnWeb no navegador.
1. Para simular uma **Solicitação com falha** adicione **/discount** à URL, o que resultará em uma mensagem de erro, uma vez que essa página não existe. Atualize a página várias vezes para gerar vários eventos.
1. No portal do Azure, no campo "Pesquisar recursos, serviços e documentos", digite **Application Insights** e selecione o **Canário-AppInsights** criado no exercício anterior. Em seguida, vá até **Alertas**. 
1. Deve haver pelo menos **1** novo alerta na lista de resultados, com uma **Gravidade 2**, digite **Alertas** para abrir o Serviço de Alertas do Azure Monitor.
1. Observe que deve haver pelo menos **1** Failed_Alert** com **Gravidade 2 – Aviso** aparecendo na lista. Isso foi disparado quando você validou o endereço URL do site não existente no exercício anterior.

    > **Observação:** se nenhum alerta aparecer ainda, aguarde mais alguns minutos. 

1. Volte para o portal do Azure DevOps e abra o projeto **EShopOnWeb-Release Gates**. Navegue até **Pipelines**, selecione **Versões** e selecione o **Novo pipeline de lançamento**.
1. Clique no botão **Criar Versão**.
1. Aguarde o início do pipeline de lançamento e **aprove** a ação de lançamento do estágio Canário.
1. Aguarde o estágio da versão Canário ser concluído com sucesso. Observe como os **Portões de pós-implantação** estão mudando para um status de **Portões de Avaliação**.  Clique no ícone **Portões de Avaliação**.
1. Em **Consultar Alertas do Azure Monitor**, observe um estado inicial com falha.
1. Deixe o pipeline de lançamento em um estado pendente pelos próximos 5 minutos. Depois de 5 minutos, repare novamente na falha da 2ª avaliação.
1. Esse é o comportamento esperado, já que existem alertas do Application Insights disparados para o aplicativo Web Canário.

    > **Observação**: como há um alerta disparado pela exceção, o portão **Consultar o Azure Monitor** falhará. Isso, por sua vez, impedirá a implantação no ambiente de **Produção**.

1. Aguarde mais 3 minutos e valide o status dos portões de liberação novamente. Como agora são mais de 8 minutos após a verificação dos portões de liberação iniciai e já se passaram mais de 8 minutos desde que o alerta inicial do Application Insights inicial foi disparado com a ação "Acionado", ele deve resultar em um portão de liberação bem-sucedido, permitindo também a implantação do estágio da versão de produção.

### Exercício 6: Remover os recursos do laboratório do Azure

Neste exercício, você removerá os recursos do Azure provisionados neste laboratório para eliminar cobranças inesperadas.

>**Observação**: lembre-se de remover todos os recursos do Azure recém-criados e que você não usa mais. Remover recursos não utilizados garante que você não veja encargos inesperados.

#### Tarefa 1: remover os recursos do Azure Lab

Nesta tarefa, você usará o Azure Cloud Shell para remover os recursos do Azure provisionados neste laboratório com o objetivo de eliminar cobranças desnecessárias.

1. No portal do Azure, abra a sessão do **Shell Bash** no painel do **Cloud Shell**.
1. Liste todos os grupos de recursos criados em todos os laboratórios deste módulo executando o seguinte comando:

    ```sh
    az group list --query "[?starts_with(name,'az400m04l09-RG')].name" --output tsv
    ```

1. Exclua todos os grupos de recursos criados em todos os laboratórios deste módulo executando o seguinte comando:

    ```sh
    az group list --query "[?starts_with(name,'az400m04l09-RG')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**Observação**: o comando é executado de modo assíncrono (conforme determinado pelo parâmetro --nowait), portanto, embora você possa executar outro comando da CLI do Azure imediatamente depois na mesma sessão Bash, levará alguns minutos antes de os grupos de recursos serem removidos.

## Revisão

Neste laboratório, você configurou pipelines de lançamento e configurou e testou portões de liberação.
