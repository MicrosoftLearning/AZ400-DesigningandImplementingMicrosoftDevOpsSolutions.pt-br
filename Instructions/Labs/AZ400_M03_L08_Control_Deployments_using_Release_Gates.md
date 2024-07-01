---
lab:
  title: Controlar as implantações usando portões de lançamento
  module: 'Module 03: Design and implement a release strategy'
---

# Controlar as implantações usando portões de lançamento

## Manual de laboratório do aluno

## Requisitos do laboratório

- Este laboratório requer o **Microsoft Edge** ou um [navegador com suporte do Azure DevOps.](https://docs.microsoft.com/azure/devops/server/compatibility)

- **Configurar uma organização do Azure DevOps:** se você ainda não tiver uma organização Azure DevOps que possa usar para este laboratório, crie uma seguindo as instruções disponíveis em [Criar uma organização ou coleção de projetos](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization).

- Identifique uma assinatura existente do Azure ou crie uma.

- Verifique se você tem uma conta Microsoft ou uma conta do Microsoft Entra com a função Proprietário na assinatura do Azure e a função Administrador Global no locatário do Microsoft Entra associado à assinatura do Azure. Para obter detalhes, veja [Listar designações de função do Azure usando o portal do Azure](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-list-portal) e [Exibir e designar funções de administrador no Azure Active Directory](https://docs.microsoft.com/azure/active-directory/roles/manage-roles-portal).

## Visão geral do laboratório

Este laboratório aborda a configuração dos portões de implantação e detalha como usá-los para controlar a execução do Azure Pipelines. Para ilustrar sua implementação, você configurará uma definição da versão com dois ambientes para um Aplicativo Web do Azure. Você implantará no ambiente do DevTest somente quando não houver bugs de bloqueio para o aplicativo e marcar o ambiente do DevTest concluído somente quando não houver alertas ativos no Application Insights do Azure Monitor.

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

#### Tarefa 1: (pular se já feita) criar e configurar o projeto de equipe

Nesta tarefa, você criará um projeto **eShopOnWeb** do Azure DevOps para ser usado por vários laboratórios.

1. No computador do laboratório, em uma janela do navegador, abra sua organização do Azure DevOps. Clique em **Novo projeto**. Dê ao seu projeto o nome **eShopOnWeb** e deixe os outros campos com padrões. Clique em **Criar**.

   ![Criar Projeto](images/create-project.png)

#### Tarefa 2: (pular se já feita) importar repositório do Git eShopOnWeb

Nesta tarefa, você importará o repositório eShopOnWeb do Git que será usado por vários laboratórios.

1. No computador do laboratório, em uma janela do navegador, abra sua organização do Azure DevOps e o projeto **eShopOnWeb** criado anteriormente. Clique em **Repos>Arquivos**, **Importar um repositório**. Selecione **Importar**. Na janela **Importar um repositório do Git**, cole a seguinte URL <https://github.com/MicrosoftLearning/eShopOnWeb.git> e clique em **Importar**:

   ![Importar repositório](images/import-repo.png)

1. O repositório está organizado da seguinte forma:
   - A pasta **.ado** contém os pipelines YAML do Azure DevOps.
   - O contêiner da pasta **.devcontainer** está configurado para o desenvolvimento usando contêineres (localmente no VS Code ou no GitHub Codespaces).
   - A pasta **infra** contém a infraestrutura Bicep e ARM como modelos de código usados em alguns cenários de laboratório.
   - A pasta **.github** contém definições de fluxo de trabalho YAML do GitHub.
   - A pasta **src** contém o site do .NET 8 usado em cenários de laboratório.

1. Vá para **Repos>Branches**.
1. Passe o mouse sobre o branch **main** e clique nas reticências à direita da coluna.
1. Clique em **Definir como branch padrão**.

#### Tarefa 3: (pular se já feita) configurar pipeline de CI como código com YAML no Azure DevOps

Nesta tarefa, você adicionará uma definição de compilação do YAML ao projeto existente.

1. Navegue de volta ao painel **Pipelines** no hub **Pipelines**.
1. Na janela **Criar seu primeiro pipeline**, clique em **Criar pipeline**.

   > **Observação**: usaremos o assistente para criar uma nova definição de pipeline do YAML com base em nosso projeto.

1. No painel **Onde está seu código?**, clique na opção **Git do Azure Repos (YAML).**
1. No painel **Selecionar um repositório**, clique em **eShopOnWeb**.
1. No painel **Configurar seu pipeline**, role para baixo e selecione **Arquivo YAML existente do Azure Pipelines**.
1. Na folha **Selecionar um arquivo YAML existente** , especifique os seguintes parâmetros:
   - Ramificação: **principal**
   - Caminho: **.ado/eshoponweb-ci.yml**
1. Clique em **Continuar** para salvar essas configurações.
1. Na tela **Revisar seu YAML de pipeline**, clique em **Executar** para iniciar o processo de Pipeline de build.
1. Aguarde a conclusão do pipeline de build. Ignore quaisquer avisos sobre o código-fonte em si, pois eles não são relevantes para este exercício de laboratório.

   > **Observação**: cada tarefa do arquivo YAML está disponível para revisão, incluindo quaisquer avisos e erros.

1. Seu pipeline terá um nome com base no nome do projeto. Vamos **renomear** o pipeline para melhor identificá-lo. Vá até **Pipelines>Pipelines** e clique no pipeline criado recentemente. Clique nas reticências e na opção **Renomear/mover**. Nomeie-o **eshoponweb-ci** e clique em **Salvar**.

### Exercício 2: criar os recursos necessários do Azure para o pipeline de lançamento

#### Tarefa 1: criar dois aplicativos Web do Azure

Nesta tarefa, você criará dois aplicativos Web do Azure que representam os ambientes **DevTest** e **Production**, nos quais você implantará o aplicativo por meio do Azure Pipelines.

1. No computador do laboratório, inicie um navegador da Web, navegue até o [**Portal do Azure**](https://portal.azure.com) e entre com a conta de usuário que tem a função Proprietário na assinatura do Azure que você usará neste laboratório e tem a função de Administrador global no locatário do Microsoft Entra associado a essa assinatura.
1. No portal do Azure, clique no ícone do **Cloud Shell**, localizado diretamente à direita da caixa de texto de pesquisa na parte superior da página.
1. Se for solicitado que você selecione **Bash** ou **PowerShell**, selecione **Bash**.

   > **Observação**: se esta for a primeira vez que você está iniciando o **Cloud Shell** e você receber a mensagem **Você não tem nenhum armazenamento montado**, selecione a assinatura que você está usando no laboratório e selecione **Criar armazenamento**.

1. No prompt **Bash**, no painel **Cloud Shell**, execute o seguinte comando para criar um grupo de recursos (substitua o espaço reservado de variável `<region>` pelo nome da região do Azure que hospedará os dois aplicativos Web do Azure, por exemplo, "westeurope" ou "centralus" ou qualquer outra região disponível de sua escolha):

   > **Observação**: possíveis locais podem ser encontrados executando o seguinte comando, use o **Nome** em `<region>` : `az account list-locations -o table`

   ```bash
   REGION='centralus'
   RESOURCEGROUPNAME='az400m04l09-RG'
   az group create -n $RESOURCEGROUPNAME -l $REGION
   ```

1. Para criar um plano do Serviço de Aplicativo

   ```bash
   SERVICEPLANNAME='az400m04l09-sp1'
   az appservice plan create -g $RESOURCEGROUPNAME -n $SERVICEPLANNAME --sku S1
   ```

1. Crie dois aplicativos Web com nomes de aplicativos exclusivos.

   ```bash
   SUFFIX=$RANDOM$RANDOM
   az webapp create -g $RESOURCEGROUPNAME -p $SERVICEPLANNAME -n RGATES$SUFFIX-DevTest
   az webapp create -g $RESOURCEGROUPNAME -p $SERVICEPLANNAME -n RGATES$SUFFIX-Prod
   ```

   > **Observação**: registre o nome do aplicativo Web DevTest. Você precisará dela posteriormente neste laboratório.

1. Aguarde até que o processo de provisionamento dos Recursos de Serviços de Aplicativo Web seja concluído e feche o painel do **Cloud Shell** .

#### Tarefa 2: configurar um recurso do Application Insights

1. No portal do Azure, use a caixa de texto **Procurar recursos, serviços e documentos** na parte superior da página para pesquisar o **Application Insights** e, na lista de resultados, selecione **Application Insights**.
1. Na folha **Application Insights**, selecione **+ Criar**.
1. Na folha **Application Insights**, na guia **Noções básicas**, especifique as seguintes configurações (deixe outras pessoas com seus valores padrão):

   | Configuração        | Valor                                                                                 |
   | -------------- | ------------------------------------------------------------------------------------- |
   | Grupo de recursos | **az400m04l09-RG**                                                                    |
   | Nome           | o nome do aplicativo Web DevTest que você registrou na tarefa anterior                     |
   | Região         | a mesma região do Azure na qual você implantou os aplicativos Web anteriormente na tarefa anterior |
   | Modo de Recurso  | **Clássico**                                                                           |

   > **Observação**: desconsidere a mensagem de substituição. Isso é necessário para evitar falhas da tarefa do DevOps Habilitar integração contínua que você usará posteriormente neste laboratório.

1. Clique em **Revisar + criar** e em **Criar**.
1. Aguarde o processo de provisionamento ser concluído.
1. No portal do Azure, navegue até o grupo de recursos **az400m04l09-RG** que você criou na tarefa anterior.
1. Na lista de recursos, clique no aplicativo Web **DevTest**.
1. Na página do aplicativo Web **DevTest**, no menu vertical à esquerda, na seção **Configurações**, clique em **Application Insights**.
1. Na folha do **Application Insights**, clique em **Ativar o Application Insights**.
1. Na seção **Alterar seu recurso**, clique na opção **Selecionar recurso existente**. Na lista de recursos existentes, selecione o recurso recém-criado do Application Insights, clique em **Aplicar** e, quando solicitado para confirmação, clique em **Sim**.
1. Aguarde até que a alteração entre em vigor.

   > **Observação**: você criará alertas de monitoramento aqui, que serão usados na parte posterior deste laboratório.

1. Na mesma opção do menu **Configurações** /  do **Application Insights** no Aplicativo Web, selecione **Exibir Dados do Application Insights**. A folha do Application Insights abre no portal do Azure.
1. Na folha de recursos do Application Insights, na seção **Monitoramento**, clique em **Alertas** e em **Criar > Regra de alerta**.
1. Na folha **Selecionar um sinal**, na caixa de texto **Pesquisar por nome de sinal**, digite **Solicitações**. Na lista de resultados, selecione **Solicitações com Falha**.
1. Na folha **Criar uma Regra de Alerta**, na seção **Condição**, deixe o **Limite** definido como **Estático**. Valide as outras configurações padrão da seguinte maneira:

   - Tipo de agregação: contagem
   - Operador: maior que
   - Unidade: Contagem

1. Na caixa de texto **Valor do limite**, digite **0** e clique em **Avançar: Ações**. Não faça alterações na folha de configurações de **Ações** e defina os seguintes parâmetros na seção **Detalhes**:

   | Configuração                                        | Valor                            |
   | ---------------------------------------------- | -------------------------------- |
   | Gravidade                                       | **2 – Aviso**                   |
   | Nome da regra de alerta                                | **RGATESDevTest_FailedRequests** |
   | Opções avançadas: resolver alertas automaticamente | **desmarcado**                      |

   > **Observação**: as regras de alerta de métrica podem levar até 10 minutos para ficarem ativas.

   > **Observação**: você pode criar várias regras de alerta em métricas diferentes, como disponibilidade < 99%, tempo de resposta do servidor > 5 segundos ou exceções de servidor > 0

1. Confirme a criação da regra de alerta clicando em **Revisar+Criar** e confirme mais uma vez clicando em **Criar**. Aguarde até que a regra de alerta seja criada com êxito.

### Exercício 3: configurar o pipeline de lançamento

Neste exercício, você configurará um pipeline de lançamento.

#### Tarefa 1: configurar tarefas de lançamento

Nesta tarefa, você configurará tarefas de lançamento como parte do Pipeline de lançamento.

1. No projeto **eShopOnWeb** no portal do Azure DevOps, no painel de navegação vertical, selecione **Pipelines** e, na seção **Pipelines**, clique em **Lançamentos**.
1. Clique em **Novo pipeline**.
1. Na janela **Selecionar um modelo**, **escolha****Implantação do Serviço de Aplicativo do Azure** (Implante seu aplicativo no Serviço de Aplicativo do Azure. Escolha entre Aplicativo Web no Windows, Linux, contêineres, aplicativos de funções ou WebJobs) na lista de modelos **Em destaque**.
1. Clique em **Aplicar**.
1. Na janela **Etapa** exibida, atualize o nome padrão da etapa "Etapa 1" para **DevTest**. Feche a janela pop-up usando o botão **X**. Agora você está no editor gráfico do Pipeline de lançamento, mostrando a Etapa DevTest.
1. Na parte superior da página, renomeie o pipeline atual de **Novo pipeline de lançamento** para **eshoponweb-cd**.
1. Passe o mouse sobre a Etapa DevTest e clique no botão **Clonar** para copiar essa etapa para uma Etapa adicional. Nomeie esta Etapa **Produção**.

   > **Observação**: o pipeline agora contém duas etapas chamadas **DevTest** e **Produção**.

1. Na guia **Pipeline**, selecione o retângulo **Adicionar um Artefato** e selecione o **eshoponweb-ci** no campo **Origem (pipeline de build).** Clique em **Adicionar** para confirmar a seleção do artefato.
1. No retângulo **Artefatos**, observe o **Gatilho de implantação contínua** (raio). Clique nele para abrir as configurações **Gatilho de implantação contínua**. Clique em **Desativado** para alternar a opção e habilitá-la. Deixe todas as outras configurações como padrão e feche o painel **Gatilho de implantação contínua** clicando na marca **x** no canto superior direito.
1. Na etapa **Ambientes do DevTest**, clique na etiqueta **1 trabalho, 1 tarefa** e revise as tarefas nessa etapa.

   > **Observação**: o ambiente DevTest tem 1 tarefa que, respectivamente, publica o pacote de artefatos no Aplicativo Web do Azure.

1. No painel **Todos os pipelines > eshoponweb-cd**, verifique se a etapa **DevTest** está selecionada. Na lista suspensa **Assinatura do Azure**, selecione sua assinatura do Azure e clique em **Autorizar**. Se solicitado, autentique-se usando a conta de usuário com a função Proprietário na assinatura do Azure.
1. Confirme se o Tipo de Aplicativo está definido como "Aplicativo Web no Windows". Em seguida, na lista suspensa **Nome do Serviço de Aplicativo**, selecione o nome do aplicativo Web **DevTest**.
1. Selecione a tarefa **Implantar Serviço de Aplicativo do Azure**. No campo **Pacote ou Pasta**, atualize o valor padrão de "$(System.DefaultWorkingDirectory)/\*\*/\*.zip" para "$(System.DefaultWorkingDirectory)/\*\*/Web.zip"

   > observe um ponto de exclamação ao lado da guia Tarefas. Isso é esperado, pois precisamos definir as configurações para a Etapa de Produção.

1. Abra o painel **Aplicativo e Definições de Configuração** e insira `-UseOnlyInMemoryDatabase true -ASPNETCORE_ENVIRONMENT Development` na caixa **Configurações do Aplicativo**.

1. No painel **Todos os pipelines > eshoponweb-cd**, navegue até a guia **Pipeline** e, desta vez, na Etapa de **Produção**, clique na etiqueta **1 trabalho, 1 tarefa**. Semelhante à etapa DevTest anterior, conclua as configurações do pipeline. Na guia Tarefas / Processo de Implantação de Produção, na lista suspensa **Assinatura do Azure**, selecione a assinatura do Azure que você usou para a Etapa **Ambiente do DevTest**, exibida em **Conexões de Serviço do Azure Disponíveis**, uma vez que já criamos a conexão de serviço antes ao autorizar o uso da assinatura.
1. Na lista suspensa **Nome do Serviço de Aplicativo**, selecione o nome do aplicativo Web de **Produção**.
1. Selecione a tarefa **Implantar Serviço de Aplicativo do Azure**. No campo **Pacote ou Pasta**, atualize o valor padrão de "$(System.DefaultWorkingDirectory)/\*\*/\*.zip" para "$(System.DefaultWorkingDirectory)/\*\*/Web.zip"
1. Abra o painel **Aplicativo e Definições de Configuração** e insira `-UseOnlyInMemoryDatabase true -ASPNETCORE_ENVIRONMENT Development` na caixa **Configurações do Aplicativo**.
1. No painel **Todos os pipelines > eshoponweb-cd**, clique em **Salvar** e, na caixa de diálogo **Salvar**, clique em **OK**.

   Agora você configurou com êxito o Pipeline de lançamento.

1. Na janela do navegador que exibe o projeto **eShopOnWeb**, no painel de navegação vertical, na seção **Pipelines**, clique em **Pipelines**.
1. No painel **Pipelines**, clique na entrada que representa o pipeline de build **eshoponweb-ci** e clique em **Executar Pipeline**.
1. No painel **Executar pipeline**, aceite as configurações padrão e clique em **Executar** para disparar o pipeline. **Aguarde o pipeline de build concluir**.

   > **Observação**: depois que o build for bem-sucedido, o lançamento será acionado automaticamente e o aplicativo será implantado em ambos os ambientes. Valide as ações de lançamento, uma vez que o pipeline de build foi concluído com êxito.

1. No painel de navegação vertical, na seção **Pipelines**, clique em **Lançamentos** e, no **painel eshoponweb-cd**, clique na entrada que representa o lançamento mais recente.
1. Na folha **eshoponweb-cd > Release-1**, acompanhe o progresso do lançamento e verifique se a implantação em ambos os aplicativos Web foi concluída com êxito.
1. Alterne para a interface do portal do Azure, navegue até o grupo**az400m04l09-RG**. Na lista de recursos, clique no aplicativo Web **DevTest**. Na folha do aplicativo Web, clique em **Procurar** e verifique se a página da Web (site de comércio eletrônico) é carregada em uma nova guia do navegador da Web.
1. Volte para a interface do portal do Azure, desta vez navegando até o grupo de recursos **az400m04l09-RG**. Na lista de recursos, clique no aplicativo Web de **Produção**. Na folha do aplicativo Web, clique em **Procurar** e verifique se a página da Web é carregada em uma nova guia do navegador da Web.
1. Feche a guia do navegador da Web que exibe o site **EShopOnWeb**.

   > **Observação**: agora o aplicativo foi configurado com CI/CD. No próximo exercício, configuraremos o Portão de qualidade como parte de um pipeline de lançamento mais avançado.

### Exercício 3: configurar as restrições de lançamento

Neste exercício, você configurará as Restrições de Qualidade no pipeline de lançamento.

#### Tarefa 1: configurar restrições de pré-implantação para aprovações

Nesta tarefa, você configurará portas de pré-implantação.

1. Mude para a janela do navegador da Web que exibe o portal do Azure DevOps e abra o projeto **eShopOnWeb**. No painel de navegação vertical, na seção **Pipelines**, clique em **Lançamentos** e, no painel **eshoponweb-cd**, clique em **Editar**.
1. No painel **Todos os pipelines > eshoponweb-cd**, na borda esquerda do retângulo que representa a etapa **Ambiente do DevTest**, clique na forma oval que representa as **condições de pré-implantação**.
1. No painel **Condições de pré-implantação**, defina o controle deslizante **Aprovações de pré-implantação** como **Habilitado** e, na caixa de texto **Aprovadores**, digite e selecione o nome da sua conta do Azure DevOps.

   > **Observação**: em um cenário da vida real, esse deve ser um alias de nome de equipe de DevOps em vez de seu próprio nome.

1. **Salve** as configurações de pré-aprovação e feche a janela pop-up.
1. Clique em **Criar lançamento** e confirme pressionando o botão **Criar** na janela pop-up.
1. Observe a mensagem de confirmação verde, dizendo que "Release-2" foi criado. Clique no link de "Release-2" para navegar até seus detalhes.
1. Observe que a Etapa **DevTest** está em estado de **Aprovação Pendente**. Clique no botão **Aprovar**. Isso iniciará a Etapa DevTest novamente.

#### Tarefa 2: configurar restrições pós-implantação para o Azure Monitor

Nesta tarefa, você habilitará o portão pós-implantação para o ambiente DevTest.

1. De volta ao painel **Todos os pipelines > eshoponweb-cd**, na borda direita do retângulo que representa a etapa **Ambiente do DevTest**, clique na forma oval que representa as **Condições pós-implantação**.
1. No painel **Condições pós-implantação**, defina o controle deslizante **Portões** como **Habilitado**, clique em **+ Adicionar** e, no menu pop-up, clique em **Consultar Alertas do Azure Monitor**.
1. No painel **Condições pós-implantação**, na seção **Consultar Alertas do Azure Monitor**, na lista suspensa **Assinatura do Azure**, selecione a entrada de **conexão de serviço** que representa a conexão com sua assinatura do Azure; e, na lista suspensa **Grupo de recursos**, selecione a entrada **az400m04l09-RG**.
1. No painel **Condições pós-implantação**, expanda a seção **Avançado** e configure as seguintes opções:

   - Tipo de filtro: **Nenhum**
   - Gravidade: **Sev0, Sev1, Sev2, Sev3, Sev4**
   - Intervalo de tempo: **Última hora**
   - Estado de alerta: **Reconhecido, Novo**
   - Condição do Monitor: **Acionado**

1. No painel **Condições pós-implantação**, expanda as **Opções de avaliação** e configure as seguintes opções:

   - Defina o valor de **Tempo entre a reavaliação dos portões** como **5 Minutos**.
   - Defina o valor de **Tempo Limite após o qual os portões falham** como **8 Minutos**.
   - Selecione a opção **Em portões com êxito, solicitar aprovações**.

   > **Observação**: o intervalo de amostragem e o tempo limite funcionam juntos para que os portões chamem suas funções em intervalos adequados e rejeitem a implantação se não tiverem êxito durante o mesmo intervalo de amostragem dentro do período de tempo limite.

1. Feche o painel **Condições pós-implantação** clicando na marca **x** no canto superior direito.
1. De volta ao painel **eshoponweb-cd**, clique em **Salvar** e, na caixa de diálogo **Salvar**, clique em **OK.**

### Exercício 5: testar os portões de lançamento

Neste exercício, você testará os portões de lançamento atualizando o aplicativo, o que acionará uma implantação.

#### Tarefa 1: atualizar e implantar o aplicativo após adicionar portões de lançamento

Nesta tarefa, você primeiro gerará alguns alertas para o aplicativo Web do DevTest, seguido do acompanhamento do processo de lançamento com os portões de lançamento habilitados.

1. No Portal do Azure, navegue até o Recurso **Aplicativo Web do DevTest** implantado anteriormente.
1. No painel Visão geral, observe o campo **URL** mostrando o Hiperlink do aplicativo Web. Clique neste link, que redirecionará você para a aplicativo Web eShopOnWeb no navegador.
1. Para simular uma **Solicitação com falha**, adicione **/discount** à URL, o que resultará em uma mensagem de erro, já que essa página não existe. Atualize esta página várias vezes para gerar vários eventos.
1. No Portal do Azure, no campo "Pesquisar recursos, serviços e documentos", insira **Application Insights** e selecione o Recurso **DevTest-AppInsights** criado no exercício anterior. Em seguida, navegue até **Alertas**.
1. Haverá pelo menos **1** novo alerta na lista de resultados com **Severidade 2**. Inserir **Alertas** para abrir o Serviço de Alertas do Azure Monitor.
1. Haverá pelo menos **1** Failed_Alert com **Severidade 2 – Aviso** aparecendo na lista. Isso foi acionado quando você validou o endereço URL do site não existente no exercício anterior.

   > **Observação:** se nenhum Alerta aparecer ainda, aguarde mais alguns minutos.

1. Retorne ao Portal do Azure DevOps e abra o Projeto **eShopOnWeb**. Navegue até **Pipelines**, selecione **Lançamentos** e selecione o **eshoponweb-cd**.
1. Clique no botão **Criar lançamento**.
1. Aguarde o início do pipeline de lançamento e **aprove** a ação de lançamento da Etapa DevTest.
1. Aguarde até que a etapa de lançamento do DevTest seja concluída com êxito. Observe como os **Portões pós-implantação** estão mudando para o status **Portões de avaliação**. Clique no ícone **Portões de avaliação**.
1. Em **Consultar alertas do Azure Monitor**, observe um estado inicial com falha.
1. Deixe o pipeline de lançamento em um estado pendente pelos próximos 5 minutos. Passados os 5 minutos, repare novamente a segunda avaliação falhando.
1. Esse é o comportamento esperado, já que há um alerta do Application Insights foi disparado para o aplicativo Web do DevTest.

   > **Observação**: como há um alerta disparado pela exceção, o portão **Consultar o Azure Monitor** falhará. Isso, por sua vez, impedirá a implantação no ambiente de **Produção**.

1. Aguarde alguns minutos e valide o status dos Portões de lançamento novamente. Dentro de alguns minutos após a verificação dos Portões de lançamento iniciais, e como o Alerta inicial do Application Insights foi disparado com a ação "Acionado", ele resultará em um Portão de lançamento bem-sucedido, permitindo a implantação da Etapa de lançamento de produção.

   > **Observação:** se o portão falhar, feche o alerta.

### Exercício 4: remover os recursos do laboratório do Azure

Neste exercício, você removerá os recursos do Azure provisionados neste laboratório para eliminar cobranças inesperadas.

> **Observação**: lembre-se de remover todos os recursos do Azure que acabam de ser criados e que você não usa mais. Remover recursos não utilizados garante que você não veja encargos inesperados.

#### Tarefa 1: remover os recursos do laboratório do Azure

Nesta tarefa, você usará o Azure Cloud Shell para remover os recursos do Azure provisionados neste laboratório para eliminar cobranças desnecessárias.

1. No portal do Azure, abra a sessão **Bash** no painel **Cloud Shell**.
1. Liste todos os grupos de recursos criados nos laboratórios deste módulo executando o seguinte comando:

   ```sh
   az group list --query "[?starts_with(name,'az400m04l09-RG')].name" --output tsv
   ```

1. Exclua todos os grupos de recursos criados em todos os laboratórios deste módulo executando o seguinte comando:

   ```sh
   az group list --query "[?starts_with(name,'az400m04l09-RG')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
   ```

   > **Observação**: o comando é executado de modo assíncrono (conforme determinado pelo parâmetro --nowait), portanto, embora você possa executar outro comando da CLI do Azure imediatamente depois na mesma sessão Bash, levará alguns minutos antes de o grupo de recursos ser removido.

## Revisão

Neste laboratório, você configurou pipelines de lançamento e, em seguida, configurou e testou portões de lançamento.
