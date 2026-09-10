# Publicar e manter versões

Repositório: https://github.com/thiagolcordeiro/home-assistant-wiz-segments

Publique somente este projeto. A raiz contém README, LICENSE, hacs.json e
custom_components/wiz_segments. Não inclua firmware, backups, dados dos testes
locais ou arquivos do projeto WLED que o originou.

1. Atualize `version` no manifesto e o histórico em `CHANGELOG.md`.
2. Execute os testes e `python tools/validate_release.py`.
3. Envie um commit à branch `main` e confira os jobs em GitHub Actions.
4. No GitHub, abra **Releases → Draft a new release**. Crie uma tag correspondente
   à versão, por exemplo `v0.2.1`, apontando para o commit verificado.
5. Descreva mudanças, testes e limitações; marque como pré-release enquanto o
   suporte permanecer experimental. Publique a release. Uma tag isolada não é
   uma release. O GitHub disponibiliza os arquivos de código automaticamente.

O HACS usa a estrutura normal do repositório; não configure `zip_release`.
Pré-releases podem exigir habilitar versões beta no HACS. Sem release, a instalação
por repositório personalizado usa o conteúdo disponível na branch padrão.
Teste a instalação em um Home Assistant separado antes de anunciar estabilidade.

Adicione descrição e tópicos do projeto no GitHub. Mantenha Issues habilitadas.
A validação do HACS depende também desses metadados remotos. O job hassfest verifica
metadados do HA; nenhum desses jobs substitui um teste real da integração.

Os usuários podem adicionar o endereço como repositório personalizado seguindo o
[README em português](../README.pt-BR.md). A inclusão no catálogo padrão é um processo
separado descrito na [documentação HACS](https://www.hacs.xyz/docs/publish/).

Para publicar alterações locais com Git, execute dentro desta pasta:

```text
git add custom_components tests tools docs .github README.md README.pt-BR.md HARDWARE.md CONTRIBUTING.md CHANGELOG.md LICENSE hacs.json .gitignore .gitattributes
git diff --cached --stat
git commit -m "Describe the change"
git push origin main
```

Revise os arquivos preparados antes do commit. Nunca adicione tokens ao código.
