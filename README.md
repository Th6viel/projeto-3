# 🌐 MyDashboards — Projeto 3
**Curso:** Design de Interação / Multimédia  
**Instituição:** ISTEC Porto  
**Unidade Curricular:** Computação Orientada a Dados e Sistemas Interativos Web  

---

## 📋 Descrição Geral

Este repositório contém dois dashboards interativos desenvolvidos em **p5.js**, que integram APIs públicas externas para visualizar e manipular dados em tempo real. O projeto explora comunicação assíncrona em JavaScript, tratamento de dados JSON e pós-processamento de imagem no canvas.

**Estrutura do repositório:**
```
MyDashboards/
├── index.html        → Página principal / portal de entrada
├── dashboard1.html   → World Pulse (painel informativo)
├── dashboard2.html   → Glitch Studio (painel de edição)
├── world_map.jpg     → Mapa-múndi em projeção equiretangular
├── iss_icon.png      → Ícone da ISS
└── README.md         → Este ficheiro
```

---

## 🚀 Como Correr Localmente

1. Instalar a extensão **Live Server** no VS Code
2. Clique direito em `index.html` → **"Open with Live Server"**
3. Abre em `http://127.0.0.1:5500`

> ⚠️ **Não abrir os ficheiros diretamente pelo browser** (`file://`) — as chamadas fetch() às APIs são bloqueadas por razões de segurança. O Live Server resolve este problema simulando um servidor local.

---

## 📊 Dashboard 1 — World Pulse

**Ficheiro:** `dashboard1.html`  
**Tipo:** Painel Informativo (Visual Non-Interactive)

### Descrição
Mapa-múndi dinâmico que visualiza dados meteorológicos, sísmicos e de rastreamento espacial em tempo real. O p5.js mapeia os dados numéricos das APIs em elementos visuais no canvas — temperatura em cor e frequência de pulso, sismos em ondas de expansão, e a ISS como ponto em movimento real.

### APIs Utilizadas

#### 1. Open-Meteo — Meteorologia
| | |
|---|---|
| **Endpoint** | `https://api.open-meteo.com/v1/forecast` |
| **Parâmetros** | `current=temperature_2m,relativehumidity_2m,windspeed_10m` |
| **Parâmetros** | `hourly=temperature_2m&forecast_days=1` |
| **Autenticação** | Nenhuma — sem chave de API |
| **Limites** | ~10.000 req/dia por IP. Dados atualizados cada 15 minutos |
| **Atualização** | A cada 10 minutos no dashboard |

**Dados obtidos:** temperatura atual (°C), humidade relativa (%), velocidade do vento (km/h) e variação horária de temperatura para as 24h do dia atual — para 10 cidades em simultâneo via `Promise.all()`.

**Comparação com alternativas:**
| API | Chave | Limite Free | CORS | Qualidade |
|---|---|---|---|---|
| **Open-Meteo** ✓ | Não | ~10.000/dia | ✅ Aberto | ECMWF + GFS |
| OpenWeatherMap | Sim | 1.000/dia | ✅ | Boa |
| WeatherAPI | Sim | 1.000.000/mês | ✅ | Boa |
| Meteostat | Sim | 2.000/mês | ✅ | Boa |

**Motivo da escolha:** A Open-Meteo é a única API meteorológica de qualidade profissional que não exige registo nem chave de API, o que a torna ideal para projetos publicados publicamente no GitHub Pages sem expor credenciais.

---

#### 2. USGS Earthquake Hazards Program — Sismos
| | |
|---|---|
| **Endpoint** | `https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/2.5_day.geojson` |
| **Formato** | GeoJSON |
| **Autenticação** | Nenhuma — feed público |
| **Limites** | Sem limite documentado. Feed atualizado cada 5 minutos |
| **Atualização** | A cada 10 minutos no dashboard |

**Dados obtidos:** longitude, latitude, profundidade (km), magnitude e localização textual de todos os sismos ≥ 2.5 nas últimas 24h a nível global.

