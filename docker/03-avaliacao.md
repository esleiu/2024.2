# Docker Compose com Django e Banco de Dados

## **Informações Gerais**

- **Assunto:** Docker, conteinerização de aplicativos
- **Disciplina:** *Sistemas Operacionais*
- **Tarefa:**
  1. Criar um projeto Django com uma aplicação web
     - Alternativa: Criar uma _branch_ no repositório do projeto integrador para configurar o acesso ao repositório de dados
  2. Criar um `Dockerfile` para o projeto Django
  3. Criar a imagem e testar o contêiner
  4. *(Opcional)* Criar um `Dockerfile` para o repositório do banco de dados
  5. Criar um `docker-compose.yml` e configurar para dois serviços: `webapp` e `db`
  6. Configurar o arquivo Django de acesso ao repositório de dados para usar o serviço Docker `db`
  7. Testar o `docker-compose.yml`
  8. Relatar minimamente o que foi feito
- **Entrega:** Cópia deste arquivo Markdown preenchido no repositório _fork_ de [sistemas-operacionais/2024.2](https://github.com/sistemas-operacionais/2024.2)

---

## **Relatório**

### **Aluno** 

- **Nome:**  wesley costa da silva
- **Matrícula:** 20222014040017

### **Relato**

Para esta atividade, foi realizada a conteinerização de um projeto Django utilizando Docker e Docker Compose. O primeiro passo foi criar um `Dockerfile` para a aplicação, definindo a imagem base do Python e copiando os arquivos necessários para o contêiner. Em seguida, foi criado um `docker-compose.yml` para orquestrar os serviços do Django e do banco de dados PostgreSQL.

Após construir as imagens com `docker compose build`, os contêineres foram iniciados com `docker compose up -d`. O banco de dados foi configurado para rodar como um serviço separado, permitindo que o Django acessasse os dados via variáveis de ambiente.

Foram testadas conexões entre os serviços e ajustadas dependências no `requirements.txt`, incluindo a instalação do `Pillow` para compatibilidade com `ImageField`. As migrações do banco foram aplicadas usando `docker compose exec webapp python manage.py migrate`, e um superusuário foi criado para acessar o painel administrativo do Django.

Após a execução dos testes, a aplicação rodou corretamente no endereço `http://localhost:8000/`, demonstrando a funcionalidade do ambiente conteinerizado.

---

## **Arquivos Docker e de Configuração do Django**

### **Dockerfile** *(localizado na raiz do projeto junto ao `manage.py`)*

```dockerfile
# Usar a imagem oficial do Python
FROM python:3.10

# Definir diretório de trabalho no contêiner
WORKDIR /app

# Copiar os arquivos do projeto
COPY . .

# Instalar dependências
RUN pip install --no-cache-dir -r requirements.txt

# Expor a porta 8000 (usada pelo Django)
EXPOSE 8000

# Comando para iniciar o servidor Django
CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
```

---

### **docker-compose.yml** *(localizado na raiz do projeto junto ao `Dockerfile`)*

```yaml
version: '3'

services:
  db:
    image: postgres:latest
    container_name: postgres_db
    restart: always
    environment:
      POSTGRES_DB: django_db
      POSTGRES_USER: django_user
      POSTGRES_PASSWORD: django_password
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

  webapp:
    build: .
    container_name: django_app
    restart: always
    depends_on:
      - db
    ports:
      - "8000:8000"
    environment:
      - DB_NAME=django_db
      - DB_USER=django_user
      - DB_PASSWORD=django_password
      - DB_HOST=db
      - DB_PORT=5432
    volumes:
      - .:/app

volumes:
  postgres_data:
```

---

### **settings.py** *(Trecho da configuração do banco de dados no Django)*

```python
import os

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.getenv('DB_NAME', 'django_db'),
        'USER': os.getenv('DB_USER', 'django_user'),
        'PASSWORD': os.getenv('DB_PASSWORD', 'django_password'),
        'HOST': os.getenv('DB_HOST', 'db'),
        'PORT': os.getenv('DB_PORT', '5432'),
    }
}
```

---

### **requirements.txt** *(Lista de dependências do Django e PostgreSQL)*

```txt
django
psycopg2-binary
Pillow
```

---

### **Comandos Importantes**

- **Construir os contêineres:**
  ```sh
  docker compose build
  ```

- **Rodar os contêineres em segundo plano:**
  ```sh
  docker compose up -d
  ```

- **Aplicar migrações no banco de dados:**
  ```sh
  docker compose exec webapp python manage.py migrate
  ```

- **Criar um superusuário do Django:**
  ```sh
  docker compose exec webapp python manage.py createsuperuser
  ```

- **Parar os contêineres:**
  ```sh
  docker compose down
  ```

---



