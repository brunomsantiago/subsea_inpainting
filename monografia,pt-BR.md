# Subsea Inpainting - Removendo overlay de vídeos de inspeção submarina

#### Aluno: [Bruno Santiago](https://github.com/garaujo94/)
#### Orientador: [Leonardo Mendoza](https://github.com/leofome8)

---

Trabalho apresentado ao curso [BI MASTER](https://ica.puc-rio.ai/bi-master) como pré-requisito para conclusão de curso e obtenção de crédito na disciplina "Projetos de Sistemas Inteligentes de Apoio à Decisão".

- [Repositório principal do trabalho](https://github.com/brunomsantiago/subsea_inpainting)
- [Conjunto de Dados - *Subsea Inpainting Dataset*](https://www.kaggle.com/brunomsantiago/subsea-inpainting-dataset)
- [Biblioteca de visualização - viajen](https://github.com/brunomsantiago/viajen)
---

### Resumo

<p align="justify">A indústria de Óleo e Gás gera centenas de horas de vídeos de inspeção submarina por dia. As grandes operadoras de petróleo possuem milhões de horas de vídeos armazenadas. Esses vídeos são filmados por véiculos remotos chamados ROVs (<i>Remote Operated Vehicles</i>) e são cruciais para o gerenciamento de integridade de ativos submarinos como dutos e ANMs (Árvores de Natal Molhadas), permitindo que as empresas operem seus campos de petróleo marítimos com segurança. Há um grande potencial de geração de valor com usos secundários desses vídeos, o que demanda processamento por técnicas visão computacional e inteligêcia artificial.</p>

<p align="justify">A maioria desses vídeos possuem metadados embutidos nas imagems, informações como data, hora e coordenadas de posição e direção do ROV. Por um lado, o fato desses metados serem embutidos garante que eles seguiram junto as imagens mesmo após operações como extração de frames ou edição de clipes. No entando, os metadados obstruem parte significativa da imagem, o que pode dificultar a visualização pelos usuários e confundir algortimos de processamento de imagem.</p>

<p align="justify">O objetivo principal deste trabalho é avaliar se métodos atuais de preenchimento de imagem estão prontos para remover os metados embutidos. Foram avaliados quatro métodos, sendo três deles baseados em redes neurais profundas. Como passo intermediário desta avaliação também foram realizadas outras duas contribuições. Um conjunto de dados com treze pequenos clipes de vídeos submarinos (<i>Subsea Inpainting Dataset</i>) e uma biblioteca em Python para visualização de clipes de vídeos em ambientes interativos de programação (<i>viajen - View Images as Animation in Jupyter and Equivalent Notebooks</i>).</p>

---

Matrícula: 192.190.096

Pontifícia Universidade Católica do Rio de Janeiro

Curso de Pós Graduação *Business Intelligence Master*
