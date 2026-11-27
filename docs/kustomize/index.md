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
<div align="center"><img width="407" height="175" alt="image" src="https://github.com/user-attachments/assets/77dc8c9b-4a18-4e82-8e9f-65c78e93bf8d" /></div>

Para entender cada campo desse arquivo:

* **apiVersion**: define a versão da API de configuração do Kustomize. Essa é a versão padrão e estável utilizada para interpretar a estrutura do arquivo kustomization.yaml;  
* **kind**: identifica o tipo de objeto que está sendo declarado, nesse caso, um kustomization;  
* **resources**: é a lista de entrada que diz ao Kustomize quais arquivos YAML (ou pastas com outros kustomization.yaml) devem ser carregados e processados;  
* **namespace**: é um campo transformador (_transformer_). Ele percorre todos os recursos listados acima e adiciona ou sobrescreve a linha **namespace: lab** dentro do bloco metadata de cada um deles (_esse campo é opcional, eu criei para personalizar o namespace_).

**kustomize build**  
O comando **kustomize build** é o comando que lê o arquivo kustomization.yaml de um diretório, junta todos os recursos listados, aplica as transformações (como namespace, labels e patches) e imprime o resultado final em formato YAML no terminal. Ele funciona como um "dry-run" (pré-visualização) e não altera nada no seu cluster Kubernetes.  

No diretório em que se encontra os arquivos, execute **`kustomize build .`** (o ponto indica que os arquivos de manifesto estão no mesmo diretório).  

Como informado anteriormente, esse comando não aplica nada no cluster, ele apenas apresenta uma saída em YAML das configurações que serão realizadas.  

Para aplicar as configurações efetivamente, você pode exportar a saída das informações para um arquivo com o comando **`kustomize build . > manifest.yaml`**  

Ou você pode propriamente implantar as configurações no cluster com os comandos **`kustomize build . | kubectl apply -f -`** ou **`kubectl apply -k .`**  

Da mesma forma que foram criados recursos, é possível eliminar os recursos, para isso, execute **`kustomize build . | kubectl delete -f`** - ou **`kubectl delete -k .`**  
<br>

## Gerenciando diretórios.  
Conforme o número de arquivos e implantações aumenta, você pode querer separar os arquivos por diretórios.  

Neste exemplo, criei o diretório **nginx**, que irá implantar as mesmas soluções dos passos anteriores.  
<div align="center"><img width="301" height="79" alt="image" src="https://github.com/user-attachments/assets/ddbb523a-4d90-4ae8-a518-e312ec8cbf1b" /></div>

Crie também o diretório **busybox**, inclua nesse diretório um arquivo de manifesto para implantar um Pod com image busybox (coloque nesse manifesto o namespace **lab**).  
<div align="center"><img width="166" height="44" alt="image" src="https://github.com/user-attachments/assets/5c916680-0a17-4b0c-990e-6295644b7e7d" /></div>

Agora, para que eu possa implantar todos esses recursos no cluster ao mesmo tempo, devemos alterar o arquivo kustomization.yaml da seguinte forma apontando para a nova estrutura de diretórios.
<div align="center"><img width="404" height="189" alt="image" src="https://github.com/user-attachments/assets/9c0bc7c6-9a65-4ea6-88f0-1d0a58ec5ba3" /></div>

Após executar **`kustomize build . | k apply -k .`**, todos os recursos foram implantados.  

Perceba que essa forma, eu não preciso colocar o caminho completo dos arquivos, apenas o caminho relativo considerando o local que eu me encontro no filesystem.
<br>

## Trabalhando com vários kustomization.yaml
Caso a sua estrutura de arquivos e implantações cresça muito, é possível deixar a configuração do kustomization.yaml mais granular ainda usando um kustomization.yaml dentro de cada diretório.  

Como exemplo, inicie criando a seguinte estrutura de diretórios.  
<div align="center"><img width="250" height="510" alt="image" src="https://github.com/user-attachments/assets/d4ca276f-d6bf-4814-909d-68209eb99f12" /></div>

O primeiro kustomization.yaml (grifado) aponta para o arquivo **namespace.yaml** para criar o namespace lab, e aponta para todos os diretórios de Pods.  
<div align="center"><img width="400" height="289" alt="image" src="https://github.com/user-attachments/assets/519c43d2-c957-45a4-8113-b3bdad82abfa" /></div>

Dentro de cada diretório de Pods, você precisa ter o manifesto **busybox.yaml** para criar o Pod propriamente, e outro arquivo **kustomization.yaml** apontando para esse arquivo de manifesto... abaixo, o exemplo do **pod1/kustomization.yaml**  
<div align="center"><img width="398" height="137" alt="image" src="https://github.com/user-attachments/assets/9e9d20d9-967c-426c-9854-5e3ef545b0bf" /></div>

Esse **pod1/kustomization.yaml** pode ser igual para todos os outros.  
Essa estrutura foi criada dessa forma apenas como exemplo de como implantar vários manifestos ao mesmo tempo com o Kustomization. Após executar **`kubectl apply -k .`**, todos esses manifestos são implantados ao mesmo tempo.
<div align="center"><img width="205" height="172" alt="image" src="https://github.com/user-attachments/assets/6e425757-bd11-4a23-8932-0a689880a632" /></div>
<br>

