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