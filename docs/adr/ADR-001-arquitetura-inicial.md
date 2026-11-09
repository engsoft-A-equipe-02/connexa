# ADR-001 — Arquitetura inicial: monólito modular com um schema por módulo

- **Situação:** aceita
- **Data:** 09-11-2026
- **Participantes:** Guilherme Caetano e Eduarda Gadanha Guerra

## Contexto

O Connexa precisa de uma arquitetura inicial para a Sprint 1. A equipe é pequena
(duas pessoas), o prazo é o de uma disciplina e o projeto parte do repositório
modelo disponibilizado pelo professor (connexa-modelo-dotnet). O modelo já
organiza o back-end em módulos (Identidade, Perfil, Materiais, Grupos,
Acompanhamento, Encontros, Painel), um projeto comum, uma API e testes, com
PostgreSQL como banco e front-end em pasta separada.

## Decisão

Adotamos o monólito modular em .NET 10 definido pelo modelo, com um único
deploy da API, módulos com fronteiras explícitas, um schema de banco por módulo
no PostgreSQL e o front-end em projeto separado.

## Alternativas consideradas
| Alternativa | Por que não foi escolhida |
|---|---|
| Microsserviços | Os limites entre os domínios do Connexa ainda não foram validados pelo uso; separá-los agora em serviços implantáveis cristaliza fronteiras que podem estar erradas. Também introduziria consistência eventual, comunicação de rede, observabilidade distribuída e múltiplos pipelines, custos que não se justificam sem requisitos de escala ou de implantação independente. |
| Monólito em camadas, sem módulos | Organizar por camada técnica deixa as regras de negócio dispersas e permite dependências livres entre áreas do sistema. A modularização por domínio, com fronteiras verificadas por teste, mantém a coesão e preserva a opção de extrair um módulo para serviço no futuro. |
| Schema único compartilhado no banco | Um schema comum permite que um módulo leia ou altere as tabelas de outro, contornando as fronteiras definidas no código. Um schema por módulo torna a propriedade dos dados explícita e verificável por teste. |
| Partir do zero, sem o repositório modelo | Estrutura, testes de fronteira, CI e instruções do agente já estão alinhados e validados no modelo. Reimplementá-los traria risco de inconsistência e não agregaria valor ao produto. |

## Consequências

- **Positivas:** um só deploy e uma só base para operar; fronteiras entre módulos
  verificadas por teste; ambiente local simples (API e PostgreSQL via Docker).
- **Negativas:** os módulos compartilham o mesmo processo e a mesma instância de
  banco; exige disciplina para não acoplar um módulo ao outro.
- **Como verificar:** o check de back-end (testes e fronteiras) no
  `verificar.ps1` e na CI, incluindo o teste de que cada módulo tem o seu
  schema no banco.