## Transformers.
Transformers no Kustomize são regras declarativas que modificam manifestos YAML do Kubernetes durante o processo de build (**kustomize build**), sem alterar os arquivos originais. Eles rodam após os generators e aplicam alterações sistemáticas em um ou vários recursos de forma reutilizável.  

No exemplo do laboratório anterior, o campo **namespace** já era um transformer, onde ele definiu o namespace **lab** em todos os Pods.  

**Os principais transformers nativos são:**  

* **Namespace (namespace):** define ou substitui o namespace em múltiplos recursos de uma só vez;  
* **Labels (labels / commonLabels):** adiciona ou altera labels em metadados e seletores de recursos;  
* **Annotations (annotations / commonAnnotations):** inclui ou atualiza anotações nos manifestos;  
* **Prefixo e sufixo (namePrefix e nameSuffix):** adiciona prefixos ou sufixos aos nomes dos recursos (como dev- ou -v2);  
* **Imagens (images):** altera nomes, tags (newTag) ou digests de imagens de contêineres por ambiente;  
* **Referências de nomes (nameReference):** atualiza automaticamente referências entre recursos quando um nome é alterado por prefixo ou sufixo;  

Abaixo, um exemplo usando **labels**:  
<div align="center"><img width="201" height="60" alt="image" src="https://github.com/user-attachments/assets/388ec1e1-1782-40fd-a3ea-e082b0f442e6" /></div>

O exemplo acima adicionará a label **lab: KodeKloud** em todos os Pods. Você pode combinar múltiplas regras dentro da lista **labels** usando as seguintes propriedades:
<div align="center"><img width="598" height="181" alt="image" src="https://github.com/user-attachments/assets/0c3f0140-ef77-441a-9d18-aca8ca5fd476" /></div>

Agora, um exemplo usando **images**.  
<div align="center"><img width="198" height="79" alt="image" src="https://github.com/user-attachments/assets/83f82b20-494b-4d42-bf2c-e4a1c7c3c202" /></div>

No exemplo acima, originalmente, os Pods usavam a imagem **busybox**, e a transformação irá usar a imagem **busybox** com a tag **1.37.0**
<br>

## Patchs.
Patches no Kustomize são instruções usadas para modificar campos específicos de recursos do Kubernetes (como aumentar réplicas, alterar limites de CPU/memória ou adicionar variáveis de ambiente) sem precisar duplicar o arquivo YAML inteiro.  

Enquanto os transformers (labels, images, namespace) aplicam regras globais e padronizadas em vários recursos ao mesmo tempo, os patches fazem cirurgias pontuais em um ou mais recursos específicos usando o campo patches no kustomization.yaml.  
<br>

**JSON 6902 Patch.**  
Patch JSON 6902 (RFC 6902) é um formato padronizado para descrever alterações em um documento JSON (ou YAML, no caso do Kubernetes e Kustomize) por meio de uma lista de operações explícitas sobre caminhos específicos (path) do recurso.  

No Kustomize, ele é uma das duas principais formas de aplicar patches (junto com o Strategic Merge Patch). Enquanto o Strategic Merge usa um trecho parcial do manifesto YAML para mesclar alterações, o JSON Patch 6902 usa instruções precisas passo a passo. Abaixo, um exemplo, alterando o nome de um Deployment.  

Na estrutura de arquivos **deploy** eu tenho os seguintes arquivos:  
<div align="center"><img width="282" height="109" alt="image" src="https://github.com/user-attachments/assets/4c124842-1f4c-49f0-8912-46fd2c2cd267" /></div>

O arquivo **kustomization.yaml** foi configurado da seguinte forma:  
<div align="center"><img width="404" height="327" alt="image" src="https://github.com/user-attachments/assets/2a3cb73d-c8a4-496a-96b8-dbb851e60640" /></div>

Para entender as configurações criadas no arquivo acima:

* **patches**: inicia a lista de modificações pontuais que o Kustomize aplicará sobre os recursos carregados.  
  * **target**: define o filtro de qual recurso especifico receberá a alteração;  
  * **kind**: restringe a busca apenas a recursos do tipo Deployment;  
  * **name**: seleciona especificamente o Deployment cujo metadata.name original é nginx-deployment (definido no arquivo nginx/nginx-deployment.yaml);  

* **patch**: inicia o bloco de instruções do patch escrito diretamente no arquivo (inline). O operador |- indica uma string de múltiplas linhas sem quebra de linha extra no final;  
  * **op**: define a operação JSON Patch como substituição (troca um valor existente por outro);  
  * **path**: aponta o caminho exato no YAML que será alterado (o campo name dentro de metadata);  
  * **value**: define o novo nome que substituirá nginx-deployment no manifesto final.  

Para usar mais de uma operação ao mesmo tempo, adicione a operação logo abaixo, como no exemplo abaixo, a operação para alterar o número de réplicas no Deployment:  
<div align="center"><img width="379" height="230" alt="image" src="https://github.com/user-attachments/assets/7964db72-e3d7-4efe-9b0f-0756678e45a2" /></div>

