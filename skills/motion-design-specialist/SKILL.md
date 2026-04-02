---
name: motion-design-specialist
description: "Especialista em Motion Design para interfaces web, landing pages, product marketing pages e fluxos de produto. Use quando precisar auditar uma interface existente, definir uma estrategia de motion por secao, componente ou etapa, escolher entre CSS, WAAPI, JS vanilla, GSAP ou motion nativo do framework, e implementar animacoes suaves, intencionais, acessiveis, mobile-aware e performance-conscious sem prejudicar legibilidade, usabilidade ou conversao."
---

# Motion Design Specialist

Projete motion como parte da UX, nao como decoracao. Audite a interface, defina um blueprint por area e implemente a menor solucao tecnica capaz de entregar clareza, feedback e ritmo.

## Workflow

1. Auditar a interface antes de animar.
2. Construir um blueprint de motion por secao, componente ou estado.
3. Escolher a stack mais leve que resolva o problema.
4. Implementar com foco em legibilidade, acessibilidade e performance.
5. Validar mobile, reduced motion e clareza do fluxo.

## Auditar

- Mapear secoes, componentes, estados, transicoes e pontos de leitura.
- Identificar o objetivo de negocio, o CTA principal e o comportamento esperado do usuario.
- Detectar a stack atual, animacoes existentes e restricoes de performance.
- Preservar o design system existente; nao reinventar a linguagem visual sem motivo.

## Construir o Blueprint

- Definir o papel do motion em cada area: hierarquia, reveal, feedback, guidance, emphasis ou continuidade.
- Registrar para cada area: objetivo do conteudo, objetivo do motion, trigger, comportamento em desktop, comportamento em mobile, fallback de reduced motion, complexidade e abordagem tecnica.
- Usar [references/analysis-template.md](references/analysis-template.md) quando houver varias secoes, storytelling ou muitos componentes.

## Escolher a Stack

- Ler [references/stack-selection.md](references/stack-selection.md) antes de adicionar qualquer dependencia.
- Preferir CSS para hover, focus, state changes e entradas simples.
- Usar `IntersectionObserver` para reveals e stagger no scroll.
- Usar WAAPI ou JS pequeno para coordenacao mais precisa sem trazer biblioteca pesada.
- Preferir motion nativo do framework quando o projeto ja usa esse caminho.
- Usar GSAP ou ScrollTrigger apenas quando scrub, pinning ou timelines sincronizadas forem realmente necessarios.

## Implementar

- Preferir `transform` e `opacity`.
- Evitar propriedades que forcam layout ou paint pesado.
- Reduzir distancia, duracao e simultaneidade em mobile.
- Isolar tokens de motion com variaveis CSS ou utilitarios previsiveis.
- Garantir `prefers-reduced-motion` e um estado totalmente funcional sem animacao.

## Validar

- Verificar legibilidade, foco, ordem de leitura e clareza do CTA.
- Testar resize, mobile real, hover inexistente, navegacao por teclado e reduced motion.
- Remover qualquer efeito que pareca decorativo, lento ou confuso.

## Heuristics

- Motion deve explicar hierarquia, continuidade ou feedback.
- Animar menos elementos costuma produzir melhor resultado.
- Stagger deve guiar o olho, nao teatralizar a tela.
- Parallax deve ser sutil, raro e opcional.
- Storytelling no scroll so faz sentido quando a narrativa depende disso.
- Loops persistentes competindo com o conteudo normalmente sao erro.
- `blur`, `filter`, `clip-path` complexo e sombra animada exigem justificativa.
- Smooth-scroll library nao eh default.

## Deliverable Shape

Quando o pedido envolver analise e execucao, responder nesta ordem:

1. audit summary
2. motion blueprint por secao, componente ou estado
3. stack choice and rationale
4. code changes
5. QA checklist

## References

- Ler [references/motion-principles.md](references/motion-principles.md) para principios de direcao.
- Ler [references/analysis-template.md](references/analysis-template.md) para estruturar a analise.
- Ler [references/stack-selection.md](references/stack-selection.md) para decidir a implementacao.
- Ler [references/implementation-patterns.md](references/implementation-patterns.md) para patterns seguros.
- Ler [references/specialists-map.md](references/specialists-map.md) quando precisar combinar esta skill com outras.
