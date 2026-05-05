<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Nest Work app</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://unpkg.com/lucide@latest"></script>
    <style>
        body { background-color: #f8fafc; font-family: 'Inter', sans-serif; -webkit-tap-highlight-color: transparent; }
        
        .sheet-preview {
            background-image: radial-gradient(#cbd5e1 1px, transparent 1px);
            background-size: 20px 20px;
            border: 2px border-dashed #94a3b8;
            position: relative;
            background-color: white;
            transition: transform 0.2s ease-out;
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
            transform-origin: top left;
        }

        .canvas-container {
            width: 100%;
            height: 100%;
            min-height: 400px;
            max-height: calc(100vh - 180px);
            overflow: auto;
            background: #f1f5f9;
            border-radius: 8px;
            border: 1px solid #e2e8f0;
            position: relative;
        }

        .zoom-wrapper {
            display: flex;
            justify-content: center;
            align-items: center;
            min-width: 100%;
            min-height: 100%;
            padding: 20px;
        }

        .zoom-control-container {
            position: absolute;
            right: 15px;
            top: 50%;
            transform: translateY(-50%);
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 8px;
            background: white;
            padding: 10px 4px;
            border-radius: 30px;
            box-shadow: 0 4px 12px rgba(0,0,0,0.1);
            z-index: 50;
        }

        .zoom-slider {
            appearance: none;
            width: 100px;
            height: 4px;
            background: #e2e8f0;
            outline: none;
            border-radius: 2px;
            transform: rotate(-90deg);
            margin: 45px -40px;
            cursor: pointer;
        }

        .zoom-slider::-webkit-slider-thumb {
            appearance: none;
            width: 14px;
            height: 14px;
            background: #2563eb;
            border-radius: 50%;
            cursor: pointer;
            border: 2px solid white;
        }

        #sheetPreview svg { width: 100%; height: 100%; display: block; }
        
        .btn-action { 
            transition: all 0.2s; 
            display: flex; 
            align-items: center; 
            justify-content: center; 
            gap: 6px; 
            font-weight: 700; 
            padding: 8px 12px; 
            border-radius: 6px; 
            font-size: 13px; 
            text-transform: uppercase;
        }

        .input-compact {
            border-bottom: 2px solid #e2e8f0;
            padding: 2px 0;
            font-size: 14px;
            font-weight: 600;
            width: 100%;
            outline: none;
            transition: border-color 0.2s;
            background: transparent;
        }

        .input-compact:focus {
            border-color: #2563eb;
        }

        .card-panel {
            background: white;
            border-radius: 10px;
            border: 1px solid #edf2f7;
            padding: 16px;
            box-shadow: 0 1px 3px rgba(0,0,0,0.05);
        }

        input[type="number"] { font-size: 14px !important; }
    </style>
</head>
<body class="p-2 md:p-4">

    <div class="max-w-[1600px] mx-auto">
        <!-- Header Compacto -->
        <header class="flex flex-col md:flex-row justify-between items-center mb-4 gap-4 bg-slate-900 p-3 rounded-xl shadow-lg">
            <div class="flex items-center gap-3 pl-2">
                <div class="bg-red-600 p-2 rounded-lg">
                    <i data-lucide="scissors" class="text-white w-5 h-5"></i>
                </div>
                <div>
                    <h1 class="text-lg font-black text-white leading-none tracking-tight">N  E  S  T      W  O  R  K <span class="text-red-500"></span></h1>
                    <p class="text-slate-400 text-[9px] font-bold uppercase tracking-[0.2em]">Otimize sua tarefa de agrupamento </p>
                </div>
            </div>
            
            <div class="flex gap-2 w-full md:w-auto">
                <label class="btn-action flex-1 md:flex-none bg-blue-600 text-white hover:bg-blue-700 cursor-pointer shadow-lg shadow-blue-900/20">
                    <i data-lucide="file-down" class="w-4 h-4"></i> <span>Importar SVG</span>
                    <input type="file" id="fileInput" accept=".svg" class="hidden" onchange="handleFileUpload(event)">
                </label>
                <button onclick="downloadSVG()" class="btn-action flex-1 md:flex-none bg-emerald-500 text-white hover:bg-emerald-600 shadow-lg shadow-emerald-900/20">
                    <i data-lucide="file-up" class="w-4 h-4"></i> <span>Exportar SVG</span>
                </button>
            </div>
        </header>

        <div class="grid grid-cols-1 lg:grid-cols-12 gap-4">
            <!-- Coluna de Controles (Lateral Esquerda) -->
            <div class="lg:col-span-3 space-y-4">
                
                <!-- Painel Combinado: Área e Cópia (Ajuste Lateral de 50%) -->
                <div class="grid grid-cols-2 gap-4">
                    <!-- Área de Trabalho (Esquerda) -->
                    <div class="card-panel">
                        <h2 class="text-[10px] font-black uppercase text-slate-400 mb-4 flex items-center gap-2">
                            <i data-lucide="maximize" class="w-3 h-3"></i> Área da chapa
                        </h2>
                        <div class="flex flex-col gap-4">
                            <div>
                                <label class="block text-[9px] font-bold text-slate-500 uppercase mb-1">Altura (mm)</label>
                                <input type="number" id="sheetHeight" value="55" oninput="updatePreview()" class="input-compact text-blue-600">
                            </div>
                            <div>
                                <label class="block text-[9px] font-bold text-slate-500 uppercase mb-1">Largura (mm)</label>
                                <input type="number" id="sheetWidth" value="45" oninput="updatePreview()" class="input-compact text-blue-600">
                            </div>
                        </div>
                    </div>

                    <!-- Configuração de Cópia (Direita) -->
                    <div class="card-panel">
                        <h2 class="text-[10px] font-black uppercase text-slate-400 mb-4 flex items-center gap-2">
                            <i data-lucide="layers" class="w-3 h-3"></i> Cópia
                        </h2>
                        <div class="flex flex-col gap-4">
                            <div>
                                <label class="block text-[9px] font-bold text-slate-500 uppercase mb-1">Quantidade</label>
                                <input type="number" id="itemQty" value="1" min="0" oninput="generateLayout()" class="input-compact text-orange-600">
                            </div>
                            <div>
                                <label class="block text-[9px] font-bold text-slate-500 uppercase mb-1">Espessura mm</label>
                                <input type="number" id="itemPadding" value="1" min="0" oninput="generateLayout()" class="input-compact text-orange-600">
                            </div>
                        </div>
                    </div>
                </div>

                <!-- Painel de Ferramentas de Precisão com campos realocados -->
                <div class="card-panel">
                    <h2 class="text-[10px] font-black uppercase text-slate-400 mb-4 flex items-center gap-2">
                        <i data-lucide="zap" class="w-3 h-3"></i> Ferramentas de Ajuste
                    </h2>
                    
                    <div id="fileDimsEmpty" class="text-center py-4 bg-slate-50 rounded-lg border-2 border-dashed border-slate-100">
                        <p class="text-[10px] font-bold text-slate-400 uppercase italic">Aguardando Vetor...</p>
                    </div>

                    <div id="fileDimsDisplay" class="hidden">
                        <div class="grid grid-cols-12 gap-3 mb-4">
                            <!-- Botões de Ação (Lado Esquerdo) -->
                            <div class="col-span-7 grid grid-cols-2 gap-2">
                                <button id="fillQtyBtn" onclick="toggleFillQuantity()" class="btn-action bg-slate-100 text-slate-600 hover:bg-orange-100 hover:text-orange-700 border border-slate-200">
                                    <i data-lucide="expand" class="w-3.5 h-3.5"></i> <span class="text-[10px]">Preencher</span>
                                </button>
                                <button id="fineTuneBtn" onclick="toggleFineTune()" class="btn-action bg-slate-100 text-slate-600 hover:bg-purple-100 hover:text-purple-700 border border-slate-200">
                                    <i data-lucide="repeat" class="w-3.5 h-3.5"></i> <span class="text-[10px]">Inverter</span>
                                </button>
                                <button id="rotate90Btn" onclick="cycleRotation()" class="btn-action bg-slate-100 text-slate-600 hover:bg-indigo-100 hover:text-indigo-700 border border-slate-200">
                                    <i data-lucide="rotate-cw" class="w-3.5 h-3.5"></i> <span id="rotateBtnLabel" class="text-[10px]">0°</span>
                                </button>
                                <button id="compactBtn" onclick="toggleCompact()" class="btn-action bg-slate-100 text-slate-600 hover:bg-blue-100 hover:text-blue-700 border border-slate-200">
                                    <i data-lucide="align-justify" class="w-3.5 h-3.5"></i> <span class="text-[10px]">Compactar</span>
                                </button>
                            </div>

                            <!-- Vetor Unitário (Realocado para a Direita) -->
                            <div class="col-span-5 flex flex-col gap-2">
                                <div class="bg-slate-50 p-2 rounded border border-slate-100 text-center">
                                    <span class="block text-[8px] font-bold text-slate-400 uppercase">Vetor A</span>
                                    <span id="infoItemH" class="text-xs font-black text-slate-700">--</span>
                                </div>
                                <div class="bg-slate-50 p-2 rounded border border-slate-100 text-center">
                                    <span class="block text-[8px] font-bold text-slate-400 uppercase">Vetor L</span>
                                    <span id="infoItemW" class="text-xs font-black text-slate-700">--</span>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>

                <div id="fileInfoDisplay" class="hidden">
                    <div class="flex items-center gap-2 p-2 bg-emerald-50 rounded-lg border border-emerald-100">
                        <i data-lucide="check-circle-2" class="w-4 h-4 text-emerald-600"></i>
                        <span id="fileName" class="text-[10px] font-black text-emerald-800 truncate uppercase">---</span>
                    </div>
                </div>
            </div>

            <!-- Coluna de Preview (Principal) -->
            <div class="lg:col-span-9">
                <div class="card-panel h-full flex flex-col relative overflow-hidden">
                    <div class="flex justify-between items-center mb-3">
                        <div>
                            <h2 class="text-sm font-black text-slate-800 uppercase tracking-tight">Preview da Área de Produção</h2>
                            <p class="text-[10px] text-slate-400 font-bold" id="sheetDimLabel">---</p>
                        </div>
                        
                        <!-- Espaço centralizado para dimensões do arquivo ocupado -->
                        <div class="flex-1 flex justify-center">
                            <div id="occupiedDimsDisplay" class="hidden flex items-center gap-2 px-4 py-1.5 bg-blue-50 border border-blue-100 rounded-full">
                                <i data-lucide="crop" class="w-3.5 h-3.5 text-blue-500"></i>
                                <span class="text-[10px] font-black text-blue-700 uppercase tracking-wide">
                                    Espaço Utilizado: <span id="occupiedW">0</span> x <span id="occupiedH">0</span> mm
                                </span>
                            </div>
                        </div>

                        <div id="statusBadge" class="px-3 py-1 bg-slate-100 text-slate-400 rounded-md text-[9px] font-black uppercase border border-slate-200">
                            OFFLINE
                        </div>
                    </div>
                    
                    <div class="canvas-container" id="scrollContainer">
                        <!-- Controle de Zoom -->
                        <div class="zoom-control-container">
                            <i data-lucide="plus" class="w-3 h-3 text-slate-400"></i>
                            <input type="range" id="zoomSlider" min="0" max="100" value="0" class="zoom-slider" oninput="handleZoom(this.value)">
                            <i data-lucide="minus" class="w-3 h-3 text-slate-400"></i>
                            <span id="zoomLabel" class="text-[9px] font-black text-blue-600 mt-2">0%</span>
                        </div>

                        <div class="zoom-wrapper" id="zoomWrapper">
                            <div id="sheetPreview" class="sheet-preview">
                                <div id="placeholder" class="text-center opacity-20 py-20">
                                    <i data-lucide="image-plus" class="w-16 h-16 mx-auto mb-2 text-slate-300"></i>
                                    <p class="font-black uppercase text-xs text-slate-400">Arraste ou importe seu SVG</p>
                                </div>
                            </div>
                        </div>
                    </div>
                    
                    <!-- Footer do Painel de Visualização -->
                    <div class="mt-3 pt-3 border-t border-slate-50 flex flex-wrap gap-4 items-center justify-between">
                        <div class="flex gap-4 items-center">
                            <div class="flex items-center gap-1.5">
                                <span class="w-2.5 h-2.5 bg-red-500 rounded-sm"></span>
                                <span class="text-[10px] font-bold text-slate-500 uppercase">Caminho de Corte</span>
                            </div>
                            <div class="h-4 w-[1px] bg-slate-200"></div>
                            <div id="resultInfo" class="text-[11px] font-black text-slate-700 uppercase font-mono">
                                0 PEÇAS PROCESSADAS
                            </div>
                        </div>
                        <div class="text-[10px] font-bold text-slate-300 uppercase">
                            NestWork Engine v2.5
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <script>
        const dom = {
            sheetWidth: document.getElementById('sheetWidth'),
            sheetHeight: document.getElementById('sheetHeight'),
            itemQty: document.getElementById('itemQty'),
            itemPadding: document.getElementById('itemPadding'),
            preview: document.getElementById('sheetPreview'),
            zoomWrapper: document.getElementById('zoomWrapper'),
            dimLabel: document.getElementById('sheetDimLabel'),
            statusBadge: document.getElementById('statusBadge'),
            resultInfo: document.getElementById('resultInfo'),
            infoItemW: document.getElementById('infoItemW'),
            infoItemH: document.getElementById('infoItemH'),
            fileName: document.getElementById('fileName'),
            fineTuneBtn: document.getElementById('fineTuneBtn'),
            fillQtyBtn: document.getElementById('fillQtyBtn'),
            compactBtn: document.getElementById('compactBtn'),
            rotate90Btn: document.getElementById('rotate90Btn'),
            rotateBtnLabel: document.getElementById('rotateBtnLabel'),
            zoomSlider: document.getElementById('zoomSlider'),
            zoomLabel: document.getElementById('zoomLabel'),
            occupiedW: document.getElementById('occupiedW'),
            occupiedH: document.getElementById('occupiedH'),
            occupiedContainer: document.getElementById('occupiedDimsDisplay')
        };

        lucide.createIcons();

        let originalNodes = []; 
        let itemSize = { w: 0, h: 0, x: 0, y: 0 };
        let isFileLoaded = false;
        let fileScale = 1; 
        let isFineTuneEnabled = false; 
        let isFillEnabled = false; 
        let isCompactEnabled = false; 
        let currentRotation = 0; 
        let currentZoom = 0; 
        let baseW = 0, baseH = 0;
        let originalFileName = ""; 
        let lastPlacedCount = 0;   
        const NS_SVG = "http://www.w3.org/2000/svg";

        function handleZoom(val) {
            currentZoom = parseInt(val);
            dom.zoomLabel.innerText = `${currentZoom}%`;
            const scaleValue = 1 + (currentZoom / 100);
            dom.preview.style.transform = `scale(${scaleValue})`;
            dom.zoomWrapper.style.width = `${baseW * scaleValue + 40}px`;
            dom.zoomWrapper.style.height = `${baseH * scaleValue + 40}px`;
        }

        function updatePreview() {
            const w = parseFloat(dom.sheetWidth.value) || 1;
            const h = parseFloat(dom.sheetHeight.value) || 1;
            dom.dimLabel.innerText = `Chapa: ${w} x ${h} mm`;

            const container = document.getElementById('scrollContainer');
            if (!container) return;
            
            const availableW = container.clientWidth - 60;
            const availableH = container.clientHeight - 60;
            const screenScale = Math.min(availableW / w, availableH / h);
            
            baseW = w * screenScale;
            baseH = h * screenScale;

            dom.preview.style.width = `${baseW}px`;
            dom.preview.style.height = `${baseH}px`;
            
            handleZoom(dom.zoomSlider.value);

            if (isFileLoaded) generateLayout();
        }

        async function handleFileUpload(event) {
            const file = event.target.files[0];
            if (!file) return;

            try {
                originalFileName = file.name.replace(/\.svg$/i, '');
                const text = await file.text();
                const parser = new DOMParser();
                const xmlDoc = parser.parseFromString(text, "image/svg+xml");
                const svgEl = xmlDoc.querySelector('svg');
                
                if (!svgEl) throw new Error("Vetor Inválido");

                const attrW = svgEl.getAttribute('width');
                const viewBoxAttr = svgEl.getAttribute('viewBox');
                const viewBoxArr = viewBoxAttr ? viewBoxAttr.split(/[\s,]+/).filter(v => v !== "").map(Number) : null;
                
                fileScale = 1;

                if (attrW && (attrW.includes('mm') || attrW.includes('cm')) && viewBoxArr) {
                    const val = parseFloat(attrW);
                    const mult = attrW.includes('cm') ? 10 : 1;
                    fileScale = (val * mult) / viewBoxArr[2];
                }

                const tempSvg = document.createElementNS(NS_SVG, "svg");
                tempSvg.style.cssText = "position:absolute;top:-9999px;left:-9999px;visibility:hidden;width:5000px;height:5000px;";
                const g = document.createElementNS(NS_SVG, "g");
                
                Array.from(svgEl.childNodes).forEach(n => {
                    g.appendChild(n.cloneNode(true));
                });

                tempSvg.appendChild(g);
                document.body.appendChild(tempSvg);
                const bbox = g.getBBox();
                document.body.removeChild(tempSvg);

                if (!viewBoxArr && attrW && !attrW.includes('%')) {
                    const val = parseFloat(attrW);
                    if (val) fileScale = val / bbox.width;
                }

                itemSize = { 
                    w: bbox.width * fileScale, 
                    h: bbox.height * fileScale,
                    x: bbox.x,
                    y: bbox.y
                };
                
                originalNodes = Array.from(g.childNodes).map(n => n.cloneNode(true));
                isFileLoaded = true;

                dom.fileName.innerText = file.name;
                document.getElementById('fileInfoDisplay').classList.remove('hidden');
                document.getElementById('fileDimsEmpty').classList.add('hidden');
                document.getElementById('fileDimsDisplay').classList.remove('hidden');
                dom.occupiedContainer.classList.remove('hidden');
                
                dom.infoItemW.innerText = `${itemSize.w.toFixed(1)} mm`;
                dom.infoItemH.innerText = `${itemSize.h.toFixed(1)} mm`;
                
                const placeholder = document.getElementById('placeholder');
                if (placeholder) placeholder.style.display = 'none';
                
                generateLayout();
            } catch (e) {
                console.error("Erro:", e);
            }
        }

        function toggleFineTune() {
            isFineTuneEnabled = !isFineTuneEnabled;
            updateBtnState(dom.fineTuneBtn, isFineTuneEnabled, 'bg-purple-600', 'border-purple-600');
            generateLayout();
        }

        function cycleRotation() {
            currentRotation = (currentRotation + 90) % 360;
            dom.rotateBtnLabel.innerText = `${currentRotation}°`;
            const isActive = currentRotation !== 0;
            updateBtnState(dom.rotate90Btn, isActive, 'bg-indigo-600', 'border-indigo-600');
            generateLayout();
        }

        function toggleFillQuantity() {
            if (!isFileLoaded) return;
            isFillEnabled = !isFillEnabled;
            updateBtnState(dom.fillQtyBtn, isFillEnabled, 'bg-orange-500', 'border-orange-500');
            
            if (isFillEnabled) {
                const sw = parseFloat(dom.sheetWidth.value);
                const sh = parseFloat(dom.sheetHeight.value);
                const gap = parseFloat(dom.itemPadding.value) || 0;
                const isVerticalRotation = currentRotation === 90 || currentRotation === 270;
                const effItemW = isVerticalRotation ? itemSize.h : itemSize.w;
                const effItemH = isVerticalRotation ? itemSize.w : itemSize.h;
                const verticalCompressionFactor = isCompactEnabled ? 0.85 : 1.0;
                const effectiveRowH = effItemH * verticalCompressionFactor;
                const usableW = sw - gap;
                const usableH = sh - gap;
                let calculatedQty = 0;
                if (usableW >= effItemW && usableH >= effItemH) {
                    const cols = Math.floor((usableW + gap) / (effItemW + gap));
                    const rows = Math.floor((usableH - effItemH + gap) / (effectiveRowH + gap)) + 1;
                    calculatedQty = Math.max(0, cols * rows);
                }
                dom.itemQty.value = calculatedQty;
            } else {
                dom.itemQty.value = 1;
            }
            generateLayout();
        }

        function toggleCompact() {
            isCompactEnabled = !isCompactEnabled;
            updateBtnState(dom.compactBtn, isCompactEnabled, 'bg-blue-600', 'border-blue-600');
            generateLayout();
        }

        function updateBtnState(btn, isActive, activeBg, activeBorder) {
            if (isActive) {
                btn.classList.replace('bg-slate-100', activeBg);
                btn.classList.replace('text-slate-600', 'text-white');
                btn.classList.replace('border-slate-200', activeBorder);
            } else {
                btn.classList.remove(activeBg, 'text-white', activeBorder);
                btn.classList.add('bg-slate-100', 'text-slate-600', 'border-slate-200');
            }
        }

        function generateLayout() {
            if (!isFileLoaded) return;
            const sw = parseFloat(dom.sheetWidth.value) || 0;
            const sh = parseFloat(dom.sheetHeight.value) || 0;
            const targetQty = parseInt(dom.itemQty.value) || 0;
            const gap = parseFloat(dom.itemPadding.value) || 0;
            
            dom.preview.innerHTML = "";
            const svg = document.createElementNS(NS_SVG, "svg");
            svg.setAttribute("id", "production-svg");
            svg.setAttribute("width", sw);
            svg.setAttribute("height", sh);
            svg.setAttribute("viewBox", `0 0 ${sw} ${sh}`);
            svg.style.width = "100%";
            svg.style.height = "100%";

            const guide = document.createElementNS(NS_SVG, "rect");
            guide.setAttribute("id", "canvas-border");
            guide.setAttribute("width", sw); guide.setAttribute("height", sh);
            guide.setAttribute("fill", "white"); guide.setAttribute("stroke", "#e2e8f0"); guide.setAttribute("stroke-width", "0.2");
            svg.appendChild(guide);

            const grid = document.createElementNS(NS_SVG, "g");
            grid.setAttribute("id", "nested-items");

            let cursorX = gap, cursorY = gap, countPlaced = 0, currentRow = 0;
            const isVerticalRotation = currentRotation === 90 || currentRotation === 270;
            const currentItemW = isVerticalRotation ? itemSize.h : itemSize.w;
            const currentItemH = isVerticalRotation ? itemSize.w : itemSize.h;
            const itemsPerRow = Math.floor((sw - gap + gap) / (currentItemW + gap));
            const verticalCompressionFactor = isCompactEnabled ? 0.85 : 1.0;

            let maxReachX = 0;
            let maxReachY = 0;

            for (let i = 0; i < targetQty; i++) {
                if (cursorX + currentItemW > sw - gap + 0.001) {
                    cursorX = gap;
                    cursorY += (currentItemH * verticalCompressionFactor) + gap;
                    currentRow++;
                }
                if (cursorY + currentItemH > sh - gap + 0.001) break;

                const pieceGroup = document.createElementNS(NS_SVG, "g");
                const shouldRotate180Extra = isFineTuneEnabled && (i % 2 !== 0);
                let xOffset = 0;
                if (isCompactEnabled && (currentRow % 2 !== 0)) {
                    const rowItemsW = (itemsPerRow * currentItemW) + ((itemsPerRow - 1) * gap);
                    xOffset = Math.max(0, sw - gap - rowItemsW - gap);
                }
                const centerX = itemSize.x + (itemSize.w/fileScale)/2;
                const centerY = itemSize.y + (itemSize.h/fileScale)/2;
                let combinedRotation = currentRotation + (shouldRotate180Extra ? 180 : 0);

                let tx = cursorX + xOffset - (itemSize.x * fileScale);
                let ty = cursorY - (itemSize.y * fileScale);
                if (isVerticalRotation) {
                    tx += (currentItemW - itemSize.w)/2;
                    ty += (currentItemH - itemSize.h)/2;
                }

                pieceGroup.setAttribute("transform", `translate(${tx}, ${ty}) scale(${fileScale}) rotate(${combinedRotation}, ${centerX}, ${centerY})`);
                originalNodes.forEach(node => pieceGroup.appendChild(node.cloneNode(true)));
                grid.appendChild(pieceGroup);
                
                maxReachX = Math.max(maxReachX, cursorX + xOffset + currentItemW);
                maxReachY = Math.max(maxReachY, cursorY + currentItemH);

                cursorX += currentItemW + gap;
                countPlaced++;
            }

            svg.appendChild(grid);
            dom.preview.appendChild(svg);
            lastPlacedCount = countPlaced;
            
            // Atualiza as dimensões ocupadas reais (adicionando gap final se necessário, mas aqui pegamos o extremo das peças)
            if (countPlaced > 0) {
                dom.occupiedW.innerText = (maxReachX).toFixed(1);
                dom.occupiedH.innerText = (maxReachY).toFixed(1);
            } else {
                dom.occupiedW.innerText = "0";
                dom.occupiedH.innerText = "0";
            }

            const isOk = (countPlaced >= targetQty && targetQty > 0);
            dom.statusBadge.innerText = targetQty === 0 ? "Aguardando" : (isOk ? "Otimizado" : "Limitado");
            dom.statusBadge.className = `px-3 py-1 ${isOk ? 'bg-emerald-50 text-emerald-600 border-emerald-100' : 'bg-red-50 text-red-600 border-red-100'} rounded-md text-[9px] font-black uppercase border`;
            dom.resultInfo.innerText = `${countPlaced} PEÇAS PROCESSADAS`;
        }

        function downloadSVG() {
            const mainSvg = document.getElementById('production-svg');
            if (!mainSvg || !isFileLoaded) return;
            const exportClone = mainSvg.cloneNode(true);
            const border = exportClone.getElementById('canvas-border');
            if (border) border.remove();
            const w = dom.sheetWidth.value, h = dom.sheetHeight.value;
            exportClone.setAttribute("width", `${w}mm`);
            exportClone.setAttribute("height", `${h}mm`);
            exportClone.setAttribute("viewBox", `0 0 ${w} ${h}`);
            const serializer = new XMLSerializer();
            let svgString = serializer.serializeToString(exportClone);
            if (!svgString.includes('xmlns=')) svgString = svgString.replace(/^<svg/, '<svg xmlns="http://www.w3.org/2000/svg"');
            const blob = new Blob([`<?xml version="1.0" encoding="UTF-8"?>\r\n` + svgString], {type: 'image/svg+xml'});
            const url = URL.createObjectURL(blob);
            const a = document.createElement('a');
            a.href = url;
            a.download = `${originalFileName} [${lastPlacedCount}]UNID.svg`;
            a.click();
            URL.revokeObjectURL(url);
        }

        window.addEventListener('resize', updatePreview);
        window.onload = updatePreview;
    </script>
</body>
</html>