Além da operação **replace**, existem as operações **add** e **remove**. Abaixo, cada um dos dois exemplos:   

Neste exemplo, eu estou adicionando a label **dev: brsouza** em **spec.template.metadata.labels** de um Deployment.  
<div align="center"><img width="455" height="347" alt="image" src="https://github.com/user-attachments/assets/3f405f68-39f9-46d8-b7ed-b51b4f71689e" /></div>

Para utilizar a operação **remove**, o componente já deve existir no Deployment... com isso, no Deployment, eu adicionei em **spec.template.metadata.labels** a label **dev: brsouza**  

Após isso, criei a transformação abaixo, para remover a label:
<div align="center"><img width="417" height="71" alt="image" src="https://github.com/user-attachments/assets/7b6dced1-4417-4308-b675-741edf4c57fd" /></div>

Com isso, o Deployment é implantado sem essa label.  
<br>

**Strategic merge patch.**  
O **strategic merge patch** é um formato de atualização parcial de recursos no Kubernetes que mescla campos de forma inteligente com base na estrutura do objeto e em chaves de mesclagem (merge keys), em vez de substituir listas inteiras.  

Abaixo, como trabalhar com o formato strategic merge patch. Neste momento, eu vou trocar a configuração dos exemplos acima, porém, usando o formato strategic merge patch.  
<div align="center"><img width="338" height="212" alt="image" src="https://github.com/user-attachments/assets/9847cb54-d5bd-4132-90a4-f6eea7ff64a7" /></div>

Pode perceber que os campos em patch são muito parecidos com um arquivo de manifesto do Deployment.  

**Patch em arquivo separado.**  
No Kustomization, você pode configurar o patch em um arquivo separado. No exemplo abaixo, eu apontei a minha configuração para o arquivos **labels.yaml**  
<div align="center"><img width="198" height="42" alt="image" src="https://github.com/user-attachments/assets/64fef4d8-b9bb-4cec-9082-80560ca1e0ac" /></div>

Depois, criei o arquivo **labels.yaml** apenas para customizar o número de réplicas.  
<div align="center"><img width="243" height="120" alt="image" src="https://github.com/user-attachments/assets/45671d11-3233-44a5-9fd1-62bd3f511d9d" /></div>

## Overlays.
No Kustomize, Overlays são configurações que permitem personalizar os mesmos manifests Kubernetes para diferentes ambientes, sem precisar duplicar todos os arquivos YAML.  

**Como funcionam?**
* **Estrutura em camadas**: a pasta **base/** guarda os manifestos comuns e o arquivo **kustomization.yaml** principal. Cada subpasta em **overlays/** representa um ambiente e contém seu próprio **kustomization.yaml** que aponta para a base;

* **Regra prática**: o que é igual em todos os ambientes fica na base; o que muda por ambiente fica no overlay;  

* **Aplicação**: você gera ou aplica o resultado final com comandos como **`kubectl kustomize overlays/prod`** ou **`kubectl apply -k overlays/prod`**  

**Exemplo de uso.**  
Considerando que eu já tenho a estrutura de arquivo abaixo, vou iniciar um exemplo de utilização da estrutura com overlays.
<div align="center"><img width="294" height="109" alt="image" src="https://github.com/user-attachments/assets/c590218a-e4a3-44e5-a922-036838a4359d" /></div>

Agora, crie o diretório **overlays/development** e crie o arquivo **kustomization.yaml** neste local com o conteúdo abaixo:    
<div align="center"><img width="246" height="156" alt="image" src="https://github.com/user-attachments/assets/23670c1a-a30c-45dd-8142-84e2222f9c29" /></div>

Essa configuração indica:  
* **resources**: reutiliza a base que está dois níveis acima;  
* **nameSuffix**: adiciona **-dev** ao nome dos recursos;  
* **replicas**: define uma réplica para o Deployment chamado nginx-deployment.  

Crie também o diretório **overlays/production** e crie o arquivo **kustomization.yaml** neste local com o conteúdo abaixo:  
<div align="center"><img width="249" height="154" alt="image" src="https://github.com/user-attachments/assets/86d13eea-6e2f-40ad-b411-50394baaee2a" /></div>

Para esse caso do **production**, a ideia é a mesma, apenas alterando o **nameSuffix** e o número de réplicas.  

Eu criei o diretório **base**, coloquei o kustomization.yaml nesse diretório e os arquivos de manifesto da aplicação também nesse diretório... a estrutura final deve ficar da seguinte forma:  
<div align="center"><img width="329" height="228" alt="image" src="https://github.com/user-attachments/assets/764b2c23-0f6f-4f76-8dde-2b8672592def" /></div>

Após executar **kubectl apply -k overlays/development/** e **kubectl apply -k overlays/production/** os Pods e os Services foram implantados no namespace **lab**
<div align="center"><img width="558" height="139" alt="image" src="https://github.com/user-attachments/assets/9b9d3a77-1e4f-48b9-a3a8-bb41687df0ff" /></div>
