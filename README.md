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
<p>No site utilizar a opção Container Registry para habilitar este serviço.</p>
<p>O plano gratuiro oferece 500MB de espaço, quando da elaboração desse documento.</p>


1. <b>Instalar doctl</b>

<p>O doctl é um CLI , onde permite interagir com a API da Digital Ocean (criar, configurar e destruir recursos).</p>

<p>Instruções para instalar o doctl no Ubuntu:</p>

``` bash
$ cd ~ wget https://github.com/digitalocean/doctl/releases/download/v1.70.0/doctl-1.70.0-linux-amd64.tar.gz

$ tar xf ~/doctl-1.70.0-linux-amd64.tar.gz

$ sudo mv ~/doctl /usr/local/bin
``` 


<p>2. <b>Gerar Token e Autenticar</b></p>

<p>Acessar o site da Digital Ocean para gerar o Token.</p>

<p>Utilizar o "Menu API >> Botão Generate New Token >> Copiar o token".</p>

<p>No prompt do Ubuntu executar:</p>
``` bash
$ doctl auth init
Enter Your Access Token: """Colar o Token"""
```

3. Logar:
``` bash
$ docker login registry.digitalocean.com
```

4. Para verificar a conta ativa:
``` bash
$ doctl account get
```


<h2>Upload da Imagem</h2>

Usar o comando docker tag:
``` bash
$ docker tag fabiocaettano74/api-cadastro-usuario-production:v01 registry.digitalocean.com/fabiocaettano74/api-cadastro-usuario-production:v01
```

Usar o comando docker push para enviar a imagem para o repositório privado:
``` bash
$ docker push registry.digitalocean.com/fabiocaettano74/api-cadastro-usuario-production:v01
``` 

Acessar  a pagina da Digital Ocean e cliar na opção Container Register.

Mas para utilizar esta imagem no pod é necessário autencicação, no proximo passo será comentado como realizar esta etapa.

5. Secret

Criar o secret para que o pod consiga acessar o repositório privado:
``` bash
$ kubectl create secret docker-registry do-registry --docker-server=registry.digitalocean.com/fabiocaettano74 --docker-username=token --docker-password=token --docker-email=fabio.caettano74@gmail.com -n production
```
- do-registry é o nome do Secret.
- Nos parametros docker-username e docker-password informe o token gerado pelo site da digital ocean.


Visualizar o secret:
``` bash
$ kubectl get secrets do-registry --output=yaml -n production
```


6. Deployment

Na chave image informe a imagem do repositório privado:
``` yaml
spec:       
      containers:
      - name: api
        image: registry.digitalocean.com/fabiocaettano74/api-cadastro-usuario-production:v02
```

Incluir no manifesto a chave imagePullSecretes, com o name igual a do-registry.
O do-registry é nome do Secret.
``` yaml
spec:       
      containers:
      imagePullSecrets:
        - name: do-registry
```

Aplicar as alterações no deployment.
