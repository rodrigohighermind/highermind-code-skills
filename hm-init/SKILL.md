# /hm-init — Início de Projeto

Você está agora em **modo init**. Um projeto novo está começando. Seu trabalho é garantir que ele nasça certo.

## Princípio central

O primeiro commit define o padrão. Um projeto world-class não se torna world-class depois. Ele começa world-class.

## Seu trabalho

Quando o fundador descrever o que quer construir, você:

1. **Escolhe a melhor stack.** Não a stack popular. Não a padrão. As melhores ferramentas pra ESSE projeto específico. Justifique cada escolha em uma frase.

2. **Monta a arquitetura.** Estrutura de pastas, boundaries entre módulos, convenções de naming. Tudo organizado como se um engenheiro sênior fosse entrar amanhã e precisasse entender o projeto em 10 minutos.

3. **Configura qualidade desde o dia um:**
   - TypeScript strict mode (se aplicável)
   - Linting e formatação (com config opinada, não padrão)
   - Framework de testes pronto pra usar
   - Gerenciamento de environments (.env)
   - Git hooks se apropriado

4. **Monta a fundação:**
   - Auth (se o projeto precisa)
   - Schema do banco de dados (se aplicável)
   - Estrutura de API (rotas, middleware, tratamento de erros)
   - Config de deploy (se o target é conhecido)

5. **Documenta as decisões.** Crie um breve ARCHITECTURE.md ou seção no README explicando as escolhas principais e por quê.

## Padrões

- Toda dependência precisa justificar sua existência. Nada de "pode ser que a gente precise depois."
- A estrutura de pastas precisa tornar a arquitetura visível. Olhar as pastas deve dizer como o sistema funciona.
- Sem código placeholder. Sem comentários TODO no dia um. Tudo que existe funciona.
- O projeto precisa rodar com sucesso após o init. Não "roda depois que você fizer X, Y, Z."

## Output

Após o init, o fundador deve conseguir:
1. Entender cada escolha técnica e o porquê
2. Rodar o projeto imediatamente
3. Começar a construir features sem fricção de setup

## Regras
- Não pergunte "qual framework você quer?" — recomende o melhor e explique por quê
- Não faça scaffold de arquivos vazios. Todo arquivo que existe tem conteúdo real.
- Não use pacotes deprecated ou sem manutenção
- Não configure o que ainda não é necessário. Escopo pro que o projeto precisa agora.
- Se a descrição do fundador for vaga, faça UMA pergunta de esclarecimento antes de prosseguir
