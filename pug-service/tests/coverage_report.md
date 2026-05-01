# Relatorio de Cobertura de Testes - pug-service

## Resumo Executivo

- Cobertura de linhas: **85.17%** (4077 cobertas, 710 perdidas).
- Cobertura de instrucoes: **86.56%** (17172 cobertas, 2667 perdidas).
- Cobertura de metodos: **94.09%**. Cobertura de classes: **98.49%**.
- Classes analisadas no CSV do JaCoCo: **265**.
- Pacotes com linhas rastreadas: **80**.

## Totais Gerais

| Metrica | Cobertas | Perdidas | Total | % Cobertura |
| :--- | ---: | ---: | ---: | ---: |
| Instruction | 17172 | 2667 | 19839 | 86.56% |
| Line | 4077 | 710 | 4787 | 85.17% |
| Complexity | 1317 | 631 | 1948 | 67.61% |
| Method | 987 | 62 | 1049 | 94.09% |
| Class | 261 | 4 | 265 | 98.49% |

## Pacotes com Menor Cobertura de Linhas

| Pacote | % Linhas | Linhas Perdidas | Linhas Cobertas | Instrucoes Perdidas | Classes |
| :--- | ---: | ---: | ---: | ---: | ---: |
| `geo.domain` | 16.13% | 26 | 5 | 76 | 1 |
| `shared.infra` | 28.57% | 10 | 4 | 26 | 1 |
| `shared.validation` | 37.50% | 5 | 3 | 18 | 2 |
| `academic.domain.enums` | 40.62% | 19 | 13 | 126 | 2 |
| `geo.domain.enums` | 41.67% | 7 | 5 | 30 | 2 |
| `geo.infra` | 47.06% | 9 | 8 | 22 | 1 |
| `project.domain.enums` | 57.33% | 32 | 43 | 230 | 5 |
| `shared.domain` | 60.00% | 10 | 15 | 32 | 1 |
| `shared.presenter.mappers` | 63.64% | 4 | 7 | 11 | 1 |
| `partner.domain` | 68.75% | 20 | 44 | 79 | 2 |

## Classes com Mais Linhas Perdidas

| Classe | % Linhas | Linhas Perdidas | Linhas Cobertas | Instrucoes Perdidas | Metodos Perdidos |
| :--- | ---: | ---: | ---: | ---: | ---: |
| `academic.domain.Student` | 56.96% | 34 | 45 | 131 | 4 |
| `project.domain.enums.ProjectsFieldErrorCodes` | 0.00% | 32 | 0 | 230 | 2 |
| `geo.domain.City` | 16.13% | 26 | 5 | 76 | 4 |
| `identity.domain.Account` | 62.90% | 23 | 39 | 92 | 2 |
| `project.domain.Project` | 78.85% | 22 | 82 | 79 | 1 |
| `identity.service.impl.UserServiceImpl` | 73.68% | 20 | 56 | 62 | 2 |
| `academic.domain.enums.AcademicFieldErrorCodes` | 0.00% | 19 | 0 | 126 | 2 |
| `project.domain.vos.ProjectInfo` | 64.81% | 19 | 35 | 71 | 2 |
| `identity.service.impl.AccountServiceImpl` | 83.90% | 19 | 99 | 67 | 3 |
| `partner.domain.Entity` | 65.38% | 18 | 34 | 73 | 2 |

## Pacotes com Maior Volume de Codigo Rastreado

| Pacote | Total de Linhas | % Linhas | Classes |
| :--- | ---: | ---: | ---: |
| `identity.service.impl` | 405 | 87.16% | 8 |
| `project.service.impl` | 329 | 91.79% | 8 |
| `academic.service.impl` | 267 | 89.14% | 6 |
| `project.domain` | 188 | 78.19% | 4 |
| `partner.service.impl` | 175 | 92.00% | 4 |
| `project.infra` | 164 | 76.22% | 4 |
| `project.presenter` | 163 | 90.18% | 5 |
| `project.infra.persistence.impl` | 142 | 84.51% | 4 |
| `academic.domain` | 139 | 70.50% | 3 |
| `project.domain.vos` | 136 | 72.79% | 5 |

## Visualizacao Mermaid

```mermaid
pie title Cobertura Geral (Linhas)
    "Cobertas" : 4077
    "Perdidas" : 710
```

## Leitura Rapida

- A tabela de pacotes destaca onde a cobertura esta mais fragil no nivel em que o build costuma falhar.
- A tabela de classes mostra os principais candidatos para novos testes quando a meta de um pacote cai.
- O volume de codigo rastreado ajuda a separar pacote pequeno com cobertura ruim de pacote grande com impacto real.
