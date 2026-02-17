<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    
    <!-- MODO APP MÓVIL -->
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
    <meta name="theme-color" content="#2563EB">
    <link rel="apple-touch-icon" href="https://cdn-icons-png.flaticon.com/512/1048/1048953.png">
    
    <title>Cotizador PVC Profesional</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <script src="https://cdn.jsdelivr.net/npm/sweetalert2@11"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html2pdf.js/0.10.1/html2pdf.bundle.min.js"></script>

    <style>
        body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; background-color: #f3f4f6; -webkit-tap-highlight-color: transparent; }
        .fade-in { animation: fadeIn 0.3s ease-in; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }

        /* CONFIGURACIÓN DE IMPRESIÓN - CARTA VERTICAL (MITAD SUPERIOR) */
        @page {
            size: letter portrait;
            margin: 0;
        }

        @media print {
            body { background-color: white; padding: 0; margin: 0; font-size: 9px; line-height: 1.1; }
            .no-print { display: none !important; }
            
            body > * { display: none; }
            #printableArea { display: flex !important; }

            input, textarea {
                border: none;
                background: transparent;
                padding: 0;
                width: auto;
                font-family: inherit;
                resize: none;
                outline: none;
            }
            input[type="number"] { text-align: right; font-weight: bold; }
            input.text-left-print { text-align: left !important; }
            
            input::-webkit-outer-spin-button, input::-webkit-inner-spin-button { -webkit-appearance: none; margin: 0; }
            
            th, td { padding: 1px 3px !important; border-bottom: 1px solid #eee; }
            
            #printableArea { 
                position: fixed;
                top: 0;
                left: 0;
                width: 215.9mm;
                height: 139.7mm; 
                padding: 8mm 12mm;
                box-sizing: border-box;
                overflow: hidden;
                flex-direction: column; 
                background: white;
                display: flex !important;
            }
            
            .header-section { flex-shrink: 0; margin-bottom: 4px; }
            .client-section { flex-shrink: 0; border-bottom: 1.5px solid #000; margin-bottom: 4px; padding-bottom: 4px; }
            .lists-container { display: flex; gap: 5mm; flex-grow: 1; overflow: hidden; }
            .spaces-column { width: 30%; border-right: 1px solid #ddd; padding-right: 2mm; }
            .costs-column { width: 70%; }
            
            .footer-section { 
                flex-shrink: 0; 
                border-top: 1.5px solid #000; 
                padding-top: 4px; 
                margin-top: auto; 
                display: block !important;
            }
            /* Texto reducido según solicitud */
            .footer-section .conditions-title { font-size: 8px; font-weight: bold; margin-bottom: 1px; text-transform: uppercase; }
            .footer-section .conditions-text { font-size: 6.8px; line-height: 1.1; text-align: justify; }
            .footer-section .social-text { font-size: 7.5px; font-weight: bold; display: flex; align-items: center; justify-content: flex-end; }
            
            * { -webkit-print-color-adjust: exact !important; print-color-adjust: exact !important; }
        }
        
        body.pdf-export-mode .no-print { display: none !important; }
        body.pdf-export-mode #printableArea {
            width: 215.9mm;
            height: 139.7mm;
            padding: 10mm;
            background: white;
            display: flex;
            flex-direction: column;
            overflow: hidden;
        }
        
        .print-visible { display: block; }
    </style>
</head>
<body class="flex items-center justify-center min-h-screen p-4 bg-gray-100 print:p-0 print:bg-white select-none md:select-auto">

    <div class="bg-white rounded-2xl shadow-xl w-full max-w-6xl overflow-hidden flex flex-col md:flex-row print:shadow-none print:w-full relative">
        
        <!-- COLUMNA IZQUIERDA: CONTROLES -->
        <div class="w-full md:w-1/3 bg-blue-50 p-6 border-r border-blue-100 no-print flex flex-col h-full overflow-y-auto max-h-screen">
            <h2 class="text-xl font-bold text-blue-800 mb-4 flex items-center">
                <i class="fa-solid fa-calculator mr-2"></i>Cotizador Galpa
            </h2>

            <div class="bg-blue-100 p-3 rounded-lg text-xs text-blue-800 mb-4 border border-blue-200">
                <div class="font-bold mb-1">Galpa Comercializadora</div>
                <div class="mb-2 opacity-75">Datos corporativos fijos.</div>
                <input type="text" id="advisorName" class="w-full p-2 text-sm border rounded border-blue-200 bg-white" placeholder="Nombre del Asesor" value="">
            </div>

            <!-- DATOS DEL CLIENTE -->
            <div class="bg-white p-4 rounded-lg shadow-sm border border-blue-100 mb-4 space-y-2">
                <h3 class="text-xs font-bold text-gray-500 uppercase mb-2">Datos del Cliente</h3>
                <input type="text" id="clientName" class="w-full p-2 text-sm border rounded border-gray-200" placeholder="Nombre Completo" oninput="syncClientData()">
                <div class="grid grid-cols-2 gap-2">
                    <input type="text" id="clientId" class="w-full p-2 text-sm border rounded border-gray-200" placeholder="NIT / Cédula" oninput="syncClientData()">
                    <input type="tel" id="clientPhone" class="w-full p-2 text-sm border rounded border-gray-200" placeholder="Teléfono" oninput="syncClientData()">
                </div>
                <div class="grid grid-cols-2 gap-2">
                    <input type="text" id="clientAddress" class="w-full p-2 text-sm border rounded border-gray-200" placeholder="Dirección" oninput="syncClientData()">
                    <input type="text" id="clientCity" class="w-full p-2 text-sm border rounded border-gray-200" placeholder="Ciudad" oninput="syncClientData()">
                </div>
            </div>
            
            <div class="space-y-3">
                <h3 class="text-xs font-bold text-gray-500 uppercase">Agregar Espacio</h3>
                <input type="text" id="inputName" class="w-full p-3 border rounded-lg border-blue-200 bg-white" placeholder="Ej: Sala, Cocina...">
                <div class="grid grid-cols-2 gap-2">
                    <input type="number" id="inputLength" class="w-full p-2 border rounded-lg border-blue-200 bg-white" placeholder="Largo (m)" oninput="calculateRowArea()">
                    <input type="number" id="inputWidth" class="w-full p-2 border rounded-lg border-blue-200 bg-white" placeholder="Ancho (m)" oninput="calculateRowArea()">
                </div>
                <div class="grid grid-cols-2 gap-2">
                    <input type="number" id="inputArea" class="w-full p-2 border rounded-lg border-blue-200 bg-white" placeholder="Área m²">
                    <input type="number" id="inputPerimeter" class="w-full p-2 border rounded-lg border-blue-200 bg-white" placeholder="Perím. ml">
                </div>
                <button onclick="addSpace()" class="w-full bg-blue-600 hover:bg-blue-700 text-white font-bold py-3 px-4 rounded-lg transition shadow-lg active:scale-95 transform text-sm">
                    <i class="fa-solid fa-plus-circle mr-2"></i>Añadir Espacio
                </button>
            </div>

            <div class="mt-4 pt-4 border-t border-blue-200">
                <div class="flex justify-between mb-1">
                    <label class="text-xs font-bold text-gray-500 uppercase">Desperdicio</label>
                    <span id="wasteValue" class="text-xs font-bold text-blue-600">5%</span>
                </div>
                <input type="range" id="wasteRange" min="0" max="30" value="5" class="w-full h-2 bg-blue-200 rounded-lg appearance-none cursor-pointer" oninput="updateWasteLabel(); updateTotals()">
            </div>
        </div>

        <!-- COLUMNA DERECHA: IMPRESIÓN (MITAD SUPERIOR) -->
        <div class="w-full md:w-2/3 flex flex-col bg-white" id="printableArea">
            
            <!-- Encabezado Fijo -->
            <div class="header-section flex justify-between items-start print:mb-2 border-b border-gray-100 pb-2 print:border-none">
                <div class="w-2/3">
                    <div class="text-gray-800 leading-tight">
                        <h2 class="font-black text-sm uppercase text-black mb-0.5">Galpa Comercializadora</h2>
                        <p class="text-[10px] mb-0.5 font-bold">1.037.617.137</p>
                        <p class="text-[9px]">Carrera 39 # 34D Sur - 52, Envigado</p>
                        <p class="text-[9px]">Carrera 75 # 45 Sur - 132, San Antonio de Prado</p>
                        <p class="text-[9px] font-bold mt-0.5"><i class="fa-solid fa-phone-flip mr-1"></i>304 443 0424</p>
                    </div>
                </div>

                <div class="text-right w-1/3">
                    <h1 class="text-lg font-bold text-gray-800 uppercase print:text-xs">Cotización</h1>
                    <p class="text-[9px] text-gray-500 font-bold mb-1" id="printDate"></p>
                    <input type="text" id="printAdvisor" class="text-[10px] text-blue-700 font-bold text-right w-full block bg-transparent" placeholder="Nombre Asesor" readonly>
                    
                    <div class="flex gap-2 justify-end mt-2 no-print">
                        <button onclick="shareQuotePDF()" class="bg-green-100 hover:bg-green-200 text-green-700 text-xs font-semibold py-1 px-2 rounded transition border border-green-200"><i class="fa-solid fa-file-pdf"></i></button>
                        <button onclick="shareQuoteNative()" class="bg-blue-100 hover:bg-blue-200 text-blue-700 text-xs font-semibold py-1 px-2 rounded transition border border-blue-200"><i class="fa-solid fa-share-nodes"></i></button>
                        <button onclick="window.print()" class="bg-gray-100 hover:bg-gray-200 text-gray-700 text-xs font-semibold py-1 px-2 rounded transition border border-gray-200"><i class="fa-solid fa-print"></i></button>
                        <button onclick="clearAll()" class="text-xs text-red-500 hover:text-red-700 underline px-2">Limpiar</button>
                    </div>
                </div>
            </div>

            <!-- Datos Cliente -->
            <div class="client-section px-0 py-1 text-[10px] mb-2">
                <div class="flex justify-between">
                    <div class="w-1/2 pr-3">
                        <p class="flex mb-1"><span class="font-bold w-12">Cliente:</span> <span id="printName" class="border-b border-gray-200 flex-grow h-4"></span></p>
                        <p class="flex"><span class="font-bold w-12">ID/NIT:</span> <span id="printId" class="border-b border-gray-200 flex-grow h-4"></span></p>
                    </div>
                    <div class="w-1/2 pl-3">
                        <p class="flex mb-1"><span class="font-bold w-12">Tel:</span> <span id="printPhone" class="border-b border-gray-200 flex-grow h-4"></span></p>
                        <p class="flex"><span class="font-bold w-12">Dir:</span> <span id="printAddress" class="border-b border-gray-200 flex-grow h-4"></span></p>
                    </div>
                </div>
            </div>

            <!-- Tablas de Datos -->
            <div class="lists-container p-2 bg-gray-50 print:bg-white print:p-0">
                <div class="spaces-column">
                    <div class="font-bold text-[9px] uppercase border-b border-black mb-1">Detalle de Áreas</div>
                    <table class="w-full text-[9px] text-left text-gray-600 print:text-black">
                        <thead>
                            <tr class="border-b border-gray-200">
                                <th class="py-0.5">Espacio</th>
                                <th class="text-center">m²</th>
                            </tr>
                        </thead>
                        <tbody id="spacesTableBody">
                            <tr id="emptyRow"><td colspan="2" class="text-center py-2 text-gray-400">Sin espacios</td></tr>
                        </tbody>
                    </table>
                </div>

                <div class="costs-column flex-grow">
                    <div class="flex justify-between items-center mb-1 border-b border-black">
                        <h3 class="text-[9px] font-bold uppercase">Materiales</h3>
                        <button onclick="addExtraRow()" class="no-print text-[9px] bg-blue-100 py-0.5 px-2 rounded-full font-bold text-blue-700"><i class="fa-solid fa-plus"></i></button>
                    </div>
                    <table class="min-w-full text-[10px]">
                        <thead class="bg-gray-50 print:bg-transparent text-[9px]">
                            <tr class="border-b border-gray-200">
                                <th class="text-left py-0.5">Descripción</th>
                                <th class="text-center">Cant.</th>
                                <th class="text-right">Unitario</th>
                                <th class="text-right">Total</th>
                            </tr>
                        </thead>
                        <tbody id="costTableBody">
                            <tr><td>Láminas PVC</td><td class="text-center"><input type="number" id="qtyPanels" value="0" oninput="updateCosts()"></td><td class="text-right"><input type="number" id="pricePanels" value="33000" oninput="updateCosts()"></td><td class="text-right font-bold" id="totalPanels">$0</td></tr>
                            <tr><td>Cornisas</td><td class="text-center"><input type="number" id="qtyCornices" value="0" oninput="updateCosts()"></td><td class="text-right"><input type="number" id="priceCornices" value="15000" oninput="updateCosts()"></td><td class="text-right font-bold" id="totalCornices">$0</td></tr>
                            <tr><td>Omegas</td><td class="text-center"><input type="number" id="qtyOmegas" value="0" oninput="updateCosts()"></td><td class="text-right"><input type="number" id="priceOmegas" value="4500" oninput="updateCosts()"></td><td class="text-right font-bold" id="totalOmegas">$0</td></tr>
                            <tr><td>Viguetas</td><td class="text-center"><input type="number" id="qtyViguetas" value="0" oninput="updateCosts()"></td><td class="text-right"><input type="number" id="priceViguetas" value="4500" oninput="updateCosts()"></td><td class="text-right font-bold" id="totalViguetas">$0</td></tr>
                            <tr><td>Ángulos</td><td class="text-center"><input type="number" id="qtyAngles" value="0" oninput="updateCosts()"></td><td class="text-right"><input type="number" id="priceAngles" value="3500" oninput="updateCosts()"></td><td class="text-right font-bold" id="totalAngles">$0</td></tr>
                            <tr><td>Torn. Estruct (x100)</td><td class="text-center"><input type="number" id="qtyScrewsStructure" value="0" oninput="updateCosts()"></td><td class="text-right"><input type="number" id="priceScrewsStructure" value="6000" oninput="updateCosts()"></td><td class="text-right font-bold" id="totalScrewsStructure">$0</td></tr>
                            <tr><td>Torn. Lámina (x100)</td><td class="text-center"><input type="number" id="qtyScrewsSheet" value="0" oninput="updateCosts()"></td><td class="text-right"><input type="number" id="priceScrewsSheet" value="6000" oninput="updateCosts()"></td><td class="text-right font-bold" id="totalScrewsSheet">$0</td></tr>
                        </tbody>
                        <tfoot class="border-t border-black">
                            <tr><td colspan="3" class="text-right font-bold uppercase pt-1">Total Global</td><td class="text-right text-xs font-black pt-1" id="grandTotal">$0</td></tr>
                        </tfoot>
                    </table>
                </div>
            </div>

            <!-- Pie de Página Fijo -->
            <div class="footer-section pt-2">
                <div class="flex justify-between items-start">
                    <div class="w-3/4">
                        <p class="conditions-title text-black">Términos y Condiciones:</p>
                        <p class="conditions-text text-gray-700">Esta cotización es válida por 3 días hábiles. El despacho de materiales se realiza únicamente tras verificar el pago total. Las medidas son responsabilidad del cliente. No se aceptan devoluciones en materiales cortados.</p>
                    </div>
                    <div class="w-1/4 text-right flex flex-col justify-end space-y-0.5">
                        <p class="social-text text-black"><i class="fa-brands fa-instagram mr-1"></i> galpacomercializadora</p>
                        <p class="social-text text-black"><i class="fa-brands fa-tiktok mr-1"></i> galpa.pvc</p>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <script>
        const PANEL_AREA = 1.785; 
        const CORNICE_LENGTH = 5.95; 
        const PROFILE_LENGTH = 2.44; 
        const OMEGA_SPACING = 0.60; 
        const VIGUETA_SPACING = 0.70; 
        
        let spaces = []; 
        let quantities = {}; 
        let extraCounter = 0;

        document.addEventListener('DOMContentLoaded', () => {
            const now = new Date();
            document.getElementById('printDate').innerText = `Fecha: ${now.toLocaleDateString('es-CO', {day:'2-digit', month:'2-digit', year:'numeric'})}`;
            
            document.getElementById('advisorName').addEventListener('input', function() {
                document.getElementById('printAdvisor').value = this.value ? 'Asesor: ' + this.value : '';
            });
            updateWasteLabel();
        });

        function calculateRowArea() {
            const l = parseFloat(document.getElementById('inputLength').value) || 0;
            const w = parseFloat(document.getElementById('inputWidth').value) || 0;
            if(l > 0 && w > 0){
                document.getElementById('inputArea').value = (l * w).toFixed(2);
                document.getElementById('inputPerimeter').value = ((l + w) * 2).toFixed(2);
            }
        }

        function syncClientData() {
            document.getElementById('printName').innerText = document.getElementById('clientName').value;
            document.getElementById('printId').innerText = document.getElementById('clientId').value;
            document.getElementById('printPhone').innerText = document.getElementById('clientPhone').value;
            let addr = document.getElementById('clientAddress').value;
            let city = document.getElementById('clientCity').value;
            document.getElementById('printAddress').innerText = addr + (city ? ` (${city})` : "");
        }

        function updateWasteLabel() { 
            document.getElementById('wasteValue').innerText = document.getElementById('wasteRange').value + '%'; 
        }

        function addSpace() {
            const area = parseFloat(document.getElementById('inputArea').value);
            if (!area || area <= 0) return;
            spaces.push({ 
                id: Date.now(), 
                name: document.getElementById('inputName').value || `Espacio ${spaces.length + 1}`, 
                area, 
                perimeter: parseFloat(document.getElementById('inputPerimeter').value) || 0 
            });
            ['inputName','inputLength','inputWidth','inputArea','inputPerimeter'].forEach(id => document.getElementById(id).value = '');
            renderTable(); updateTotals();
        }

        function removeSpace(id) { 
            spaces = spaces.filter(s => s.id !== id); 
            renderTable(); updateTotals(); 
        }

        function renderTable() {
            const tbody = document.getElementById('spacesTableBody'); tbody.innerHTML = '';
            if (spaces.length === 0) { 
                tbody.innerHTML = '<tr><td colspan="2" class="text-center py-2 text-gray-400">Sin espacios</td></tr>'; 
                return; 
            }
            spaces.forEach(s => {
                const tr = document.createElement('tr'); 
                tr.innerHTML = `<td class="py-0.5">${s.name}</td><td class="text-center font-bold">${s.area.toFixed(2)}</td><td class="text-right no-print"><button onclick="removeSpace(${s.id})" class="text-red-500"><i class="fa-solid fa-trash-can"></i></button></td>`;
                tbody.appendChild(tr);
            });
        }

        function updateTotals() {
            let tArea = spaces.reduce((s, i) => s + i.area, 0);
            let tPerim = spaces.reduce((s, i) => s + i.perimeter, 0);
            const factor = 1 + (parseFloat(document.getElementById('wasteRange').value) || 0) / 100;
            let aW = tArea * factor; let pW = tPerim * factor;

            const q = {
                Panels: Math.ceil(aW / PANEL_AREA), 
                Cornices: Math.ceil(pW / CORNICE_LENGTH),
                Omegas: Math.ceil((aW / OMEGA_SPACING) / PROFILE_LENGTH), 
                Viguetas: Math.ceil((aW / VIGUETA_SPACING) / PROFILE_LENGTH),
                Angles: Math.ceil(pW / PROFILE_LENGTH), 
                ScrewsStructure: aW > 0 ? Math.ceil(aW / 10) : 0, 
                ScrewsSheet: aW > 0 ? Math.ceil(aW / 10) : 0
            };

            Object.keys(q).forEach(k => document.getElementById('qty' + k).value = q[k]);
            updateCosts();
        }

        function addExtraRow() {
            extraCounter++;
            const tr = document.createElement('tr');
            tr.className = "extra-row border-b border-gray-100"; tr.id = `rowExtra${extraCounter}`;
            tr.innerHTML = `<td><input type="text" placeholder="Material" class="w-full text-xs" id="nameExtra${extraCounter}"></td><td class="text-center"><input type="number" value="1" oninput="updateCosts()" id="qtyExtra${extraCounter}"></td><td class="text-right"><input type="number" value="0" oninput="updateCosts()" id="priceExtra${extraCounter}"></td><td class="text-right font-bold" id="totalExtra${extraCounter}">$0</td><td class="no-print"><button onclick="this.parentElement.parentElement.remove();updateCosts();" class="text-red-400"><i class="fa-solid fa-times"></i></button></td>`;
            document.getElementById('costTableBody').appendChild(tr);
            updateCosts();
        }

        function updateCosts() {
            let total = 0;
            ['Panels','Cornices','Omegas','Viguetas','Angles','ScrewsStructure','ScrewsSheet'].forEach(k => {
                const q = parseFloat(document.getElementById('qty'+k).value) || 0;
                const p = parseFloat(document.getElementById('price'+k).value) || 0;
                const t = q * p; 
                document.getElementById('total'+k).innerText = formatCurrency(t); 
                total += t;
            });
            document.querySelectorAll('.extra-row').forEach(row => {
                const id = row.id.replace('rowExtra', '');
                const q = parseFloat(document.getElementById('qtyExtra'+id).value) || 0;
                const p = parseFloat(document.getElementById('priceExtra'+id).value) || 0;
                const t = q * p; 
                document.getElementById('totalExtra'+id).innerText = formatCurrency(t); 
                total += t;
            });
            document.getElementById('grandTotal').innerText = formatCurrency(total);
        }

        function formatCurrency(n) { return '$ ' + n.toLocaleString('es-CO'); }

        async function shareQuotePDF() {
            document.body.classList.add('pdf-export-mode');
            const opt = { 
                filename: `Cotizacion_Galpa_${document.getElementById('clientName').value || 'Cliente'}.pdf`, 
                jsPDF: { format: 'letter', orientation: 'portrait' },
                html2canvas: { scale: 3, useCORS: true }
            };
            try {
                const pdfBlob = await html2pdf().set(opt).from(document.getElementById('printableArea')).output('blob');
                if (navigator.share) {
                    await navigator.share({ 
                        files: [new File([pdfBlob], opt.filename, { type: 'application/pdf' })],
                        title: 'Cotización Galpa'
                    });
                } else { 
                    const a = document.createElement('a'); a.href = URL.createObjectURL(pdfBlob); a.download = opt.filename; a.click(); 
                }
            } catch (err) {
                console.error(err);
            } finally { 
                document.body.classList.remove('pdf-export-mode'); 
            }
        }

        function clearAll() { 
            Swal.fire({ title: '¿Limpiar todo?', icon: 'warning', showCancelButton: true }).then(r => {
                if(r.isConfirmed) {
                    spaces = []; 
                    document.querySelectorAll('.extra-row').forEach(e => e.remove());
                    renderTable(); updateTotals();
                }
            });
        }
    </script>
</body>
</html>