# Representação Baseada em Grafos de Infraestrutura como Código: Possibilitando o Raciocínio Semântico para Sistemas Conteinerizados

Este artigo propõe um novo framework que integra um Grafo de Conhecimento baseado em ontologia com o Model Context Protocol (MCP) para o gerenciamento semântico de infraestruturas conteinerizadas. A estrutura aborda uma lacuna nas plataformas de orquestração ao fornecer uma representação semântica unificada da topologia, alimentada por processo de Extração, Transformação e Carga (ETL) que analisa arquivos do Docker Compose e disponibiliza o conhecimento para Large Language Models via uma camada de ferramentas. Essa abordagem permite que usuários realizem consultas em linguagem natural para obter insights profundos sobre a configuração da infraestrutura. A abordagem foi validada por meio de um experimento controlado simulando um cenário DevOps, no qual o sistema identificou com sucesso falhas de integridade em volumes e riscos de movimentação lateral — problemas que a análise tradicional dificilmente detectaria. Os resultados demonstram a superioridade da estrutura na auditoria automatizada e na detecção de conflitos, preenchendo a lacuna entre artefatos brutos de Infraestrutura como Código e o raciocínio dinâmico do sistema para um gerenciamento de contêineres mais inteligente e explicável.

---

# Estrutura do README.md

Este README.md está organizado da seguinte forma:

- **Selos Considerados**: Descrição dos selos pretendidos.
- **Informações Básicas**: Requisitos de hardware e software.
- **Dependências**: Lista de dependências e versões.
- **Preocupações com Segurança**: Riscos e medidas de segurança.
- **Instalação**: Passos para baixar e instalar o artefato.
- **Teste Mínimo**: Exemplo básico de execução.
- **Experimentos**: Detalhes para reproduzir as reivindicações do artigo.
- **LICENSE**: Licença do projeto.

## Estrutura do Projeto

```text
├── example.mkv        # Vídeo demonstrativo dos cenários propostos
├── docker-compose.yaml    # Arquivo de configuração do Docker Compose para iniciar o banco de dados Neo4j
├── LICENSE                # Arquivo de licença do projeto (MIT)
├── README.md              # Este arquivo, contendo instruções e documentação
├── requirements.txt       # Lista de dependências Python necessárias
├── .env_example           # Exemplo de arquivo de configuração de ambiente
├── mcp_server/
│   └── base_ontology.ttl  # Arquivo de ontologia em formato TTL, define as classes e relações para o grafo de conhecimento da infraestrutura Docker, utilizado implicitamente na construção do parser.
├── parser_neo4j/
│   └── parser.py          # Script do parser que processa os arquivos Docker Compose e constrói o grafo
│   └── docker_composes/
│        ├── case_1.yaml   # Cenário A: Análise de Impacto em Cadeia
│        ├── case_2.yaml   # Cenário B: Integridade de Volumes
│        ├── case_3.yaml   # Cenário C: Detecção de Movimentação Lateral
│        └── case_4.yaml   # Cenário D: Conflito de Portas
```

# Selos Considerados

Os autores julgam como considerados no processo de avaliação os selos:

- Artefatos Disponíveis (SeloD)
- Artefatos Funcionais (SeloF)
- Artefatos Sustentáveis (SeloS)
- Experimentos Reprodutíveis (SeloR)

Com base nos códigos e documentação disponibilizados neste e nos repositórios relacionados.

---

# Informações Básicas

O artefato requer um ambiente com Docker e Docker Compose para executar o banco de dados Neo4j, Python 3.9+ para os scripts, e um cliente MCP compatível (como Claude Desktop) para interação. Não há requisitos específicos de hardware além dos necessários para executar containers Docker e um LLM localmente.

---

# Dependências

- **Docker**: Plataforma de contêinerização para executar o banco de dados Neo4j.
- **Python 3.9+**: Linguagem de programação utilizada.
- **Neo4j 5.20.0**: Banco de dados de grafos.
- **Bibliotecas Python** (listadas em requirements.txt):
  - neo4j: 6.0.3
  - python-dotenv: 1.2.1
  - PyYAML: 6.0.3
  - mcp: 1.22.0

---

# Preocupações com Segurança

A execução do artefato envolve a criação de um container Neo4j com autenticação básica. A senha é definida via variável de ambiente NEO4J_PASSWORD no arquivo .env. Recomenda-se usar uma senha forte e não reutilizar em ambientes de produção. O container Neo4j é executado localmente e não expõe portas adicionais além das necessárias (7474 e 7687). Não há riscos de execução remota ou acesso não autorizado se seguido as instruções.

---

# Instalação

Siga as instruções abaixo para instanciar o grafo e popular a base de dados com os artefatos avaliados no artigo:

**1. Clone o repositório e acesse a pasta do projeto:**

```bash
git clone https://github.com/ComputerNetworks-UFRGS/compose-kg-mcp.git
cd compose-kg-mcp
```

**2. Configure as Variáveis de Ambiente:**
Renomeie o arquivo .env_example para .env na raiz do projeto e defina a senha do Neo4j.

```
cp .env_example .env
# Edite .env para definir NEO4J_PASSWORD
```

**3. Inicialize o Banco de Dados Neo4j:**

```
docker compose up -d
```

**4. Instale as Dependências Python:**
Recomendamos o uso de um ambiente virtual para não gerar conflitos.

