# WiZ Segments

[English](README.md) | **Português brasileiro**

Integração personalizada para controlar uma fita WiZ RGBIC diretamente pelo IP,
sem ESP e sem WLED. Cada segmento configurado vira uma entidade de luz no Home
Assistant, com liga/desliga, brilho, vermelho, verde, azul, branco frio e branco quente.

**Versão experimental 0.3.0.** O controle independente de RGB e o mapeamento correto
de **branco quente e branco frio estão confirmados visualmente** na WiZ 605568,
módulo `ESP25_MHORGB_01`, firmware `1.38.0`. A versão 0.2.1 foi validada no Home
Assistant; as novas opções e animações da 0.3.0 ainda precisam de validação em uso.
Outros módulos são recusados durante a configuração.
Este projeto é independente e não é uma integração oficial da WiZ.

## Requisitos

- Home Assistant 2026.3 ou posterior: versão mínima declarada para os ícones locais;
  ainda não representa uma matriz de versões testadas em execução.
- Fita compatível configurada no aplicativo WiZ e acessível pela rede local.
- Comunicação UDP na porta 38899 entre o Home Assistant e a fita. Reserve o IP
  no DHCP do roteador; não é necessário expor portas para a internet.
- HACS instalado, caso escolha essa forma de instalação. Siga o
  [guia oficial do HACS](https://www.hacs.xyz/docs/use/) para instalar e configurar
  o próprio HACS conforme seu tipo de instalação do Home Assistant.

## Instalação pelo HACS

1. Abra **HACS → menu ⋮ → Repositórios personalizados**.
2. Informe `https://github.com/thiagolcordeiro/home-assistant-wiz-segments`,
   selecione o tipo **Integração** e adicione.
3. Procure **WiZ Segments**, abra o projeto e escolha **Baixar**.
4. Reinicie o Home Assistant.
5. Vá a **Configurações → Dispositivos e serviços → Adicionar integração** e
   procure **WiZ Segments**.

A inclusão é por repositório personalizado; não significa aprovação no catálogo
padrão do HACS. [Instruções oficiais](https://www.hacs.xyz/docs/faq/custom_repositories/).

## Instalação manual

Baixe o código do repositório ou de uma release e copie somente a pasta
`custom_components/wiz_segments` para `/config/custom_components/wiz_segments`.
O arquivo final precisa estar em
`/config/custom_components/wiz_segments/manifest.json`.
Reinicie o Home Assistant e adicione a integração como descrito acima.

## Configuração inicial

Informe o IP da sua fita, a quantidade de blocos físicos instalados e o modo
personalizado. No hardware testado, cada bloco tem **6 LEDs**. Uma fita completa
com 150 LEDs possui 25 blocos. Informe a quantidade efetivamente instalada;
o comprimento não é identificado automaticamente.

O modo padrão é **258**, validado no teste. Ele deve estar livre de modos
personalizados salvos pelo aplicativo WiZ; um modo ocupado pode aceitar o comando
e ignorar suas cores. A configuração cria até três segmentos cobrindo toda a fita
e não altera sua iluminação.

## Criar e editar segmentos

Abra **Configurar** na integração e escolha adicionar, editar ou remover segmento.
Confirme em **Salvar alterações**. As edições ficam pendentes até salvar;
a iluminação muda no próximo comando de luz.

Os limites são inclusivos e começam em 1:

| Blocos | LEDs físicos |
|---|---|
| 1–3 | 1–18 |
| 4–6 | 19–36 |
| 7–9 | 37–54 |
| 10–12 | 55–72 |

Reduza ou remova um segmento existente antes de criar outro no mesmo espaço.
Não são permitidas sobreposições, intervalos fora da fita ou remoção de todos
os segmentos. O limite implementado é **12 regiões no comando**, incluindo
lacunas apagadas e o final sem uso. Portanto, nem todo desenho permite 12 entidades.
Renomear ou redimensionar mantém a identidade da entidade; remover exclui sua
entidade do registro depois de salvar. Ajuste automações que usem entidades removidas.

O controle mínimo é um bloco de seis LEDs, sem endereçamento individual dentro
do bloco. A versão 0.3.0 oferece as animações descritas abaixo; comandos nativos de transição não são suportados.

## Cores, brancos e brilho

As entidades usam `RGBWW`. O cartão padrão depende da versão da interface do
Home Assistant e não inclui necessariamente cinco controles separados.
Para valores exatos, use **Ferramentas do desenvolvedor → Ações** ou automações.
Substitua `light.wiz_rgbic_segment_1` pelo ID real da sua entidade.

```yaml
action: light.turn_on
target:
  entity_id: light.wiz_rgbic_segment_1
data:
  brightness: 128
  rgbww_color: [0, 0, 0, 0, 255]
```

A ordem é **[vermelho, verde, azul, branco frio, branco quente]**.
Cada canal aceita 0–255; `brightness` é um ajuste independente de 0–255.

| Resultado | `rgbww_color` |
|---|---|
| Vermelho | `[255, 0, 0, 0, 0]` |
| Verde | `[0, 255, 0, 0, 0]` |
| Azul | `[0, 0, 255, 0, 0]` |
| Branco quente | `[0, 0, 0, 0, 255]` |
| Branco frio | `[0, 0, 0, 255, 0]` |
| Branco formado por RGB | `[255, 255, 255, 0, 0]` |

```yaml
action: light.turn_off
target:
  entity_id: light.wiz_rgbic_segment_1
```

Cada comando recompõe a fita inteira, preservando os outros segmentos sob controle
da integração. Lacunas ficam apagadas. Os campos de branco do protocolo têm ordem
inversa à do Home Assistant; a integração faz a conversão. O mapeamento foi
validado visualmente: `[0, 0, 0, 0, 255]` produz branco quente e
`[0, 0, 0, 255, 0]` produz branco frio. A validação confirma os controles
funcionais, sem determinar a composição física dos emissores.
Temperatura em Kelvin, misturas simultâneas dos brancos e curvas ópticas não foram
calibradas. Preferências RGB da versão 0.1.0 são carregadas com os brancos zerados.

## Aplicativo WiZ, estado e reinicialização

O dispositivo não devolve as cores por segmento. As entidades apresentam estado
presumido; uma confirmação UDP não prova a saída visual. Há consulta de estado
geral a cada 10 segundos. Cores e brilho preferidos são armazenados localmente,
mas iniciar ou reconfigurar a integração não liga a fita automaticamente.

Após reinício, falha de comunicação, mudança de layout ou cena externa, o estado
individual pode ficar desconhecido enquanto a fita estiver ligada. Ligue um
segmento para retomar o controle: os demais segmentos desconhecidos ficam apagados.
Desligar isoladamente um segmento nessa situação é recusado porque não é possível
preservar uma cena que não pode ser lida. Quando a fita informa que está desligada,
todas as entidades indicam desligado.

Evite comandos simultâneos do aplicativo WiZ, WLED e outras integrações para a
mesma fita. Alterações externas no mesmo modo personalizado não são detectáveis
com segurança. Atualizações de firmware podem mudar o comportamento.

## Atualizar, reverter e remover

Faça backup do Home Assistant antes de atualizar. Pelo HACS, abra WiZ Segments,
baixe a versão desejada e reinicie. Para reverter, use a opção de baixar novamente
selecionando uma release anterior disponível, ou restaure o backup. Na instalação
manual, substitua a pasta da integração pelos arquivos da versão desejada e reinicie.

Para remover, exclua a entrada em Dispositivos e serviços e depois remova o download
no HACS (ou a pasta instalada manualmente), reiniciando em seguida. A remoção da
integração não envia um comando para desligar a fita; desligue antes se desejar.

## Solução de problemas

| Sintoma | Verificação |
|---|---|
| Integração não aparece | Confira o caminho de `manifest.json`, reinicie e recarregue a página. |
| Não conecta | Confira IP, energia, isolamento Wi-Fi/VLAN e UDP 38899 a partir do servidor HA. |
| Módulo incompatível | Envie modelo, módulo e firmware em uma issue; não force outro módulo. |
| Comando aceito sem mudar cores | Confira se o modo 258 está ocupado por uma cena salva. |
| Trechos têm comprimento errado | Confira a quantidade instalada de blocos de 6 LEDs. |
| Layout rejeitado | Revise sobreposições e conte também lacunas e cauda no limite de 12. |
| Estado desconhecido | Ligue um segmento para retomar o controle conforme explicado acima. |
| Branco não aparece no cartão | Teste `rgbww_color` pela ação, com RGB zerado. |

Consulte **Configurações → Sistema → Registros** e procure `wiz_segments`.
Ao relatar problemas, inclua versões do HA e da integração, modelo/firmware,
limites dos segmentos e resultado esperado/observado. Remova IPs, MACs, e-mails e
tokens dos registros antes de publicar.

## Tamanho e quantidade dinâmica de segmentos (0.3.0)

Em **Configurar → Editar segmento**, informe **Quantidade de blocos**. Cada bloco
contém seis LEDs na fita compatível. **Organizar segmentos automaticamente**
alinha os segmentos desde o bloco 1, na ordem atual, preservando os IDs e eliminando
lacunas. Novos segmentos são acrescentados ao final. Nesse modo, **Primeiro bloco**
é ignorado; desative a organização automática para escolher uma posição específica.
A soma precisa caber na fita: a configuração não cria LEDs físicos adicionais.

Use **Distribuir quantidade de segmentos** para dividir toda a fita em 1–12
segmentos, respeitando também a quantidade instalada de blocos. Para 18 blocos,
seis segmentos terão três blocos cada. Depois, edite os tamanhos individualmente.
A distribuição substitui tamanhos e lacunas anteriores, mantém os primeiros nomes
e IDs na ordem física e remove os últimos segmentos ao reduzir a quantidade.
Tudo fica pendente até **Salvar alterações**. Revise automações que usem entidades removidas.

## Efeitos animados (0.3.0)

Cada luz passa a oferecer uma lista de efeitos no Home Assistant:

| Efeito | Comportamento |
|---|---|
| `off` | Cor RGBWW fixa; para a animação sem desligar a luz. |
| `Rainbow` | Arco-íris em movimento nas regiões disponíveis do segmento. |
| `Chase` | Uma região brilhante percorre um fundo fraco na cor RGBWW escolhida. |
| `Breathe` | Variação gradual de brilho na cor RGBWW escolhida. |
| `Color loop` | Todo o segmento percorre um ciclo de cores RGB. |

```yaml
action: light.turn_on
target:
  entity_id: light.wiz_rgbic_segment_1
data:
  brightness: 128
  effect: Rainbow
```

Escolha `effect: "off"` para voltar à cor fixa ou use `light.turn_off` para desligar.
Um comando de cor sem efeito explícito interrompe a animação; um comando apenas
de brilho a mantém. Rainbow e Color loop geram suas próprias cores RGB;
Breathe e Chase usam os canais RGBWW configurados.

Em **Configurar → Conexão e comprimento da fita**, ajuste a duração do ciclo
(1–60 segundos, padrão 6) e a frequência (1–5 atualizações por segundo, padrão 2).
Essas opções valem para os efeitos da fita e precisam ser salvas. Quadros são
enviados em sequência; respostas lentas reduzem a frequência real, sem acumular
uma fila. Quadros de animação não são gravados em disco nem geram um novo estado
do Home Assistant a cada atualização.

São efeitos originais gerados pelo servidor, inspirados em animações comuns,
sem incorporar o motor do WLED. Segmentos fixos, lacunas e cauda reservam primeiro
suas regiões no comando. Os segmentos animados dividem o espaço restante do limite
de 12 regiões. Por isso, trechos longos podem se mover em grupos maiores; sem
regiões livres, Rainbow vira um ciclo de cor e Chase não consegue se deslocar
dentro daquele segmento. Use menos segmentos/lacunas para obter mais detalhes.
Não há controle individual dentro de um bloco de seis LEDs, nem garantia da
mesma fluidez do WLED.

O servidor HA precisa continuar funcionando. Efeitos param após detectar cena
externa, desligamento, falha de comunicação, recarga ou encerramento do HA, sem
retomada automática. Ao parar o HA, as últimas cores enviadas podem continuar
acesas. Alterações externas no mesmo modo não são detectadas com segurança.
A versão 0.2.1 foi validada no Home Assistant; as novas animações ainda
precisam de validação visual na fita.

## Desenvolvimento e licença

```text
python -m unittest discover -s tests -v
python tools/validate_release.py
```

Os testes locais cobrem quadros RGBWW, limites e lacunas, brilho, transporte UDP,
falhas, concorrência e comportamento do coordenador com substitutos mínimos do HA.
A comunicação local, as regiões RGB independentes e o mapeamento correto dos
brancos quente e frio foram validados no hardware compatível. A versão 0.2.1
também foi validada no Home Assistant. As novas opções e animações da versão
0.3.0 ainda precisam de validação em uso.

Veja [observações de hardware](HARDWARE.md), [contribuições](CONTRIBUTING.md) e
[publicação de versões](docs/PUBLISHING.pt-BR.md). `tools/probe.py` consulta a fita
sem alterar a iluminação por padrão; o teste visual opcional usa temporizador.

Licença [MIT](LICENSE). Implementação e testes desenvolvidos com assistência de IA,
sem copiar código do [projeto de investigação do protocolo](https://github.com/TechAntohere/WizScreenSyncController/).
