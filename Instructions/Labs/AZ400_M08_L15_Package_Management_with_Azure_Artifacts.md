---
lab:
  title: Package Management with Azure Artifacts
  module: 'Module 08: Design and implement a dependency management strategy'
---

# Package Management with Azure Artifacts

## Manual de laboratório do aluno

## Requisitos do laboratório

- Este laboratório requer o **Microsoft Edge** ou um [navegador compatível com o Azure DevOps.](https://docs.microsoft.com/azure/devops/server/compatibility)

- **Configurar uma organização do Azure DevOps:** se você ainda não tiver uma organização Azure DevOps que possa usar para este laboratório, crie uma seguindo as instruções disponíveis em [Pré-requisitos do laboratório AZ-400](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/AZ400_M00_Validate_lab_environment.html).

- **Configurar o projeto de amostra do EShopOnWeb:** se você ainda não tiver um projeto de exemplo do EShopOnWeb que possa usar para este laboratório, crie um seguindo as instruções disponíveis em [Pré-requisitos do laboratório AZ-400](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/AZ400_M00_Validate_lab_environment.html).

- A Edição 2022 do Visual Studio Community está disponível na [página de Downloads do Visual Studio](https://visualstudio.microsoft.com/downloads/). A instalação do Visual Studio 2022 deve incluir cargas de trabalho do **ASP<nolink>.NET e desenvolvimento Web**, **desenvolvimento Azure** e **desenvolvimento entre plataformas do .NET Core**.

## Visão geral do laboratório

O Azure Artifacts facilita a descoberta, a instalação e a publicação de pacotes NuGet, npm e Maven no Azure DevOps. Está profundamente integrado com outros recursos do Azure DevOps, como Compilação, tornando o gerenciamento de pacotes uma parte perfeita de seus fluxos de trabalho existentes.

## Objetivos

Após concluir este laboratório, você poderá:

- Criar e se conectar a um feed.
- Criar e publicar um pacote NuGet.
- Importar um pacote NuGet.
- Atualizar um pacote NuGet.

## Tempo estimado: 40 minutos

## Instruções

### Exercício 0: configurar os pré-requisitos do laboratório

Neste exercício, revisaremos como validar os pré-requisitos de laboratório tendo uma organização do Azure DevOps pronta e o projeto EShopOnWeb criado. Consulte as instruções acima para obter mais detalhes.

#### Tarefa 1: Configurar a solução EShopOnWeb no Visual Studio

Nesta tarefa, você configurará o Visual Studio para se preparar para o laboratório.

1. Verifique se você está exibindo o projeto de equipe **EShopOnWeb** no portal do Azure DevOps.

    > **Observação**: você pode acessar a página do projeto diretamente navegando até a URL [https://dev.azure.com/`<your-Azure-DevOps-account-name>`/EShopOnWeb](https://dev.azure.com/`<your-Azure-DevOps-account-name>`/EShopOnWeb), onde o espaço reservado `<your-Azure-DevOps-account-name>` representa o nome da organização do Azure DevOps.

2. No menu vertical no lado esquerdo do painel **EShopOnWeb**, clique em **Repos**.
3. No painel **Arquivos**, clique em **Clonar**, selecione a seta suspensa ao lado de **Clonar no VS Code** e, no menu suspenso, selecione **Visual Studio**.
4. Se for perguntado se deseja continuar, clique em **Abrir**.
5. Se solicitado, entre com a conta de usuário que você usou para configurar a organização do Azure DevOps.
6. Na interface do Visual Studio, na janela pop-up **Azure DevOps**, aceite o caminho local padrão (C:\EShopOnWeb) e clique em **Clonar**. Isso importará automaticamente o projeto para o Visual Studio.
7. Deixe a janela do Visual Studio aberta para uso no laboratório.

### Exercício 1: trabalhar com o Azure Artifacts

Neste exercício, você aprenderá a trabalhar com o Azure Artifacts usando as seguintes etapas:

- Criar e se conectar a um feed.
- Criar e publicar um pacote NuGet.
- Importar um pacote NuGet.
- Atualizar um pacote NuGet.

#### Tarefa 1: criar e se conectar a um feed

Nesta tarefa, você criará e se conectará a um feed.

1. Na janela do navegador da Web que exibe as configurações do projeto no portal do Azure DevOps, no painel de navegação vertical, selecione **Artefatos**.
2. Com o hub **Artefatos** exibido, clique em **+ Criar feed** na parte superior do painel.

    > **Observação**: esse feed será uma coleção de pacotes NuGet disponíveis para usuários dentro da organização e ficará ao lado do feed NuGet público como um par. O cenário deste laboratório se concentrará no fluxo de trabalho para usar Azure Artifacts, portanto, as decisões reais de arquitetura e desenvolvimento são meramente ilustrativas.  Esse feed incluirá funcionalidades comuns que podem ser compartilhadas entre projetos nesta organização.

3. No painel **Criar novo feed**, na caixa de texto **Nome**, digite **EShopOnWebShared**, na seção **Escopo**, selecione a opção **Organização**, deixe as outras configurações com os valores padrão e clique em **Criar**.

    > **Observação**: um usuário que queira se conectar a esse feed NuGet deve configurar seu ambiente.

4. De volta ao hub **Artefatos**, clique em **Conectar ao feed**.
5. No painel **Conectar ao feed**, na seção **NuGet**, selecione **Visual Studio** e no painel **Visual Studio**, copie a URL **Origem**. (https://pkgs.dev.azure.com/<Azure-DevOps-Org-Name>_packaging/EShopOnWebShared/nuget/v3/index.json)
6. Alterne de volta para a janela do **Visual Studio**.
7. Na janela do Visual Studio, clique no cabeçalho de menu **Ferramentas**, no menu suspenso, selecione **Gerenciador de pacotes NuGet** e no menu de cascata, selecione **Configurações do gerenciador de pacotes**.
8. Na caixa de diálogo **Opções**, clique em **Origem do pacote** e clique no sinal de adição para adicionar uma nova origem do pacote.
9. Na parte inferior da caixa de diálogo, na caixa de texto **Nome**, substitua **Origem do pacote** com **EShopOnWebShared** e na caixa de texto **Origem**, cole a URL que você copiou no portal do Azure DevOps.
10. Clique em **Atualizar** e depois clique em **OK** para finalizar a adição.

    > **Observação**: o Visual Studio agora está conectado ao novo feed.

#### Tarefa 2: criar e publicar um pacote NuGet desenvolvido internamente

Nesta tarefa, você criará e publicará um pacote NuGet personalizado desenvolvido internamente.

1. Na janela do Visual Studio usada para configurar a nova origem do pacote, no menu principal, clique em **Arquivo**, no menu suspenso, clique em **Novo** e, em seguida, no menu em cascata, clique em **Projeto**.

    > **Observação**: agora criaremos um assembly compartilhado que será publicado como um pacote NuGet para que outras equipes possam integrá-lo e se manter atualizadas sem precisar trabalhar diretamente com a origem do projeto.

2. Na página **Modelos de projeto recentes** do painel **Criar um novo projeto**, use a caixa de texto de pesquisa para localizar o modelo **Biblioteca de classes**, selecione o modelo para C# e clique em **Avançar**.
3. Na página **Biblioteca de classes** do painel **Criar um novo projeto**, especifique as seguintes configurações e clique em **Criar**:

    | Configuração | Valor |
    | --- | --- |
    | Nome do projeto | **EShopOnWeb.Shared** |
    | Localização | aceitar o valor padrão |
    | Solução | **Criar nova solução** |
    | Solução nome | **EShopOnWeb.Shared** |

    Deixe a configuração **Colocar solução e o projeto no mesmo diretório** ativada.

4. Clique em Avançar. Aceite o **.NET 6.0 (suporte de longo prazo)** como opção do Framework.
5. Confirme a criação do projeto pressionando o botão **Criar**.
6. Na interface do Visual Studio, no painel **Gerenciador de soluções**, clique com o botão direito do mouse em **Class1.cs**, no menu de clique com o botão, selecione **Excluir** e quando for solicitada a confirmação, clique em **OK**.
7. Pressione **Ctrl+Shift+B** ou **clique com o botão direito do mouse no projeto EShopOnWeb.Shared** e selecione **Compilar** para compilar o projeto.

    > **Observação**: na próxima tarefa, usaremos o **NuGet.exe** para gerar um pacote NuGet diretamente do projeto compilado, mas ele requer que o projeto seja compilado primeiro.

8. Alterne para a guia do navegador da Web que está que exibe o portal do Azure DevOps.
9. Navegue até o painel **Conectar ao feed**, na seção **NuGet** e selecione **NuGet.exe**. Isso exibirá o painel **NuGet.exe**.
10. No painel **NuGet.exe**, clique em **Obter as ferramentas**.
11. No painel **Obter as ferramentas**, clique no link **Baixar o NuGet mais recente**. Isso abrirá automaticamente uma nova guia do navegador exibindo a página **Versões disponíveis de distribuição do NuGet**.
12. Na página **Versões disponíveis de distribuição do NuGet**, selecione **nuget.exe - recomendado o v6.x mais recente** e faça download do executável para a pasta local **EShopOnWeb.Shared Project** (se você manteve as localizações padrão de pasta, ela deve ser C:\EShopOnWeb\EShopOnWeb.Shared).
13. Selecione o arquivo **nuget.exe** e abra as propriedades dele clicando com o botão direito do mouse no arquivo e selecionando **Propriedades** no menu de contexto.
14. Na janela de contexto Propriedades, na guia **Geral**, selecione **Desbloquear** na seção Segurança. Confirme pressionando **Aplicar** e **OK**.
15. Na estação de trabalho de laboratório, abra o menu Iniciar e procure **Windows PowerShell**. Em seguida, no menu em cascata, clique em **Abrir o Windows PowerShell como administrador**.
16. Na janela **Administrador: Windows PowerShell**, navegue até a pasta EShopOnWeb.Shared executando o seguinte comando:

    ```text
    cd c:\EShopOnWeb\EShopOnWeb.Shared
    ```

    Execute o seguinte para criar um arquivo **.nupkg** do projeto.

    > **Observação**: este é um atalho para empacotar os bits do NuGet para implantação. O NuGet é altamente personalizável. Para saber mais, consulte a [página de criação de pacotes NuGet](https://docs.microsoft.com/nuget/create-packages/overview-and-workflow).

    ```text
    .\nuget.exe pack ./EShopOnWeb.Shared.csproj
    ```

    > **Observação**: desconsidere quaisquer avisos exibidos na janela **Administrador: Windows PowerShell**.

    > **Observação**: o NuGet cria um pacote mínimo com base nas informações que ele é capaz de identificar do projeto. Por exemplo, observe que o nome é **EShopOnWeb.Shared.1.0.0.nupkg**. Esse número de versão foi recuperado da assembly.

17. Após a criação bem-sucedida do pacote, execute o seguinte para publicar o pacote no feed do**EShopOnWebShared**:

    > **Observação**: você precisa fornecer uma **chave de API**, que pode ser qualquer cadeia de caracteres não vazia. Estamos usando **AzDO** aqui. Quando solicitado, entre na sua organização do Azure DevOps.

    ```text
    .\nuget.exe push -source "EShopOnWebShared" -ApiKey AzDO EShopOnWeb.Shared.1.0.0.nupkg
    ```

18. Aguarde a confirmação da operação de envio de pacote bem-sucedida.
19. Alterne para a janela do navegador da Web que exibe o portal do Azure DevOps, no painel de navegação vertical, selecione **Artefatos**.
20. No painel do hub **Artefatos**, clique na lista suspensa no canto superior esquerdo e, na lista de feeds, selecione a entrada **EShopOnWebShared**.

    > **Observação**: o feed **EShopOnWebShared** deve incluir o pacote NuGet recém-publicado.

21. Clique no pacote NuGet para exibir os detalhes.

#### Tarefa 3: importar um pacote NuGet de código aberto para o feed de pacotes do Azure DevOps

Além de desenvolver seus próprios pacotes, por que não usar o NuGet de código aberto (https://www.nuget.org) biblioteca de pacotes DotNet? Com alguns milhões de pacotes disponíveis, sempre haverá algo útil para o aplicativo.

Nesta tarefa, usaremos um pacote de exemplo genérico "Olá, Mundo", mas você pode usar a mesma abordagem para outros pacotes na biblioteca.

1. Na mesma janela do PowerShell, execute o seguinte comando **nuget** para instalar o pacote de exemplo:

    ```text
    .\nuget install HelloWorld -ExcludeVersion
    ```

2. Verifique a saída do processo de instalação. Na primeira linha, ela mostra os diferentes feeds que tentará para baixar o pacote:

    ```text
    Feeds used:
      https://api.nuget.org/v3/index.json
      https://pkgs.dev.azure.com/<AZURE_DEVOPS_ORGANIZATION>/eShopOnWeb/_packaging/EShopOnWebPFeed/nuget/v3/index.json
    ```

3. Em seguida, ele mostrará uma saída adicional em relação ao processo de instalação.

    ```text
    Installing package 'Helloworld' to 'C:\eShopOnWeb\EShopOnWeb.Shared'.
      GET https://api.nuget.org/v3/registration5-gz-semver2/helloworld/index.json
      OK https://api.nuget.org/v3/registration5-gz-semver2/helloworld/index.json 114ms
    MSBuild auto-detection: using msbuild version '17.5.0.10706' from 'C:\Program Files\Microsoft Visual Studio\2022\Professional\MSBuild\Current\bin'.
      GET https://pkgs.dev.azure.com/pdtdemoworld/7dc3351f-bb0c-42ba-b3c9-43dab8e0dc49/_packaging/188ec0d5-ff93-4eb7-b9d3-293fbf759f06/nuget/v3/registrations2-semver2/helloworld/index.json
      OK https://pkgs.dev.azure.com/pdtdemoworld/7dc3351f-bb0c-42ba-b3c9-43dab8e0dc49/_packaging/188ec0d5-ff93-4eb7-b9d3-293fbf759f06/nuget/v3/registrations2-semver2/helloworld/index.json 698ms
    
    Attempting to gather dependency information for package 'Helloworld.1.3.0.17' with respect to project 'C:\eShopOnWeb\EShopOnWeb.Shared', targeting 'Any,Version=v0.0'
    Gathering dependency information took 21 ms
    Attempting to resolve dependencies for package 'Helloworld.1.3.0.17' with DependencyBehavior 'Lowest'
    Resolving dependency information took 0 ms
    Resolving actions to install package 'Helloworld.1.3.0.17'
    Resolved actions to install package 'Helloworld.1.3.0.17'
    Retrieving package 'HelloWorld 1.3.0.17' from 'nuget.org'.
      GET https://api.nuget.org/v3-flatcontainer/helloworld/1.3.0.17/helloworld.1.3.0.17.nupkg
      OK https://api.nuget.org/v3-flatcontainer/helloworld/1.3.0.17/helloworld.1.3.0.17.nupkg 133ms
    Installed HelloWorld 1.3.0.17 from https://api.nuget.org/v3/index.json with content hash 1Pbk5sGihV5JCE5hPLC0DirUypeW8hwSzfhD0x0InqpLRSvTEas7sPCVSylJ/KBzoxbGt2Iapg72WPbEYxLX9g==.
    Adding package 'HelloWorld.1.3.0.17' to folder 'C:\eShopOnWeb\EShopOnWeb.Shared'
    Added package 'HelloWorld.1.3.0.17' to folder 'C:\eShopOnWeb\EShopOnWeb.Shared'
    Successfully installed 'HelloWorld 1.3.0.17' to C:\eShopOnWeb\EShopOnWeb.Shared
    Executing nuget actions took 686 ms
    ```

4. O pacote HelloWorld foi instalado em uma subpasta **HelloWorld**, na pasta EShopOnWeb.Shared. No Gerenciador de soluções do **Visual Studio**, navegue até o projeto **EShopOnWeb.Shared** e observe a subpasta **HelloWorld**. Clique na pequena seta à esquerda da subpasta para abrir a lista de pastas e arquivos.
5. Observe a subpasta **lib**, com um arquivo de assinatura **signature.p7s**, que prova a origem do pacote. Em seguida, observe o próprio arquivo de pacote **HelloWorld.nupkg**.

#### Tarefa 4: carregar o pacote NuGet de código aberto no Azure Artifacts

Vamos considerar este pacote um pacote "aprovado" para nossa equipe de DevOps reutilizar, carregando-o no feed de pacotes do Azure Artifacts criado anteriormente.

1. Na janela do PowerShell, execute o seguinte comando:

    ```text
    .\nuget.exe push -source "EShopOnWebShared" -ApiKey AzDO c:\EShopOnWeb\EShopOnWeb.Shared\HelloWorld\HelloWorld.nupkg
    ```

    > **Observação**: isso resulta em uma mensagem de erro:

    ```text
    Response status code does not indicate success: 409 (Conflict - 'HelloWorld 1.3.0.17' cannot be published to the feed because it exists in at least one of the feed's upstream sources. Publishing this copy would prevent you from using 'HelloWorld 1.3.0.17' from 'NuGet Gallery'. For more information, see https://go.microsoft.com/fwlink/?linkid=864880 (DevOps Activity ID: AE08BE89-C2FA-4FF7-89B7-90805C88972C)).
    ```

Quando você criou o feed de pacotes de artefatos do Azure DevOps, por design, ele permite **fontes upstream**, como nuget.org no exemplo dotnet. No entanto, nada impede que sua equipe de DevOps crie um feed de pacotes **"somente interno"**.

1. Navegue até o portal do Azure DevOps, navegue até **Artefatos** e selecione o feed do**EShopOnWebShared**.
2. Clique em **Pesquisar fontes upstream**
3. Na janela **Ir para um pacote upstream**, selecione **NuGet** como Tipo de pacote e insira **HelloWorld** no campo de pesquisa.
4. Confirme pressionando o botão **Pesquisar**.
5. Isso resulta em uma lista de todos os pacotes HelloWorld com as diferentes versões disponíveis.
6. Clique na **tecla de seta para a esquerda** para retornar ao feed do **EShopOnWebShared**.
7. Clique na engrenagem para abrir as **Configurações de feed**. Na página Configurações do Feed, selecione **Fontes upstream**.
8. Observe os diferentes gerenciadores de pacotes upstream para diferentes linguagens de desenvolvimento. Selecione **Galeria NuGet** na lista. Pressione o botão **Excluir** e, em seguida, pressione o botão **Salvar**.

9. Com essas mudanças salvas, será possível carregar o pacote **HelloWorld** usando o NuGet.exe da janela do PowerShell, reiniciando o comando a seguir:

    ```text
     .\nuget.exe push -source "EShopOnWebShared" -ApiKey AzDO c:\EShopOnWeb\EShopOnWeb.Shared\HelloWorld\HelloWorld.nupkg
    ```

    > **Observação**: isso agora deve resultar em um carregamento bem-sucedido 

    ```text
    Pushing HelloWorld.nupkg to 'https://pkgs.dev.azure.com/pdtdemoworld/7dc3351f-bb0c-42ba-b3c9-43dab8e0dc49/_packaging/188ec0d5-ff93-4eb7-b9d3-293fbf759f06/nuget/v2/'...
      PUT https://pkgs.dev.azure.com/<AZUREDEVOPSORGANIZATION>/7dc3351f-bb0c-42ba-b3c9-43dab8e0dc49/_packaging/188ec0d5-ff93-4eb7-b9d3-293fbf759f06/nuget/v2/
    MSBuild auto-detection: using msbuild version '17.5.0.10706' from 'C:\Program Files\Microsoft Visual Studio\2022\Professional\MSBuild\Current\bin'.
      Accepted https://pkgs.dev.azure.com/pdtdemoworld<AZUREDEVOPSORGANIZATION>/7dc3351f-bb0c-42ba-b3c9-43dab8e0dc49/_packaging/188ec0d5-ff93-4eb7-b9d3-293fbf759f06/nuget/v2/ 1645ms
    Your package was pushed.
    PS C:\eShopOnWeb\EShopOnWeb.Shared>
    ```

10. No portal do Azure DevOps, **atualize** a página Feed do pacote de artefatos. A lista de pacotes mostra o pacote desenvolvido sob medida **EShopOnWeb.Shared**, além do pacote de origem pública **HelloWorld**.
11. Na Solução **EShopOnWeb.Shared** do Visual Studio, clique com o botão direito do mouse no projeto **EShopOnWeb.Shared** e selecione **Gerenciar pacotes NuGet** no menu de contexto.
12. Na janela Gerenciador de pacotes NuGet, valide se a ** Origem do pacote** está definida como **EShopOnWebShared**.
13. Clique em **Procurar** e aguarde até que a lista de pacotes NuGet seja carregada.
14. A lista mostrará também o pacote desenvolvido sob medida **EShopOnWeb.Shared**, além do pacote de origem pública **HelloWorld**.

## Revisão

Neste laboratório, você aprendeu a trabalhar com o Azure Artifacts usando as seguintes etapas:

- Criar e se conectar a um feed.
- Criar e publicar um pacote NuGet.
- Importar um pacote NuGet desenvolvido sob medida.
- Importar um pacote NuGet de origem pública.
