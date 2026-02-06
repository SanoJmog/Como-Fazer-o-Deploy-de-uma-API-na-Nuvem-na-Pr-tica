# Como-Fazer-o-Deploy-de-uma-API-na-Nuvem-na-Pr-tica
Um passo a passo resumido pelo colega GPT de como fazer esse Deploy!

1️⃣ Pré-requisitos (antes de tudo)

Antes de clicar em qualquer coisa, você precisa ter:

🔹 Conta no Azure

Uma Subscription ativa

Acesso ao Azure Portal

🔹 Projeto no Azure DevOps

Organização criada

Um Project (ex: MinhaAPI)

🔹 Código da API versionado

Repositório Git (Azure Repos ou GitHub)

API já funcionando localmente (dotnet run, por exemplo)

2️⃣ Criar a infraestrutura no Azure (lado “cloud”)

Aqui você prepara onde a API vai rodar.

🧱 2.1 Criar um App Service

No Azure Portal:

Create a resource

Web App

Configurações principais:

Publish: Code

Runtime stack: .NET 8 (ou a versão da sua API)

Operating System: Linux (recomendado)

Region: Brazil South (ou a mais próxima)

Criar ou usar um App Service Plan

👉 Esse App Service será o “servidor” da sua API.

🔐 2.2 (Opcional mas recomendado) Variáveis de ambiente

No App Service:

Settings → Configuration

Adicione:

ConnectionStrings

Secrets

URLs externas

Isso evita deixar segredo no código 💀

3️⃣ Preparar o projeto para deploy

No código da API:

🔹 Conferir:

Program.cs usando builder.Services

Porta dinâmica (no Linux, o Azure define a porta)

Nada hardcoded (ex: localhost:5000)

🔹 Testar build local:
dotnet build
dotnet publish -c Release


Se isso falhar local, vai falhar no pipeline também.

4️⃣ Criar Service Connection no Azure DevOps

Isso é o “Azure DevOps pedindo permissão pro Azure”.

Azure DevOps → Project Settings

Service connections

New service connection

Tipo: Azure Resource Manager

Authentication: Automatic

Escolhe:

Subscription

Resource Group

Nomeia algo tipo:

Azure-Connection-AppService


✅ Sem isso, o pipeline não consegue fazer deploy.

5️⃣ Criar o Pipeline (CI/CD)

Agora vem a mágica ✨

🧪 5.1 Criar pipeline

Pipelines → New pipeline

Escolha onde está o código (Azure Repos / GitHub)

Escolha:

.NET Core


O Azure já sugere um YAML base.

⚙️ 5.2 Pipeline YAML (exemplo completo)

Esse pipeline:

Compila

Publica

Faz deploy no App Service

trigger:
- main

variables:
  buildConfiguration: 'Release'

stages:
- stage: Build
  displayName: Build API
  jobs:
  - job: Build
    pool:
      vmImage: 'ubuntu-latest'
    steps:
    - task: UseDotNet@2
      inputs:
        packageType: 'sdk'
        version: '8.x'

    - script: dotnet restore
      displayName: Restore

    - script: dotnet build --configuration $(buildConfiguration)
      displayName: Build

    - script: dotnet publish -c $(buildConfiguration) -o $(Build.ArtifactStagingDirectory)
      displayName: Publish

    - task: PublishBuildArtifacts@1
      inputs:
        pathToPublish: '$(Build.ArtifactStagingDirectory)'
        artifactName: 'drop'

- stage: Deploy
  displayName: Deploy to Azure
  dependsOn: Build
  jobs:
  - job: Deploy
    pool:
      vmImage: 'ubuntu-latest'
    steps:
    - task: DownloadBuildArtifacts@0
      inputs:
        artifactName: 'drop'

    - task: AzureWebApp@1
      inputs:
        azureSubscription: 'Azure-Connection-AppService'
        appName: 'nome-do-seu-app-service'
        package: '$(Pipeline.Workspace)/drop'

6️⃣ Testar o deploy

Após o pipeline rodar:

Vá no Azure Portal

Abra o App Service

Copie a URL pública

Teste:

https://sua-api.azurewebsites.net/swagger


Se abriu, deploy concluído com sucesso 🎉

7️⃣ Boas práticas (nível profissional)

Depois de funcionar, pense nisso:

🔐 Segurança

Secrets no Azure Key Vault

HTTPS obrigatório

Authentication (JWT / Azure AD)

🔄 Ambientes

Pipeline com:

dev

hml

prod

App Service separado por ambiente

📊 Observabilidade

Application Insights

Logs no Azure Portal

8️⃣ Visão mental do fluxo (pra fixar)
Commit no Git
   ↓
Azure DevOps Pipeline
   ↓
Build + Publish
   ↓
Deploy automático
   ↓
API rodando no Azure