**Nota sobre visualização:** O feed regista tipicamente entre 20 a 80 sismos por dia — a maioria de magnitude 2.5 a 3.9, imperceptíveis ao ser humano mas detetáveis por sismógrafos. O toggle de ondas permite ao utilizador controlar a animação.

**Comparação com alternativas:**
| API | Chave | Cobertura | CORS |
|---|---|---|---|
| **USGS** ✓ | Não | Global | ✅ |
| EMSC | Não | Europa/Mediterrâneo | ✅ |
| Iris Earthquake Science | Não | Global | ✅ |

**Motivo da escolha:** A USGS tem cobertura global, formato GeoJSON padronizado, é operada pelo governo dos EUA e é considerada a referência mundial em dados sísmicos.

---

#### 3. WhereTheISS — Posição da Estação Espacial Internacional
| | |
|---|---|
| **Endpoint** | `https://api.wheretheiss.at/v1/satellites/25544` |
| **Parâmetro** | `25544` = NORAD ID da ISS |
| **Autenticação** | Nenhuma — sem chave de API |
| **Limites** | Sem limite documentado |
| **Atualização** | A cada 5 segundos no dashboard |

**Dados obtidos:** latitude, longitude, altitude (km) e velocidade (km/h) da ISS em tempo real.

**Fallback orbital:** Quando a API não responde, a posição é calculada matematicamente com base no período orbital (92.68 min) e inclinação (51.64°) da ISS. Não é astronomicamente exato mas é visualmente coerente.

**Comparação com alternativas:**
| API | HTTPS | Chave | CORS |
|---|---|---|---|
| **WhereTheISS** ✓ | ✅ | Não | ✅ |
| Open Notify | ❌ HTTP | Não | ✅ |
| N2YO | ✅ | Sim | ✅ |

**Motivo da escolha:** A Open Notify (a mais conhecida) usa HTTP em vez de HTTPS, o que causa erros de segurança em páginas modernas. A WhereTheISS oferece HTTPS e CORS abertos sem chave.

### Funcionalidades
- 🌍 Mapa-múndi com imagem real em projeção equiretangular com tint escuro
- 🌡️ 10 cidades com pulso reativo à temperatura (cor + frequência)
- 📊 Clique numa cidade → painel esquerdo com gráfico SVG de variação horária (24h), mínima e máxima do dia
- ⚡ Sismos com ondas de expansão animadas (toggle on/off) e painel de detalhes ao clique
- 🛰️ ISS a mover-se em tempo real com ícone e coordenadas
- 🗂️ Menu de cidades clicável no topo para acesso rápido

### Decisões Técnicas
- `Promise.all()` para pedidos paralelos às 10 cidades — reduz tempo de carregamento de ~10s para ~2s
- `frameRate(24)` em vez de 60 para otimização em hardware com menos recursos
- Dados mock com fallback automático — dashboard funciona mesmo sem acesso às APIs
- Projeção equiretangular simples: `x = (lon+180)/360*W`, `y = (90-lat)/180*H`

---

## 🎨 Dashboard 2 — Glitch Studio

**Ficheiro:** `dashboard2.html`  
**Tipo:** Painel de Edição e Geração Criativa (Interactive)

### Descrição
Ferramenta de edição criativa que permite pesquisar fotografias profissionais por palavras-chave e aplicar filtros de pós-processamento diretamente na matriz de píxeis do canvas p5.js. Explora o conceito de glitch art através de manipulação algorítmica de imagem.

### API Utilizada

#### Unsplash — Repositório de Fotografia Profissional
| | |
|---|---|
| **Endpoint** | `https://api.unsplash.com/photos/random` |
| **Parâmetros** | `query={termo}&orientation=landscape&client_id={ACCESS_KEY}` |
| **Autenticação** | Access Key gratuita (registo em unsplash.com/developers) |
| **Limites** | 50 req/hora em modo Demo |
| **Formato** | JSON com URLs da imagem em múltiplas resoluções |

**Dados obtidos:** URL da imagem em resolução `regular` (~1080px), nome do autor e metadados da fotografia.

