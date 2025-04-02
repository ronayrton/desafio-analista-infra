# Etapa 1 - Tarefa: Criar um contêiner para uma aplicação simples e expor as métricas via Jolokia.

## Objetivo

O objetivo desta etapa é configurar e disponibilizar uma aplicação Jenkins rodando no Apache Tomcat, garantindo que métricas de desempenho estejam acessíveis via Jolokia.

## Passos

### 1. Preparação do Ambiente

Instalar Docker (se ainda não estiver instalado).

Criar um Dockerfile para construir a imagem contendo o Jenkins e o Jolokia.

### 2. Construção da Imagem

Utilizar um Dockerfile baseado na imagem oficial do Tomcat 9.

Baixar e adicionar o Jenkins na pasta de aplicações do Tomcat.

Baixar e adicionar o Jolokia para expor métricas via JMX.

Expor a porta 8080 para acesso ao Tomcat e Jenkins.

### 3. Execução do Container

Após criar o Dockerfile, construir a imagem e executar o container:

- Construir a imagem
docker build -t jenkins-tomcat .

- Executar o container
docker run -d -p 8080:8080 --name jenkins-infra jenkins-tomcat

### 4. Testes e Validação

Acessar o Jenkins via http://localhost:8080/jenkins.

Verificar a resposta do Jolokia via http://localhost:8080/jolokia/version.

Validar se as métricas estão acessíveis usando:

curl http://localhost:8080/jolokia/read/java.lang:type=Memory

Observação: Ao acessar http://localhost:8080/jolokia/version, foi apresentada uma tela solicitando usuário e senha. Caso necessário, consulte a documentação do Jolokia para configurar autenticação ou desativá-la conforme a política de segurança do ambiente.

## Considerações Finais

Certificar-se de que os serviços estão rodando corretamente.

Garantir que as métricas do Jolokia possam ser acessadas remotamente, caso necessário.

Caso tenha dúvidas ou precise de ajustes, consulte a documentação oficial do Jenkins e do Jolokia.

## Referências

[Documentação do Jenkins](https://www.jenkins.io/sigs/docs/)

[Documentação do Tomcat](https://tomcat.apache.org/tomcat-9.0-doc/)

[Documentação do Jolokia](https://jolokia.org/documentation.html)