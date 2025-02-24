# Docker - Introdução

## Informações gerais
- **Público alvo**: alunos da disciplina de SO (Sistemas Operacionais) do curso de TADS (Superior em Tecnologia em Análise e Desenvolvimento de Sistemas) no CNAT-IFRN (Instituto Federal de Educação, Ciência e Tecnologia do Rio Grande do Norte - Campus Natal-Central).
- disciplina: **SO** Sistemas Operacionais
- Assunto: Docker e conteinização de aplicações
- professor: [Leonardo A. Minora](https://github.com/leonardo-minora)
- texto gerado pelo [Microsoft Copilot](https://copilot.microsoft.com/)

## Sumário
1. Aula de 14/02 - Docker
2. Aula de 17/02 - Docker compose


## 1. Aula de 14/02 - Docker
1.1. Cliente e servidor em python
1.2. Construindo com docker

### 1.1. Cliente e servidor em python

#### Servidor HTTP
`servidor-http.py`

```python
import http.server

# Definir o conteúdo HTML fixo
html_fixo = """
<!DOCTYPE html>
<html>
<head>
    <title>Servidor HTTP Multithread</title>
</head>
<body>
    <h1>Olá, mundo!</h1>
    <p>Este é um servidor HTTP multithread em Python.</p>
</body>
</html>
"""

class MeuManipulador(http.server.SimpleHTTPRequestHandler):
    def do_GET(self):
        # Imprimir informações da requisição no console
        print(f"Requisição recebida de: {self.client_address}")
        print(f"Caminho solicitado: {self.path}")
        print("Cabeçalhos da requisição:")
        for nome, valor in self.headers.items():
            print(f"{nome}: {valor}")

        # Enviar resposta de código 200
        self.send_response(200)
        self.send_header("Content-type", "text/html")
        self.end_headers()
        # Enviar o conteúdo HTML fixo
        self.wfile.write(html_fixo.encode("utf-8"))

# Definir o endereço e a porta do servidor
endereco = ("", 8000)

# Criar o servidor
with http.server.HTTPServer(endereco, MeuManipulador) as httpd:
    print("Servidor HTTP rodando na porta 8000...")
    # Manter o servidor rodando
    httpd.serve_forever()

```

#### Cliente HTTP com Threads
`cliente-http.py`

```python
import http.client
from concurrent.futures import ThreadPoolExecutor

def fazer_requisicao_get(id):
    print(f'### thread id {id} - Iniciou')
    # Conectar ao servidor localhost na porta 3000
    conexao = http.client.HTTPConnection("localhost", 3000)
    print(f'### thread id {id} - Abriu conexão')

    # Fazer a requisição GET
    conexao.request("GET", f"/{id}")
    print(f'### thread id {id} - Enviou requisição')

    # Obter a resposta
    resposta = conexao.getresponse()

    # Ler o conteúdo da resposta
    # conteudo = resposta.read()

    # Imprimir o status e o conteúdo da resposta
    print(f"Status ({id}): {resposta.status}")
    print(f"Motivo ({id}): {resposta.reason}")
    # print("Conteúdo:")
    # print(conteudo.decode("utf-8"))

    # Fechar a conexão
    conexao.close()
    print(f'### thread id {id} - Terminou')

# Chamar a função para fazer a requisição GET
# fazer_requisicao_get()

# Usar ThreadPoolExecutor para gerenciar threads
# with ThreadPoolExecutor(max_workers=2) as executor:
#     futuro1 = executor.submit(fazer_requisicao_get, 1)
#     futuro2 = executor.submit(fazer_requisicao_get, 2)

#     futuro1.result()
#     futuro2.result()

futuro = [0] * 10
with ThreadPoolExecutor(max_workers=10) as executor:
    for thread_id in range(10):
        futuro[thread_id] = executor.submit(fazer_requisicao_get, thread_id)

    for thread_id in range(10):
        futuro[thread_id].result()

```

### 1.2. Construindo com docker

#### Servidor Docker
1. Colocar o arquivo Servidor HTTP `servidor-http.py` e o arquivo Dockerfile abaixo no mesmo diretório (pasta) `servidor`
2. Executar o comando `docker build -t http-server-python .` para **construir** a **imagem**
3. Executar o comando `docker run -p 3000:8000 --name servidor http-server-python` para executar o **container**
   - Executar o comando noutro terminal `docker stop servidor` para **parar** o **container**
   - Para executar o container novamente, em um terminal executar `docker start servidor`

`Dockerfile`

```
# Usar uma imagem base do Python
FROM python:3-slim

# Definir o diretório de trabalho no contêiner
WORKDIR /app

# Copiar o script Python para o contêiner
COPY servidor-http.py .

# Expor a porta 8000
EXPOSE 8000

# Executar o script Python
CMD ["python", "servidor-http.py"]

```

#### Cliente Docker

1. Colocar o arquivo cliente HTTP `cliente-http.py` e o arquivo Dockerfile abaixo no mesmo diretório (pasta) `cliente`
2. Executar o comando `docker build -t http-cliente-python .` para **construir** a **imagem**
3. Executar o comando `docker run -i --name cliente http-cliente-python` para executar o **container**
   - Executar o comando noutro terminal `docker stop cliente` para **parar** o **container**
   - Para executar o container novamente, em um terminal executar `docker start cliente`

**Obs** a execução ocasiona um erro porque a requisição é para `localhost:3000`, ou seja, a _máquina_ é a imagem do cliente http que não tem uma porta 8000 aberta.


`Dockerfile`

```Dockerfile
# Usar uma imagem base do Python
FROM python:3-slim

# Definir o diretório de trabalho no contêiner
WORKDIR /app

# Copiar o script Python para o contêiner
COPY cliente-http.py .

# Executar o script Python
CMD ["python", "cliente-http.py"]

```

## 2. Aula de 17/02 - Docker compose

[docker compose](https://docs.docker.com/compose/)

1. Para o cliente http
   - Criar uma 2a versão do cliente http, conforme nome de arquivo e código abaixo
   - Modificar o `Dockerfile` conforme abaixo porque o arquivo cliente http modificou
2. Para composição dos serviços, nome do docker compose para imagens, no diretório princial criar o arquivo `docker-compose.yml`
3. Executar na linha de comando `docker-compose up --build`

`cliente-http-python-v2.py`

```python
import requests

response = requests.get('http://servidor:8000')
print(response.text)

```

`docker-compose.yml`

```yaml
version: '3'

services:
  servidor:
    build: ./servidor
    ports:
      - "8000:8000"

  cliente:
    build: ./cliente
    depends_on:
      - servidor
```