**Nota sobre a chave de API:** A Access Key está implementada como input do utilizador em runtime — não está hardcoded no código. Isto garante que o repositório público não expõe credenciais. A Secret Key da Unsplash não é utilizada — é apenas necessária para fluxos OAuth (autenticação de utilizadores), que não é o caso deste projeto.

**Comparação com alternativas:**
| API | Chave | Tipo | CORS | Qualidade |
|---|---|---|---|---|
| **Unsplash** ✓ | Sim (gratuita) | Fotografia profissional | ✅ | Alta |
| Pexels | Sim (gratuita) | Stock photos | ✅ | Alta |
| Pollinations.ai | Não* | Geração IA | ✅* | Variável |
| DALL-E (OpenAI) | Sim (pago) | Geração IA | ✅ | Muito alta |
| Stability AI | Sim (pago) | Geração IA | ✅ | Alta |

> *A Pollinations.ai foi inicialmente considerada por ser gratuita e sem chave, mas revelou instabilidade frequente (erros 500) durante o desenvolvimento e passou a requerer autenticação Bearer. A Unsplash foi escolhida pela fiabilidade e por estar explicitamente referida no enunciado como opção válida.

**Motivo da escolha:** A Unsplash oferece fotografias de alta qualidade que servem como excelente material base para o pós-processamento artístico. O enunciado menciona explicitamente "repositórios de fotografia profissional com base em tags de pesquisa (Unsplash API)" como exemplo válido para o Dashboard 2.

### Filtros de Pós-Processamento (pixels[])

Todos os filtros manipulam diretamente o array `pixels[]` do p5.js — sem bibliotecas externas de processamento de imagem.

| Filtro | Algoritmo | Intensidade |
|---|---|---|
| **Chromatic Aberration** | Deslocamento horizontal dos canais R e B em direções opostas | 0–20px de offset |
| **Scanline Glitch** | Deslocamento determinístico de linhas horizontais com `sin(y)` | 0–10 níveis |
| **Wave Distortion** | Deslocamento horizontal por função seno: `shift = sin(y * 0.04) * amplitude` | 0–20px amplitude |
| **Dither / Posterize** | Redução de paleta: `round(valor / step) * step` por canal RGB | 2–8 níveis de cor |
| **Pixelate** | Cor média de blocos NxN aplicada a todos os píxeis do bloco | 1–30px por bloco |

**Paint Mode:** Pincel de distorção local — arrastar o rato sobre a imagem dispersa píxeis de forma aleatória na área sob o cursor (raio 40px), acumulando o efeito a cada passagem.

### Funcionalidades
- 🔍 Pesquisa de imagens por palavras-chave em inglês
- ⚙️ 5 filtros com controlos `[−] valor [+]` numa toolbar inferior
- 🖌️ Paint Mode — pincel de glitch aplicado localmente
- ↺ Reset para restaurar imagem original
- 💾 Export PNG do canvas atual

### Decisões Técnicas
- `img.get()` cria uma cópia independente da imagem original — os filtros são sempre aplicados sobre o original, nunca acumulados sobre versões já filtradas
- Flag `needsProcess` evita recalcular filtros a cada frame — só recalcula quando um valor muda
- `new Uint8ClampedArray(px)` como snapshot antes de filtros que lêem e escrevem píxeis em simultâneo (Wave, Chromatic) — evita artefactos de leitura de pixels já modificados

---

## ⚙️ Requisitos Técnicos

- Browser moderno (Chrome, Firefox, Edge)
- Live Server (desenvolvimento local) ou GitHub Pages (produção)
- Unsplash Access Key para o Dashboard 2 (gratuita em unsplash.com/developers)

## 📸 Capturas de Ecrã



---

## 📝 Notas de Segurança

A Access Key da Unsplash é introduzida pelo utilizador em runtime e nunca está armazenada no código ou no repositório. Em produção real, a solução ideal seria um servidor backend (ex: Netlify Functions) que fizesse os pedidos à API sem expor a chave no frontend.

---
