# Helm.
## O que é o Helm e o Chart.

O **Helm** é um gerenciador de pacotes para Kubernetes que facilita a definição, instalação e atualização de aplicações complexas no Kubernetes. Com ele você pode definir no detalhe como a sua aplicação será instalada, quais configurações serão utilizadas e como será feita a atualização da aplicação.  

Um **Chart** é um pacote que contém informações necessárias para criar instâncias de aplicações Kubernetes. É com ele que iremos definir como a nossa aplicação será instalada, quais configurações serão utilizadas e como será feita a atualização da aplicação.  

Um **Chart**, normalmente é composto por um conjunto de arquivos que definem a aplicação, e um conjunto de templates que definem como a aplicação será instalada no Kubernetes.  

**Artifacthub.**  
Neste contexto de Helm e Chart, um portal muito útil é o **[Artifacthub](https://artifacthub.io/)**. O Artifacthub é um site que centraliza pacotes para Kubernetes, como:  

* Helm Charts;  
* Operators;  
* Plugins;  
* etc.  

👉 Pense nele como:  

* App Store do Kubernetes;  
* ou tipo um "Google dos Charts".  

## Instalando o Helm.  
Verifique a documentação e as formas de instalação na página do **[helm.sh](https://helm.sh/)**.  
Utilizando o script, a instalação é bem simples... siga os passos abaixo para instalação.  

```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-4
chmod 700 get_helm.sh
./get_helm.sh
```

## Relação de comandos do Helm.  
Abaixo, os principais comandos para utilização do Helm:  
