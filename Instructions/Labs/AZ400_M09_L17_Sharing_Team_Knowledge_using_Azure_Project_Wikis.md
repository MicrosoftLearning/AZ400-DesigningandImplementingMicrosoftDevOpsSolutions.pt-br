---
lab:
  title: Sharing Team Knowledge using Azure Project Wikis
  module: 'Module 09: Implement continuous feedback'
---

# Sharing Team Knowledge using Azure Project Wikis

## Manual de laboratório do aluno

## Requisitos do laboratório

- Este laboratório requer o **Microsoft Edge** ou um [navegador com suporte do Azure DevOps](https://docs.microsoft.com/azure/devops/server/compatibility).

- **Configurar uma organização do Azure DevOps:** se você ainda não tiver uma organização do Azure DevOps que possa usar para este laboratório, crie uma seguindo as instruções disponíveis em [Pré-requisitos do laboratório AZ-400](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/AZ400_M00_Validate_lab_environment.html).

- **Configure o projeto eShopOnWeb de exemplo:** Se você ainda não tiver o projeto eShopOnWeb de exemplo que poderá usar neste laboratório, crie um seguindo as instruções disponíveis em [Pré-requisitos do Laboratório AZ-400](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/AZ400_M00_Validate_lab_environment.html).

## Visão geral do laboratório

Neste laboratório, você criará e configurará um wiki no Azure DevOps, incluindo o gerenciamento de conteúdo markdown e a criação de um diagrama do Mermaid.

## Objetivos

Após concluir este Laboratório, você poderá:

- Crie um wiki em um Projeto do Azure.
- Adicionar e editar markdown.
- Crie um diagrama do Mermaid.

## Tempo estimado: 45 minutos

## Instruções

### Exercício 0: configurar os pré-requisitos do laboratório

Neste exercício, lembre-se de validar os pré-requisitos do laboratório, ter uma organização do Azure DevOps pronta e ter criado o projeto eShopOnWeb. Consulte as instruções acima para obter mais detalhes.

### Exercício 1: publicar código como wiki

Neste exercício, você passará pela publicação de um repositório do Azure DevOps como wiki e pelo gerenciamento do wiki publicado.

> **Observação**: o conteúdo que você mantém em um repositório Git pode ser publicado em um wiki do Azure DevOps. Por exemplo, o conteúdo escrito para dar suporte a um Software Development Kit, documentação do produto ou arquivos LEIA-ME pode ser publicado diretamente em um wiki. Você tem a opção de publicar vários wikis no mesmo projeto de equipe do Azure DevOps.

#### Tarefa 1: Publicar um branch de um repositório do Azure DevOps como wiki

Nesta tarefa, você publicará um branch de um repositório do Azure DevOps como wiki.

> **Observação**: se o wiki publicado corresponder a uma versão do produto, você poderá publicar novas branches à medida que lança novas versões do produto.

1. No menu vertical à esquerda, clique em **Repos**, na seção superior do painel **Arquivos**, verifique se o repositório **eShopOnWeb** está selecionado (escolha-o na lista suspensa, na parte superior com o ícone do Git). Na lista suspensa de branches (na parte superior de "Arquivos" com o ícone de branch), selecione **principal** e revise o conteúdo do branch principal.
1. À esquerda do **painel Arquivos**, na listagem da pasta repo e da hierarquia de arquivos, expanda a pasta **src** e navegue até a subpasta para **Web-> wwwroot -> images**. Na subpasta **Imagens**, localize a entrada **brand.png**, passe o mouse com o ponteiro do mouse sobre sua extremidade direita para revelar o símbolo de reticências verticais (três pontos) que representa o menu **Mais**, clique em **Download** para baixar o **arquivo brand.png**para a pasta local **Downloads** no computador do laboratório.

    >**Observação**: você usará essa imagem no próximo exercício.

1. Armazenaremos os arquivos de origem do Wiki em uma pasta separada dentro da estrutura de pastas atual do Repos. Em **Repos**, selecione **Arquivos**. Observe o título do repositório **eShopOnWeb** na parte superior da estrutura de pastas. **Selecione as reticências (3 pontos),** escolha **Novo/Pasta** e forneça **Documentos** como título para o nome da Nova pasta. Como um repositório não permite que você crie uma pasta vazia, forneça **READ.ME ** como Novo nome de arquivo.
1. Confirme a criação da pasta e do arquivo **pressionando o botão Criar**.
1. O arquivo READ.ME será aberto no modo de exibição interno. Como isso é armazenado 'como código', você precisa **confirmar** as alterações clicando no **botão Confirmar**. Na janela Confirmar, confirme mais uma vez pressionando **Confirmar**.
1. No menu vertical do Azure DevOps no lado esquerdo, clique em **Visão geral**, na seção **Visão geral**, selecione **Wiki**, selecione **Publicar código como wiki*.
1. No painel de **Publicar código como wiki**, especifique as seguintes configurações e clique em **Publicar**:

    | Configuração | Valor |
    | ------- | ----- |
    | Repositório | **eShopOnWeb** |
    | Branch | **main** |
    | Pasta | **/Documentos** |
    | Nome Wiki | **eShopOnWeb (Documentos)** |

    >**Observação**: isso abrirá automaticamente a seção Wiki e publicará **o editor**, onde você pode fornecer um título de página Wiki, bem como adicionar o conteúdo real. Observe que você é encorajado a usar o formato MarkDown, mas use a faixa de opções para ajudá-lo com parte da sintaxe de layout MarkDown.

1. No campo **Título** da Página Wiki, digite "Boas-vindas à nossa Loja de Varejo Online!"

1. No corpo da Página Wiki, cole o seguinte texto:

    ```text
    ##Welcome to Our Online Retail Store!
    At our online retail store, we offer a **wide range of products** to meet the **needs of our customers**. Our selection includes everything from *clothing and accessories to electronics, home decor, and more*.
    
    We pride ourselves on providing a seamless shopping experience for our customers. Our website offers the following benefits:
    1. user-friendly,
    1. and easy to navigate, 
    1. allowing you to find what you're looking for,
    1. quickly and easily. 
    
    We also offer a range of **_payment and shipping options_** to make your shopping experience as convenient as possible.
    
    ### about the team
    Our team is dedicated to providing exceptional customer service. If you have any questions or concerns, our knowledgeable and friendly support team is always available to assist you. We also offer a hassle-free return policy, so if you're not completely satisfied with your purchase, you can easily return it for a refund or exchange.
    
    ### Physical Stores
    |Location|Area|Hours|
    |--|--|--|
    | New Orleans | Home and DIY  |07.30am-09.30pm  |
    | Seattle | Gardening | 10.00am-08.30pm  |
    | New York | Furniture Specialists  | 10.00am-09.00pm |
    
    ## Our Store Qualities
    - We're committed to providing high-quality products
    - Our products are offered at affordable prices 
    - We work with reputable suppliers and manufacturers 
    - We ensure that our products meet our strict standards for quality and durability. 
    - Plus, we regularly offer sales and discounts to help you save even more.
    
    #Summary
    Thank you for choosing our online retail store for your shopping needs. We look forward to serving you!
    ```

1. Este texto de exemplo fornece uma visão geral de vários dos recursos comuns de sintaxe MarkDown, que vão desde Título e legendas (##), negrito (**), itálico (*), como criar tabelas e muito mais.

1. Quando terminar, **pressione** o botão Salvar no canto superior direito.

1. **Atualize** seu navegador ou selecione qualquer outra opção do portal DevOps e retorne à seção Wiki. Observe que agora você é apresentado com o Wiki **EshopOnWeb (Documentos)**, bem como ter o **Bem-vindo à nossa Loja de Varejo Online** como **HomePage** do Wiki.

#### Tarefa 2: Gerenciar o conteúdo de um wiki publicado

Nesta tarefa, você gerenciará o conteúdo do wiki que você publicou na tarefa anterior.

1. No menu vertical à esquerda, clique em **Repos** e verifique se o menu suspenso na seção superior do painel **Arquivos** exibe o repositório **eShopOnWeb** e o **main** branch. Na hierarquia de pastas do repositório, selecione a pasta **Documentos** e escolha o arquivo **Welcome-to-our-Online-Retail-Store!.md**.
1. Observe como o formato MarkDown é visível aqui como formato de texto bruto, permitindo que você continue editando o conteúdo do arquivo a partir daqui também.

> **Observação**: uma vez que os arquivos fonte Wiki são tratados como código-fonte, lembre-se de todas as práticas de controle do código-fonte tradicional (Clone, Pull Requests, Aprovações e muito mais), agora também podem ser aplicadas às páginas Wiki.

### Exercício 2: crie e gerencie um projeto wiki

Neste exercício, você passará pela criação e gerenciamento de um wiki do projeto.

> **Observação**: você pode criar e gerenciar wiki independentemente dos repositórios existentes.

#### Tarefa 1: Criar um wiki de projeto incluindo um diagrama de Mermaid e uma imagem

Nesta tarefa, você criará um wiki do projeto e adicionará a ele um diagrama de Mermaid e uma imagem.

1. No computador de laboratório, com o portal do Azure DevOps exibindo o **painel Wiki** do projeto **eShopOnWeb** e o conteúdo do wiki **eShopOnWeb (Documentos)** selecionado, na parte superior do painel, clique no cabeçalho da lista suspensa **eShopOnWeb (Documentos)** (o ícone de seta para baixo) e, na lista suspensa, selecione **Criar wiki do projeto**.
1. Na caixa de texto **Título da página**, digite **Design do projeto**.
1. Coloque o cursor no corpo da página, clique no ícone mais à esquerda na barra de ferramentas que representa a configuração de cabeçalho e, na lista suspensa, clique em **Cabeçalho 1**. Isso adicionará automaticamente o caractere de hash (**#**) no início da linha.
1. Logo após o caractere **#** recém-adicionado, digite **Autenticação e Autorização** e pressione a tecla **Enter**.
1. Clique no ícone mais à esquerda na barra de ferramentas que representa a configuração de cabeçalho e, na lista suspensa, clique em **Cabeçalho 2**. Isso adicionará automaticamente o caractere de hash (**##**) no início da linha.
1. Logo após o caractere **##** recém-adicionado, digite **Fluxo de Autorização do Azure DevOps Oauth 2.0** e pressione a tecla **Enter**.
1. **Copie e cole** o código a seguir para inserir um diagrama de sereia em seu wiki.

    ```text
    ::: mermaid
    sequenceDiagram
     participant U as User
     participant A as Your app
     participant D as Azure DevOps
     U->>A: Use your app
     A->>D: Request authorization for user
     D-->>U: Request authorization
     U->>D: Grant authorization
     D-->>A: Send authorization code
     A->>D: Get access token
     D-->>A: Send access token
     A->>D: Call REST API with access token
     D-->>A: Respond to REST API
     A-->>U: Relay REST API response
    :::
    ```

    >**Observação**: para obter detalhes sobre a sintaxe da Mermaid, consulte [Sobre a Mermaid ](https://mermaid-js.github.io/mermaid/#/)

1. À direita do painel do editor, no painel de visualização, clique em **Carregar diagrama** e revise o resultado.

    >**Observação**: a saída deve ser semelhante ao fluxograma que ilustra como [autorizar o acesso a APIs REST com OAuth 2.0](https://docs.microsoft.com/azure/devops/integrate/get-started/authentication/oauth)

1. No canto superior direito do painel do editor, clique no cursor virado para baixo ao lado do botão **Salvar** e, no menu suspenso, clique em **Salvar com mensagem de revisão**.
1. Na caixa de diálogo **Salvar página**, digite **a seção Autenticação e autorização com o diagrama Mermaid do OAuth 2.0** e clique em **Salvar**.
1. No painel **Editor de Design do Projeto**, coloque o cursor no final do elemento Mermaid que você adicionou anteriormente nesta tarefa, pressione a **tecla Enter** para adicionar uma linha extra, clique no ícone mais à esquerda na barra de ferramentas que representa a configuração do cabeçalho e, na lista suspensa, clique em **Cabeçalho 2**. Isso adicionará automaticamente o caractere de hash duplo (**##**) no início da linha.
1. Logo após o caractere **##** recém-adicionado, digite **Interface do usuário** e pressione a tecla **Enter**.
1. No painel **Editor de Design do Projeto**, na barra de ferramentas, clique no ícone de clipe de papel que representa a ação **Inserir um arquivo**, na caixa de diálogo **Abrir**, navegue até a pasta **Downloads**, selecione o arquivo **Brand.png** que você baixou no exercício anterior e clique em **Abrir**.
1. De volta ao painel **Editor de Design do Projeto**, revise o painel de visualização e verifique se a imagem foi exibida corretamente.
1. No canto superior direito do painel do editor, clique no cursor virado para baixo ao lado do botão **Salvar** e, no menu suspenso, clique em **Salvar com mensagem de revisão**.
1. Na caixa de diálogo **Salvar página**, digite **seção Interface do usuário com a imagem do Tailwind Traders** e clique em **Salvar**.
1. De volta ao painel do editor, no canto superior direito, clique em **Fechar**.

#### Tarefa 2: Gerenciar um projeto wiki

Nesta tarefa, você gerenciará o wiki do projeto recém-criado.

>**Observação**: você começará revertendo a alteração mais recente para a página wiki.

1. No computador de laboratório, com o portal do Azure DevOps exibindo o **painel Wiki** do projeto **eShopOnWeb** e o conteúdo do wiki **Design do Projeto** selecionado, no canto superior direito, clique no símbolo de reticências verticais e, no menu suspenso, clique em **Exibir revisões**.
1. No painel **Revisões**, clique na entrada que representa a alteração mais recente.
1. No painel resultante, examine a comparação entre a versão anterior e a atual do documento, clique em **Reverter**, quando for solicitada a confirmação, clique em **Reverter** novamente e clique em **Procurar Página**.
1. De volta ao painel **Design do Projeto**, verifique se a alteração foi revertida com êxito.

    >**Observação**: agora você adicionará outra página ao wiki do projeto e defini-lo como a página inicial do wiki.

1. No painel **Design do Projeto**, no canto inferior esquerdo, clique em **+ Nova página**.
1. No painel do editor de página, na caixa de texto **Título da página**, digite **Visão Geral do Design do Projeto**, clique em **Salvar** e clique em **Fechar**.
1. De volta ao painel que lista as páginas dentro do wiki do projeto **Design do Projeto**, localize a entrada **Visão Geral do Design do Projeto**, selecione-a com o ponteiro do mouse, arraste-a e solte-a acima da entrada da página **Design do Projeto**.
1. Confirme as alterações pressionando o **botão Mover** na janela exibida.
1. Verifique se a entrada **Visão geral do design do projeto** está listada como a página de nível superior com o ícone inicial designando-a como a página inicial do wiki.

## Revisão

Neste laboratório, você criou e configurou um wiki no Azure DevOps, incluindo o gerenciamento de conteúdo markdown e a criação de um diagrama do Mermaid.
