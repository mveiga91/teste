Objetivo:
Automatizar o processo de compilação e deploy do código em uma VM AWS usando GitHub Actions.

1. Criar Repositório no GitHub

    Acesse GitHub e crie um novo repositório.

    Clone localmente:

    git clone https://github.com/seu-usuario/repositorio.git
    cd repositorio

2. Configurar GitHub Actions

Crie o arquivo .github/workflows/deploy.yml:
yaml

name: Deploy to AWS VM

on:
  push:
    branches: [main]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Instalar dependências
        run: npm install  # Altere para seu gerenciador (pip, maven, etc.)
      
      - name: Compilar projeto
        run: npm run build  # Altere para seu comando de build
      
      - name: Deploy na VM via SSH
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.VM_IP }}
          username: ${{ secrets.VM_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            cd /caminho/do/app
            git pull
            npm install --production
            pm2 restart app

3. Configurar Secrets no GitHub

No repositório:

    Settings > Secrets and variables > Actions

    Adicione:

        VM_IP: IP público da VM (ec2-1-2-3-4.compute-1.amazonaws.com)

        VM_USER: Usuário da VM (ubuntu)

        SSH_PRIVATE_KEY: Chave privada da VM (conteúdo do arquivo .pem da AWS).

4. Pré-requisitos na VM

Certifique-se de:

    Ter Node.js (ou o runtime da sua aplicação) instalado.

    PM2 (para gerenciamento de processos):
    bash

    npm install -g pm2

    Permissões de SSH configuradas (chave pública no ~/.ssh/authorized_keys).

5. Testar o Pipeline

    Faça um push para o branch main:

    git add .github/workflows/deploy.yml
    git commit -m "Adiciona pipeline de deploy"
    git push origin main

    Verifique a execução em Actions no GitHub.

   Resultados:
   ![image](https://github.com/user-attachments/assets/7c5b9d7c-a31f-46c2-8497-afe406c88d46)
   ![image](https://github.com/user-attachments/assets/0810ff8e-8899-4bb8-ae22-acb9d97301f2)
   ![image](https://github.com/user-attachments/assets/5f9b4517-b984-4b6f-b351-4ee7ec6bdbbf)


6 -  Objetivo
Configurar balanceamento de carga com Nginx/HAProxy para distribuir requisições entre múltiplas instâncias de aplicação de forma eficiente.
   Resultados Alcançados
   Status: IMPLEMENTADO COM SUCESSO

✅ Nginx instalado e configurado como Load Balancer
✅ 2 aplicações Node.js rodando simultaneamente (portas 3000/3001)
✅ Balanceamento Round-Robin funcionando perfeitamente
✅ Distribuição eficiente de requisições (50/50)
✅ Configuração upstream otimizada
✅ Proxy reverso implementado

Arquitetura Implementada:
Internet/Cliente
        ↓
   Nginx (porta 80)
   Load Balancer
        ↓
┌─────────────────────┐
│    upstream backend │
├─────────────────────┤
│ App1 → 127.0.0.1:3000 │
│ App2 → 127.0.0.1:3001 │
└─────────────────────┘

Configuração Técnica
Ambiente:

SO: Ubuntu (AWS EC2)
Web Server: Nginx 1.x
Runtime: Node.js 18.x
Algoritmo: Round-Robin
Portas: 80 (Load Balancer), 3000/3001 (Apps)


Configuração Nginx:
upstream backend {
    server 127.0.0.1:3000;
    server 127.0.0.1:3001;
}

server {
    listen 80;
    server_name _;
    
    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}

Aplicações Backend:

App 1: Servidor HTTP Node.js na porta 3000
App 2: Servidor HTTP Node.js na porta 3001
Execução: Background com nohup para persistência

Teste de Balanceamento (10 requisições):
![image](https://github.com/user-attachments/assets/bf1df6f0-646a-4cd4-bcee-b20a6e45f36f)

Resultados dos Testes:

✅ Alternância Perfeita: 100% das requisições alternadas
✅ Distribuição Equilibrada: 50% para cada instância
✅ Zero Downtime: Todas as requisições atendidas
✅ Latência Baixa: Respostas instantâneas
✅ Alta Disponibilidade: Sistema resiliente

Status dos processos:
![image](https://github.com/user-attachments/assets/6015b3c8-a060-4646-83db-6fb5ec82d183)


Configuração validada:
![image](https://github.com/user-attachments/assets/0e83efb6-6703-4e66-9bde-47fc541687f3)


Teste de carga:
![image](https://github.com/user-attachments/assets/26b6d815-0d0e-48e5-8a9a-6fe3bff17b5b)

