# CLI Integradora para Arcee, TESS e MCP

Este projeto implementa uma interface de linha de comando (CLI) que integra APIs de terceiros, incluindo Arcee AI, TESS e o SDK oficial do Model Context Protocol (@sdk-mcp). O projeto segue a estrutura Clean Architecture e os princípios do Domain-Driven Design (DDD).

## Visão Geral da Integração

Esta CLI **não reimplementa** os serviços que integra, mas serve como uma camada de abstração que permite:

1. Acesso unificado a múltiplos serviços (Arcee, TESS, MCP)
2. Padronização de interfaces para facilitar o uso
3. Adição de funcionalidades específicas à CLI sem modificar as APIs subjacentes

## Arquitetura de Componentes

### MCP (Model Context Protocol)

O MCP é um protocolo padronizado que facilita a comunicação entre modelos de linguagem e ferramentas externas. Este projeto utiliza o SDK oficial do MCP (@sdk-mcp) para implementar essa comunicação, sem modificar ou reimplementar o protocolo.

### Servidor TESS-MCP

O servidor TESS-MCP é o componente responsável por implementar a integração entre o TESS (Tool Execution Subsystem) e o MCP. Este servidor:

- Orquestra e gerencia ferramentas que podem ser utilizadas por modelos de IA
- Expõe APIs para listar e executar agentes de IA
- Gerencia upload e organização de arquivos
- Permite interação com modelos de linguagem como o "tess-ai-light"

O TESS trabalha em conjunto com o MCP, permitindo que modelos solicitem e utilizem ferramentas de forma padronizada.

### CLI (Interface de Linha de Comando)

A CLI **consome** as APIs do servidor TESS-MCP, Arcee AI e MCP, fornecendo uma interface unificada para o usuário final. Ela não reimplementa essas funcionalidades, apenas oferece uma forma padronizada de acessá-las.

### Arcee AI

O Arcee AI é uma plataforma completa de IA que inclui:

- **Arcee Orchestra**: plataforma agentic para construir fluxos de trabalho de IA
- **Arcee Conductor**: roteador inteligente que seleciona o modelo mais adequado e eficiente em custo para cada prompt
- **Small Language Models (SLMs)**: modelos de linguagem otimizados para tarefas específicas

