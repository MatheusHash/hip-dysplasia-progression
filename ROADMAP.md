# Displasia de quadril — Estadiamento e risco de progressão

Projeto pessoal de pesquisa em IA aplicada a raio-x de quadril com displasia. Objetivo: estimar o estágio atual da condição e o risco/tempo de progressão para osteoartrose (OA), com foco em melhorar a qualidade de vida a longo prazo dos pacientes ajudando a identificar o momento certo de intervenção.

Não é uma ferramenta diagnóstica e não substitui avaliação médica.

## 1. Requisitos

**Funcionais (o que o sistema faz, nesta fase)**
- Receber um raio-x de quadril (AP de pelve) e extrair parâmetros radiográficos (CE angle, índice acetabular, etc.).
- Classificar a gravidade atual da displasia a partir desses parâmetros.
- Estimar risco/tempo esperado de progressão para OA, para pacientes pós-maturidade esquelética, com base em modelos publicados na literatura clínica.
- Produzir uma saída interpretável (números + classificação + faixa de risco), não uma imagem.

**Não-funcionais / limites explícitos**
- Não é ferramenta diagnóstica — resultado é exploratório/educacional, sempre com disclaimer.
- Precisa ser auditável: cada número que o sistema produz deve ser rastreável a uma fórmula/fonte da literatura, não uma "caixa preta" sem explicação.
- Não depende de dataset de imagem longitudinal de adulto (não existe disponível publicamente).

**Restrições**
- Projeto solo, sem orientador, sem dataset próprio ainda, sem prazo formal (projeto pessoal, separado do TCC).
- Sem acesso a validação clínica de especialista no momento — todo "ground truth" vem de literatura publicada, não de julgamento próprio.

## 2. Visão geral do pipeline

```
Raio-x (entrada)
     │
     ▼
[Fase 1] Detecção de landmarks → cálculo de ângulos → estadiamento
     │
     ▼
[Fase 2] Parâmetros do paciente + coeficientes de risco da literatura
     │         → estimativa de risco/tempo de progressão para OA
     ▼
Saída: estágio atual + risco estimado (número/faixa, com fonte citada)

     ╎ (futuro, não agora)
     ╎
     ▼
[Fase 3 — deferida] Visualização da progressão prevista sobre a imagem original
```

## 3. Roadmap (marcos, não datas — dado que dataset/acesso é incerto)

### M0 — Fundamentação teórica
- Formalizar por escrito: quais parâmetros radiográficos serão usados (CE angle, índice acetabular, outros), quais classificações (Tönnis OA 0-3, etc.), e quais estudos servirão de base de risco (Hasegawa et al., coorte CHECK, outros que surgirem).
- Entregável: documento de referência técnica (não código) com fórmulas/definições e fontes.
- Critério de pronto: conseguir explicar, pra alguém de fora, de onde vem cada número que o sistema produz.

### M1 — Pipeline de estadiamento, provado em dataset pediátrico
- Usar datasets públicos (MTDDH, dataset da Jordânia) como teste de método, mesmo sendo população diferente do foco final (adulto).
- Objetivo: validar que a detecção de landmarks + cálculo de ângulos funciona e dá números consistentes com as anotações do dataset.
- Critério de pronto: erro entre ângulo calculado pelo pipeline e ângulo anotado dentro de margem clinicamente aceitável (limiar a definir com base na literatura).

### M2 — Adaptação/validação em contexto adulto
- Buscar imagens de adulto disponíveis (mesmo em pequena quantidade) + raio-x próprio como estudo de caso.
- Avaliar se o pipeline de landmarks/ângulos generaliza pra anatomia adulta ou precisa de ajuste.
- Decisão em aberto: avaliar contato com grupo de pesquisa universitário para acesso a mais dados — revisitar essa decisão só depois do M1, com protótipo funcionando para mostrar.

