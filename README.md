📱 FGC Mobile Export Tool

Uma ferramenta web leve e poderosa para converter código HTML em imagens PNG de alta fidelidade, otimizada para layouts mobile (400px).

O projeto resolve problemas comuns de renderização de fontes e cortes de layout utilizando uma engine baseada em SVG (dom-to-image-more) e oferece funcionalidades avançadas como fatiamento manual de imagens longas.

✨ Funcionalidades Principais

Motor de Renderização SVG: Utiliza dom-to-image-more para garantir que fontes, ícones e espaçamentos sejam idênticos ao navegador, corrigindo falhas do html2canvas.

Qualidade Retina (2x): Exporta imagens com o dobro da densidade de pixels para nitidez perfeita em telas de alta resolução.

Fatiamento Manual (Slicing): Interface visual para definir onde a imagem deve ser cortada, ideal para conteúdos longos (ex: tabelas de jogos) que não podem ser cortados no meio de um card.

Empacotamento ZIP: Gera automaticamente um arquivo .zip contendo todas as fatias nomeadas sequencialmente.

Nomeação Automática: Gera nomes de arquivos baseados na data atual (ex: jogos_do_dia-18-01-2026-FGC).

Correção de Segurança (CORS): Sanitização automática de links CSS (como FontAwesome) para evitar erros de segurança ao exportar.

📚 Biblioteca de Snippets (Ferramentas Extraídas)

Abaixo estão as funções principais isoladas para sua biblioteca pessoal de desenvolvimento.

1. Motor de Renderização de Alta Fidelidade (DOM -> Canvas)

Esta função substitui o html2canvas padrão. Ela converte o DOM em SVG e depois em Canvas, preservando a renderização exata do navegador.

Dependência: dom-to-image-more.js

/**
 * Converte um nó DOM em um elemento Canvas usando renderização SVG.
 * Resolve problemas de glifos cortados e posicionamento incorreto de fontes.
 * * @param {HTMLElement} domNode - O elemento HTML a ser renderizado.
 * @param {number} width - Largura base da imagem (ex: 400).
 * @param {number} height - Altura total do conteúdo.
 * @param {number} scale - Fator de escala (Padrão: 2 para telas Retina/HD).
 * @returns {Promise<HTMLCanvasElement>} Retorna uma Promise com o objeto Canvas.
 */
async function generateHighFidelityCanvas(domNode, width, height, scale = 2) {
    // Configuração crítica para garantir a escala correta sem borrar
    const config = {
        width: width * scale,
        height: height * scale,
        style: {
            transform: `scale(${scale})`, // Força o navegador a desenhar maior
            transformOrigin: 'top left',
            width: `${width}px`,
            height: `${height}px`
        }
    };

    try {
        // 1. Converte DOM para DataURL (PNG base64)
        const dataUrl = await domtoimage.toPng(domNode, config);
        
        // 2. Carrega a imagem em memória
        const img = new Image();
        img.src = dataUrl;
        await new Promise(resolve => img.onload = resolve);

        // 3. Desenha a imagem em um Canvas real para manipulação posterior
        const canvas = document.createElement('canvas');
        canvas.width = width * scale;
        canvas.height = height * scale;
        const ctx = canvas.getContext('2d');
        ctx.drawImage(img, 0, 0);
        
        return canvas;
    } catch (err) {
        console.error("Erro na renderização SVG:", err);
        throw err;
    }
}


2. Fatiador Manual com Exportação ZIP

Esta lógica pega um canvas gigante, corta em pedaços baseados em coordenadas Y manuais e empacota tudo em um ZIP.

Dependência: jszip.js

/**
 * Fatia um canvas grande em múltiplos arquivos e baixa como ZIP.
 * * @param {HTMLCanvasElement} sourceCanvas - O canvas original em alta resolução.
 * @param {number[]} cutPointsY - Array com as posições Y (pixels) onde cortar.
 * @param {string} baseFilename - Nome base para o arquivo (ex: "jogos-hoje").
 * @param {number} domWidth - Largura original do DOM (para cálculo de escala).
 */
