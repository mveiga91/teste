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

        VM_IP: IP público da VM (ex: ec2-1-2-3-4.compute-1.amazonaws.com)

        VM_USER: Usuário da VM (ex: ubuntu)

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

6. Balanceamento de Carga (Opcional)

Se precisar de Nginx como load balancer:

    Instale e configure o Nginx na VM:

sudo apt install nginx

Configure /etc/nginx/sites-available/app:
nginx

upstream app_servers {
    server localhost:3000;  # Instância 1
    server localhost:3001;  # Instância 2
}
server {
    listen 80;
    location / {
        proxy_pass http://app_servers;
    }
}

Reinicie o Nginx:

    sudo systemctl restart nginx

    Para problemas de SSH, verifique:
    bash

ssh -i chave.pem ubuntu@IP_DA_VM
