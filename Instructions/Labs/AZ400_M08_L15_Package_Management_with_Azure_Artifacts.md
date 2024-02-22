---
lab:
  title: Package Management with Azure Artifacts
  module: 'Module 08: Design and implement a dependency management strategy'
---

# Package Management with Azure Artifacts

## Manual de laboratório do aluno

## Requisitos do laboratório

- Este laboratório requer o **Microsoft Edge** ou um [navegador com suporte do Azure DevOps](https://docs.microsoft.com/azure/devops/server/compatibility).

- **Configurar uma organização do Azure DevOps:** se você ainda não tiver uma organização do Azure DevOps que possa usar para este laboratório, crie uma seguindo as instruções disponíveis em [Pré-requisitos do laboratório AZ-400](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/AZ400_M00_Validate_lab_environment.html).

- **Configure o projeto eShopOnWeb de exemplo:** Se você ainda não tiver o projeto eShopOnWeb de exemplo que poderá usar neste laboratório, crie um seguindo as instruções disponíveis em [Pré-requisitos do Laboratório AZ-400](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/AZ400_M00_Validate_lab_environment.html).

- O Visual Studio 2022 Community Edition está disponível na [página de Downloads do Visual Studio](https://visualstudio.microsoft.com/downloads/). A instalação do Visual Studio 2022 deve incluir as cargas de trabalho **ASP<nolink>.NET e desenvolvimento da Web**, **desenvolvimento do Azure** e **desenvolvimento entre plataformas do .NET Core**.

- **SDK do .NET Core:** [Baixe e instale o SDK do .NET Core (2.1.400+)](https://go.microsoft.com/fwlink/?linkid=2103972)

- **Provedor de credenciais do Azure Artifacts:** [Baixe e instale o provedor de credenciais](https://go.microsoft.com/fwlink/?linkid=2099625).

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

Neste exercício, lembre-se de validar os pré-requisitos do laboratório, ter uma organização do Azure DevOps pronta e ter criado o projeto eShopOnWeb. Consulte as instruções acima para obter mais detalhes.

#### Tarefa 1: Configuração da solução eShopOnWeb no Visual Studio

Nesta tarefa, você configurará o Visual Studio para se preparar para o laboratório.

1. Certifique-se de que você esteja exibindo o projeto de equipe do **eShopOnWeb** no portal do Azure DevOps.

    > **Observação**: Você pode acessar a página do projeto diretamente navegando até a URL [https://dev.azure.com/`<your-Azure-DevOps-account-name>`/eShopOnWeb](https://dev.azure.com/`<your-Azure-DevOps-account-name>`/eShopOnWeb), em que o espaço reservado `<your-Azure-DevOps-account-name>` representa o nome da organização do Azure DevOps.

1. No menu vertical no lado esquerdo do painel **eShopOnWeb**, clique em **Repos**.
1. No painel **Arquivos**, clique em **Clonar**, selecione a seta suspensa ao lado de **Clonar no VS Code** e, no menu suspenso, selecione **Visual Studio**.
1. Se for perguntado se deseja continuar, clique em **Abrir**.
1. Se solicitado, entre com a conta de usuário que você usou para configurar sua organização do Azure DevOps.
1. Na interface do Visual Studio, na janela pop-up do **Azure DevOps**, aceite o caminho local padrão (C:\eShopOnWeb) e clique em **Clonar**. Isso importará automaticamente o projeto para o Visual Studio.
1. Deixe a janela do Visual Studio aberta para usar em seu laboratório.

### Exercício 1: trabalhar com o Azure Artifacts

Neste exercício, você aprenderá a trabalhar com o Azure Artifacts usando as seguintes etapas:

- Criar e se conectar a um feed.
- Criar e publicar um pacote NuGet.
- Importar um pacote NuGet.
- Atualizar um pacote NuGet.

#### Tarefa 1: criar e conectar-se a um feed

Nesta tarefa, você criará e se conectará a um feed.

1. Na janela do navegador da Web que exibe suas configurações de projeto no portal do Azure DevOps, no painel de navegação vertical, selecione **Artefatos**.
1. Com o hub **Artefatos** exibido, clique em **+ Criar feed** na parte superior do painel.

    > **Observação**: esse feed será uma coleção de pacotes do NuGet disponíveis para usuários dentro da organização e ficará ao lado do feed do NuGet público como um par. O cenário neste laboratório se concentrará no fluxo de trabalho para usar o Azure Artifacts, portanto, as decisões de arquitetura e desenvolvimento são meramente ilustrativas.  Esse feed incluirá funcionalidades comuns que podem ser compartilhadas entre projetos nesta organização.

1. No painel **Criar novo feed**, na caixa de texto **Nome**, digite **eShopOnWebShared**, na seção **Escopo**, selecione a opção **Organização**, deixe outras configurações com seus valores padrão e clique em **Criar**.

    > **Observação**: qualquer usuário que queira se conectar a esse feed do NuGet deve configurar seu ambiente.

1. De volta ao hub **Artefatos**, clique em **Conectar ao feed**.
1. No painel **Conectar ao feed**, na seção **NuGet**, selecione **Visual Studio** e, no painel **Visual Studio**, copie a URL de **Origem**. `https://pkgs.dev.azure.com/Azure-DevOps-Org-Name/_packaging/eShopOnWebShared/nuget/v3/index.json`
1. Volte para a janela do **Visual Studio**.
1. Na janela do Visual Studio, clique no cabeçalho do menu **Ferramentas**. No menu suspenso, selecione **Gerenciador de Pacotes NuGet** e, no menu em cascata, selecione **Configurações do Gerenciador de Pacotes**.
1. Na caixa de diálogo **Opções**, clique em **Origens dos pacotes** e clique no sinal de adição para adicionar uma nova origem de pacote.
1. Na parte inferior da caixa de diálogo, na caixa de texto **Nome**, substitua a **Origem do pacote** por **eShopOnWebShared** e, na caixa de texto **Origem**, cole a URL copiada no portal do Azure DevOps. 
1. Clique em **Atualizar** e em **OK** para finalizar a adição.

    > **Observação**: o Visual Studio agora está conectado ao novo feed.

#### Tarefa 2: criar e publicar um pacote NuGet desenvolvido internamente

Nesta tarefa, você criará e publicará um pacote NuGet personalizado desenvolvido internamente.

1. Na janela do Visual Studio usada para configurar a nova origem do pacote, no menu principal, clique em **Arquivo**. No menu suspenso, clique em **Novo** e, em seguida, no menu em cascata, clique em **Projeto**.

    > **Observação**: agora criaremos um assembly compartilhado que será publicado como um pacote NuGet para que outras equipes possam integrá-lo e manter-se atualizadas sem precisar trabalhar diretamente com a origem do projeto.

1. Na página **Modelos de projetos recentes** do painel **Criar um novo projeto**, use a caixa de texto de pesquisa para localizar o modelo **Biblioteca de Classes**, selecione o modelo para C# e clique em **Avançar**.
1. Na página **Biblioteca de Classes** do painel **Criar um novo projeto**, especifique as seguintes configurações e clique em **Criar**:

    | Configuração | Valor |
    | --- | --- |
    | Nome do projeto | **eShopOnWeb.Shared** |
    | Localização | aceitar o valor padrão |
    | Solução | **Criar nova solução** |
    | Solução nome | **eShopOnWeb.Shared** |

    Marque a caixa de seleção para **colocar a solução e o projeto no mesmo diretório**.

1. Clique em Avançar. Aceite o **.NET 8.0** como a opção de estrutura.
1. Confirme a criação do projeto pressionando botão **Criar**.
1. Na interface do Visual Studio, no painel **Gerenciador de Soluções**, clique com o botão direito do mouse em **Class1.cs**. No menu de contexto, selecione **Excluir** e, quando for solicitada a confirmação, clique em **OK.**
1. Pressione **Ctrl+Shift+B** ou **Clique com o botão direito do mouse no projeto EShopOnWeb.Shared** e selecione **Compilar** para compilar o projeto.
1. Na estação de trabalho de laboratório, abra o menu Iniciar e procure **Windows PowerShell**. Em seguida, no menu em cascata, clique em **Abrir o Windows PowerShell como administrador**.
1. **No Administrador: Janela do Windows PowerShell**, navegue até a pasta eShopOnWeb.Shared executando o seguinte comando:

    ```text
    cd c:\eShopOnWeb\eShopOnWeb.Shared
    ```

    > **Observação**: A pasta **eShopOnWeb.Shared** é o local do arquivo **eShopOnWeb.Shared.csproj**. Se você escolheu um local diferente, navegue até esse local.

1. Execute o seguinte comando para criar um arquivo **.nupkg** no projeto.

    ```powershell
    dotnet pack .\eShopOnWeb.Shared.csproj
    ```

    > **Observação**: O comando **dotnet pack** cria o projeto e cria um pacote NuGet na pasta **bin\Release**.

    > **Observação**: desconsidere quaisquer avisos exibidos na janela **Administrador: Windows PowerShell**.

    > **Observação**: o dotnet pack cria um pacote mínimo com base nas informações que ele pode identificar do projeto. Por exemplo, observe que o nome é **eShopOnWeb.Shared.1.0.0.nupkg**. Esse número de versão foi recuperado do assembly.

1. Na janela do PowerShell, execute o seguinte comando para abrir a pasta **bin\Release**:

    ```powershell
    cd .\bin\Release
    ```

1. Execute o seguinte para publicar o pacote no feed **eShopOnWebShared**:

    > **Importante**: Você precisa instalar o provedor de credenciais para que o seu sistema operacional possa se autenticar com o Azure DevOps. Você pode encontrar as instruções de instalação no [Provedor de credenciais do Azure Artifacts](https://go.microsoft.com/fwlink/?linkid=2099625). Você pode fazer a instalação executando o seguinte comando na janela do PowerShell: `iex "& { $(irm https://aka.ms/install-artifacts-credprovider.ps1) } -AddNetfx"`

    > **Observação**: você precisa fornecer uma **chave de API**, que pode ser qualquer cadeia de caracteres não vazia. Estamos usando **az** aqui. Quando solicitado, entre na sua organização do Azure DevOps.

    ```powershell
    dotnet nuget push --source "eShopOnWebShared" --api-key az eShopOnWeb.Shared.1.0.0.nupkg
    ```

    > **Observação**: Se o prompt não aparecer, você poderá adicionar o parâmetro **--interactive** ao comando.

1. Aguarde a confirmação da operação de envio de pacote bem-sucedida.
1. Alterne para a janela do navegador da Web que exibe o portal do Azure DevOps e, no painel de navegação vertical, selecione **Artefatos**.
1. No painel do hub **Artefatos**, clique na lista suspensa no canto superior esquerdo e, na lista de feeds, selecione a entrada **eShopOnWebShared**.

    > **Observação**: O feed **eShopOnWebShared** deve incluir o pacote NuGet publicado recentemente.

1. Clique no pacote NuGet para exibir os detalhes.

#### Tarefa 3: importar um pacote NuGet de código aberto para o feed de pacotes do Azure DevOps

Além de desenvolver seus próprios pacotes, por que não usar a biblioteca de pacotes DotNet do NuGet de Código Aberto (<https://www.nuget.org>)? Com alguns milhões de pacotes disponíveis, sempre haverá algo útil para o seu aplicativo.

Nesta tarefa, usaremos um pacote de exemplo genérico "Newtonsoft.Json", mas você pode usar a mesma abordagem para outros pacotes na biblioteca.

1. Na mesma janela do PowerShell, navegue até a pasta **eShopOnWeb.Shared** e execute o seguinte comando **dotnet** para instalar o pacote de exemplo:

    ```powershell
    dotnet add package Newtonsoft.Json
    ```

1. Verifique a saída do processo de instalação. Ele mostra os diferentes feeds dos quais tentará baixar o pacote:

    ```powershell
    Feeds used:
      https://api.nuget.org/v3/registration5-gz-semver2/newtonsoft.json/index.json
      https://pkgs.dev.azure.com/<AZURE_DEVOPS_ORGANIZATION>/eShopOnWeb/_packaging/eShopOnWebShared/nuget/v3/index.json
    ```

1. Em seguida, ele mostrará uma saída adicional em relação ao processo de instalação em si.

    ```powershell
    Determining projects to restore...
    Writing C:\Users\AppData\Local\Temp\tmpxnq5ql.tmp
    info : X.509 certificate chain validation will use the default trust store selected by .NET for code signing.
    info : X.509 certificate chain validation will use the default trust store selected by .NET for timestamping.
    info : Adding PackageReference for package 'Newtonsoft.Json' into project 'c:\eShopOnWeb\eShopOnWeb.Shared\eShopOnWeb.Shared.csproj'.
    info :   GET https://api.nuget.org/v3/registration5-gz-semver2/newtonsoft.json/index.json
    info :   OK https://api.nuget.org/v3/registration5-gz-semver2/newtonsoft.json/index.json 124ms
    info : Restoring packages for c:\eShopOnWeb\eShopOnWeb.Shared\eShopOnWeb.Shared.csproj...
    info :   GET https://api.nuget.org/v3/vulnerabilities/index.json
    info :   OK https://api.nuget.org/v3/vulnerabilities/index.json 84ms
    info :   GET https://api.nuget.org/v3-vulnerabilities/2024.02.15.23.23.24/vulnerability.base.json
    info :   GET https://api.nuget.org/v3-vulnerabilities/2024.02.15.23.23.24/2024.02.17.11.23.35/vulnerability.update.json
    info :   OK https://api.nuget.org/v3-vulnerabilities/2024.02.15.23.23.24/vulnerability.base.json 14ms
    info :   OK https://api.nuget.org/v3-vulnerabilities/2024.02.15.23.23.24/2024.02.17.11.23.35/vulnerability.update.json 30ms
    info : Package 'Newtonsoft.Json' is compatible with all the specified frameworks in project 'c:\eShopOnWeb\eShopOnWeb.Shared\eShopOnWeb.Shared.csproj'.
    info : PackageReference for package 'Newtonsoft.Json' version '13.0.3' added to file 'c:\eShopOnWeb\eShopOnWeb.Shared\eShopOnWeb.Shared.csproj'.
    info : Writing assets file to disk. Path: c:\eShopOnWeb\eShopOnWeb.Shared\obj\project.assets.json
    log  : Restored c:\eShopOnWeb\eShopOnWeb.Shared\eShopOnWeb.Shared.csproj (in 294 ms).
    ```

1. O pacote Newtonsoft.Json foi instalado nos Pacotes como **Newtonsoft.Json**. No **Gerenciador de Soluções** do Visual Studio, navegue até o projeto **eShopOnWeb.Shared**, expanda Dependências e observe o **Newtonsoft.Json** nos Pacotes. Clique na pequena seta à esquerda dos pacotes para abrir a pasta e a lista de arquivos.

#### Tarefa 4: carregar o pacote NuGet de código aberto no Azure Artifacts

Vamos considerar este pacote um pacote "aprovado" para nossa equipe de DevOps reutilizar carregando-o no feed de Pacotes do Azure Artifacts criado anteriormente.

1. No Visual Studio, clique com o botão direito do mouse no novo pacote **Newtonsoft.Json** e selecione **Abrir pasta no Explorador de Arquivos** no menu de contexto. Você verá o novo pacote **Newtonsoft.Json** com a extensão **.nupkg**.
2. Copie o caminho completo da barra de endereços da janela do Explorador de Arquivos.
3. Na janela do PowerShell, execute o seguinte comando substituindo o caminho pelo que você copiou:

    ```powershell
    dotnet nuget push --source "eShopOnWebShared" --api-key az C:\eShopOnWeb\eShopOnWeb.Shared\Newtonsoft.Json\newtonsoft.json.13.0.3.nupkg
    ```

    > **Observação**: isso resulta em uma mensagem de erro:

    ```text
    Response status code does not indicate success: 409 (Conflict - 'Newtonsoft.Json 1.3.0.17' cannot be published to the feed because it exists in at least one of the feed's upstream sources. Publishing this copy would prevent you from using 'Newtonsoft.Json 1.3.0.17' from 'NuGet Gallery'. For more information, see https://go.microsoft.com/fwlink/?linkid=864880 (DevOps Activity ID: AE08BE89-C2FA-4FF7-89B7-90805C88972C)).
    ```

Quando você criou o Feed de Pacotes de Artefatos do Azure DevOps, por padrão, ele permite **fontes upstream**, como nuget.org no exemplo dotnet. No entanto, nada impede que sua equipe de DevOps crie um Feed de Pacotes **"somente interno"**.

1. Navegue até o portal do Azure DevOps, navegue até **Artefatos** e selecione o feed **eShopOnWebShared**.
1. Clique em **Pesquisar fontes upstream**
1. Na janela **Ir para um pacote upstream**, selecione **NuGet** como o tipo de pacote e insira **Newtonsoft.Json** no campo de pesquisa.
1. Confirme pressionando o botão **Pesquisar**.
1. Isso resulta em uma lista de todos os pacotes Newtonsoft.Json com as diferentes versões disponíveis.
1. Clique na tecla de **tecla de direção esquerda** para retornar ao feed **eShopOnWebShared**.
1. Clique na engrenagem para abrir as **Configurações do Feed**. Na página Configurações do Feed, selecione **Fontes Upstream**.
1. Observe os diferentes Gerenciadores de Pacotes Upstream para diferentes linguagens de desenvolvimento. Selecione **Galeria do NuGet** na lista. Pressione o botão **Excluir** e, em seguida, pressione o botão **Salvar**.

1. Com essas alterações salvas, será possível carregar o pacote **Newtonsoft.Json** usando o NuGet.exe da janela do PowerShell, relançando o seguinte comando:

    ```text
     dotnet nuget push --source "eShopOnWebShared" --api-key az C:\eShopOnWeb\eShopOnWeb.Shared\Newtonsoft.Json\newtonsoft.json.13.0.3.nupkg
    ```

    > **Observação**: Isso agora deve resultar em um upload bem-sucedido.

    ```text
    Pushing newtonsoft.json.13.0.3.nupkg to 'https://pkgs.dev.azure.com/<AZURE_DEVOPS_ORGANIZATION>/_packaging/5faffb6c-018b-4452-a4d6-72c6bffe79db/nuget/v2/'...
    PUT https://pkgs.dev.azure.com/<AZURE_DEVOPS_ORGANIZATION>/_packaging/5faffb6c-018b-4452-a4d6-72c6bffe79db/nuget/v2/
    Accepted https://pkgs.dev.azure.com/<AZURE_DEVOPS_ORGANIZATION>/_packaging/5faffb6c-018b-4452-a4d6-72c6bffe79db/nuget/v2/ 3160ms
    Your package was pushed.
    ```

1. No Portal do Azure DevOps, **atualize** a página Feed de Pacotes de Artefatos. A lista de pacotes mostra o pacote personalizado **eShopOnWeb.Shared**, bem como o pacote de origem pública **Newtonsoft.Json**.
1. Na solução do Visual Studio **eShopOnWeb.Shared**, clique com o botão direito do mouse no projeto **eShopOnWeb.Shared** e selecione **Gerenciar pacotes NuGet** no menu de contexto.
1. Na janela do Gerenciador de pacotes NuGet, valide se a **origem do pacote** está definida como **eShopOnWebShared**.
1. Clique em **Procurar** e aguarde até que a lista de Pacotes NuGet seja carregada.
1. Essa lista também mostrará o pacote de desenvolvimento personalizado **eShopOnWeb.Shared**, bem como o pacote de origem pública **Newtonsoft.Json**.

## Revisão

Neste laboratório, você aprendeu a trabalhar com o Azure Artifacts usando as seguintes etapas:

- Criar e se conectar a um feed.
- Criar e publicar um pacote NuGet.
- Importar um pacote NuGet personalizado.
- Importar um pacote NuGet de origem pública.