async function sliceAndZipCanvas(sourceCanvas, cutPointsY, baseFilename, domWidth = 400) {
    const zip = new JSZip();
    const totalHeight = sourceCanvas.height;
    const canvasWidth = sourceCanvas.width;
    
    // Calcula a proporção entre o DOM e o Canvas (caso seja Retina/Scale 2x)
    const scaleFactor = canvasWidth / domWidth;
    
    // Prepara os pontos de corte: [0, ...cortes ajustados pela escala, final]
    const cuts = [0, ...cutPointsY.map(y => y * scaleFactor), totalHeight];
    
    // Remove duplicatas e ordena
    const uniqueCuts = [...new Set(cuts)].sort((a, b) => a - b);

    let partNumber = 1;

    // Loop para criar cada fatia
    for (let i = 0; i < uniqueCuts.length - 1; i++) {
        const startY = uniqueCuts[i];
        const endY = uniqueCuts[i+1];
        const sliceHeight = endY - startY;

        // Ignora fatias vazias ou muito pequenas
        if (sliceHeight <= 1) continue;

        // Cria um canvas temporário para a fatia
        const sliceCanvas = document.createElement('canvas');
        sliceCanvas.width = canvasWidth;
        sliceCanvas.height = sliceHeight;
        const ctx = sliceCanvas.getContext('2d');

        // Recorta: drawImage(img, sx, sy, sw, sh, dx, dy, dw, dh)
        ctx.drawImage(sourceCanvas, 0, startY, canvasWidth, sliceHeight, 0, 0, canvasWidth, sliceHeight);

        // Converte para blob/base64 e adiciona ao ZIP
        const imgData = sliceCanvas.toDataURL("image/png").split(',')[1];
        
        // Nome do arquivo interno: base-001.png
        const fileName = `${baseFilename}-${String(partNumber).padStart(3, '0')}.png`;
        zip.file(fileName, imgData, {base64: true});
        
        partNumber++;
    }

    // Gera o arquivo ZIP final e dispara o download
    const zipContent = await zip.generateAsync({type:"blob"});
    const link = document.createElement('a');
    link.href = URL.createObjectURL(zipContent);
    link.download = `${baseFilename}.zip`;
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
}


3. Gerador de Nome de Arquivo (Data Automática)

Gera nomes de arquivos padronizados com a data atual, ideal para relatórios diários ou agendas.

/**
 * Gera uma string de nome de arquivo com a data atual formatada.
 * Formato: prefixo-DD-MM-AAAA-sufixo.extensao
 * * @param {string} prefix - Texto inicial (ex: "jogos_do_dia").
 * @param {string} suffix - Texto final (ex: "FGC").
 * @param {string} extension - Extensão do arquivo (sem ponto).
 * @returns {string} O nome formatado.
 */
function generateDatedFilename(prefix, suffix, extension) {
    const today = new Date();
    
    // PadStart garante que dias/meses tenham 2 dígitos (01, 05, etc)
    const day = String(today.getDate()).padStart(2, '0');
    // Month é base 0 em JS, por isso +1
    const month = String(today.getMonth() + 1).padStart(2, '0');
    const year = today.getFullYear();
    
    return `${prefix}-${day}-${month}-${year}-${suffix}.${extension}`;
}


4. Sanitizador CSS (Correção CORS)

Corrige automaticamente problemas de segurança que impedem a exportação de imagens quando se usa bibliotecas externas (como FontAwesome) via CDN.

/**
 * Injeta o atributo 'crossorigin="anonymous"' em links de CSS no HTML.
 * Essencial para evitar SecurityError no dom-to-image/html2canvas.
 * * @param {string} htmlString - O código HTML cru.
 * @returns {string} O HTML sanitizado e seguro para renderização.
 */
function sanitizeCssLinks(htmlString) {
    return htmlString.replace(
        /<link([^>]+)href=["']([^"']+)["']([^>]*)/gi, 
        (match, beforeAttributes, url, afterAttributes) => {
            // Se já tem a propriedade, não faz nada
            if (match.includes('crossorigin')) return match;
            
            // Verifica se é uma folha de estilo
            if (match.includes('rel="stylesheet"') || match.includes("rel='stylesheet'")) {
                // Reconstrói a tag com o atributo necessário
                return `<link${beforeAttributes}href="${url}"${afterAttributes} crossorigin="anonymous"`;
            }
            return match;
        }
    );
}


🚀 Como usar o projeto

Cole seu HTML: Use a aba "Editor" para colar o código da sua tabela ou agenda.

Pré-visualize: Vá para a aba "Preview". A largura será fixada em 400px (padrão mobile).

Defina Cortes (Opcional):

Clique em "Definir Cortes".

Toque/Clique na imagem onde deseja separar as fatias (linhas vermelhas aparecerão).

Use o "X" vermelho para remover uma linha errada.

Exporte:

Baixar Inteiro: Gera um PNG único, longo e contínuo.

Baixar ZIP: Gera um arquivo .zip contendo as imagens fatiadas nos pontos que você definiu.

🛠 Tecnologias Utilizadas

Tailwind CSS: Para estilização da interface (layout, botões, modais).

FontAwesome: Para ícones da interface (tesoura, download, código).

dom-to-image-more: Engine principal de renderização (substituiu html2canvas por melhor fidelidade).

JSZip: Para empacotamento das fatias de imagem no navegador.
