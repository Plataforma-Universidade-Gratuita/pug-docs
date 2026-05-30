# Relatorio de Cobertura de Testes - pug-service

## Resumo Executivo

- Cobertura de linhas: **86.49%** (4744 cobertas, 741 perdidas).
- Cobertura de instrucoes: **87.94%** (20840 cobertas, 2858 perdidas).
- Cobertura de metodos: **94.33%**. Cobertura de classes: **98.16%**.
- Classes analisadas no CSV do JaCoCo: **326**.
- Pacotes com linhas rastreadas: **97**.

## Totais Gerais

| Metrica | Cobertas | Perdidas | Total | % Cobertura |
| :--- | ---: | ---: | ---: | ---: |
| Instruction | 20840 | 2858 | 23698 | 87.94% |
| Line | 4744 | 741 | 5485 | 86.49% |
| Complexity | 1561 | 775 | 2336 | 66.82% |
| Method | 1082 | 65 | 1147 | 94.33% |
| Class | 320 | 6 | 326 | 98.16% |

## Pacotes com Menor Cobertura de Linhas

| Pacote | % Linhas | Linhas Perdidas | Linhas Cobertas | Instrucoes Perdidas | Classes |
| :--- | ---: | ---: | ---: | ---: | ---: |
| `geo.domain` | 16.13% | 26 | 5 | 76 | 1 |
| `shared.infra` | 28.57% | 10 | 4 | 26 | 1 |
| `shared.validation` | 37.50% | 5 | 3 | 18 | 2 |
| `geo.domain.enums` | 41.67% | 7 | 5 | 30 | 2 |
| `geo.infra` | 47.06% | 9 | 8 | 22 | 1 |
| `project.domain.enums` | 57.89% | 32 | 44 | 230 | 5 |
| `project.domain` | 62.56% | 73 | 122 | 265 | 4 |
| `shared.presenter.mappers` | 63.64% | 4 | 7 | 11 | 1 |
| `academic.presenter.mappers` | 64.80% | 44 | 81 | 160 | 3 |
| `identity.infra` | 68.27% | 33 | 71 | 88 | 4 |

## Classes com Mais Linhas Perdidas

| Classe | % Linhas | Linhas Perdidas | Linhas Cobertas | Instrucoes Perdidas | Metodos Perdidos |
| :--- | ---: | ---: | ---: | ---: | ---: |
| `project.domain.Project` | 50.00% | 52 | 52 | 155 | 4 |
| `project.domain.enums.ProjectsFieldErrorCodes` | 0.00% | 32 | 0 | 230 | 2 |
| `academic.presenter.mappers.FormerStudentPresenter` | 63.86% | 30 | 53 | 129 | 3 |
| `geo.domain.City` | 16.13% | 26 | 5 | 76 | 4 |
| `identity.domain.Account` | 63.33% | 22 | 38 | 88 | 2 |
| `identity.service.impl.UsersServiceImpl` | 73.68% | 20 | 56 | 62 | 2 |
| `project.service.impl.ProjectServiceImpl` | 78.26% | 20 | 72 | 103 | 0 |
| `project.domain.vos.ProjectInfo` | 64.81% | 19 | 35 | 71 | 2 |
| `identity.service.impl.AccountsServiceImpl` | 83.90% | 19 | 99 | 67 | 3 |
| `partner.domain.Entity` | 65.38% | 18 | 34 | 73 | 2 |

## Pacotes com Maior Volume de Codigo Rastreado

| Pacote | Total de Linhas | % Linhas | Classes |
| :--- | ---: | ---: | ---: |
| `identity.service.impl` | 413 | 88.38% | 8 |
| `project.service.impl` | 355 | 88.45% | 8 |
| `academic.service.impl` | 264 | 93.56% | 6 |
| `project.presenter` | 217 | 96.77% | 5 |
| `project.infra.read.impl` | 214 | 94.39% | 3 |
| `project.domain` | 195 | 62.56% | 4 |
| `identity.infra.read.impl` | 175 | 99.43% | 3 |
| `academic.infra.read.impl` | 173 | 99.42% | 3 |
| `project.infra` | 164 | 76.22% | 4 |
| `project.infra.persistence.impl` | 150 | 84.00% | 4 |

## Visualizacao Mermaid

```mermaid
pie title Cobertura Geral (Linhas)
    "Cobertas" : 4744
    "Perdidas" : 741
```

## Leitura Rapida

- A tabela de pacotes destaca onde a cobertura esta mais fragil no nivel em que o build costuma falhar.
- A tabela de classes mostra os principais candidatos para novos testes quando a meta de um pacote cai.
- O volume de codigo rastreado ajuda a separar pacote pequeno com cobertura ruim de pacote grande com impacto real.
