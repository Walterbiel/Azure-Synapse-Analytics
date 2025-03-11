# Azure-Synapse-Analytics

1️⃣ Estrutura do Processo
O pipeline do Azure DevOps deve:

Identificar o ambiente correto com base no branch ou variável de ambiente.
Passar esse valor como um parâmetro ao notebook do Azure Synapse.
Executar o notebook automaticamente via Azure Synapse Pipeline.
2️⃣ Criando o Pipeline no Azure DevOps
Você precisará criar um arquivo YAML no Azure DevOps, geralmente chamado azure-pipelines.yml, para definir a automação.

Pipeline YAML (azure-pipelines.yml)
Crie um pipeline no DevOps que:

Identifique o ambiente com base no branch (dev, test, prod).
Execute o pipeline do Azure Synapse passando o parâmetro environment.
yaml
Copy
Edit
trigger:
  branches:
    include:
      - main
      - dev
      - test

variables:
  - name: environment
    ${{ if eq(variables['Build.SourceBranchName'], 'main') }}:
      value: 'prod'
    ${{ if eq(variables['Build.SourceBranchName'], 'dev') }}:
      value: 'dev'
    ${{ if eq(variables['Build.SourceBranchName'], 'test') }}:
      value: 'test'

stages:
  - stage: Deploy_Synapse_Notebook
    displayName: "Executar Notebook no Azure Synapse"
    jobs:
      - job: RunNotebook
        displayName: "Executar Notebook"
        pool:
          vmImage: 'ubuntu-latest'
        steps:
          - task: AzureSynapseNotebookTask@0
            displayName: "Rodar Notebook no Synapse"
            inputs:
              azureSubscription: "<NOME_DO_SERVIÇO_CONEXÃO>"
              workspaceName: "<NOME_DO_WORKSPACE>"
              notebookName: "<NOME_DO_NOTEBOOK>"
              parameters: "{ \"environment\": \"$(environment)\" }"
3️⃣ Explicação do Código
🔹 Definição do Ambiente Automaticamente
O pipeline identifica o ambiente baseado no branch do Git.
main → Produção (prod)
dev → Desenvolvimento (dev)
test → Teste (test)
🔹 Passando Parâmetros para o Notebook
A tarefa AzureSynapseNotebookTask executa o notebook e injeta o valor do ambiente na variável environment.
🔹 Execução no Synapse
O notebook receberá environment automaticamente via dbutils.widgets.get("environment").
4️⃣ Configuração no Notebook
Dentro do seu notebook no Azure Synapse, adicione a célula Parameters:

python
Copy
Edit
from pyspark.sql import SparkSession

spark = SparkSession.builder.getOrCreate()

dbutils.widgets.text("environment", "dev", "Environment")
env = dbutils.widgets.get("environment")

print(f"Executando no ambiente: {env}")
5️⃣ Testando
Commit no branch dev → Deve rodar no ambiente dev.
Commit no branch test → Deve rodar no ambiente test.
Commit no branch main → Deve rodar no ambiente prod.

![image](https://github.com/user-attachments/assets/0a0cbcd0-a670-4054-b891-7f2d0094eee5)



