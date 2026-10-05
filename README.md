# Displasia de quadril — Estadiamento e risco de progressão

Projeto pessoal de pesquisa em IA aplicada a raio-x de quadril com displasia. Objetivo: estimar o estágio atual da condição e o risco/tempo de progressão para osteoartrose (OA), com foco em ajudar a identificar o momento certo de intervenção e melhorar a qualidade de vida a longo prazo dos pacientes.

**Não é uma ferramenta diagnóstica e não substitui avaliação médica.**

Para o detalhamento de requisitos, fases, marcos, riscos e trade-offs, ver [ROADMAP.md](./ROADMAP.md).

## Stack técnico

### Linguagem principal: Python

Processamento de imagem médica, visão computacional e modelagem estatística (análise de sobrevivência/risco) são dominados pelo ecossistema Python — não é preferência, é onde as ferramentas maduras existem (OpenCV, scikit-image, PyTorch, lifelines). O stack de projetos pessoais do autor (TypeScript/Next.js/NestJS) não se aplica aqui: é um projeto de pesquisa/ciência de dados, não uma aplicação web.

### Por fase

**M0 — Fundamentação**
Nenhuma ferramenta técnica além de Markdown/texto.

**M1 — Detecção de landmarks + cálculo de ângulos**

Duas abordagens possíveis, com trade-off real entre elas:
- **Visão computacional clássica (OpenCV)**: pouco dado necessário, resultado totalmente interpretável, mais frágil a variação anatômica entre pacientes.
- **Deep learning para keypoint detection (CNN com transfer learning)**: é o padrão usado pelos datasets/papers de referência (anotações em formato COCO), mas exige mais dado e cuidado para não overfitar nos datasets pequenos disponíveis.

**Recomendação**: começar pela abordagem clássica (OpenCV) como baseline rápido e debugável, evoluindo para deep learning apenas quando o baseline mostrar limitações concretas nos dados reais — evita assumir complexidade antes de entender o problema.

Se/quando for necessário deep learning: **PyTorch** (padrão de fato em pesquisa de imagem médica) e, se fizer sentido, **MONAI** (framework sobre PyTorch especializado em imagem médica — lida com DICOM, aumentos de dado apropriados para radiografia, etc.).

**M3 — Modelo de risco de progressão**

**lifelines** (biblioteca Python de análise de sobrevivência — Kaplan-Meier, Cox proportional hazards), o mesmo tipo de modelo usado nos estudos de referência (Hasegawa et al., coorte CHECK).

**Dados tabulares (M1-M4)**: pandas + numpy. Sem banco de dados nesse estágio — poucas centenas/milhares de parâmetros extraídos cabem em CSV/DataFrame; introduzir um banco agora seria complexidade sem necessidade.

**M5 — Apresentação do resultado**

Decisão a tomar apenas quando chegar nessa fase: notebook/relatório simples, ou Streamlit/Gradio (prototipagem rápida de interface em Python), ou — se fizer sentido integrar com o stack pessoal — uma API Python consumida por um front em Next.js.

### Infraestrutura e práticas

- **Ambiente**: `venv` + `requirements.txt` (ou `poetry`). Sem Docker/Kubernetes nesta fase — complexidade sem motivo para um projeto solo.
- **Compute**: Google Colab ou Kaggle Notebooks (GPU gratuita) para qualquer experimento de deep learning do M1/M2, sem necessidade de hardware próprio.
- **Jupyter notebooks** para exploração (M1-M4); promover para scripts `.py` apenas o que estabilizar.
- **Dados**: datasets de imagem (mesmo públicos) não são commitados no repositório — documentar fonte/script de download em `data/README.md` e usar `.gitignore`.
- **Testes**: pytest para as funções de cálculo de ângulo/classificação, validadas contra valores de referência conhecidos (ex: casos de exemplo dos papers).
- **Fora de escopo por agora**: MLOps (MLflow, DVC), CI/CD, deploy em nuvem — overengineering para a fase atual (M0-M4). Revisitar se o projeto crescer em escala de dados ou colaboradores.