### M3 — Modelo de estimativa de risco de progressão
- A partir dos parâmetros extraídos (M1/M2) + coeficientes publicados (ex: CE angle < 25° e risco/tempo de progressão), construir o estimador de risco.
- Técnica a avaliar quando os dados concretos estiverem em mãos (regressão, modelo de sobrevivência tipo Cox, regras por limiares da literatura) — não comprometer antes disso.
- Critério de pronto: para um paciente hipotético com parâmetros X, o sistema produz estimativa de risco na mesma ordem de grandeza reportada nos estudos de referência (ex: sobrevida de 83,7% em 10 anos do Hasegawa et al., para perfil de risco equivalente).

### M4 — Consolidação e validação cruzada com literatura
- Testar o pipeline completo (M1→M3) contra casos descritos em papers (parâmetros do paciente + desfecho real) como validação indireta, na ausência de coorte própria.
- Entregável: relatório de validação — onde o sistema acerta, onde erra, e por quê.

### M5 — Apresentação do resultado
- Definir a interface de saída (relatório, painel simples, etc.) — decisão de implementação, não de arquitetura de IA; pode ficar para depois dos modelos estarem prontos.

### M6 (futuro, fora do MVP) — Fase 3: visualização de progressão
- Só entra depois do M4 validado. Pré-requisito: modelo de risco confiável, já que a visualização seria derivada desse risco (deformação guiada da imagem original), não gerada do zero.

## 4. Riscos e dependências

| Risco | Impacto | Mitigação |
|---|---|---|
| Nenhum dataset de adulto disponível | Bloqueia M2 | Usar literatura (coeficientes publicados) em vez de treinar modelo próprio; raio-x próprio como sanity-check |
| Sem validação clínica de especialista | Resultado pode estar sutilmente errado sem ninguém notar | Validação indireta via casos de papers (M4); deixar claro nas limitações do projeto |
| Projeto solo, sem prazo formal | Risco de nunca "fechar" fases | Marcos com critério de pronto explícito, não data |
| Confundir classificações (pediátrica vs. adulta) | Comprometeria M1→M2 se usar fórmula errada | Resolvido no M0: documentar explicitamente qual classificação se aplica onde |

## 5. Trade-offs explícitos

- **Precisão própria vs. literatura publicada**: risco de progressão ancorado em modelos estatísticos publicados em vez de treinar algo do zero. Menos "autoral", mas é a única opção cientificamente honesta sem dataset longitudinal próprio.
- **Validar em pediátrico primeiro**: ganha velocidade de iteração (dataset existe), mas adiciona um passo de verificação de generalização para adulto no M2.
- **Deixar Fase 3 de fora agora**: reduz escopo e risco do projeto nunca ter um resultado validado, mas atrasa o entregável visualmente mais impactante.

## 6. O que revisitar conforme o projeto maturar

- Depois do M1 com resultado concreto: decidir se busca parceria universitária para dados de adulto.
- Depois do M3 validado: só então escolher a técnica de visualização da Fase 3 (deformação de imagem guiada por parâmetros vs. outra abordagem).
- Se em algum momento envolver dado de paciente real (além do próprio): avaliar CEP/ética antes de qualquer coisa, mesmo que seja só para validação.

## Referências levantadas até agora

- [A dataset for quality evaluation of pelvic X-ray and diagnosis of developmental dysplasia of the hip (MTDDH)](https://www.nature.com/articles/s41597-025-05146-x)
- [Detection of developmental dysplasia of the hip in X-ray images using deep transfer learning](https://pmc.ncbi.nlm.nih.gov/articles/PMC9375244/)
- [Hip Dysplasia Diagnosed After Skeletal Maturity: Factors Associated With Progression to Osteoarthritis (Hasegawa et al.)](https://pmc.ncbi.nlm.nih.gov/articles/PMC11726487/)
- [Hip dysplasia as risk factor for clinically relevant and radiographic hip osteoarthritis: 10-year results from the CHECK cohort](https://academic.oup.com/rheumatology/article/64/1/149/7459912)
- [The lateral center-edge angle as radiographic selection criteria for periacetabular osteotomy](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC7382803/)
