---
lab:
  title: Creating a Release Dashboard
  module: 'Module 04: Design and implement a release strategy'
---

# Creating a Release Dashboard

# Manual de laboratório do aluno

## Requisitos do laboratório

- Este laboratório requer o **Microsoft Edge** ou um [navegador compatível com o Azure DevOps](https://docs.microsoft.com/en-us/azure/devops/server/compatibility?view=azure-devops#web-portal-supported-browsers).

- **Configurar uma organização do Azure DevOps:** se você ainda não tiver uma organização Azure DevOps que possa usar para este laboratório, crie uma seguindo as instruções disponíveis em [Criar uma organização ou coleção de projetos](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/create-organization?view=azure-devops).

- Identifique uma assinatura existente do Azure ou crie uma.

- Verifique se você tem uma conta Microsoft ou uma conta do Azure AD com função de proprietário na assinatura do Azure. Para conhecer os detalhes, consulte [Listar atribuições de função do Azure usando o portal do Azure](https://docs.microsoft.com/en-us/azure/role-based-access-control/role-assignments-list-portal).

## Visão geral do laboratório

Neste laboratório, você aprenderá a criar um painel de versão e usar a API REST para recuperar dados de versão do Azure DevOps, que podem ser disponibilizados para seus aplicativos ou painéis personalizados.

O laboratório usa o recurso Azure DevOps Starter, que cria automaticamente um projeto do Azure DevOps que cria e implanta um aplicativo no Azure.

## Objetivos

Após concluir este laboratório, você poderá:

- Criar um painel de versão.
- Use a API REST para consultar informações de versão.

## Tempo estimado: 45 minutos

## Instruções

### Exercício 1: criar um painel de versão

Neste exercício, você criará um painel de versão em uma organização do Azure DevOps.

#### Tarefa 1: criar um recurso do Azure DevOps Starter

Nesta tarefa, você criará um recurso do Azure DevOps Starter na sua assinatura do Azure. Isso criará automaticamente um projeto correspondente em sua organização do Azure DevOps.

1. No computador do laboratório, inicie um navegador da Web, navegue até o [**portal do Azure**](https://portal.azure.com) e entre com a conta de usuário que tem a função proprietário ou colaborador na assinatura do Azure que você usará neste laboratório.
1. No portal do Azure, pesquise e selecione o tipo de recurso do **DevOps Starter** e, na folha do **DevOps Starter**, clique em **+ Criar**.
1. Na folha **DevOps Starter**, no painel **Iniciar do zero com um novo aplicativo**, selecione o bloco **.NET** e, na parte superior, ao lado de **Configurar o DevOps Starter com o GitHub**, altere as configurações, clique **aqui** e selecione **Azure DevOps**, **Concluído** e **Próximo: Estrutura >**.
1. Na folha **DevOps Starter**, no painel **Escolha uma estrutura de aplicativo**, selecione o bloco **ASP<nolink>.NET Core**, mova o controle deslizante **Adicionar um banco de dados** para a posição **Ativado** e clique em **Próximo: Serviço >**.
1. Na folha **DevOps Starter**, no painel **Selecionar um serviço do Azure para implantar o aplicativo**, verifique se o bloco **aplicativo Web do Windows** está selecionado e clique em **Próximo: Criar >**.
1. Na folha **DevOps Starter**, no painel **Quase lá**, especifique as seguintes configurações:

    | Configuração | Valor |
    | ------- | ----- |
    | Nome do projeto | **Criar um painel de versão** |
    | Organização do Azure DevOps | nome da organização do Azure DevOps que você usará neste laboratório |
    | Assinatura | nome da assinatura do Azure que você usará neste laboratório |
    | Nome do aplicativo Web | qualquer nome globalmente exclusivo de 2 a 60 caracteres composto por letras e dígitos e hifens, começando e terminando com uma letra ou um dígito |
    | Localização | nome da região do Azure na qual você pretende implantar um aplicativo Web do Azure e um banco de dados SQL do Azure |

1. Na folha **DevOps Starter**, no painel **Quase lá**, clique em **Configurações adicionais**.
1. No painel **Configurações adicionais**, especifique as configurações a seguir e clique em **OK**.

    | Configuração | Valor |
    | ------- | ----- |
    | Grupo de recursos | **az400m10l02-rg** |
    | Tipo de preço | **F1 Gratuito** |
    | Localização do Application Insights | o nome da mesma região do Azure que você escolheu para a localização do aplicativo Web do Azure |
    | Nome do servidor | qualquer nome globalmente exclusivo de 3 a 63 caracteres composto por letras e dígitos e hifens, começando e terminando com uma letra ou um dígito |
    | Inserir o nome de usuário | **dbadmin** |
    | Localização | o nome da mesma região do Azure que você escolheu para a localização do aplicativo Web do Azure |
    | Nome do Banco de Dados | **az400m10l02-db** |

1. De volta à folha do **DevOps Starter**, no painel **Quase lá**, clique em **Concluído** e em **Revisar + Criar**.

    > **Observação**: aguarde até que a implantação seja concluída. O provisionamento do recurso do **DevOps Starter** deve levar cerca de 2 minutos.

1. Depois de receber a confirmação de que o recurso do DevOps Starter foi provisionado, clique no botão **Ir para o recurso**. Isso redirecionará o navegador para a folha do DevOps Starter.
1. Na folha do DevOps Starter, acompanhe o progresso do pipeline de CI/CD até que ele seja concluído com êxito.

    > **Observação**: a criação do aplicativo Web do Azure e do banco de dados SQL do Azure correspondente pode levar cerca de 5 minutos. O processo cria automaticamente um projeto do Azure DevOps que inclui um repositório pronto para implantação, bem como os pipelines de build e lançamento. Os recursos do Azure são criados como parte do pipeline de implantação acionado automaticamente.

#### Tarefa 2: criar versões do Azure DevOps

Nesta tarefa, você criará várias versões do Azure DevOps, incluindo uma que resultará em uma implantação com falha.

1. No navegador da Web que exibe o portal do Azure, na página DevOps Starter, na barra de ferramentas, clique em **Página inicial do projeto**. Isso abrirá automaticamente outra guia do navegador com o projeto **Criar um Painel de Versão** no portal do Azure DevOps. Se for solicitado a entrar, autentique-se com as suas credenciais de organização do Azure DevOps.

    > **Observação**: primeiro, você criará uma nova versão que será implantada com sucesso.

1. No portal do Azure DevOps, no menu vertical do lado esquerdo, clique em **Repos**, na lista de pastas no repositório, navegue até a pasta **Páginas \\aspnet-core-dotnet-core\\ do aplicativo** e clique na entrada **Index.cshtml**.
1. No painel **Index.cshtml**, clique em **Editar**, na linha **20**, substitua `<div class="description line-2"> Your ASP.NET Core app is up and running on Azure</div>` por `<div class="description line-2"> Your ASP.NET Core app v1.1 is up and running on Azure</div>`, clique em **Confirmar** e, no painel **Confirmar**, clique em **Confirmar** novamente. Isso vai disparar automaticamente o pipeline de build.
1. No portal do Azure DevOps, no painel de navegação vertical no lado esquerdo, clique em **Pipelines**.
1. Na guia **Recentes** do painel **Pipelines**, clique na entrada **az400m10l02-CI**, na guia **Execuções** do painel **az400m10l02-CI**, selecione a execução mais recente, na guia **Resumo** da execução, na seção **Trabalhos**, clique em **Build** e monitore o trabalho até a conclusão ser bem-sucedida.
1. Quando o trabalho for concluído, no portal do Azure DevOps, no painel de navegação vertical no lado esquerdo, na seção **Pipelines**, clique em **Versões**.
1. No painel **az400m10l02-CD**, na guia **Versões**, clique na entrada **Versão-2**, na guia **Pipeline** do painel **Versão-2**, clique no estágio **dev**, no painel **dev**, clique em **Exibir logs** e monitore o progresso da implantação até a conclusão ser bem-sucedida.

    > **Observação**: agora, você criará uma nova versão que terá a implantação com falha. A falha será causada pelo teste de assemblies internos, que consideram a alteração associada à nova versão inválida.

1. No portal do Azure DevOps, no menu vertical do lado esquerdo, clique em **Repos**, na lista de pastas no repositório, navegue até a pasta **Páginas \\aspnet-core-dotnet-core\\ do aplicativo** e clique na entrada **Index.cshtml**.
1. No painel **Index.cshtml**, clique em **Editar**. Na linha **4**, substitua `ViewData["Title"] = "Home Page - ASP.NET Core";` por `ViewData["Title"] = "Home Page v1.2 - ASP.NET Core";`. Clique em **Confirmar**. No painel **Confirmar**, clique em **Confirmar** novamente. Isso vai disparar automaticamente o pipeline de build.
1. No portal do Azure DevOps, no painel de navegação vertical no lado esquerdo, clique em **Pipelines**.
1. Na guia **Recentes** do painel **Pipelines**, clique na entrada **az400m10l02-CI**, na guia **Execuções** do painel **az400m10l02-CI**, selecione a execução mais recente, na guia **Resumo** da execução, na seção **Trabalhos**, clique em **Build** e monitore o trabalho até a conclusão ser bem-sucedida.
1. Quando o trabalho for concluído, no portal do Azure DevOps, no painel de navegação vertical no lado esquerdo, na seção **Pipelines**, clique em **Versões**.
1. No painel **az400m10l02-CD**, na guia **Versões**, clique na entrada **Versão-3**, na guia **Pipeline** do painel **Versão-3**, clique no estágio **dev**, no painel **dev**, clique em **Exibir logs** e monitore o progresso da implantação até a falha durante o estágio de **Testar Assemblies**.

#### Tarefa 3: criar um painel de versão do Azure DevOps

Nesta tarefa, você criará um painel e adicionará a ele widgets relacionados à versão.

1. No portal do Azure DevOps, no menu vertical do lado esquerdo, clique em **Visão geral**, na seção **Visão geral**, clique em **Painéis** e clique em **Adicionar um widget**.
1. No painel **Adicionar widget**, role para baixo pela lista de widgets, selecione a entrada **Status de implantação** e clique em **Adicionar**.
1. Use o procedimento descrito na etapa anterior para adicionar os widgets **Detalhes da integridade da versão**, **Visão geral da integridade da versão** e **Visão geral do pipeline da versão**.
    > **Observação**: instale os **Detalhes de integridade da versão** e a **Visão geral da integridade da versão** da [Integridade do projeto de equipe](https://marketplace.visualstudio.com/items?itemName=ms-devlabs.TeamProjectHealth) do Marketplace.
1. Use o mouse para arrastar a **Visão geral do pipeline de lançamento** para a direita do widget **Status da implantação** para evitar a necessidade de rolagem vertical pelo painel e clique em **Edição concluída**.
1. De volta ao painel, no retângulo que representa o widget **Status de implantação**, clique em **Configurar widget**.
1. No painel **Configuração**, especifique as seguintes configurações (deixe as demais com os valores padrão) e selecione em **Salvar**:

    | Configuração | Valor |
    | ------- | ----- |
    | Pipeline de build | **az400m10l02-CI** |
    | Pipelines de lançamento vinculados | **az400m10l02-CD; az400m10l02-CD\dev** |

1. De volta ao painel de controle, passe o mouse sobre o canto superior direito do retângulo que representa o widget **Visão geral da integridade da versão** para revelar o sinal de reticências do menu **Mais ações**, clique nele e, no menu suspenso, clique em **Configurar**.  
1. No painel **Configuração**, especifique as seguintes configurações (deixe as demais com os valores padrão) e selecione em **Salvar**:

    | Configuração | Valor |
    | ------- | ----- |
    | Selecione as definições de versão | **az400m10l02-CD** |

1. De volta ao painel de controle, passe o mouse sobre o canto superior direito do retângulo que representa o **widget Detalhes** de integridade da liberação para revelar o sinal de reticências que representa o **menu Mais ações** , clique nele e, no menu suspenso, clique em **Configurar**.  
1. No painel **Configuração**, especifique as seguintes configurações (deixe as demais com os valores padrão) e selecione em **Salvar**:

    | Configuração | Valor |
    | ------- | ----- |
    | Definição | **az400m10l02-CD** |

1. De volta ao painel de controle, passe o mouse sobre o canto superior direito do retângulo que representa o **widget Visão geral** do pipeline de liberação para revelar o sinal de reticências que representa o **menu Mais ações** , clique nele e, no menu suspenso, clique em **Configurar**.  
1. No painel **Configuração**, especifique as seguintes configurações (deixe as demais com os valores padrão) e selecione em **Salvar**:

    | Configuração | Valor |
    | ------- | ----- |
    | Pipeline de lançamento | **az400m10l02-CD** |

1. De volta ao painel, clique em **Atualizar** para atualizar o conteúdo exibido pelos widgets.

    > **Observação**: os links nos widgets permitem que você navegue diretamente para os painéis correspondentes no portal do Azure DevOps.

### Exercício 2: consultar informações de versão por meio da API REST.

Neste exercício, você consultará informações de liberação via API REST usando o Postman.

#### Tarefa 1: gerar um token de acesso pessoal do Azure DevOps

Nesta tarefa, você gerará um token de acesso pessoal do Azure DevOps que será usado para autenticar do aplicativo Postman que você instalará na próxima tarefa deste exercício.

1. No computador do laboratório, na janela do navegador da Web que exibe o portal do Azure DevOps, no canto superior direito da página do Azure DevOps, clique no ícone **Configurações do usuário**, no menu suspenso, clique em **Tokens de acesso pessoais** no painel **Tokens de acesso pessoal** e clique em **+ Novo Token**.
1. No painel **Criar um novo token de acesso pessoal**, clique no link **Mostrar todos os escopos** e especifique as seguintes configurações e clique em **Criar** (deixe todos os outros com os valores padrão):

    | Configuração | Valor |
    | --- | --- |
    | Nome | **Laboratório para criar um painel de versão** |
    | Escopo | **Versão** |
    | Permissões | **Ler** |
    | Escopo | **Compilar** |
    | Permissões | **Ler** |

1. No painel **Sucesso**, copie o valor do token de acesso pessoal para a área de transferência.

    > **Observação**: lembre-se de registrar o valor do token. Você não poderá recuperá-lo depois de fechar este painel.

1. No painel **Sucesso**, clique em **Fechar**.

#### Tarefa 2: consultar informações de versão por meio da API REST usando o Postman

Nesta tarefa, você consultará informações de versão por meio da API REST usando o Postman.

1. No computador do laboratório, inicie um navegador da Web e navegue até a [página de download do Postman](https://www.postman.com/downloads/), clique no botão **Baixar aplicativo**, no menu suspenso, clique em **Windows de 64 bits**, clique no arquivo baixado e execute a instalação. Quando a instalação for concluída, o aplicativo da área de trabalho Postman será iniciado automaticamente.
1. No painel **Criar uma conta do Postman**, digite seu endereço de email, um nome de usuário e senha e clique em **Criar conta gratuita**.

    > **Observação**: você receberá um email do Postman para ativar a conta Postman e concluir o processo de provisionamento da conta.

1. Depois de entrar, na janela do aplicativo da área de trabalho do Postman, no canto superior esquerdo, clique em **+Novo**, no painel **Criar Novo**, clique em **Solicitar**, no painel **SALVAR SOLICITAÇÃO**, na caixa de texto **Nome da solicitação**, digite **Get-Releases**, clique em **+ Criar Coleção**, na caixa de texto **Nomear sua coleção**, digite **consultas az400m10l02 do Azure DevOps**, clique na marca de seleção no lado direito, depois no botão **Salvar em consultas az400m10l02 do Azure DevOps**.
1. Abra outra janela do navegador da Web e navegue até [a página do Microsoft Docs **Versões – Lista**](https://docs.microsoft.com/en-us/rest/api/azure/devops/release/releases/list?view=azure-devops-rest-6.0) e revise o conteúdo.
1. Volte para o aplicativo de área de trabalho do Postman, no painel de ferramentas na seção superior direita da janela do aplicativo, clique no cabeçalho da guia **Autorização**, na lista suspensa **TIPO**, selecione a entrada **Autenticação Básica** e, na caixa de texto **Senha**, cole o valor do token copiado na tarefa anterior (não defina o valor da caixa de texto **Nome de usuário**).
1. No painel de ferramentas na seção superior direita da janela do aplicativo, verifique se **GET** aparece na lista suspensa, na caixa de texto **Inserir URL de solicitação**, digite o seguinte e clique em **Enviar** (substitua o valor de `<organization_name>` pelo nome da organização de Azure DevOps) para listar todas as versões:

    ```url
    https://vsrm.dev.azure.com/<organization_name>/Creating%20a%20Release%20Dashboard/_apis/release/releases?api-version=6.0
    ```

1. Revise a saída listada na guia **Corpo** na seção inferior direita da janela do aplicativo e verifique se ela inclui a listagem das versões que você criou no exercício anterior deste laboratório.
1. Alterne para a janela do navegador da Web que exibe o conteúdo do Microsoft Docs e navegue até [a página do Microsoft Docs **Implantações – Lista** Documentos Microsoft](https://docs.microsoft.com/en-us/rest/api/azure/devops/release/deployments/list?view=azure-devops-rest-6.0) e revise o conteúdo.
1. No painel de ferramentas na seção superior direita da janela do aplicativo, verifique se **GET** aparece na lista suspensa, na caixa de texto **Inserir URL de solicitação**, digite o seguinte e clique em **Enviar** (substitua o valor de `<organization_name>` pelo nome da organização de Azure DevOps) para listar todas as implantações:

    ```url
    https://vsrm.dev.azure.com/<organization_name>/Creating%20a%20Release%20Dashboard/_apis/release/deployments?api-version=6.0
    ```

1. Revise a saída listada na guia **Corpo** na seção inferior direita da janela do aplicativo e verifique se ela inclui a listagem das implantações que você iniciou no exercício anterior deste laboratório.
1. No painel de ferramentas na seção superior direita da janela do aplicativo, verifique se **GET** aparece na lista suspensa, na caixa de texto **Inserir URL de solicitação**, digite o seguinte e clique em **Enviar** (substitua o valor de `<organization_name>` pelo nome da organização de Azure DevOps) para listar todas as implantações:

    ```url
    https://vsrm.dev.azure.com/<organization_name>/Creating%20a%20Release%20Dashboard/_apis/release/deployments?DeploymentStatus=failed&api-version=6.0
    ```

1. Revise a saída listada na guia **Corpo** na seção inferior direita da janela do aplicativo e verifique se ela inclui a implantação com falha que você iniciou no exercício anterior deste laboratório.

### Exercício 3: Remover os recursos do laboratório do Azure

Neste exercício, você removerá os recursos do Azure provisionados neste laboratório para eliminar cobranças inesperadas.

>**Observação**: lembre-se de remover todos os recursos do Azure recém-criados e que você não usa mais. Remover recursos não utilizados garante que você não veja encargos inesperados.

#### Tarefa 1: remover os recursos do Azure Lab

Nesta tarefa, você usará o Azure Cloud Shell para remover os recursos do Azure provisionados neste laboratório com o objetivo de eliminar cobranças inesperadas.

1. No portal do Azure, abra a sessão do **Shell Bash** no painel do **Cloud Shell**.
1. Liste todos os grupos de recursos criados em todos os laboratórios deste módulo executando o seguinte comando:

    ```sh
    az group list --query "[?starts_with(name,'az400m10l02-rg')].name" --output tsv
    ```

1. Exclua todos os grupos de recursos criados em todos os laboratórios deste módulo executando o seguinte comando:

    ```sh
    az group list --query "[?starts_with(name,'az400m10l02-rg')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**Observação**: o comando é executado de modo assíncrono (conforme determinado pelo parâmetro --nowait), portanto, embora você possa executar outro comando da CLI do Azure imediatamente depois na mesma sessão Bash, levará alguns minutos antes de os grupos de recursos serem removidos.

## Revisão

Neste laboratório, você aprendeu como criar e configurar o painel de versão e como usar a API REST para recuperar dados de versão do Azure DevOps.
