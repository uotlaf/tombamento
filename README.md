# 🏢 Sistema de Tombamento Patrimonial

Este projeto consiste em uma plataforma web robusta e centralizada para o **gerenciamento, rastreamento e controle do ciclo de vida de bens patrimoniais (tombamentos)** de um setor, instituição ou empresa [110]. O sistema substitui controles manuais ou planilhas inconsistentes, fornecendo a cada bem uma identidade digital que associa sua comprovação visual (foto), localização física, responsável atual e um histórico cronológico de movimentações auditável e imutável [110].

---

## 🚀 Faseamento do Projeto

O desenvolvimento do sistema segue um plano estruturado em seis fases essenciais [110]:
1. **Fase 1: Concepção e Visão do Projeto** — Definição do problema de negócio e do domínio de Gestão de Ativos Físicos [110, 111].
2. **Fase 2: Levantamento e Modelagem Teórica** — Especificação detalhada de Requisitos Funcionais e Não Funcionais [111, 112].
3. **Fase 3: Arquitetura e Modelagem Lógica** — Planejamento da arquitetura orientada a serviços e do Modelo Entidade-Relacionamento [113, 114].
4. **Fase 4: Modelagem Física e Infraestrutura** — Criação do repositório Git, migrations físicas do banco de dados (Flyway) e estruturação do projeto [114, 115].
5. **Fase 5: Execução e Desenvolvimento** — Desenvolvimento incremental do Setup/Security, cadastros básicos, upload de mídias e regras de movimentação imutável [115, 116].
6. **Fase 6: Testes, Homologação e Deploy** — Implantação de testes automatizados sistemáticos e publicação em nuvem (AWS/Render/etc.) [116, 117].

---

## 📐 Escolhas Arquiteturais & Tecnológicas

O sistema foi projetado sob o padrão de **Arquitetura em Camadas** para garantir divisão estrita de responsabilidades, modularidade e facilidade de manutenção [6, 8, 96]:

### 💻 Stack Tecnológico
*   **Frontend**: React.js desenvolvido utilizando a IDE **VS Code** [23, 113].
*   **Backend**: Java com **Spring Boot** desenvolvido utilizando a IDE **IntelliJ IDEA** (relação otimizada de indexação do ecossistema Spring) [18, 23, 113].
*   **Banco de Dados**: **PostgreSQL** para ambiente de produção/desenvolvimento [33, 113] e **H2 Database** em memória para ambiente de testes isolados [14, 54, 115].
*   **Versionamento do Banco**: **Flyway** para gerenciar a evolução física do schema por meio de *migrations* versionadas no código [115].

### 🏛️ Padrão de Comunicação (RESTful)
A comunicação é totalmente desacoplada no padrão **Cliente-Servidor (Request/Reply)** através de uma **API RESTful** [3, 11, 113]:
*   **POST** (`201 Created`): Criação de recursos (ex: cadastro de bens, novos usuários) [14, 16, 73].
*   **GET** (`200 OK`): Consulta de recursos (ex: busca rápida de bens por QR Code) [11, 14].
*   **PUT** (`200 OK` / `204 No Content`): Atualização total/parcial de dados cadastrais [14, 16, 79].
*   **DELETE** (`204 No Content`): Remoção de cadastros básicos que não contenham vínculos de segurança ou histórico patrimonial [14, 16, 80].

---

## 💾 Modelo Conceitual e de Integridade

O modelo de dados físico e lógico do banco de dados (ilustrado em detalhes no artefato de mídia `modelo-conceitual-patrimonio.png`) foi projetado sob rigorosas restrições relacionais:

1.  **Integridade de Entidade (Primary Keys)**: Todas as tabelas principais possuem chaves primárias numéricas autoincrementadas (`id_usuario`, `id_localizacao`, `id_bem`, `id_movimentacao`) [34, 107, 108].
2.  **Integridade Referencial (Foreign Keys)**: A tabela associativa central `movimentacao_historico` vincula as tabelas de bens, usuários (origem e destino) e localizações de forma estrita, impedindo registros órfãos [108, 114].
3.  **Integridade de Domínio e de Check (CK)**: Validações rígidas no PostgreSQL, tais como `CHECK (valor >= 0)` para bens e tipagens atômicas (`NUMERIC(12,2)` para dados monetários precisos e `DATE` para aquisições) [35, 102].
4.  **Generalização e Especialização (Herança JPA)**:
    *   `BEM_PATRIMONIAL` (Superclasse): Contém os dados genéricos dos ativos patrimoniais (tombamento, descrição, valor, status, foto_url).
    *   `EQUIPAMENTO_TI` (Subclasse / Especialização): Adiciona endereço MAC, IP e tipo do dispositivo tecnológico.
    *   `MOBILIARIO` (Subclasse / Especialização): Adiciona atributos de dimensões e material.
    *   *Mapeamento*: Implementado via estratégia `@Inheritance(strategy = InheritanceType.JOINED)`.
