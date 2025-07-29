---
lab:
  title: Configurar pools de agentes e entender os estilos de pipeline
  module: 'Module 02: Implement CI with Azure Pipelines and GitHub Actions'
---

# Configurar pools de agentes e entender os estilos de pipeline

## Requisitos do laboratório

- Este laboratório requer o **Microsoft Edge** ou um [navegador com suporte do Azure DevOps.](https://docs.microsoft.com/azure/devops/server/compatibility)

- **Configurar uma organização do Azure DevOps:** se você ainda não tiver uma organização Azure DevOps que possa usar para este laboratório, crie uma seguindo as instruções disponíveis em [Criar uma organização ou coleção de projetos](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization).

- [Página de download do Git for Windows](https://gitforwindows.org/). Ele será instalado como parte dos pré-requisitos deste laboratório.

- [Visual Studio Code](https://code.visualstudio.com/). Ele será instalado como parte dos pré-requisitos deste laboratório.

## Visão geral do laboratório

Os pipelines baseados em YAML permitem que você implemente totalmente CI/CD como código, em que as definições de pipeline residem no mesmo repositório que o código que faz parte do seu projeto do Azure DevOps. Os pipelines baseados em YAML dão suporte a uma ampla variedade de recursos que fazem parte dos pipelines clássicos, como solicitações de pull, revisões de código, histórico, branch e modelos.

Independentemente da escolha do estilo de pipeline, para criar seu código ou implantar sua solução usando o Azure Pipelines, você precisa de um agente. Um agente hospeda recursos de computação que executam um trabalho por vez. Os trabalhos podem ser executados diretamente no computador host do agente ou em um contêiner. Você pode executar seus trabalhos usando agentes hospedados pela Microsoft, que são gerenciados para você, ou implementar um agente auto-hospedado que você configura e gerencia por conta própria.

Neste laboratório, você aprenderá a implementar e usar agentes auto-hospedados com pipelines YAML.

## Objetivos

Após concluir este laboratório, você poderá:

- Implementar pipelines baseados em YAML.
- Implementar agentes auto-hospedados.

## Tempo estimado: 30 minutos

## Instruções

### Exercício 0: (pular se já foi feito) Configurar os pré-requisitos do laboratório

Neste exercício, você configurará os pré-requisitos para o laboratório, que consistem em um novo projeto do Azure DevOps com um repositório baseado no [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb).

#### Tarefa 1: (pular se feita) criar e configurar o projeto de equipe

Nesta tarefa, você criará um projeto **eShopOnWeb** do Azure DevOps para ser usado por vários laboratórios.

1. No computador do laboratório, em uma janela do navegador, abra sua organização do Azure DevOps. Clique em **Novo projeto**. Dê ao seu projeto o nome **eShopOnWeb** e deixe os outros campos com padrões. Clique em **Criar**.

#### Tarefa 2: (pular se feita) importar repositório do Git eShopOnWeb

Nesta tarefa, você importará o repositório eShopOnWeb do Git que será usado por vários laboratórios.

1. No computador do laboratório, em uma janela do navegador, abra sua organização do Azure DevOps e o projeto **eShopOnWeb** criado anteriormente. Clique em **Repos > Arquivos**, **Importar um repositório**. Selecione **Importar**. Na janela **Importar um repositório do Git**, cole a seguinte URL <https://github.com/MicrosoftLearning/eShopOnWeb.git> e clique em **Importar**:

1. O repositório está organizado da seguinte forma:
    - A pasta **.ado** contém os pipelines YAML do Azure DevOps.
    - O contêiner da pasta **.devcontainer** está configurado para o desenvolvimento usando contêineres (localmente no VS Code ou no GitHub Codespaces).
    - A pasta **infra** contém modelos de infraestrutura como código Bicep e ARM usados em alguns cenários de laboratório.
    - A pasta **.github** contém definições de fluxo de trabalho do GitHub YAML.
    - A pasta **src** contém o site do .NET 8 usado nos cenários de laboratório.

#### Tarefa 3: (pular se feita) definir o branch main como branch padrão

1. Vá para **Repos > Branches**.
1. Passe o mouse sobre o branch **main** e clique nas reticências à direita da coluna.
1. Clique em **Definir como branch padrão**.

### Exercício 1: criar agentes e configurar pools de agentes

Neste exercício, você criará uma VM (máquina virtual) do Azure e a usará para criar um agente e configurar pools de agentes.

#### Tarefa 1: Criar e conectar-se a uma VM do Azure

1. No seu navegador, abra o portal do Azure em `https://portal.azure.com`. Se solicitado, entre usando uma conta com a função Proprietário em sua assinatura do Azure.

1. Na caixa **Pesquisar recursos, serviços e documentos (G+/)**, digite **`Virtual Machines`** e selecione na lista suspensa.

1. Selecione o botão **Criar**.

1. Selecione as **Predefinições**.

    ![Captura de tela da criação de uma máquina virtual com uma configuração predefinida.](images/create-virtual-machine-preset.png)

1. Selecione **Desenvolvimento/Teste** como o ambiente de carga de trabalho e **Uso Geral** como o tipo de carga de trabalho.

1. Selecione o botão **Continuar a criar uma VM**, na guia **Noções básicas**, execute as seguintes ações e selecione **Gerenciamento**:

   | Configuração | Ação |
   | -- | -- |
   | Lista suspensa **Assinatura** | Selecione sua assinatura do Azure. |
   | Seção **Grupo de recursos** | Crie um novo grupo de recursos chamado **rg-eshoponweb-agentpool**. |
   | Caixa de texto **Nome da máquina virtual**  | Insira o nome de sua preferência, por exemplo, **`eshoponweb-vm`**. |
   | Lista suspensa **Região** | Escolha a região do [azure](https://azure.microsoft.com/explore/global-infrastructure/geographies) mais próxima a você. For example, "eastus", "eastasia", "westus", etc. |
   | Lista suspensa **Opções de disponibilidade** | Selecione **Nenhuma redundância de infraestrutura necessária**. |
   | Lista suspensa **Tipos de segurança** | Selecione a opção **Máquinas virtuais de inicio confiável**. |
   | Lista suspensa **Imagem** | Selecione o **Datacenter do Windows Server 2022: imagem Edição do Azure – x64 Gen2**. |
   | Lista suspensa **Tamanho** | Selecione o tamanho**Padrão** mais econômico para fins de teste. |
   | Caixa de texto **Nome de usuário** | Digite o nome de usuário de sua preferência |
   | Caixa de texto **Senha** | Digite a senha de sua preferência |
   | Seção **Portas de entrada públicas** | Selecione **Permitir portas selecionadas**. |
   | Lista suspensa **Selecionar portas de entrada** | Selecione **RDP (3389)**. |

1. Na guia **Gerenciamento**, na seção **Identidade**, selecione a caixa de seleção **Habilitar identidade gerenciada atribuída pelo sistema** e selecione **Examinar + criar**:

1. Na guia **Revisar + criar**, selecione **Criar**.

   > **Observação**: Aguarde o processo de provisionamento ser concluído. Isso deverá levar cerca de dois minutos.

1. No portal do Azure, navegue até a página exibindo a configuração da VM do Azure recém-criada.

1. Na página VM do Azure, selecione **Conectar**; no menu suspenso, clique em **Conectar** e, em seguida, em **Baixar arquivo RDP**.

1. Use o arquivo RDP baixado para estabelecer uma sessão de Área de Trabalho Remota com o sistema operacional em execução na VM do Azure.

#### Tarefa 2: Criar um pool de agentes

1. Na sessão da Área de Trabalho Remota para a VM do Azure, inicie o navegador da Web do Microsoft Edge.

1. No navegador da Web, navegue até o portal do Azure DevOps em `https://aex.dev.azure.com` e entre para acessar sua organização.

   > **Observação**: se for a primeira vez que você acessa o portal do Azure DevOps, talvez seja necessário criar seu perfil.

1. Abra o projeto **eShopOnWeb** e selecione **Configurações do projeto** no menu inferior esquerdo.

1. Em **Pipelines > Pools de agentes**, selecione o botão **Adicionar pool**.

1. Escolha o tipo de pool **Auto-hospedado**.

1. Forneça um nome para o pool de agentes, como **eShopOnWebSelfPool**, e adicione uma descrição opcional.

1. Deixe a opção **Conceder permissão de acesso a todos os pipelines** desmarcada.

   ![Captura de tela mostrando opções de adicionar pool de agentes com o tipo auto-hospedado.](images/create-new-agent-pool-self-hosted-agent.png)

   > **Observação**: não é recomendado conceder permissão de acesso a todos os pipelines para ambientes de produção. Ela só é usada neste laboratório para simplificar a configuração do pipeline.

1. Selecione o botão **Criar** para criar o pool de agentes.

#### Tarefa 3: Baixar e extrair os arquivos de instalação do agente

1. No portal do Azure DevOps, selecione o pool de agentes recém-criado e, em seguida, selecione a guia **Agentes**.

1. Selecione o botão **Novo agente** e, em seguida, o botão **Baixar** em **Baixar agente** na nova janela pop-up.

   > **Observação**: siga as instruções de instalação para instalar o agente.

   > **Observação**: O nome do arquivo zip que você baixou com o botão **Baixar** deve ser semelhante ao seguinte `vsts-agent-win-x64-X.YYY.Z.zip`(no momento da redação deste laboratório, o nome do arquivo é `vsts-agent-win-x64-4.255.0.zip`). O nome do arquivo será usado posteriormente em um dos comandos de instalação do agente.

1. Inicie uma sessão do PowerShell e execute os comandos a seguir para criar um **agente** nomeado de pasta.

   ```powershell
   mkdir agent ; cd agent        
   ```

   > **Observação**: verifique se você está na pasta em que deseja instalar o agente, por exemplo, C:\agent.

1. Execute o seguinte comando para extrair o conteúdo dos arquivos do instalador do agente baixado:

   ```powershell
   Add-Type -AssemblyName System.IO.Compression.FileSystem ; [System.IO.Compression.ZipFile]::ExtractToDirectory("$HOME\Downloads\vsts-agent-win-x64-4.255.0.zip", "$PWD")
   ```

   > **Observação**: se você baixou o agente em um local diferente (ou a versão baixada difere), ajuste o comando acima de acordo.

   > **Observação**: Verifique se o nome do arquivo zip especificado dentro do comando `ExtractToDirectory` é o mesmo que o nome do arquivo zip que você baixou anteriormente.

#### Tarefa 4: Criar um token PAT

> **Observação**: antes de configurar o agente, você precisa criar um token PAT (a menos que tenha um existente). Para criar um token PAT, siga as etapas abaixo:

1. Na sessão da Área de Trabalho Remota da VM do Azure, abra outra janela do navegador, navegue até o portal do Azure DevOps em `https://aex.dev.azure.com` e acesse sua organização e projeto.

1. Selecione **Configurações do usuário** no menu superior direito (diretamente à esquerda do ícone de avatar do usuário).

1. Selecione o item de menu **Tokens de acesso pessoal**.

   ![Captura de tela mostrando o menu de tokens de acesso pessoal.](images/personal-access-token-menu.png)

1. Selecione o botão **Novo Token**.

1. Forneça um nome para o token, como **eShopOnWebToken**.

1. Selecione a organização do Azure DevOps para a qual você deseja usar o token.

1. Defina a data de validade do token (usado apenas para configurar o agente).

1. Selecione o escopo personalizado definido.

1. Selecione para mostrar todos os escopos.

1. Selecione o escopo **Pools de Agentes (Ler e Gerenciar)**.

1. Selecione o botão **Criar** para criar o token.

1. Copie o valor do token e salve-o em um local seguro (você não poderá vê-lo novamente. Você só pode regenerar o token).

   ![Captura de tela mostrando a configuração do token de acesso pessoal.](images/personal-access-token-configuration.png)

   > [!IMPORTANT]
   > Use a opção de privilégio mínimo, **Pools de Agentes (Ler &Gerenciar)**, somente para a configuração do agente. Além disso, certifique-se de definir a data de validade mínima para o token se essa for a única finalidade do token. Você pode criar outro token com os mesmos privilégios se precisar configurar o agente novamente.

#### Tarefa 5: Configurar o agente

1. Na sessão da Área de Trabalho Remota para a VM do Azure, volte para a janela do PowerShell. Se necessário, altere o diretório atual para aquele no qual você extraiu os arquivos de instalação do agente anteriormente neste exercício.

1. Para configurar o agente para ser executado sem vigilância, invoque o seguinte comando:

   ```powershell
   .\config.cmd
   ```

   > **Observação**: se você quiser executar o agente de forma interativa, use `.\run.cmd`.

1. Para configurar o agente, execute as seguintes ações quando solicitado:

   - Insira a URL da organização do Azure DevOps (**URL do servidor**) no formato `https://dev.azure.com/{your organization name}`.
   - Aceite o tipo de autenticação padrão (**`PAT`**).
   - Insira o valor do token PAT que você criou na etapa anterior.
   - Insira o nome do pool de agentes **`eShopOnWebSelfPool`** que você criou anteriormente neste exercício.
   - Insira o nome do evento **`eShopOnWebSelfAgent`**.
   - Aceite a pasta de trabalho do agente padrão (_work).
   - Insira **Y** para configurar o agente a ser executado como serviço.
   - Insira **Y** para habilitar SERVICE_SID_TYPE_UNRESTRICTED para o serviço do agente.
   - Insira **`NT AUTHORITY\SYSTEM`** para definir o contexto de segurança do serviço.

   > [!IMPORTANT]
   > Em geral, você deve seguir o princípio do privilégio mínimo ao configurar o contexto de segurança do serviço.

   - Aceite a opção padrão (**N**) para permitir que o serviço seja iniciado imediatamente após a conclusão da configuração.

   ![Captura de tela mostrando a configuração do agente.](images/agent-configuration.png)

   > **Observação**: o processo de configuração do agente levará alguns minutos para ser concluído. Feito isso, você verá uma mensagem indicando que o agente está sendo executado como um serviço.

   > [!IMPORTANT] Se você vir uma mensagem de erro indicando que o agente não está em execução, talvez seja necessário iniciar o serviço manualmente. Para fazer isso, abra o miniaplicativo **Serviços** no Painel de Controle do Windows, localize o serviço chamado **Agente do Azure DevOps (eShopOnWebSelfAgent)** e inicie-o.

   > [!IMPORTANT] Se o agente não for iniciado, talvez seja necessário escolher uma pasta diferente para o diretório de trabalho do agente. Para fazer isso, execute novamente o script de configuração do agente e escolha uma pasta diferente.

1. Verifique o status do agente alternando para o navegador da Web exibindo o portal do Azure DevOps, navegando até o pool de agentes e clicando na guia **Agentes**. Você deve ver o novo agente na lista.

   ![Captura de tela mostrando o status do agente.](images/agent-status.png)

   > **Observação**: para obter mais detalhes sobre agentes do Windows, consulte: [Agentes do Windows auto-hospedados](https://learn.microsoft.com/azure/devops/pipelines/agents/windows-agent)

   > [!IMPORTANT]
   > Para que o agente possa criar e implantar recursos do Azure a partir dos pipelines do Azure DevOps (que você percorrerá nos próximos laboratórios), é necessário instalar a CLI do Azure no sistema operacional da VM do Azure que está hospedando o agente.

1. Abra um navegador da Web e acesse a página `https://learn.microsoft.com/en-us/cli/azure/install-azure-cli-windows?tabs=azure-cli&pivots=msi#install-or-update`.

1. Baixe e instale a CLI do Azure.

1. (Opcional) Se preferir, execute o seguinte comando do PowerShell para instalar a CLI do Azure:

   ```powershell
   $ProgressPreference = 'SilentlyContinue'; Invoke-WebRequest -Uri https://aka.ms/installazurecliwindows -OutFile .\AzureCLI.msi; Start-Process msiexec.exe -Wait -ArgumentList '/I AzureCLI.msi /quiet'; Remove-Item .\AzureCLI.msi
   ```

   > **Observação**: se você estiver usando uma versão diferente da CLI do Azure, talvez seja necessário ajustar o comando acima de acordo.

1. No navegador da Web, navegue até a página do instalador do SDK do Microsoft .NET 8.0 em `https://dotnet.microsoft.com/en-us/download/dotnet/thank-you/sdk-8.0.403-windows-x64-installer`.

   > [!IMPORTANT]
   > Você precisa instalar o SDK do .NET 8.0 (ou mais recente) na VM do Azure que está hospedando o agente. Isso é necessário para criar o aplicativo eShopOnWeb nos próximos laboratórios. Todas as outras ferramentas ou SDKs necessários para o build do aplicativo também devem ser instalados na VM do Azure.

1. Baixe e instale o SDK do Microsoft .NET 8.0.

### Exercício 2: Criar Azure Pipelines baseado em YAML

Neste exercício, você criará um pipeline de build do ciclo de vida do aplicativo usando um modelo baseado em YAML.

#### Tarefa 1: criar um pipeline YAML do Azure DevOps

Nesta tarefa, você criará um pipeline baseado em YAML para o projeto **eShopOnWeb**.

1. No navegador da Web que exibe o portal do Azure DevOps com o projeto **eShopOnWeb** aberto, no painel de navegação vertical do lado esquerdo, clique em **Pipelines**.
1. Clique no botão **Criar pipeline** se você ainda não tiver nenhum outro pipeline criado ou clique em **Novo pipeline** para criar um novo.

1. Na página **Onde está seu código?**, clique em **Azure Repos Git **.
1. No painel **Selecionar um repositório**, clique em **eShopOnWeb**.
1. Na tela **Configurar o pipeline**, selecione **Arquivo YAML existente do Azure Pipelines**.
1. Em **Selecionar um arquivo YAML existente**, selecione **main** para Branch e **/.ado/eshoponweb-ci-pr.yml** para o caminho.
1. Clique em **Continuar**.
1. No painel **Revisar seu pipeline YAML**, revise o pipeline de exemplo. Este é um pipeline de build de aplicativo .NET bastante simples, que faz o seguinte:

   - Um único estágio: Compilar
   - Um único trabalho: Build
   - Quatro tarefas dentro do trabalho de build:
   - Dotnet Restore
   - Dotnet Build
   - Dotnet Test
   - Dotnet Publish

1. No painel **Revisar seu pipeline YAML**, clique no símbolo de acento circunflexo para baixo ao lado do botão **Executar**, clique em **Salvar**.

    > **Observação**: estamos apenas criando a definição de pipeline por enquanto, sem executá-lo. Primeiro, você configurará um pool de agentes do Azure DevOps e executará o pipeline em um exercício posterior.

#### Tarefa 2: Atualizar o pipeline YAML com o pool de agente auto-hospedado

1. No portal do Azure DevOps, navegue até o projeto **eShopOnWeb** e selecione **Pipelines** no menu do lado esquerdo.
1. Clique no botão **Editar** do pipeline que você criou na tarefa anterior.
1. No painel de edição **eShopOnWeb**, no pipeline baseado em YAML já existente, remova a linha 13 que diz **vmImage: ubuntu-latest**, designando ao pool de agentes de destino o seguinte conteúdo, designando o pool de agentes auto-hospedado recém-criado:

    ```yaml
    pool: 
      name: eShopOnWebSelfPool
      demands: Agent.Name -equals eShopOnWebSelfAgent
    ```

    > **AVISO**: Tenha cuidado ao copiar/colar, certifique-se de ter o mesmo recuo mostrado acima.

    ![Captura de tela mostrando a sintaxe do pool YAML.](images/eshoponweb-ci-pr-agent-pool.png)

1. No painel de edição do **eShopOnWeb**, no canto superior direito do painel, clique em **Validar e salvar**. Em seguida, clique em **Salvar**.
1. No painel de edição **eShopOnWeb**, no canto superior direito do painel, clique em **Executar**.

    > **Observação**: o pipeline será executado no pool de agentes auto-hospedado que você criou no exercício anterior.
1. Abra a execução do pipeline e monitore o trabalho até sua conclusão.

    > **Observação**: se você receber uma solicitação de permissões, clique em **Permitir** para permitir que o pipeline seja executado.

1. Depois que a execução do pipeline for concluída, examine a saída e verifique se o pipeline foi executado.

## Revisão

Neste laboratório, você aprendeu a implementar e usar agentes auto-hospedados com pipelines YAML.
