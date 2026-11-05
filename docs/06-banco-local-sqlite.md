# 6. Construção e implementação do banco de dados local

## 6.1 Banco escolhido

O banco de dados local será o **SQLite**, executado no próprio dispositivo Android. No aplicativo Flutter, o acesso será feito pelo pacote `sqflite`, com o pacote `path` para compor o caminho do arquivo.

O banco local permitirá consultar e registrar os dados necessários às funcionalidades offline previstas. A sincronização com os demais dispositivos ocorrerá por meio de uma API quando houver conexão.

## 6.2 Arquivo e inicialização

O arquivo do banco terá o nome `sr_auto.db` e ficará no diretório privado de bancos de dados do aplicativo Android, sem acesso direto por outros aplicativos comuns.

Na primeira inicialização, a aplicação deverá:

1. Inicializar o mecanismo SQLite.
2. Obter o diretório privado de banco de dados do aplicativo.
3. Abrir ou criar `sr_auto.db`.
4. Definir a versão inicial do banco.
5. Habilitar as chaves estrangeiras com `PRAGMA foreign_keys = ON`.
6. Criar as tabelas e os índices necessários.
7. Disponibilizar uma camada de acesso aos dados para as telas e serviços do aplicativo.

## 6.3 Construção das tabelas

A criação será realizada por comandos SQL executados pelo `sqflite` durante a rotina `onCreate` da abertura do banco. Serão criadas as oito tabelas:

- `USUARIOS`: identificação e perfil de acesso.
- `CLIENTES`: dados de contato do cliente.
- `VEICULOS`: veículos vinculados aos clientes.
- `SERVICOS`: catálogo de serviços e preços de referência.
- `AGENDAMENTOS`: solicitações de horário.
- `ATENDIMENTOS`: registros dos serviços realizados.
- `ITENS_ATENDIMENTO`: serviços que compõem cada atendimento.
- `FILA_SINCRONIZACAO`: alterações aguardando envio à API.

As colunas, tipos SQLite, chaves e restrições estão detalhados em [05-modelo-er.md](05-modelo-er.md).

## 6.4 Tipos de dados e integridade

- Identificadores: `TEXT`, no formato UUID.
- Textos: `TEXT`.
- Datas e horários: `TEXT`, no padrão ISO-8601.
- Valores monetários: `INTEGER`, armazenados em centavos.
- Quantidades: `INTEGER`.
- Indicadores booleanos: `INTEGER`, usando 0 ou 1.

As tabelas utilizarão `PRIMARY KEY`, `FOREIGN KEY`, `NOT NULL`, `UNIQUE`, `DEFAULT` e `CHECK` conforme a regra de cada campo. As chaves estrangeiras deverão estar habilitadas em cada conexão.

## 6.5 Operações de dados (CRUD)

A camada de acesso ao banco disponibilizará operações para:

- **Create:** inserir clientes, veículos, serviços, agendamentos e atendimentos autorizados.
- **Read:** consultar cadastros, catálogo, agenda e histórico.
- **Update:** atualizar informações e estados dos registros.
- **Delete:** quando permitido, realizar exclusão lógica ou registrar a operação para sincronização, preservando o histórico necessário.

As operações serão chamadas pelos serviços da aplicação, evitando que as telas executem SQL diretamente.

## 6.6 Transações e migrações

Operações que envolvam mais de uma tabela serão executadas em transações. Por exemplo, ao registrar um atendimento, o aplicativo deverá inserir o atendimento e seus itens na mesma transação. Se uma etapa falhar, a operação será revertida para evitar dados incompletos.

Mudanças futuras no modelo serão realizadas por migrações versionadas, utilizando `onUpgrade` do `sqflite`. Cada versão deverá preservar os registros existentes e aplicar apenas as alterações necessárias.

## 6.7 Funcionamento offline

O catálogo de serviços e os dados necessários ao usuário autenticado poderão ser mantidos localmente para consulta sem internet. As operações offline permitidas serão gravadas no SQLite e identificadas com estado de sincronização pendente.

A confirmação definitiva de horários e outras operações que dependam da disponibilidade central da oficina ocorrerá após validação pela API. O aplicativo deverá informar ao usuário quando uma solicitação estiver pendente, confirmada ou apresentar erro de sincronização.

## 6.8 Sincronização com a API

A tabela `FILA_SINCRONIZACAO` registrará cada operação que precisa ser enviada, incluindo identificador, entidade, registro, tipo de operação, conteúdo, data de criação e quantidade de tentativas.

Quando a conexão estiver disponível:

1. O aplicativo identifica operações pendentes.
2. Envia as operações à API com autenticação.
3. A API valida permissões e regras de negócio.
4. A API retorna o resultado de cada operação.
5. O aplicativo marca como sincronizadas apenas as operações confirmadas.
6. Falhas temporárias permanecem na fila para nova tentativa.

Identificadores únicos e chaves de idempotência deverão reduzir duplicidades. Conflitos de atualização deverão ser sinalizados e tratados sem sobrescrever dados silenciosamente.

## 6.9 Segurança e privacidade

O banco ficará no diretório privado do aplicativo. Senhas não serão armazenadas em texto puro. Tokens de autenticação deverão ser guardados em armazenamento seguro do Android, por exemplo, com `flutter_secure_storage`, e não em tabelas comuns do SQLite.

O aplicativo deverá restringir consultas e alterações conforme o perfil do usuário. Dados pessoais serão utilizados apenas para as finalidades do sistema, respeitando a legislação de proteção de dados.

## 6.10 Escopo desta etapa

Nesta primeira etapa, a entrega apresenta a escolha do SQLite, o modelo ER, os tipos de dados, as restrições e o plano de implementação. A implementação efetiva do aplicativo e a integração operacional com a API serão realizadas nas etapas posteriores.
