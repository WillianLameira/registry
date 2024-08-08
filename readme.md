Aqui está um exemplo de arquivo `README.md` para documentar o processo de criação do Docker Registry com autenticação usando Docker Compose:

```markdown
# Docker Registry com Autenticação

Este projeto configura um Docker Registry local com autenticação básica e HTTPS usando Docker Compose.

## Requisitos

- Docker
- Docker Compose
- OpenSSL (para gerar certificados)
- Apache HTTP Server (`httpd`) para gerar arquivos de autenticação

## Passo a Passo

### 1. Preparar o Ambiente

1. **Crie um diretório para o projeto:**

   ```bash
   mkdir registry
   cd registry
   ```

2. **Crie os diretórios para certificados e autenticação:**

   ```bash
   mkdir -p certs auth
   ```

### 2. Gerar Certificados SSL

1. **Gere um certificado autoassinado usando OpenSSL:**

   ```bash
   openssl req -newkey rsa:4096 -nodes -sha256 -keyout certs/cert-docker.key -x509 -days 365 -out certs/cert-docker.crt -subj /CN=myregistry.com
   ```

2. **Adicione a entrada no arquivo `/etc/hosts` para `myregistry.com`:**

   ```plaintext
   127.0.0.1 myregistry.com
   ```

3. **Copie o certificado para o diretório do Docker:**

   ```bash
   sudo mkdir -p /etc/docker/certs.d/myregistry.com:5001
   sudo cp certs/cert-docker.crt /etc/docker/certs.d/myregistry.com:5001/
   ```

### 3. Gerar Arquivo de Autenticação

1. **Use a imagem do Apache HTTP Server para gerar o arquivo `htpasswd`:**

   ```bash
   docker run --rm httpd:2.4-alpine htpasswd -Bbn test password > auth/htpasswd
   ```

### 4. Configurar o Docker Compose

1. **Crie um arquivo `docker-compose.yml` com o seguinte conteúdo:**

   ```yaml
   version: '3.8'

   services:
     registry:
       image: registry:2
       container_name: private-registry
       ports:
         - "5001:5000"  # Mapeia a porta 5001 para a porta 5000 do contêiner
       environment:
         REGISTRY_HTTP_TLS_CERTIFICATE: /certs/cert-docker.crt
         REGISTRY_HTTP_TLS_KEY: /certs/cert-docker.key
         REGISTRY_AUTH: htpasswd
         REGISTRY_AUTH_HTPASSWD_REALM: "Registry Realm"
         REGISTRY_AUTH_HTPASSWD_PATH: /auth/htpasswd
       volumes:
         - ./certs:/certs
         - ./auth:/auth
         - registry-data:/var/lib/registry

   volumes:
     registry-data:
   ```

2. **Inicie o serviço Docker Registry:**

   ```bash
   docker-compose up -d
   ```

### 5. Testar o Docker Registry

1. **Verifique se o Docker Registry está funcionando localmente:**

   ```bash
   curl -k https://localhost:5001/v2/_catalog
   ```

2. **Tagueie e faça o push de uma imagem para o registry:**

   ```bash
   docker pull busybox
   docker tag busybox myregistry.com:5001/my-busybox
   docker login myregistry.com:5001
   docker push myregistry.com:5001/my-busybox
   ```

3. **Teste o pull da imagem:**

   ```bash
   docker rmi busybox
   docker rmi myregistry.com:5001/my-busybox
   docker pull myregistry.com:5001/my-busybox
   ```

### 6. Parar e Remover o Serviço

Para parar e remover o contêiner do Docker Registry:

```bash
docker-compose down
```

## Problemas Conhecidos

- **Certificado SSL Autoassinado:** O navegador pode mostrar um aviso de certificado não confiável. Adicione uma exceção para o certificado no navegador ou importe o certificado no sistema.

- **Porta em Uso:** Se a porta `5001` estiver em uso, altere o mapeamento da porta no arquivo `docker-compose.yml` para uma porta diferente.

## Contribuições

Sinta-se à vontade para contribuir com melhorias ou correções. Envie um pull request ou abra um issue para discutir mudanças.

## Licença

Este projeto está licenciado sob a [MIT License](LICENSE).
```

Este `README.md` fornece um guia detalhado para criar, configurar e testar um Docker Registry com autenticação. Ajuste conforme necessário para se adequar às suas necessidades e à configuração específica do seu projeto.