# Pipeline DevSecOps — UFLA

Projeto da atividade prática de DevOps da Universidade Federal de Lavras, com verificações de segurança integradas ao CI/CD por GitHub Actions.

## Repositório

- Repositório: https://github.com/veronicarsfranca/ufla-devops-shop
- Pull Request de trabalho: https://github.com/veronicarsfranca/ufla-devops-shop/pull/1

## Objetivo

Executar verificações automatizadas durante Pull Requests para identificar segredos expostos, problemas de qualidade e vulnerabilidades em dependências e na imagem Docker antes da integração das alterações à branch principal.

## Ferramentas e verificações

### GitHub Actions

O workflow `.github/workflows/pipeline-seguro.yml` coordena as etapas de validação. Ele é acionado em `push` para `main` e em Pull Requests direcionados à `main`.

### Testes e lint

- Instala as dependências definidas em `requirements-dev.txt`.
- Executa Ruff para análise estática de estilo e problemas comuns.
- Executa os testes com `pytest` e relatório de cobertura.

### Gitleaks

Analisa o histórico Git completo (`fetch-depth: 0`) em busca de possíveis segredos, como tokens e credenciais. A verificação deve falhar quando detectar um vazamento. Não utilizar credenciais reais em testes.

### SonarCloud

Executa análise estática do código e aguarda o resultado do Quality Gate (`sonar.qualitygate.wait=true`). O token de acesso é configurado como segredo do GitHub Actions (`SONAR_TOKEN`), não deve ser versionado no repositório.

### Trivy

Constrói a imagem Docker `ufla-devops-shop:ci` e verifica vulnerabilidades do sistema operacional e das bibliotecas (`vuln-type: os,library`).

A etapa está configurada para bloquear vulnerabilidades `HIGH` e `CRITICAL` com correção disponível (`ignore-unfixed: true` e `exit-code: '1'`). Vulnerabilidades sem correção são ignoradas para efeito de bloqueio, conforme essa configuração. Verifique os relatórios da execução para confirmar os achados.

### Syft / SBOM

A etapa de geração de SBOM usa a action `anchore/sbom-action` para produzir um inventário SPDX JSON da imagem Docker, com componentes e versões. O arquivo configurado é `sbom.spdx.json`, publicado como artifact `sbom-spdx` da execução do workflow, com retenção configurada para 30 dias.

## Regras de bloqueio

O pipeline deve ser considerado aprovado somente quando as verificações obrigatórias terminarem com sucesso. A etapa Trivy retorna código de saída diferente de zero quando encontra vulnerabilidades `HIGH` ou `CRITICAL` corrigíveis. O SonarCloud aguarda o Quality Gate e falha quando ele não é aprovado.

Para impedir que uma alteração seja integrada apesar de um check vermelho, configure a proteção da branch `main` no GitHub para exigir os checks relevantes, se essa configuração estiver disponível para o repositório.

## Evidências para a atividade

Inclua nesta seção links ou capturas de tela reais obtidos nas execuções do GitHub Actions. Não substitua evidências reais por exemplos.

1. **Repositório e workflow:** https://github.com/veronicarsfranca/ufla-devops-shop
2. **Execução do pipeline:** adicionar o link da execução bem-sucedida do workflow `Pipeline seguro`.
3. **Pull Request bloqueado:** adicionar a captura do PR com o check de segurança reprovado e o trecho do relatório do Trivy que identifica a vulnerabilidade.
4. **Execução após a correção:** adicionar o link ou captura de uma execução aprovada depois de corrigir a dependência vulnerável.
5. **SBOM:** na execução do workflow, abrir a seção **Artifacts** e baixar `sbom-spdx` para confirmar a publicação do arquivo `sbom.spdx.json`.

### Demonstração de bloqueio e correção

A atividade exige duas evidências distintas:

- Uma execução em branch/PR de teste em que uma dependência com vulnerabilidade conhecida e correção disponível provoque a falha do Trivy e impeça a integração.
- Uma execução posterior, após corrigir a dependência, em que as verificações obrigatórias passem.

Não faça merge da versão vulnerável para `main` nem publique uma imagem vulnerável em produção. Registre o relatório da execução reprovada antes de corrigir a dependência.

## Segredos e configuração

Configure `SONAR_TOKEN` em **Settings → Secrets and variables → Actions** no repositório. Não grave tokens, senhas ou credenciais em arquivos versionados. O Gitleaks deve continuar analisando o histórico completo.

## Checklist final

- [ x ] Workflow executado em Pull Request.
- [ x ] Gitleaks aprovado.
- [ x ] SonarCloud Quality Gate aprovado.
- [ x ] Trivy aprovado após a correção.
- [ x ] Artifact `sbom-spdx` disponível e arquivo `sbom.spdx.json` baixado/validado.
- [ x ] Captura do Pull Request bloqueado e relatório de vulnerabilidade guardados.
- [ x ] Captura/link da execução aprovada após a correção guardado.
- [ x ] Proteção da branch `main` configurada, quando disponível.
