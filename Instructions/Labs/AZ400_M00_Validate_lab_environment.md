---
lab:
  title: Validar o ambiente de laboratório
  module: 'Module 0: Welcome'
---

# Validar o ambiente de laboratório

## Manual de laboratório do aluno

## Instruções para criar uma Organização do Azure DevOps (você só precisa fazer isso uma vez)

### Comece aqui caso você não tenha uma assinatura do Azure:
1. Obtenha um novo **código promocional** do Azure Pass do instrutor ou de outra fonte.
1. Use uma sessão privada do navegador para obter uma nova **conta pessoal da Microsoft (MSA)** em [https://account.microsoft.com](https://account.microsoft.com).
1. Usando a mesma sessão do navegador, vá para [https://www.microsoftazurepass.com](https://www.microsoftazurepass.com) resgatar seu Azure Pass usando sua conta Microsoft (MSA). Para obter detalhes, consulte [Resgatar um Microsoft Azure Pass](https://www.microsoftazurepass.com/Home/HowTo?Length=5). Siga as instruções para o resgate.

### Comece aqui caso você tenha uma assinatura do Azure:

1. Abra um navegador e navegue até [https://portal.azure.com](https://portal.azure.com), em seguida, pesquise o Azure DevOps** na parte superior da tela** do portal do Azure. Na página resultante, clique em **Organizações do Azure DevOps**.
1. Em seguida, clique no link rotulado Minhas Organizações ** Azure DevOps** ou navegue diretamente para [https://aex.dev.azure.com](https://aex.dev.azure.com).
1. Na página **Precisamos de mais alguns detalhes** selecione **Continuar**.
1. Na caixa suspensa à esquerda, escolha **Diretório Padrão** , em vez de "Conta Microsoft".
1. Se solicitado (*"Precisamos de mais alguns detalhes"),* forneça seu nome, endereço de email e local e clique em **Continuar**.
1. De volta com [https://aex.dev.azure.com](https://aex.dev.azure.com) **Diretório** padrão selecionado, clique no botão **azul Criar nova organização.**
1. Aceite os *Termos de Serviço* clicando em **Continuar**.
1. Se solicitado ("Quase concluído"),* deixe o nome da organização do Azure DevOps como padrão (* ele precisa ser um nome globalmente exclusivo) e escolha um local de hospedagem próximo a você na lista.
1. Quando a organização recém-criada for aberta no **Azure DevOps**, clique em **Configurações da organização** no canto inferior esquerdo.
1. Na tela **Configurações da organização** clique em **Faturamento** (abrir esta tela leva alguns segundos).
1. Clique em **Configurar cobrança** e, no lado direito da tela, selecione a **assinatura do Azure Pass - Patrocínio** e clique em Salvar **para** vincular a assinatura à organização.
1. Depois que a tela mostrar a ID de Assinatura do Azure vinculada na parte superior, altere o número de **trabalhos paralelos pagos** para o **MS Hosted CI/CD** de 0 para **1**. Em seguida, clique no botão **SALVAR** na parte inferior.
1. Em **configurações da Organização**, vá para a seção **Pipelines** e clique em **Configurações**.
1. Alterne a chave para **Off** para **Desativar a criação de pipelines de build clássicos** and **Desativar a criação dos pipelines de build clássicos**
    > Observação: a opção **Desativar a criação de pipelines de versão clássica** para **Ativada** h oculta as opções de criação de pipeline de versão clássica, como o menu **Liberar** na seção **Pipeline** de projetos DevOps.
1. Em **Configurações da Organização**, vá para a seção **Security** e clique em **Políticas**.
1. Altere a opção para **Ativado** para **acesso a aplicativos de terceiros via OAuth**
    > Observação: a configuração OAuth ajuda a habilitar ferramentas como o DemoDevOpsGenerator para registrar extensões. Sem isso, vários laboratórios podem falhar devido à falta das extensões necessárias.
1. Alterne a opção para **Ativado** para **Permitir projetos públicos**
    > Observação: as extensões usadas em alguns laboratórios podem exigir um projeto público para permitir o uso da versão gratuita.
1. **Aguarde pelo menos 3 horas antes de usar os recursos** de CI/CD para que as novas configurações sejam refletidas no back-end. Caso contrário, você ainda verá a mensagem *"Nenhum paralelismo hospedado foi comprado ou concedido".*

## Instruções para criar um Projeto do Azure DevOps (você só precisa fazer isso uma vez)

### Exercício 0: configurar os pré-requisitos do laboratório

> **Observação**: certifique-se de concluir as etapas para criar sua Organização do Azure DevOps antes de continuar com essas etapas.

Neste exercício, você configurará os pré-requisitos para o laboratório, que consistem em um novo projeto do Azure DevOps com um repositório baseado no [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb).

#### Tarefa 1: criar e configurar o projeto de equipe

Nesta tarefa, você criará um **projeto do Azure DevOps eShopOnWeb** para ser usado por vários laboratórios.

1. No computador do laboratório, em uma janela do navegador, abra sua organização do Azure DevOps. Clique em **Novo Projeto**. Dê ao projeto as seguintes configurações:
    - nome: **eShopOnWeb**
    - visibilidade: **Privada**
    - Avançado: controle de versão: **Git**
    - Avançado: processo de Item de Trabalho: **Scrum**

2. Clique em **Criar**.

    ![Criar Projeto](images/create-project.png)

#### Tarefa 2: importar repositório Git do eShopOnWeb

Nesta tarefa você importará o repositório Git do eShopOnWeb para ser usado por vários laboratórios.

1. No computador do laboratório, em uma janela do navegador, abra sua organização do Azure DevOps **eShopOnWeb**. Clique no repositório **Repos>Files** , **Importe um**. Selecione **Importar**. Na janela **Importar um repositório Git**, cole a seguinte URL https://github.com/MicrosoftLearning/eShopOnWeb.git e clique em **Importar**:

    ![Importar repositório](images/import-repo.png)

2. O repositório está organizado da seguinte forma:
    - A pasta **.ado** contém os pipelines YAML do Azure DevOps.
    - O contêiner da pasta **.devcontainer** está configurado para o desenvolvimento usando contêineres (localmente no VS Code ou no GitHub Codespaces).
    - A pasta **.azure** contém a infraestrutura Bicep&ARM como modelos de código usados em alguns cenários do laboratório.
    - **A pasta .github** definições de fluxo de trabalho YAML GitHub do contêiner da pasta.
    - A pasta **src** contém o site .NET 6 usado nos cenários do laboratório.

Agora você concluiu as etapas de pré-requisito necessárias para continuar com os diferentes laboratórios individuais para este curso AZ-400.
