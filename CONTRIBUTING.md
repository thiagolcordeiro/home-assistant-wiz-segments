# Contributing / Contribuindo

Issues and pull requests in English or Brazilian Portuguese are welcome.
Include Home Assistant/integration versions, hardware module and firmware,
segment layout, expected behavior and actual observations. Remove private network
addresses, MAC addresses and credentials from logs. Do not infer optical output
from a successful UDP response.

Issues e pull requests em português ou inglês são bem-vindos. Informe versões,
módulo, firmware, layout e resultados. Remova endereços privados, MACs e credenciais.
Uma resposta UDP positiva não comprova o resultado visual.

Run / Execute:

```text
python -m unittest discover -s tests -v
python tools/validate_release.py
```

Keep both READMEs and translation keys synchronized. Add meaningful regression
tests for protocol/state changes. Tests must not contact real devices by default.
Report real Home Assistant testing separately from isolated adapter tests.
Contributions are distributed under the repository's MIT license.

Mantenha os dois READMEs e as traduções sincronizados. Inclua testes de regressão
para mudanças de protocolo/estado. Os testes padrão não devem acessar dispositivos
reais. Relate testes no Home Assistant separadamente dos testes isolados.
Contribuições são distribuídas sob a licença MIT do repositório.
