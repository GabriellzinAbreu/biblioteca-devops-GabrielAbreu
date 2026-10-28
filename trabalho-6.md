Trabalho 6 — Estratégia de CD, Deploy e Rollback

Disciplina: Cloud Computing e DevOps Avançado

1. Contexto e requisitos

Vamos utilizar o mesmo sistema de biblioteca da atividade anterior. Nele, os usuários podem consultar livros e realizar empréstimos.

O problema é que, no início do semestre, o sistema recebe mais acessos. Como uma nova atualização vai modificar a parte de empréstimos, precisamos pensar em uma forma de atualizar sem prejudicar quem estiver utilizando.

Para esse cenário, temos algumas condições:

- O sistema não pode ficar indisponível por mais de 5 minutos.
- O banco de dados será compartilhado.
- Precisamos acompanhar os erros e o tempo de resposta.
- Deve existir uma maneira de voltar à versão anterior caso aconteça alguma falha.

2. Fluxo de entrega

A ideia é utilizar o GitHub para controlar as versões e iniciar o processo de integração contínua.

Quando uma alteração for enviada, os testes automatizados deverão ser executados. Se forem aprovados, será gerado o artefato da nova versão, que passará por homologação antes de chegar à produção.

flowchart TD
    A[Commit no GitHub] --> B[Testes CI]
    B --> C{Passou?}
    C -->|Nao| D[Corrigir erros]
    C -->|Sim| E[Gerar artefato]
    E --> F[Homologacao]
    F --> G{Aprovado?}
    G -->|Nao| D
    G -->|Sim| H[Deploy na Green]
    H --> I[Smoke tests]
    I --> J{Tudo certo?}
    J -->|Nao| K[Manter Blue]
    J -->|Sim| L[Liberar nova versao]
    L --> M[Monitoramento]
    M --> N{Falha grave?}
    N -->|Sim| O[Rollback para Blue]
    N -->|Nao| P[Manter nova versao]

Se houver algum erro nos testes, a atualização será interrompida. A versão antiga continuará disponível enquanto o problema é corrigido.

3. Estratégia de deploy

Escolhemos o Blue/Green, pois a biblioteca precisa continuar funcionando durante a atualização.

Nesse modelo, o Blue será o ambiente com a versão 1.0.0, enquanto o Green receberá a versão 1.1.0.

Primeiro vamos preparar e testar a Green. Quando estiver funcionando corretamente, os acessos poderão ser direcionados para ela.

O principal motivo dessa escolha é a possibilidade de voltar para a versão antiga sem precisar reinstalar toda a aplicação. A desvantagem é ter que manter dois ambientes disponíveis, o que exige mais recursos.

4. Versionamento e promoção

Para identificar as atualizações, vamos utilizar os números de versão:

- 1.0.0: versão atual do sistema.
- 1.1.0: atualização com alterações nos empréstimos.

Cada versão terá uma tag no GitHub e ficará associada ao commit correspondente.

O artefato da versão 1.1.0 será gerado apenas uma vez. Depois, o mesmo arquivo passará por homologação e produção, evitando diferenças entre as versões utilizadas nos ambientes.

A liberação para produção dependerá dos testes e da autorização do responsável pela implantação. As configurações de cada ambiente serão mantidas separadamente do artefato.

5. Critérios de sucesso

Não basta o sistema abrir normalmente. Também precisamos conferir se ele continua funcionando bem depois da atualização.

Para isso, definimos alguns critérios:

- Os testes obrigatórios precisam passar.
- Consultas e empréstimos devem funcionar sem erros.
- Taxa de erros abaixo de 1%.
- Tempo de resposta de até 800 ms para pelo menos 95% das requisições.
- Indisponibilidade máxima de 5 minutos.

Os valores de 1% e 800 ms são limites que propomos para o exercício, já que não temos métricas reais da biblioteca para usar como referência.

Depois de liberar a versão, vamos acompanhar os erros, a latência e a quantidade de empréstimos concluídos. A ideia é observar principalmente os primeiros 15 minutos e comparar com o funcionamento anterior.

