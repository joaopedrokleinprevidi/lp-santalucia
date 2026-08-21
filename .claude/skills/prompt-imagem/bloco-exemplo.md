# Bloco de handoff preenchido — exemplo canônico

Referência de formato do `prompt-imagem`. Copie a forma, não o conteúdo. Os oito campos
obrigatórios estão no SKILL.md; aqui eles aparecem preenchidos.

```markdown
### 04 · `estrutura`

**Papel visual:** mostrar que a estrutura existe sem afirmar equipamento que o cliente pode não
ter. O assunto concreto é a sala, não o aparelho de marca.

**Proporção:** 16:9 full-bleed  ·  **Salvar em:** `design/renders/04-estrutura.png`

**ANEXAR no ChatGPT, nesta ordem, no mesmo envio do prompt:**

| # | Arquivo | Serve de referência para | Não serve para |
|---|---|---|---|
| 1 | `assets-source/logo-belezacompleta.png` | cor exata da marca (#4E2A96, #FCB400) | desenhar o logo na cena |
| 2 | `assets-source/post-banho-tosa.jpg` | tratamento de foto: contraste, saturação, temperatura | enquadramento nem assunto |
| 3 | `assets-source/salao-interior.jpg` | material e luz reais do lugar | reproduzir esta sala |

**Frase de anexo (colar depois do prompt):**

    Use the attached images as reference for color, light and material treatment only.
    Do not copy their framing, composition, crop or subject placement.
    Do not reproduce any logo, mark, lettering or symbol seen in them.

**Prompt completo:**

    [STYLE ANCHOR verbatim]

    SUBJECT: A quiet grooming room seen from the doorway, with nobody in it.
    COMPOSITION: The stainless table runs across the lower third, level and centered. The
    upper half is one calm unbroken wall of violet paint — that band is the negative space
    the eyebrow and headline sit on. Keep the table surface clear of props.
    DETAIL: A cream tiled wall holding a soft reflection of one amber wall lamp, a coiled
    hose on a powder-coated violet hook, a folded towel at the far edge of the table.
    FRAMING: Eye-level frontal medium-wide, camera square to the table, no tilt, 50mm.
    EXCLUDE: [bloco base] + bottles with labels, wall posters, screens, clocks, price lists.

**Onde cai a copy:** eyebrow e H1 na faixa superior (parede); três cards sobre a faixa central.

**Teste de aceitação — regere se:** aparecer qualquer letra, número ou rótulo; a câmera não
estiver frontal e nivelada; a imagem reproduzir o crop do post anexado; o roxo virar lavagem
sobre a cena inteira em vez de estar em objetos.
```

## Seção sem imagem — também é um bloco

Bloco vazio parece esquecimento e alguém preenche com gradiente. Declare:

```markdown
### 09 · `depoimentos` — SEM IMAGEM, DECLARADO

Não existe assunto concreto: o conteúdo é nome de pessoa e frase. Foto ilustrativa por baixo de
depoimento sugere que aquela é a pessoa que falou, o que é falso. Carrega em tipografia grande
sobre a cor de superfície.
```

## Ordem do arquivo

1. O Style Anchor, uma vez, no topo, dentro de um bloco de código para copiar sem reformatar.
2. Um bloco por seção, na ordem em que serão colados nas ferramentas — a mesma ordem das seções
   no story map da Fase 6.
3. Seções sem imagem no lugar delas na sequência, não no fim: a ordem do arquivo é o roteiro de
   trabalho do dev, e um bloco fora de lugar é uma seção que ele pula.
