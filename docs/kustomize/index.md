# Kustomize.

## O que é o Kustomize?
O Kustomize é uma ferramenta de gerenciamento de configurações nativa do ecossistema Kubernetes que permite customizar arquivos YAML de manifesto sem a necessidade de usar motores de templates (como o Helm) ou alterar os arquivos originais.  

**Eu preciso instalar o kustomize?**  
Não necessariamente, pois o Kustomize já vem embutido nativamente no kubectl (desde a versão 1.14) através dos comandos **kubectl kustomize** e **kubectl apply -k**. No entanto, instalar o binário independente (kustomize) é recomendado para ter acesso aos recursos mais recentes.

Para instalar o binário do kustomize, execute o script de instalação oficial com o comando:  

```
curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh" | bash
```

Para ver a versão do kustomize, execute **`kustomize version`**
<br>

## O que é o kustomization.yaml
O **kustomization.yaml** é o arquivo de configuração principal do Kustomize que funciona como um manifesto de receitas. Ele define quais arquivos YAML do Kubernetes devem ser lidos e quais transformações, filtros ou patches devem ser aplicados a eles antes de gerar o resultado final.  

Sempre que você executa **kustomize build** ou **kubectl apply -k** em um diretório, o Kustomize procura obrigatoriamente por um arquivo chamado kustomization.yaml (ou kustomization.yml).  

**Iniciando um primeiro exemplo.**  
Para iniciar um primeiro exemplo de uso do kustomize, crie um diretório, como **deploy**, nesse diretório devem ser adicionados quatro arquivos de manifestos, o deployment de uma aplicação, o service para essa aplicação, um arquivo para criar o namespace e o arquivo **kustomization.yaml**  

Em relação ao kustomization.yaml, crie o arquivo confirme exemplo abaixo:  
<div align="center"><img width="405" height="177" alt="image" src="https://github.com/user-attachments/assets/71873eaf-76a2-4521-85e6-7011e0754239" /></div>

Para entender cada campo desse arquivo:

* **apiVersion**: define a versão da API de configuração do Kustomize. Essa é a versão padrão e estável utilizada para interpretar a estrutura do arquivo kustomization.yaml;  
* **kind**: identifica o tipo de objeto que está sendo declarado, nesse caso, um kustomization;  
* **resources**: é a lista de entrada que diz ao Kustomize quais arquivos YAML (ou pastas com outros kustomization.yaml) devem ser carregados e processados;  
* **namespace**: é um campo transformador (_transformer_). Ele percorre todos os recursos listados acima e adiciona ou sobrescreve a linha **namespace: lab** dentro do bloco metadata de cada um deles (esse campo é opcional, eu criei para personalizar o namespace).

**kustomize build**  
O comando **kustomize build** é o comando que lê o arquivo kustomization.yaml de um diretório, junta todos os recursos listados, aplica as transformações (como namespace, labels e patches) e imprime o resultado final em formato YAML no terminal. Ele funciona como um "dry-run" (pré-visualização) e não altera nada no seu cluster Kubernetes.  

No diretório em que se encontra os arquivos, execute **`kustomize build .`** (o ponto indica que os arquivos de manifesto estão no mesmo diretório).  

Como informado anteriormente, esse comando não aplica nada no cluster, ele apenas apresenta uma saída em YAML das configurações que serão realizadas.  

Para aplicar as configurações efetivamente, você pode exportar a saída das informações para um arquivo com o comando **`kustomize build . > manifest.yaml`**  

Ou você pode propriamente implantar as configurações no cluster com os comandos **`kustomize build . | kubectl apply -f -`** ou **`kubectl apply -k .`**  

Da mesma forma que foram criados recursos, é possível eliminar os recursos, para isso, execute **`kustomize build . | kubectl delete -f`** - ou **`kubectl delete -k .`**  
<br>

## Gerenciando diretórios.


