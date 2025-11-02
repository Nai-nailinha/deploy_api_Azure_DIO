## ☁️ Como fazer deploy de uma API no Azure App Service

Este passo a passo funciona para .NET, Node e Python.
Requisitos: Azure CLI instalada e uma conta no Azure.

### **1️⃣ Login e Resource Group**
```
az login
az group create -n rg-api-prod -l brazilsouth
```
### **2️⃣ Publicar direto do código (az webapp up)**

Rode os comandos **dentro da pasta do projeto**.

### 💠 .NET 8 (ASP.NET Core Web API)
```
az webapp up -n minha-api-app \
  -g rg-api-prod -l brazilsouth \
  --sku B1 --runtime "DOTNET|8.0"
```
### 🟩Node 20 (Express/Nest)

**garanta no código:** 

*app.listen(process.env.PORT || 8080)*
```
az webapp up -n minha-api-app \
  -g rg-api-prod -l brazilsouth \
  --sku B1 --runtime "NODE|20-lts"
```

### 🐍Python 3.10 (FastAPI/Flask)

**tenha requirements.txt; o App Service descobre o start via Oryx**
```
az webapp up -n minha-api-app \
  -g rg-api-prod -l brazilsouth \
  --sku B1 --runtime "PYTHON|3.10"
```

Ao final, a CLI mostra a URL pública:
```
https://minha-api-app.azurewebsites.net
```

### **3️⃣ Variáveis de ambiente (App Settings)**
```
az webapp config appsettings set -g rg-api-prod -n minha-api-app \
  --settings ASPNETCORE_ENVIRONMENT=Production CONNECTION_STRING="..."
```
### **4️⃣ CORS (se houver front-end separado)**

No portal do Azure → App Service → CORS → adicione:
```
https://meu-front.com
```
### **5️⃣ Logs e diagnóstico**

* Log stream (tempo real): Portal → App Service → Log stream
* Application Insights (opcional, recomendado): habilite para métricas e traces

### **6️⃣ Testar**
```
curl -i https://minha-api-app.azurewebsites.net/health
```
Crie um endpoint &/health* simples na API para checar uptime.

### **🔁 (Opcional) CI/CD com GitHub Actions**

1- No Portal → App Service → Get publish profile (baixe o XML).

2- No GitHub → Settings → Secrets and variables → Actions → New repository secret

* Name: AZURE_WEBAPP_PUBLISH_PROFILE
* Value: cole o XML.
  
3- Crie o arquivo **.github/workflows/deploy.yml**:
```
name: Deploy API to Azure App Service
on:
  push:
    branches: [ "main" ]
  workflow_dispatch:

jobs:
  build-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # .NET 8 — troque pela stack da sua API se for Node/Python
      - uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '8.0.x'
      - run: dotnet publish -c Release -o publish

      - uses: azure/webapps-deploy@v2
        with:
          app-name: minha-api-app            # mesmo nome do App Service
          publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
          package: ./publish
```

Para **Node**, troque a etapa de build por *actions/setup-node* e seu comando de build;
Para **Python**, gere um pacote (*zip*) com o conteúdo necessário e aponte em *package:*.

### 🧹 Limpeza (se precisar remover tudo)
```
az group delete -n rg-api-prod -y
```

### ❗ Troubleshooting rápido

* **Erro de porta (Node/Python)**: garanta que o servidor use *process.env.PORT*.
* **404 após deploy**: confira o path da API (ex.: */api* ou */health*) e o startup.
* **CORS bloqueando**: adicione o domínio do front nas configurações de CORS do App Service.
* **Logs não aparecem**: habilite Log stream no portal e tente novamente.


☕💻 Feito com café, código e neurônios por [Enaile Lopes](https://www.linkedin.com/in/enailelopes/)  
💜 #TIcomCaféENeurônios | 🌐 [ticomcafeeneuronios.com.br](https://ticomcafeeneuronios.com.br)
