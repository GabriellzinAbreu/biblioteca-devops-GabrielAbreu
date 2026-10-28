Atividade 1 — Plano de Deploy e Rollback

1. Cenário

O sistema de biblioteca é utilizado para consultar livros e realizar empréstimos. No início do semestre, o número de acessos aumenta, então é importante que as atualizações não deixem o sistema indisponível por mais de 5 minutos.

A nova versão terá alterações na parte de empréstimos. Por isso, é necessário escolher uma forma segura de atualizar o sistema e definir o que fazer caso aconteça algum problema.

2. Plano de Deploy

2.1. Artefato e versão

Vamos considerar que a versão atual do sistema é a 1.0.0 e a nova será a 1.1.0.

A nova versão será identificada pelo número da versão e pelo commit no GitHub, para facilitar o controle das alterações.

2.2. Ambientes

Serão utilizados três ambientes:

- Desenvolvimento: onde as alterações são feitas.
- Homologação: onde a nova versão será testada.
- Produção: onde o sistema fica disponível para os usuários.

2.3. Estratégia escolhida

Escolhemos a estratégia Blue/Green, pois ela permite manter a versão antiga funcionando enquanto a nova é preparada.

Assim, podemos testar a nova versão antes de liberar o acesso dos usuários. Caso apresente algum problema, será possível voltar para a versão anterior.

2.4. Pré-condições

Antes da implantação, precisamos verificar se:

- Os testes da nova versão foram aprovados.
- Existe um backup do banco de dados.
- A nova versão é compatível com o banco compartilhado.
- O ambiente da nova versão está funcionando corretamente.

2.5. Passos de implantação

1. Preparar a versão 1.1.0.
2. Realizar os testes no ambiente de homologação.
3. Implantar a nova versão no ambiente Green.
4. Verificar se as funções principais estão funcionando.
5. Direcionar os usuários para a nova versão.
6. Acompanhar os erros e o tempo de resposta após a atualização.

2.6. Smoke tests

Após a implantação, serão verificados:

- Acesso ao sistema.
- Consulta de livros.
- Realização de empréstimos.
- Registro correto dos empréstimos no banco.

Esses testes ajudam a verificar rapidamente se as principais funcionalidades continuam funcionando.

2.7. Critérios de promoção

A versão só será liberada se os testes principais forem aprovados e não forem encontrados problemas graves.

Como critérios para este planejamento, definimos uma taxa de erros abaixo de 1% e um tempo de resposta de até 800 ms para pelo menos 95% das requisições.

2.8. Responsáveis

O desenvolvedor ficará responsável pelas alterações e correções. A pessoa responsável pelos testes verificará as funcionalidades, e o responsável pela implantação acompanhará a atualização e o funcionamento do sistema.

3. Plano de Rollback

3.1. Quando realizar o rollback

O rollback será realizado caso a nova versão apresente falhas importantes, principalmente na função de empréstimos.

Também poderá ser iniciado se a taxa de erros ultrapassar 2% durante 2 minutos consecutivos ou se houver lentidão que prejudique o uso do sistema.

3.2. Como voltar à versão anterior

Como a estratégia escolhida é Blue/Green, o tráfego será direcionado novamente para o ambiente Blue, que continuará com a versão 1.0.0.

Depois disso, serão verificadas as funções principais para confirmar que o sistema voltou a funcionar.

3.3. Configurações

As configurações da versão anterior serão mantidas para permitir a reversão, caso seja necessário.

3.4. Banco de dados

Como o banco de dados é compartilhado, as alterações deverão ser compatíveis com as duas versões.

Também será mantido um backup. Porém, a restauração do banco só deverá acontecer se realmente for necessária, pois pode causar perda de informações registradas recentemente.

3.5. Tempo de recuperação

A meta será recuperar o sistema em até 5 minutos, respeitando o limite de indisponibilidade informado no cenário.

Esse tempo deverá ser verificado em testes antes de uma implantação real.

3.6. Comunicação

Se ocorrer algum problema, a equipe técnica e os funcionários da biblioteca serão informados sobre a falha e a necessidade de voltar à versão anterior.

3.7. Registro de evidências

Serão registrados o horário do problema, a versão utilizada, os erros encontrados, a decisão de rollback e o resultado das verificações após a recuperação.

4. Conclusão

A estratégia Blue/Green foi escolhida porque permite atualizar o sistema com menor risco de interrupção. Com os testes e o plano de rollback, a equipe terá uma forma de identificar problemas e recuperar o funcionamento da biblioteca caso a nova versão apresente falhas.
