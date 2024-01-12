---
lab:
  title: Configuring Agent Pools and Understanding Pipeline Styles
  module: 'Module 03: Implement CI with Azure Pipelines and GitHub Actions'
---

# Configuring Agent Pools and Understanding Pipeline Styles

# Manual de laboratório do aluno

## Requisitos do laboratório

- Este laboratório requer o **Microsoft Edge** ou um [navegador compatível com o Azure DevOps.](https://docs.microsoft.com/en-us/azure/devops/server/compatibility?view=azure-devops#web-portal-supported-browsers)

- **Configurar uma organização do Azure DevOps:** se você ainda não tiver uma organização Azure DevOps que possa usar para este laboratório, crie uma seguindo as instruções disponíveis em [Criar uma organização ou coleção de projetos](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/create-organization?view=azure-devops).

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

## Tempo estimado: 45 minutos

## Instruções

### Exercício 0: configurar os pré-requisitos do laboratório

Neste exercício, você configurará o pré-requisito para o laboratório, que consiste no projeto de equipe pré-configurado do Parts Unlimited com base em um modelo do Gerador de Demonstração do Azure DevOps.

#### Tarefa 1: Configurar o projeto de equipe

Nesta tarefa, você usará o Gerador de Demonstração de Azure DevOps para gerar um novo projeto com base no modelo **PartsUnlimited**.

1. No computador do laboratório, inicie um navegador da Web e navegue até [Gerador de Demonstração do Azure DevOps](https://azuredevopsdemogenerator.azurewebsites.net). Este site de utilitário automatizará o processo de criação de um novo projeto do Azure DevOps em sua conta que é pré-preenchido com conteúdo (itens de trabalho, repositórios etc.) necessário para o laboratório.

    > **Observação**: para obter mais informações, consulte o site <https://docs.microsoft.com/en-us/azure/devops/demo-gen>.

1. Clique em **Entrar** e entre usando sua conta Microsoft associada a uma assinatura do Azure DevOps.
1. Se necessário, na página **Gerador de Demonstração do Azure DevOps**, clique em **Aceitar** para aceitar as solicitações de permissão para acessar sua assinatura do Azure DevOps.
1. Na página **Criar Novo Projeto**, na caixa de texto **Novo Nome do Projeto** do Projeto, digite **Configuring Agent Pools and Understanding Pipeline Styles**, na lista suspensa **Selecionar organização**, selecione sua organização do Azure DevOps e clique em **Escolher modelo**.
1. Na página **Escolher um modelo**, clique no modelo **PartsUnlimited** e, em seguida, clique em **Selecionar modelo**.
1. Clique em **Criar projeto**.

    > **Observação**: aguarde o processo ser concluído. Isso deverá levar cerca de dois minutos. Caso o processo falhe, navegue até sua organização de DevOps, exclua o projeto e tente novamente.

1. **Na página Criar Novo Projeto**, clique em **Navegar para o projeto**.

### Exercício 1: criar Azure Pipelines baseados em YAML

Neste exercício, você converterá um Pipeline clássico do Azure em um baseado em YAML.

#### Tarefa 1: Criar um pipeline YAML do Azure DevOps

Nesta tarefa, você criará um pipeline YAML do Azure DevOps baseado em modelo.

1. No navegador da Web que exibe o portal de Azure DevOps com o projeto **Configuring Agent Pools and Understanding Pipeline Styles** aberto, no painel de navegação vertical no lado esquerdo, clique em **Pipelines**.
1. Na guia **Recente** do painel **Pipelines**, clique em **Novo pipeline**.
1. Na página **Onde está seu código?**, selecione **Azure Repos Git**.
1. No painel **Selecionar um repositório**, clique em **PartsUnlimited**.
1. No painel **Revisar seu pipeline YAML**, revise o pipeline de exemplo, clique no símbolo de acento circunflexo para baixo ao lado do **botão Executar**, clique em **Salvar**.

### Exercício 2: Gerenciar pools de agentes Azure DevOps

Neste exercício, você implementará o agente de Azure DevOps auto-hospedado.

#### Tarefa 1: Configurar um agente auto-hospedado do Azure DevOps.

Nesta tarefa, você configurará a VM LOD como um agente de auto-hospedagem do Azure DevOps e a usará para executar um pipeline de build.

1. Na máquina Virtual de Laboratório (VM de laboratório) ou em seu próprio computador, inicie um navegador da Web, navegue até o [portal de Azure DevOps](https://dev.azure.com) e entre usando a conta Microsoft associada à sua organização de Azure DevOps.
1. No portal Azure DevOps, no canto superior direito da página Azure DevOps, clique no ícone **Configurações do usuário**; dependendo de você ter ou não os recursos de visualização ativados, você deverá ver um item de **Segurança** ou de **Tokens de acesso pessoal** no menu; se você vir **Segurança**, clique nele e selecione **Tokens de acesso pessoal**. No painel **Tokens de Acesso Pessoal**, clique em **+ Novo Token**.
1. No painel **Criar novo token de acesso pessoal**, clique no link **Mostrar todos os escopos**, especifique as seguintes configurações e clique em **Criar** (deixe todos os outros com seus valores padrão):

    | Configuração | Valor |
    | --- | --- |
    | Nome | Laboratório **Configuring Agent Pools and Understanding Pipeline Styles** |
    | Escopo (personalizado) | **Agent Pools** (mostrar mais escopos abaixo, se necessário)|
    | Permissões | **Ler e gerenciar** |

1. No painel **Êxito**, copie o valor do token de acesso pessoal para a Área de Transferência.

    > **Observação**: Certifique-se de copiar o token. Você não poderá recuperá-lo depois de fechar este painel.

1. No painel **Êxito**, clique em **Fechar**.
1. No painel **Token de Acesso Pessoal** do portal Azure DevOps, clique no símbolo de **Azure DevOps** no canto superior esquerdo e, em seguida, clique em **Configurações da organização** no canto inferior esquerdo.
1. No lado esquerdo do painel **Visão geral**, no menu vertical, na seção **Pipelines**, clique em **Agent Pools**.
1. No painel **Agent Pools**, no canto superior direito, clique em **Adicionar pool**.
1. No painel **Adicionar pool de agentes**, na lista suspensa **Tipo de pool**, selecione **Auto-hospedado**, na caixa de texto **Nome**, digite **az400m05l05a-pool** e clique em **Criar**.
1. De volta ao painel **Agent Pools**, clique na entrada que representa o **az400m05l05a-pool** recém-criado.
1. Na guia **Trabalhos** do painel **az400m05l05a-pool**, clique no botão **Novo agente**.
1. No painel **Obter o agente**, verifique se as guias **Windows** e x64** estão selecionadas e **clique em **Download** para baixar o arquivo zip que contém os binários do agente para baixá-lo na pasta **Downloads** local dentro do seu perfil de usuário.

    > **Observação**: se você receber uma mensagem de erro neste momento indicando que as configurações atuais do sistema impedem que você baixe o arquivo, na janela do Internet Explorer, no canto superior direito, clique no símbolo da roda dentada que designa o cabeçalho do menu **Configurações**; no menu suspenso, selecione **Opções da Internet**; na caixa de diálogo **Opções da Internet**, clique em **Avançado**; na guia **Avançado**, clique em **Redefinir**; na caixa de diálogo **Redefinir Configurações do Internet Explorer**, clique em **Redefinir** novamente; clique em **Fechar** e tente baixar novamente.

1. Inicie o Windows PowerShell como administrador e, o console **Administrador: console do Windows PowerShell**, execute as seguintes linhas para criar o diretório **C:\\agent** e extrair o conteúdo do arquivo morto baixado nele.

    ```powershell
    cd \
    mkdir agent ; cd agent
    $TARGET = Get-ChildItem "$Home\Downloads\vsts-agent-win-x64-*.zip"
    Add-Type -AssemblyName System.IO.Compression.FileSystem
    [System.IO.Compression.ZipFile]::ExtractToDirectory($TARGET, "$PWD")
    ```

1. No mesmo console do **Administrador: Windows PowerShell**, execute o seguinte para configurar o agente:

    ```powershell
    .\config.cmd
    ```

1. Quando solicitado, especifique os valores das seguintes configurações:

    | Configuração | Valor |
    | ------- | ----- |
    | Insira o servidor de URL | a URL da sua organização de Azure DevOps, no formato **<https://dev.azure.com/>`<organization_name>`**, onde `<organization_name>` representa o nome da sua organização de Azure DevOps |
    | Digite o tipo de autenticação (pressione Enter para PAT) | **Enter** |
    | Insira seu token de acesso pessoal | O token de acesso que você registrou anteriormente nesta tarefa |
    | Insira o pool de agentes (pressione Enter para padrão) | **az400m05l05a-pool** |
    | Insira o nome do agente | **az400m05-vm0** |
    | Entre na pasta de trabalho (pressione Enter para _work) | **Enter** |
    | **(Somente se mostrado)** Digite Executar um descompactamento para tarefas para cada etapa. (pressione Enter para N) | **AVISO**: pressione Enter** somente **se a mensagem for mostrada|
    | Inserir agente de execução como serviço? (S/N) (pressione Enter para N) | **Y** |
    | digite habilitar SERVICE_SID_TYPE_UNRESTRICTED (S/N) (pressione Enter para N) | **Y** |
    | Digite a conta de usuário a ser usada para o serviço (pressione Enter para NT AUTHORITY\NETWORK SERVICE) | **Enter** |
    | Digite se deseja impedir que o serviço seja iniciado imediatamente após a conclusão da configuração? (S/N) (pressione Enter para N) | **Enter** |

    > **Note**: você pode executar seu agente auto-hospedado como um serviço ou um processo interativo. Talvez você queira começar com o modo interativo, pois isso simplifica a verificação da funcionalidade do agente. Para uso em produção, você deve considerar a execução do agente como um serviço ou como um processo interativo com logon automático habilitado, já que ambos persistem seu estado de execução e garantem que o agente seja iniciado automaticamente se o sistema operacional for reiniciado.

1. Alterne para a janela do navegador que exibe o portal de Azure DevOps e feche o painel **Obter o agente**.
1. De volta à guia **Agentes** do painel **az400m05l05a-pool**, observe que o agente recém-configurado está listado com o status **Online**.
1. Na janela do navegador da Web que exibe o portal de Azure DevOps, no canto superior esquerdo, clique no rótulo de **Azure DevOps**.
1. Na janela do navegador que exibe a lista de projetos, clique no bloco que representa o projeto **Configuring Agent Pools and Understanding Pipeline Styles**.
1. No painel **Configuring Agent Pools and Understanding Pipeline Styles**, no painel de navegação vertical do lado esquerdo, na seção **Pipelines**, clique em **Pipelines**.
1. Na guia **Recente** do painel **Pipelines**, selecione **PartsUnlimited** e, no painel **PartsUnlimited**, selecione **Editar**.
1. No painel de edição **PartsUnlimited**, no pipeline existente baseado em YAML, substitua a linha `vmImage: windows-2019` que designa o pool de agentes de destino pelo conteúdo seguinte, designando o pool de agentes auto-hospedado recém-criado:

    ```yaml
    name: az400m05l05a-pool
    demands:
    - agent.name -equals az400m05-vm0
    ```
    > **AVISO**: Tenha cuidado com copiar/colar, certifique-se de ter o mesmo recuo mostrado acima.


1. Para `Task: NugetToolInstaller@0`, clique em **Configurações (link que está sendo exibido acima da tarefa na cor cinza)**, modifique **a versão do NuGet.exe para instalar o** > **4.0.0** e clique em **Adicionar.**
1.  No painel de edição **PartsUnlimited**, no canto superior direito do painel, clique em **Salvar** e, no painel **Salvar**, clique em **Salvar** novamente. Isso vai disparar automaticamente o pipeline de build.
1.  No portal do Azure DevOps, no painel de navegação vertical no lado esquerdo, na seção **Pipelines**, clique em **Pipelines**.
1.  Na guia **Recente** do painel **Pipelines**, clique na entrada **PartsUnlimited**; na guia **Execuções** do painel **PartsUnlimited**, selecione a execução mais recente; no painel **Resumo** da execução, role para baixo até a parte inferior; na seção **Trabalhos**, clique em **Fase 1** e monitore o trabalho até sua conclusão bem-sucedida.



## Revisão

Neste laboratório, você aprendeu a implementar e usar agentes auto-hospedados com pipelines YAML.
