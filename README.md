# Laboratório Prático: GitHub Actions com act

**Nome:** Gustavo Palomo Moura
**RA:** 2500355
**Email:** gustavopalomomou@gmail.com

---

## Validação da Parte 2

O job `teste-basico` se encontra no **Stage 0**, conforme a saída do `act -l`.

---

## Validação da Parte 3 — Saída do `act -l`

```
Stage  Job ID             Job name           Workflow name       Workflow file              Events
0      teste-basico       teste-basico       Hello Local         01-hello-local.yml         workflow_dispatch,push
0      setup-e-lint       setup-e-lint       Pipeline Principal  02-pipeline-principal.yml  push
1      scan-de-seguranca  scan-de-seguranca  Pipeline Principal  02-pipeline-principal.yml  push
1      testes-unitarios   testes-unitarios   Pipeline Principal  02-pipeline-principal.yml  push
2      build-e-deploy     build-e-deploy     Pipeline Principal  02-pipeline-principal.yml  push
```

O DAG está montado corretamente:
- **Stage 0:** `setup-e-lint` (sem dependências, roda primeiro)
- **Stage 1:** `testes-unitarios` e `scan-de-seguranca` (ambos dependem do `setup-e-lint`, rodam em paralelo)
- **Stage 2:** `build-e-deploy` (depende de `testes-unitarios` e `scan-de-seguranca`)

---

## Validação da Parte 4 — Comprovação de Paralelismo

Com base na saída do terminal, é possível comprovar que o Job 2 (`testes-unitarios`) e o Job 3 (`scan-de-seguranca`) rodaram em paralelo observando que:

1. **As mensagens de log dos dois jobs aparecem intercaladas na saída do terminal**, e não sequenciais. Ou seja, linhas do `[Pipeline Principal/testes-unitarios]` e do `[Pipeline Principal/scan-de-seguranca]` se alternam, mostrando que ambos estavam sendo executados simultaneamente.

2. **O act criou dois contêineres Docker separados ao mesmo tempo** (um para cada job) logo após o Stage 0 (`setup-e-lint`) ser concluído. Na saída do dry-run (`act -n`), é possível ver que as operações de `docker pull`, `docker create` e `docker run` para ambos os jobs ocorrem de forma intercalada, comprovando que foram iniciados em paralelo.

3. **Ambos os jobs foram concluídos antes do `build-e-deploy` iniciar**, confirmando que estavam no mesmo Stage (Stage 1) e que o act os executou simultaneamente, respeitando o DAG configurado pelas dependências `needs`.