Para mais informações sobre TESS, consulte a [documentação oficial da API TESS](https://docs.tess.pareto.io/api/introduction).

## Arquitetura do Projeto

O projeto segue a Clean Architecture com as seguintes camadas:

### 1. Domínio

- `domain/interfaces`: Interfaces que definem contratos para integração com APIs externas
- `domain/services`: Serviços de domínio que orquestram as chamadas para APIs externas
- `domain/exceptions`: Exceções de domínio para tratamento de erros

### 2. Aplicação

- `application/use_cases`: Casos de uso que coordenam a lógica de negócio

### 3. Infraestrutura

- `infrastructure/mcp_client`: Cliente que integra o SDK oficial do MCP (@sdk-mcp)
- `infrastructure/providers`: Adaptadores para as APIs externas (TESS, Arcee)

### 4. Interface de Usuário

- `src/commands`: Comandos CLI para interação com os casos de uso
- `src/adapters`: Adaptadores para compatibilidade entre implementações

## Arquitetura de Adaptadores

### Papel dos Adaptadores

Os adaptadores no projeto são **componentes arquiteturais intencionais** que servem a propósitos específicos:

1. **Desacoplamento entre sistemas**:
   - MCP e TESS são sistemas com responsabilidades distintas que evoluem independentemente
   - Os adaptadores permitem que essas evoluções ocorram sem quebrar a integração

2. **Tradução entre interfaces**:
   - Convertem chamadas e dados entre o formato esperado por cada sistema
   - Protegem o domínio de mudanças nas APIs externas

3. **Implementação do padrão Ports & Adapters**:
   - Facilitam testes unitários e simulações
   - Permitem substituir implementações sem afetar o domínio

### Tipos de Adaptadores

O projeto utiliza dois tipos principais de adaptadores:

1. **Adaptadores de Infraestrutura**: 
   - Localização: `infrastructure/providers`
   - Propósito: Conectar as interfaces do domínio com APIs externas
   - Exemplo: `TessApiProvider` adapta a API do TESS para a interface `ITessProvider` do domínio

2. **Adaptadores de Compatibilidade**:
   - Localização: `src/adapters`
   - Propósito: Manter compatibilidade com código existente enquanto usa as novas implementações
   - Exemplo: `MCPRunClient` adapta a nova implementação para código que ainda usa a interface antiga

### Otimização de Adaptadores

Nosso foco é manter adaptadores eficientes através de:

- Interfaces padronizadas com contratos claros
- Minimização de conversões desnecessárias
- Cobertura adequada de testes
- Documentação das responsabilidades de cada adaptador

## Funcionalidades da CLI

A CLI fornece acesso unificado aos seguintes serviços:

### MCP e TESS

A CLI consome as APIs do servidor TESS-MCP, permitindo:

- Listar todas as ferramentas disponíveis
- Buscar ferramentas específicas
- Obter detalhes de ferramentas e agentes
- Executar ferramentas e agentes com parâmetros
- Fazer upload de arquivos para uso com agentes

### Arcee AI

A CLI permite interagir com a API do Arcee AI:

- Gerar conteúdo com modelos de linguagem
- Interagir através de uma interface de chat
- Selecionar entre diferentes modelos disponíveis
- Utilizar o roteamento inteligente do Arcee Conductor

## Uso

```bash
# Ferramentas MCP (via SDK @sdk-mcp)
arcee mcp-tools listar
arcee mcp-tools buscar "processamento"
arcee mcp-tools detalhes tool1
arcee mcp-tools executar tool1 '{"param1": "valor1"}'

# Servidor TESS-MCP
arcee tess listar
arcee tess executar agent123 "Como posso otimizar este código?"

# API Arcee AI
arcee chat "Explique o conceito de Clean Architecture"
arcee generate "Escreva um código Python para ordenar uma lista"
```

## Garantindo Compatibilidade e Evolução

O projeto usa adaptadores para permitir evolução contínua enquanto mantém compatibilidade:

```python
# Código existente (continua funcionando)
from src.tools.mcpx_simple import MCPRunClient

client = MCPRunClient(session_id="abc123")
tools = client.get_tools()

# Código novo (implementação atual)
from infrastructure.mcp_client import MCPClient

client = MCPClient()
tools = client.list_tools()
```

Este modelo de adaptadores permite:
- Evolução gradual sem quebrar código existente
- Melhoria contínua da implementação interna
- Facilidade de migração para novos clientes

## Desenvolvimento

Para desenvolver novos recursos de integração:

1. Defina interfaces no domínio (`domain/interfaces`) que espelhem as APIs externas
2. Implemente serviços no domínio (`domain/services`) que orquestrem chamadas às APIs
3. Crie casos de uso na aplicação (`application/use_cases`) para funcionalidades específicas
4. Implemente clientes/adaptadores na infraestrutura (`infrastructure`) para comunicação com APIs externas
5. Exponha funcionalidades via CLI (`src/commands`)

Este fluxo de desenvolvimento garante uma separação clara entre o código do projeto e as APIs externas integradas.

## Novidades

### 25/03/2024

- **Novo script para interação direta com API TESS**: Implementado o script `tess_api_cli.py` que permite listar, consultar e executar agentes TESS diretamente, sem depender do comando `test_api_tess`.
- **Suporte a múltiplos modelos de linguagem**: O script suporta diversos modelos de linguagem, incluindo GPT-4o e Claude 3.5 Sonnet.
- **Exemplos práticos para geração de anúncios**: Adicionados exemplos para cafeteria, agência de marketing digital e corretora de seguros.
- **Documentação detalhada**: Criados guias de integração e dicas avançadas para otimização de anúncios Google Ads usando a API TESS.

### 20/03/2024

- **Interface Streamlit simplificada**: Removidas as abas superiores desnecessárias, mantendo apenas "Arcee CLI" e "Ajuda".
- **Correção de bugs no adaptador TESS**: Resolvidos problemas na comunicação com a API TESS. 