5.  **Armazenamento de Mídia Otimizado (Sem Inchaço de Tabelas)**: As imagens capturadas por celulares durante o inventário são enviadas para um servidor de mídias especializado (S3 ou volume físico) [116]. O PostgreSQL armazena exclusivamente o caminho lógico (`foto_url` como String), blindando o banco contra lentidão por *table bloat* (evitando campos binários `bytea` pesados).
6.  **Desnormalização Controlada (Performance)**: A tabela `localizacao` reúne prédio, setor/andar e sala em uma estrutura física unificada, evitando JOINs excessivos nas chamadas frequentes da API REST.

---

## 📂 Estrutura do Projeto Backend (Java)

```text
com.seusetor.patrimonio/
│
├── 📂 config           # Configurações do Spring Security, Filtros JWT e CORS [91, 92, 93]
├── 📂 controller       # Endpoints REST (UsuarioController, BemController) que expõem os serviços [71, 72]
├── 📂 model/
│   ├── 📂 dto          # Objetos de Transferência de Dados (UsuarioDTO, BemDTO) [61, 73, 74]
│   ├── 📂 entidades    # Classes mapeadas do JPA com anotações de persistência e Lombok [36, 42, 43]
│   └── 📂 repositorio  # Interfaces de persistência herdadas do JpaRepository (Spring Data JPA) [45, 46]
│
├── 📂 service          # Camada com regras de negócio, validações rigorosas e controle transacional [57, 59, 60]
│   └── 📂 exceptions   # Modelo de exceções de domínio customizadas (RegraNegocioRunTime) [58, 59]
│
└── 📝 PatrimonioApplication.java # Classe principal executável (Spring Boot standalone) [18, 24, 25]
```

---

## 🔒 Segurança (Spring Security e JWT)

O backend possui uma arquitetura de segurança stateless para bloquear acessos não autorizados à API [87, 88]:
*   **BCryptPasswordEncoder**: As senhas de acesso são criptografadas por hashing de via única com adição de ruído (sal) antes de serem gravadas no banco de dados [90].
*   **Filtro de Autenticação (JWT Generator)**: Intercepta a rota `/login` (pública) e, após validar as credenciais, gera um token assinado digitalmente (Header.Payload.Signature) e o anexa ao cabeçalho da resposta (`token` no Header) [88, 92, 93, 94].
*   **Filtro de Autorização (JWT Filter)**: Analisa todas as requisições subsequentes para as rotas privadas da API, extraindo o token do cabeçalho HTTP e validando sua autenticidade antes de liberar a requisição [92, 94].

---

## 🧪 Estratégia Sistemática de Testes Automatizados

A confiabilidade operacional das regras de negócio patrimoniais baseia-se em testes automatizados repetíveis e independentes do banco produtivo [53, 54, 95]:

### 1. Testes de Unidade na Camada Service (com JUnit & Mockito)
*   **Localização**: `src/test/java/com/seusetor/patrimonio/service/`
*   Isolam a lógica de negócio simulando o comportamento do banco por meio de mocks do repositório [83, 84].
*   *Testes Obrigatórios*:
    *   Tentativa de salvar um ativo sem número de tombamento (deve gerar erro de validação) [68, 69].
    *   Tentativa de cadastrar um número de tombamento já existente no sistema (deve lançar `RegraNegocioRunTime`) [59, 69].
    *   Transferência de custódia de bem sem justificativa formal ou sem os dados do responsável destino [59, 62].

### 2. Testes de Integração na Camada Repository (com Banco H2)
*   **Localização**: `src/test/java/com/seusetor/patrimonio/model/repositorio/`
*   Verificam a consistência das restrições de chaves primárias, restrições únicas (`tombamento_unique`) e de herança (`JOINED`) usando o banco em memória H2 com o perfil específico de teste (`@ActiveProfiles("test")`) [51, 54, 55].

### 3. Testes nos Controllers (com MockMvc)
*   **Localização**: `src/test/java/com/seusetor/patrimonio/controller/`
*   Simulam requisições HTTP (GET por tombamento, POST para novos bens) e avaliam as respostas do controlador (Status HTTP `201 Created` ou `400 Bad Request`) sem levantar fisicamente o servidor web [83, 84, 85].

---

## 🔧 Como Executar a Aplicação

### Pré-requisitos
*   Java JDK 17 ou superior.
*   Maven 3.8+ instalado.
*   PostgreSQL ativo localmente ou via container Docker.

### 1. Configurando as Variáveis de Ambiente
Crie ou edite o arquivo `src/main/resources/application.properties` para apontar para sua base de dados PostgreSQL ativa [40]:
```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/patrimonio
spring.datasource.username=seu_usuario
spring.datasource.password=sua_senha
spring.datasource.driver-class-name=org.postgresql.Driver
```

### 2. Executando as Migrations do Banco de Dados
Ao iniciar o Spring Boot, o Flyway lerá o script em `src/main/resources/db/migration/V1__criar_tabelas_patrimonio.sql` e aplicará a modelagem relacional de forma automática no PostgreSQL [115].

### 3. Rodando o Servidor de Backend
Na raiz do projeto java, execute o comando Maven para levantar a aplicação com o container Tomcat embutido na porta `8080` [18, 22]:
```bash
mvn spring-boot:run
```

### 4. Executando a Suíte de Testes Automatizados
Para rodar toda a suíte de testes de serviços, repositórios e endpoints REST, execute [53, 55]:
```bash
mvn test
```
