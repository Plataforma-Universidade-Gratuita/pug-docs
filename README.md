# PUG Docs — Plataforma Universidade Gratuita (PUG)

> Repositório: `pug-docs`  
> Escopo: *briefing* de produto, visão de arquitetura e restrições operacionais da plataforma de controle de horas do Universidade Gratuita.

---

## 1) O que é

Plataforma multiaplicativo para registrar, validar e auditar horas de contrapartida exigidas pelo **Universidade Gratuita** de Santa Catarina. Conecta três atores:

- **Administradores institucionais**: configuram entidades, projetos, alocações e relatórios.
- **Estudantes (beneficiários)**: aderem a projetos, acompanham progresso, geram QR de presença.
- **Parceiros (entidades/empresas)**: aceitam alunos e validam presenças.

Entrega alvo: **1º dez 2025**.

---

## 2) Por que existe

- Substituir controle manual por presenças verificáveis com QR, geolocalização e horário.
- Prover fonte única de verdade para conformidade e relatórios.
- Oferecer fluxos móveis simples com progresso e prazos claros.

---

## 3) Objetivos

### Primários
- Acompanhar horas requeridas vs. concluídas por aluno e por projeto.
- Aplicar controle de acesso por papel e autenticação externa.
- Gerar relatórios auditáveis em XLSX/PDF.

### Secundários
- Reduzir carga administrativa.
- Melhorar transparência e notificações entre atores.
- Atender LGPD e normas institucionais.

---

## 4) Resultados esperados

- **Operacional**: alunos comprovam horas no prazo; parceiros validam rápido; admins monitoram portfólio com filtros e exportações.
- **Conformidade**: trilhas à prova de violação (quem, quando, onde).
- **Escalabilidade**: inclusão de novos projetos, entidades e turmas sem mudanças estruturais.
- **Confiabilidade**: ≥99,5% de *uptime* e P95 de API < 500 ms em carga normal.

---

## 5) Escopo do produto

### No escopo
- Portais por papel: Admin (web), Estudante (mobile), Parceiro (mobile).
- Jornada do estudante: buscar projetos, solicitar adesão, participar, acompanhar, sair se necessário.
- Jornada do parceiro: aceitar/negar, expulsar, validar presenças, acompanhar horas.
- Admin: CRUD de entidades/projetos/cursos/áreas, alocações, análises, exportações.
- Validação de presença: **QR code + localização + timestamp**.
- Notificações de adesão/aceite/expulsão/saída e marcos.
- Exportações (XLSX/PDF) para alunos, parceiros e admins.

### Fora do escopo (v1)
- Acompanhamento pós-colação além das regras do programa.
- Programas não-UG ou outros estados.
- Upload de arquivos.
- Projetos 100% remotos baseados só em entregáveis sem validação de presença.
- Deploy em PRD.
- Manutenção do código pós entrega da v1.

---

## 6) Arquitetura do sistema

- **Aplicativos**:
  - Web (Admin): Next.js 15, React LTS, TypeScript, Tailwind, Shadcn UI, Zustand, TanStack Query, Zod, i18next, Jest, Sonner.
  - Mobile (Aluno, Parceiro): Expo, React Native, TypeScript, Zustand e contrapartes compatíveis das outras *libs* da versão web.
- **Serviços**: Java 21 + Quarkus (REST), Hibernate ORM Panache, Bean Validation, SmallRye OpenAPI.
- **Dados**: PostgreSQL (primário), MongoDB (sessões, aceite de termos, auditorias).
- **Migrações/Docs/Testes**: Flyway, Javadoc, TSDoc, JUnit, Mockito, Postman.
- **Segurança**: JWT.

---

## 7) Modelo de dados (alto nível)

- **Usuários** com **papéis**: `ADMIN`, `FORMER_STUDENT`, `PARTNER`.
- **Estudantes** vinculados a **cursos**, com **cotas de horas** e datas de início/vencimento.
- **Entidades** em **cidades**; **staff** representam entidades.
- **Projetos** pertencem a entidades e áreas; podem ter **alocações** de horas e **locais**.
- **Matrículas** ligam aluno↔projeto com estados: `PENDING`, `ACCEPTED`, `DENIED`, `CONCLUDED`, `EXITED`, `REMOVED`.
- **Presenças** vinculadas à matrícula com duração, geolocalização, hash do QR e status: `PENDING`, `VALIDATED`, `REJECTED`.

---

## 8) Fluxos principais

- **Aluno**: descobre projetos elegíveis → solicita → é aceito → participa → gera QR no local → parceiro valida → progresso atualiza → exporta relatório final.
- **Parceiro**: analisa solicitações → aceita/nega → valida presenças via QR → monitora horas → exporta relatório por projeto.
- **Admin**: cria entidades/projetos/alocações → monitora indicadores com filtros → exporta relatórios agregados.

---

## 9) Requisitos não funcionais

- Acessibilidade WCAG 2.1 AA.
- Disponibilidade ≥99,5% mensal.
- Latência de API P95 ≤ 500 ms.
- Escalabilidade horizontal.
- Cobertura de testes ≥80% (backend).
- Versionamento de API com SemVer.
- Contêineres para portabilidade de deploy.
- LGPD: consentimento, minimização, criptografia em repouso e trânsito, exclusão sob solicitação.

---

## 10) Segurança

- RBAC + validação de login externo.
- JWT com access tokens de curta duração e refresh quando aplicável.
- Validação estrita de entrada (Bean Validation/Zod).
- CSP, sanitização anti-XSS, statements preparados via ORM.
- Rate limiting e proteção contra DoS em endpoints públicos.
- Mobile: armazenamento seguro de tokens (Keychain/Keystore) e checagens de integridade.

---

## 11) Relatórios

- Alunos: declaração de conclusão (XLSX/PDF).
- Parceiros: presenças e progresso por projeto.
- Admins: idem + estados incompletos e análises do portfólio.

---

## 12) Glossário

- **Horas de contrapartida**: serviço à comunidade devido por beneficiários.
- **Alocação**: bloco de horas ofertadas atrelado a projeto e janela temporal.
- **Presença**: registro validado de participação com duração e geodados.

---

## 13) Referências

- Requisitos: “Plataforma de Gerenciamento de Horas” (documento interno, mais antigo e menos atualizado).
- ER e enums: [DER](./assets/PUG_MER.png) e [notas de esquema](./assets/PUG_MER.txt).
- Arquitetura de sistema: [System Architecture](./assets/PUG-System-Architecture.png)

---

## 14) Licença e titularidade

Software institucional para operações do UG. Licenciamento e distribuição conforme política da instituição contratante.

---

> Documento vivo. Atualize com mudanças de esquema, fluxos ou políticas.
