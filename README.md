<h1>Kubedev - Modulo 16 - Exercicio</h1>

<h1>Coloque a imagem da sua API em um Docker Hub privado</h1>

<h2>Ambiente de Desenvolvimento:</h2>

1. Ferramentas
* Windows 10 Pro;
* WSL 2;
* Ubuntu 20.4;
* Visual Studio Code;
* Api para o teste.
* Docker;
* Kubernetes;
* doctl;
* Cadastro na Digital Ocean.


<h2>Repositório Privado</h2>
<p>O repositório privado utilizado é o da Digital Ocean.</p>

<p>1. <b>Container Registry</b></p>
<p>No site utilizar a opção Container Registry para habilitar este serviço.</p>
<p>O plano gratuiro oferece 500MB de espaço, quando da elaboração desse documento.</p>


<p>2. <b>Instalar doctl</b></p>

<p>O doctl é um CLI , onde permite interagir com a API da Digital Ocean (criar, configurar e destruir recursos).</p>

<p>Instruções para instalar o doctl no Ubuntu:</p>

``` bash
$ cd ~ wget https://github.com/digitalocean/doctl/releases/download/v1.70.0/doctl-1.70.0-linux-amd64.tar.gz
```

``` bash
$ tar xf ~/doctl-1.70.0-linux-amd64.tar.gz
```

``` bash
$ sudo mv ~/doctl /usr/local/bin
``` 


<p>3. <b>Gerar Token e Autenticar</b></p>

<p>Acessar o site da Digital Ocean para gerar o Token.</p>

<p>Utilizar o "Menu API >> Botão Generate New Token >> Copiar o token".</p>

<p>No prompt do Ubuntu executar:</p>

``` bash
$ doctl auth init
Enter Your Access Token: Colar o Token
```

<p>4. <b>Logar</b></p>

``` bash
$ docker login registry.digitalocean.com
```

<p>5. <b>Para verificar a conta ativa:</b></p>

``` bash
$ doctl account get
```


<h2>Upload da Imagem</h2>

<p>1. <b>Docker tag</b></p>

``` bash
$ docker tag fabiocaettano74/api-cadastro-usuario-production:v01 registry.digitalocean.com/fabiocaettano74/api-cadastro-usuario-production:v01
```

<p>2. <b>Docker push</b></p>

<p>Usar o comando docker push para enviar a imagem para o repositório privado:</p>

``` bash
$ docker push registry.digitalocean.com/fabiocaettano74/api-cadastro-usuario-production:v01
``` 

<p>No menu Container Register , no site da Digital Ocean,  é possivel visualizar a imagem.</p>


<h2> Secret </h2>

<p>Mas para o pod utilizar a imagem é necessário uma autenticação, por tratar-se de um container privado.</p>


<p>1. <b>Criar o Secret</b></p>

``` bash
$ kubectl create secret docker-registry do-registry --docker-server=registry.digitalocean.com/fabiocaettano74 --docker-username=token --docker-password=token --docker-email=fabio.caettano74@gmail.com -n production
```

- O do-registry é o nome do Secret.
- O docker-server é composto registry.digitalocean.com/nomeDoRepositorio;
- Nos parametros docker-username e docker-password informe o token gerado pelo site da digital ocean;

<p>2. <b>Visualizar o Secret</b></p>

``` bash
$ kubectl get secrets do-registry --output=yaml -n production
```


<h2> Deployment </h2>

<p>Na chave image informe a imagem do repositório privado:</p>

``` yaml
spec:       
      containers:
      - name: api
        image: registry.digitalocean.com/fabiocaettano74/api-cadastro-usuario-production:v02
```

<p>Incluir no manifesto a chave imagePullSecretes, e na chave name informe o nome do Secret:</p>

``` yaml
spec:       
      containers:
      imagePullSecrets:
        - name: do-registry
```