```
python -m venv venv
source venv/bin/activate  # Em windows utilize: venv\Scripts\activate
pip install -r requirements.txt
```

**5. Execute a Ingestão do Grafo (ETL):**
Esta etapa processa os arquivos em `parser_neo4j/docker_composes/` (Cenários 1, 2, 3 e 4) e constrói o grafo.

```
python parser_neo4j/parser.py
```

_Uma mensagem de "Processamento concluído!" indicará que a base de conhecimento está pronta._

---

# Teste Mínimo

O teste mínimo proposto para validar a integração se dá via a conexão do servidor MCP com o Claude Desktop do avaliador.
Atualmente o Claude Desktop só está disponível para Windows e MacOS. Porém, há diversos clientes compatíveis, podendo ser acessado em: https://modelcontextprotocol.io/clients.

Para o Claude, abra as configurações do Claude Desktop e adicione seu servidor MCP, substituíndo `CAMINHO/ABSOLUTO/PARA/` pelo local exato onde você clonou este repositório:

```
# Sem python virtual environment
{
  "mcpServers": {
    "docker-infra-manager": {
      "command": "python",
      "args": [
        "/CAMINHO/ABSOLUTO/PARA/compose-kg-mcp/mcp_server/server.py"
      ]
    }
  }
}

# Com python virtual environment - linux
{
  "mcpServers": {
    "docker-infra-manager": {
      "command": "bash",
      "args": [
        "-c",
        "cd /CAMINHO/ABSOLUTO/PARA/compose-kg-mcp && source venv/bin/activate && python mcp_server/server.py"
      ]
    }
  }
}

# Com python virtual environment - windows
{
  "mcpServers": {
    "docker-infra-manager": {
      "command": "CAMINHO\\ABSOLUTO\\PARA\\compose-kg-mcp\\venv\\Scripts\\python.exe",
      "args": [
        "CAMINHO\\ABSOLUTO\\PARA\\compose-kg-mcp\\mcp_server\\server.py"
      ]
    }
  }
}
```

Reinicie o Claude Desktop e em um novo chat, verifique se o MCP foi ativo e se as ferramentas estão prontas para uso. Execute um prompt simples como "Liste os serviços disponíveis no grafo" para confirmar a conectividade.

---

# Experimentos

No artigo, apresentamos 4 casos de estudo para validar as reivindicações. Cada subseção abaixo detalha a reivindicação, arquivos de configuração, comandos, tempo estimado, recursos esperados e resultado esperado.

## Cenário A: Análise de Impacto em Cadeia

**Arquivo de Configuração:** `parser_neo4j/docker_composes/case_1.yaml`

**Ferramenta MCP Utilizada:** find_impact_analysis, get_service_details

Prompt: _'Utilize suas ferramentas MCP para fazer uma análise de impacto em cadeia. Se o serviço 'ecommerce-db' falhar, quais outros serviços da infraestrutura serão afetados direta ou indiretamente?'_

**Tempo Estimado:** 1 minuto.

**Resultado Esperado:** O agente deve rastrear a cadeia completa de dependências e identificar que, além do `inventory-api` (dependente direto), o serviço final de frontend `storefront-ui` será severamente impactado pela falha do banco de dados.

## Cenário B: Integridade de Volumes

**Arquivo de Configuração:** `parser_neo4j/docker_composes/case_2.yaml`

**Ferramenta MCP Utilizada:** check_volume_conflicts, get_service_details

Prompt: _'Faça uma varredura na infraestrutura usando as ferramentas e me informe se existe algum conflito crítico de volume, ou seja, múltiplos serviços gravando no mesmo Host Path.'_

**Tempo Estimado:** 1 minuto.

**Resultado Esperado:** O agente deve alertar preventivamente para uma condição crítica de corrupção de dados (colisão de `HostPath`). Ele identificará que dois projetos distintos estão configurados erroneamente para montar o mesmo diretório local simultaneamente para a persistência de dados.

## Cenário C: Detecção de Movimentação Lateral

**Arquivo de Configuração:** `parser_neo4j/docker_composes/case_3.yaml`

**Ferramenta MCP Utilizada:** analyze_network_reachability, get_service_details

Prompt: _'Execute uma análise de alcançabilidade de rede (network reachability) entre o 'load-balancer' e o 'vault-database'. Existe um caminho possível de ataque lateral entre eles?'_

**Tempo Estimado:** 1 minuto.

**Resultado Esperado:** O sistema deve detetar uma violação da política de segmentação de rede. O agente traçará o caminho de transição, revelando que um atacante que comprometa o `load-balancer` (na zona pública) poderia alcançar o `vault-database` (na zona segura) utilizando o serviço `legacy-auth-service` como ponto de pivô.

## Cenário D: Conflito de Portas

**Arquivo de Configuração:** `parser_neo4j/docker_composes/case_4.yaml`

**Ferramenta MCP Utilizada:** check_port_conflicts, get_service_details

Prompt: _''Audite todas as portas da infraestrutura. Existe algum conflito de portas instanciadas no host? Se sim, quais serviços estão em colisão?_

**Tempo Estimado:** 1 minuto.

**Resultado Esperado:** O agente consultará o grafo global e identificará que dois nós de serviços distintos convergem para a mesma porta no host. O sistema alertará sobre a impossibilidade física desta implantação simultânea.

---

# LICENSE

Este projeto está licenciado sob a Licença MIT - consulte o arquivo [LICENSE](LICENSE) para obter mais detalhes.
