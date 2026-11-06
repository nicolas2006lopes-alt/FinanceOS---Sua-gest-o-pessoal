# FinanceOS---Sua-gestão-pessoal
Meu primeiro siestema de gestão financeira
# Rigo Finance — Personal Edition V1.0

Aplicativo Next.js, React e TypeScript com Better Auth, MFA TOTP e persistência criptografada em SQLite ou PostgreSQL. Contas, receitas, despesas, transferências, cartões, investimentos e metas são gerenciados manualmente. Não há conexão bancária ou dependência da Pluggy.

## Executar

Requisitos: Node.js 24.15+ e pnpm 11.25.

```sh
pnpm install --frozen-lockfile
pnpm setup
pnpm db:migrate
pnpm dev
```

Abra http://localhost:3000. O setup preserva valores existentes de `.env.local`. Não altere a chave de criptografia de uma base existente. A migração cria backup SQLite antes de modificar documentos. Para PostgreSQL, faça e valide um `pg_dump` antes de executar o script com `--backup-confirmed`.

A rota `/` é pessoal: entre ou crie sua conta. Novos usuários começam sem dados fictícios e recebem um assistente opcional de configuração. Nome, orçamento, primeira conta e meta são salvos no servidor. A demonstração fica isolada em `/demo/mobile`, com exemplos somente em memória.

## Regras financeiras

- Valores em centavos inteiros. Saldo atual = saldo inicial + movimentações realizadas até hoje. Editar ou excluir uma movimentação recalcula o saldo, sem aplicar incrementos repetidos.
- Lançamentos marcados como previstos não alteram os saldos nem os totais realizados. Os realizados com data futura entram no mês selecionado e na previsão, mas não alteram o saldo disponível antes da data. Marcar como realizado não muda a data do lançamento.
- Transferências criam duas pontas atômicas em contas diferentes e não contam como receita ou despesa. A revisão de importações sugere pares de mesmo valor e mesma data; nenhuma data é alterada.
- Compras no cartão afetam despesas e dívida, sem debitar a conta imediatamente. Cada parcela tem o valor informado no formulário; o campo é **valor de cada parcela**, não o valor total. Meses curtos usam o último dia válido.
- Pagamento de fatura debita a conta, reduz a dívida na data registrada e não duplica despesas. Pagamentos parciais são aceitos; valores acima do saldo da fatura são rejeitados. Informe o mês de referência correto. Fechamento e vencimento aceitam dias 1 a 28.
- Parcelas futuras comprometem o limite. Faturas não calculam juros rotativos, multas ou conciliação bancária.
- A marca de recorrência é informativa; não gera lançamentos automaticamente. Investimentos usam valores informados, sem cotações externas. Metas não somam seu acumulado novamente ao patrimônio.
- Aportes e resgates movimentam a conta selecionada e o investimento numa única gravação, sem virar receita ou despesa. O cadastro inicial e a atualização de valor de mercado apenas registram a posição; não movimentam contas. Para dinheiro saindo de uma conta já cadastrada, use **Aportar / resgatar**. Investimentos com histórico não podem ser excluídos pela interface.
- O orçamento mensal geral e os limites por categoria usam as despesas realizadas do período. BRL é a moeda suportada nesta edição.
- Contas com histórico podem ser arquivadas. Exclusão de conta/cartão com movimentações é bloqueada para preservar vínculos.

## Recursos preservados

Login, cadastro, MFA, isolamento por usuário, exportação JSON, importação manual CSV/OFX com prévia e deduplicação, filtros, relatórios, orçamentos, investimentos, metas e assistente local. A IA remota opcional continua exigindo chave própria e consentimento explícito; não faz parte da integração bancária removida.

CSV UTF-8: cabeçalhos `data,descricao,valor,id`, vírgula ou ponto e vírgula; valores negativos indicam saídas. Limites: 2 MB e 5.000 linhas. Exemplos em `examples/`. Revise categorias, pagamentos e transferências após importar.

## Arquitetura e segurança

`src/lib/finance.ts` concentra validação e cálculos; `src/lib/store.ts` mantém armazenamento AES-256-GCM com identidade do usuário como AAD e versão otimista. `/api/data` valida sessão, origem, relações e regras financeiras. Conflitos retornam 409; recarregue os dados antes de repetir. A autenticação normal e MFA permanecem em `src/lib/auth.ts`.

A migração preserva saldos anteriores calculando o saldo inicial equivalente. Documentos originais são arquivados criptografados; exportação disponível em Configurações. Quando existem dados originais e Sandbox, os originais têm prioridade e ambos ficam preservados. Tabelas bancárias históricas não são apagadas. Os antigos endpoints bancários retornam 410 e não chamam provedores.

`.env.local` e `data/` são ignorados pelo Git. `.env.example` não contém segredos. Backups precisam ser protegidos e conservados junto à chave de criptografia, sem publicá-la.

## Validação

```sh
pnpm typecheck
pnpm test
# Servidor local iniciado em outro terminal:
pnpm test:api
pnpm build
```

Não existe comando de lint configurado. Os testes HTTP criam usuários descartáveis `@example.test`, verificam autenticação, MFA, CSRF, isolamento e persistência e limpam os próprios dados ao final.

## Produção

Use PostgreSQL, HTTPS, backups testados e variáveis protegidas para `DATABASE_URL`, `BETTER_AUTH_URL`, `BETTER_AUTH_SECRET` e `DATA_ENCRYPTION_KEY`. Não use `NEXT_PUBLIC_` para segredos. O PostgreSQL precisa ser validado no ambiente de implantação; esta entrega valida SQLite local. Não há migração automática entre SQLite e PostgreSQL.

Na Netlify, publique o projeto Next.js completo com `netlify.toml`, não a pasta `netlify-preview` ou o ZIP de demonstração. O servidor recusa iniciar na Netlify sem PostgreSQL para evitar perda de dados em disco temporário. Veja [PERSONAL-EDITION.md](docs/PERSONAL-EDITION.md) para configuração, arquivos modificados, validação e pendências de produção.

```sh
node node_modules/tsx/dist/cli.mjs scripts/migrate.ts --backup-confirmed
pnpm build
pnpm start
```

`pnpm start` escuta em loopback; use proxy HTTPS em produção. Verificação/recuperação de e-mail, políticas de privacidade e operações LGPD requerem configuração de implantação. Consulte `docs/MANUAL-MIGRATION.md` para entrega, backups e resultados dos testes. Documentos Open Finance existentes são históricos e não devem ser usados para configurar esta versão.

## Showcase mobile executivo

Acesse `http://localhost:3000/demo/mobile` para a apresentação interativa do Rigo. Quatro viewports, tema claro/escuro, modo apresentação e exemplos isolados em memória. A rota principal continua em `/`. Detalhes e testes em [MOBILE-SHOWCASE.md](docs/MOBILE-SHOWCASE.md).
