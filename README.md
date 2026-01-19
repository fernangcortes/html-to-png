# html-to-png

📱 HTML para PNG Mobile

Uma ferramenta web client-side poderosa para converter tabelas, agendas e layouts HTML em imagens PNG de alta fidelidade. Otimizada para formatos mobile (400px), ideal para compartilhamento em redes sociais (WhatsApp, Instagram Stories).

Acesse a ferramenta: Basta abrir o arquivo index.html em qualquer navegador moderno.

✨ Funcionalidades Principais

🎨 Motor de Renderização SVG (High Fidelity)

Utiliza a biblioteca dom-to-image-more para renderizar o HTML dentro de um container SVG (foreignObject).

O que faz: Garante que fontes, ícones (FontAwesome), sombras e espaçamentos sejam reproduzidos exatamente como aparecem no navegador, superando as limitações do antigo html2canvas.

👁️ Qualidade Retina (2x Scaling)

O motor de exportação é configurado nativamente com scale: 2.

O que faz: Gera imagens com o dobro da densidade de pixels, garantindo nitidez cristalina em telas de alta resolução (Retina/Super AMOLED) sem serrilhados nos textos.

✂️ Fatiamento Manual (Custom Slicing)

Interface visual interativa para dividir imagens verticais longas.

O que faz: Permite clicar na imagem para definir linhas de corte exatas. O sistema garante que você nunca corte uma linha de texto ou um card ao meio, respeitando o contexto do seu conteúdo.

📦 Empacotamento Inteligente (Auto-ZIP)

Integração com a biblioteca JSZip.

O que faz: Ao exportar cortes múltiplos, a ferramenta não enche sua pasta de downloads com vários arquivos. Ela agrupa todas as fatias (fatia_001.png, fatia_002.png) em um único arquivo .zip organizado.

📅 Nomenclatura Dinâmica

Gerador automático de nomes de arquivo baseado na data do sistema.

O que faz: Exporta arquivos seguindo o padrão jogos_do_dia-DD-MM-AAAA-FGC, eliminando a necessidade de renomear arquivos manualmente após o download.

🌙 Modo Escuro Automático (Dark Mode)

Suporte nativo a temas de sistema via Tailwind CSS.

O que faz: Detecta se o dispositivo do usuário está em modo claro ou escuro e adapta toda a interface (editor, botões, modais) para cores confortáveis, mantendo a área de preview (o "papel") branca para fidelidade de impressão.

🔒 Sanitização de CSS (CORS Fix)

Sistema de injeção automática de atributos de segurança.

O que faz: Ao colar o código, a ferramenta varre links de CSS (como CDNs de fontes) e injeta crossorigin="anonymous". Isso previne o bloqueio de renderização por políticas de segurança do navegador (CORS Policy).

💾 Salvar App (Portable Tool)

Funcionalidade de auto-replicação limpa.

O que faz: Permite baixar o próprio código-fonte da ferramenta como um arquivo HTML único e limpo (sem os dados que você digitou), facilitando o backup ou a distribuição da ferramenta para outros usuários.

🛠️ Tecnologias Utilizadas

HTML5 & JavaScript ES6+: Núcleo da aplicação, sem frameworks pesados.

Tailwind CSS (CDN): Estilização utilitária rápida e responsiva.

dom-to-image-more: Engine de captura de imagem via SVG.

JSZip: Compressão de arquivos no navegador.

FontAwesome: Ícones vetoriais para a interface.

🚀 Como Usar

Cole: Insira seu código HTML na aba Editor.

Visualize: Alterne para a aba Preview para ver como ficará (largura fixa em 400px).

Exporte:

Baixar PNG: Gera uma imagem única contínua.

Cortes: Entre no modo de corte, clique na imagem para marcar as divisões e baixe o ZIP.

📝 Licença e Créditos

Desenvolvido para uso livre e local.
Desenvolvido por FGC