6. Smoke tests

Antes de direcionar os usuários para a nova versão, faremos algumas verificações básicas no ambiente Green:

1. Abrir o sistema e conferir se a página carrega.
2. Realizar o login.
3. Consultar os livros disponíveis.
4. Fazer um empréstimo de teste.
5. Verificar se o empréstimo foi registrado corretamente.

Como o banco é compartilhado, os testes deverão utilizar registros controlados, para não interferir nos empréstimos reais.

Se algum teste importante falhar, a Green não será liberada.

7. Plano de rollback

Mesmo com os testes, ainda existe a possibilidade de algum problema aparecer depois que os usuários começarem a utilizar a nova versão.

O rollback será necessário principalmente se a função de empréstimos parar de funcionar ou se o sistema ficar indisponível.

Também vamos considerar uma taxa de erros acima de 2% durante dois minutos como sinal de que a atualização precisa ser reavaliada. Em caso de falha crítica, não vamos esperar esse período: o retorno deverá começar imediatamente.

Como será feito

1. O responsável identifica a falha e decide pela reversão.
2. Os acessos são redirecionados para o ambiente Blue.
3. A equipe confere se o login, a consulta de livros e os empréstimos voltaram a funcionar.
4. O problema é registrado para análise e correção.

As configurações da versão anterior deverão permanecer disponíveis.

Cuidados com o banco de dados

Como as duas versões utilizam o mesmo banco, precisamos evitar mudanças que impeçam a versão antiga de funcionar.

Por exemplo, se a atualização adicionar um campo novo, podemos manter os campos antigos durante essa transição, em vez de removê-los imediatamente.

Também será necessário manter um backup verificado. Porém, não vamos restaurar o banco automaticamente durante o rollback, porque isso poderia apagar empréstimos realizados recentemente.

Tempo de recuperação

Nossa estimativa inicial será:

Etapa| Tempo
Identificar a falha| 1 minuto
Voltar os acessos para Blue| 2 minutos
Verificar o funcionamento| 1 minuto
Total| 4 minutos

Essa é uma previsão para o planejamento, não um tempo comprovado. Em uma implantação real, precisaríamos testar o procedimento para verificar se ele realmente atende ao limite de 5 minutos.

A equipe técnica e os funcionários da biblioteca também deverão ser informados caso ocorra uma falha ou reversão.

8. Riscos e prevenção

Durante a atualização, alguns problemas podem acontecer.

Erros na função de empréstimos: como essa parte será modificada, existe o risco de algum empréstimo não ser registrado corretamente. Vamos testar essa função antes de liberar a versão.

Incompatibilidade com o banco: se a atualização remover algum campo utilizado pela versão antiga, o rollback poderá falhar. Por isso, as alterações deverão manter a compatibilidade entre as versões.

Lentidão: o sistema pode apresentar problemas de desempenho, principalmente nos períodos de maior acesso. A equipe vai acompanhar a taxa de erros e o tempo de resposta.

Falha na troca dos ambientes: se o redirecionamento não funcionar, os usuários poderão ter dificuldades para acessar o sistema. O procedimento de troca e retorno deverá ser testado antes da implantação.

9. Evidências e responsáveis

Para saber se a implantação funcionou, vamos precisar conferir os resultados dos testes e as métricas do sistema.

As principais evidências previstas são:

- Resultado dos testes da CI.
- Resultado dos smoke tests.
- Logs com os erros encontrados.
- Registros de latência e taxa de erros.
- Identificação da versão implantada.
- Registro da aprovação ou do rollback, caso aconteça.

Quanto aos responsáveis, o desenvolvedor ficará encarregado das alterações e correções. Outra pessoa deverá verificar os testes, enquanto o responsável pela implantação vai autorizar a liberação, acompanhar as métricas e realizar o rollback se for necessário.

Com essas informações, a equipe poderá avaliar se a versão 1.1.0 está funcionando como esperado ou se será melhor manter a 1.0.0 até corrigir os problemas.
