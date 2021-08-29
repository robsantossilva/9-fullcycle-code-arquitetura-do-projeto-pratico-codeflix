# Arquitetura do projeto prático - Codeflix
---
- Uma espécie de Netflix
- Assinatura do serviço pelo cliente
- Catálogo de vídeos para navegação
- Playback de vídeos
- Busca full text no catálogo

--

- Processamento e encoding dos vídeos
- Administração do catálogo de vídeos
- Administração do serviço de assinatura
- Autenticação

### Decisões de projeto e de arquitetura

**Microsserviços**
- Arquitetura baseada em microsserviços
- Tecnologia adequada para cada contexto (ex: Golang para processar vídeos)
- Não existe uma única verdade na escolha das tecnologias
- Microsserviços podem ser substituidos por outros com tecnologias diferentes
- Cada microsserviço terá seu próprio processo de CI/CD

**API Gateway**
- Acesso externo aos microsserviços através do Ingress do Kubernetes /Istio como API Gateway
- Único ponto de acesso direto as aplicações
- Controle de tráfego
- Rate limit
- Políticas de Retry
- Etc

**Service Discovery**
- Não haverá a necessidade de trabalhar com um sistema de Service Discovery como "Consul"
- O projeto utilizará o Kubernetes para orquestrar os containers, logo Service Discovery já faz parte do processo

**Escala horizontal**
- O processo de escala poderá ser configurado a nível de microsserviço
- Todos os microsserviços trabalharão de forma "Stateless"
- Quando utilizado upload de qualquer tipo de asset, o mesmo será armazenado em um Cloud Storage
- O processo de escala se dará no aumento na quantidade de PODs do Kubernetes
- O processo de autoscaling também será utilizado de um recurso chamado HPA (Horizontal Pod Autoscaler)
- Todos os logs gerados serão persistidos em sistemas externos como Prometheus e Elasticsearch

**Consistência eventual**
- Grande parte da comunicação entre os microsserviços será assíncrona
- Cada microsserviço possuirá sua própria base de dados
- Eventualmente os dados poderão ficar inconsistentes, desde que não haja prejuízo direto ao negócio

**Duplicação de dados**
- Eventualmente um microsserviço poderá persistir dados já existentes em outro microsserviço em seu banco de dados
- Essa duplicação ocorre para deixar o microsserviço mais autônomo possivel
- O microsserviço duplicará apenas os dados necessários para seu contexto

**Mensageria**
- Como grande parte da comunicação entre os microsserviços é assíncrona, um sistema de mensageria é necessário
- O RabbitMQ foi escolhido para esse caso
- Por que não o Apache Kafka ou Amazon SQS, entre outros?
    - Apache Kafka também poderia ser utilizado nesse caso, por outro lado, muitos recursos que ele poderia prover não seriam utilizados
    - Evitaremos ao máximo o Lock-in nos cloud providers, logo, Amazon SQS e similares foram descartados
    - Não há uma verdade única sobre a escolha realizada

**Resiliência e self healing**
- Para garantir resiliência caso um ou mais microsserviços fiquem fora do ar, as filas serão essenciais
- Caso uma mensagem venha em um padrão não esperado para determinado microsserviço, o microsserviço poderá rejeitá-la e automaticamente a mesma poderpa ser encaminhada para uma dead-letter queue.
- Pelo fato do Kubernetes e Istio possuitem recursos de Circuit Breaker, Liveness e Readiness probes:
    - Se um container tiver um crash, automaticamente ele será reiniciado ou mesmo recriado
    - Caso um container não aguente determinado tráfego, temos a opção de trabalhar com Circuit Breaker para impedir que ele receba mais requisições enquanto está se "curando"

**Autenticação**
- Serviço centralizador de identidade opensource: KEYCLOAK
- OpenID Connect
- Customização do tema
  - Utilização do create-react-app
- Compartilhamento de chave pública com os serviços para verificação de autenticação dos tokens
- Diversos tipos de ACL
- Flow de autenticação para frontend e backend