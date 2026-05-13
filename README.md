# Finance
<!DOCTYPE html>
<html lang="pt-pt">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>FinanceFlow - Gestão de Despesas</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/d3/7.8.5/d3.min.js"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Inter', sans-serif; background-color: #f8fafc; }
        .chart-container { min-height: 300px; position: relative; }
        .tooltip {
            position: absolute;
            background: rgba(0, 0, 0, 0.8);
            color: white;
            padding: 8px 12px;
            border-radius: 8px;
            pointer-events: none;
            font-size: 12px;
            z-index: 100;
            display: none;
        }
        .expense-row:hover .delete-btn { opacity: 1; }
        input::-webkit-outer-spin-button, input::-webkit-inner-spin-button { -webkit-appearance: none; margin: 0; }
        
        /* Estilo para Scrollbar */
        ::-webkit-scrollbar { width: 6px; }
        ::-webkit-scrollbar-track { background: #f1f1f1; }
        ::-webkit-scrollbar-thumb { background: #cbd5e1; border-radius: 10px; }
        ::-webkit-scrollbar-thumb:hover { background: #94a3b8; }
    </style>
</head>
<body class="text-slate-800">

    <div class="min-h-screen pb-12">
        <!-- Header -->
        <header class="bg-white border-b border-slate-200 sticky top-0 z-30">
            <div class="max-w-5xl mx-auto px-4 h-16 flex items-center justify-between">
                <div class="flex items-center gap-2">
                    <div class="w-8 h-8 bg-indigo-600 rounded-lg flex items-center justify-center text-white font-bold">€</div>
                    <h1 class="text-xl font-bold tracking-tight text-slate-900">FinanceFlow</h1>
                </div>
                <div class="flex items-center gap-4">
                    <button onclick="exportData()" class="text-xs font-medium text-indigo-600 hover:text-indigo-800 transition-colors">Exportar Dados</button>
                    <div class="h-4 w-px bg-slate-200"></div>
                    <div class="text-sm font-medium text-slate-500" id="current-date-display"></div>
                </div>
            </div>
        </header>

        <main class="max-w-5xl mx-auto px-4 mt-8 grid grid-cols-1 lg:grid-cols-12 gap-8">
            
            <!-- Coluna Esquerda: Inputs e Estatísticas -->
            <div class="lg:col-span-5 space-y-6">
                <!-- Card de Estatísticas Rápidas -->
                <div class="bg-indigo-600 rounded-2xl p-6 text-white shadow-xl shadow-indigo-100">
                    <p class="text-indigo-100 text-sm font-medium mb-1">Total Gasto Acumulado</p>
                    <h2 class="text-4xl font-bold" id="total-monthly-amount">0,00 €</h2>
                    <div class="mt-4 flex items-center gap-2 text-xs text-indigo-100">
                        <span class="bg-white/20 px-2 py-1 rounded-full" id="expense-count">0 despesas</span>
                    </div>
                </div>

                <!-- Formulário de Despesa -->
                <div class="bg-white rounded-2xl p-6 border border-slate-200 shadow-sm">
                    <h3 class="text-lg font-semibold mb-4 text-slate-900">Registar Despesa</h3>
                    <form id="expense-form" class="space-y-4">
                        <div>
                            <label class="block text-sm font-medium text-slate-600 mb-1">Descrição</label>
                            <input type="text" id="desc" required placeholder="Ex: Renda Maio" 
                                class="w-full px-4 py-2 rounded-xl border border-slate-200 focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 outline-none transition-all">
                        </div>
                        <div class="grid grid-cols-2 gap-4">
                            <div>
                                <label class="block text-sm font-medium text-slate-600 mb-1">Valor (€)</label>
                                <input type="number" id="amount" step="0.01" required placeholder="0.00"
                                    class="w-full px-4 py-2 rounded-xl border border-slate-200 focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 outline-none transition-all">
                            </div>
                            <div>
                                <label class="block text-sm font-medium text-slate-600 mb-1">Categoria</label>
                                <select id="category" class="w-full px-4 py-2 rounded-xl border border-slate-200 focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 outline-none transition-all cursor-pointer">
                                    <option value="Renda de Casa">🏠 Renda de Casa</option>
                                    <option value="Luz">⚡ Luz</option>
                                    <option value="Água">💧 Água</option>
                                    <option value="Telecomunicações">🌐 Telecomunicações</option>
                                    <option value="Alimentação">🛒 Alimentação</option>
                                    <option value="Transporte">🚗 Transporte</option>
                                    <option value="Lazer">🎬 Lazer</option>
                                    <option value="Saúde">🏥 Saúde</option>
                                    <option value="Outros">📦 Outros</option>
                                </select>
                            </div>
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-slate-600 mb-1">Data</label>
                            <input type="date" id="date" required
                                class="w-full px-4 py-2 rounded-xl border border-slate-200 focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 outline-none transition-all">
                        </div>
                        <button type="submit" class="w-full bg-indigo-600 text-white font-semibold py-3 rounded-xl hover:bg-indigo-700 transition-colors shadow-lg active:scale-[0.98]">
                            Adicionar Despesa
                        </button>
                    </form>
                </div>

                <div class="text-center">
                    <button onclick="clearAllData()" class="text-xs text-slate-400 hover:text-red-500 transition-colors">Limpar todos os dados</button>
                </div>
            </div>

            <!-- Coluna Direita: Visualização e Histórico -->
            <div class="lg:col-span-7 space-y-6">
                <!-- Card de Visualização -->
                <div class="bg-white rounded-2xl p-6 border border-slate-200 shadow-sm overflow-hidden">
                    <div class="flex flex-col sm:flex-row sm:items-center justify-between mb-6 gap-4">
                        <div>
                            <h3 class="text-lg font-semibold text-slate-900" id="chart-title">Distribuição Mensal</h3>
                            <p class="text-xs text-slate-400" id="chart-subtitle">Gasto por categoria</p>
                        </div>
                        <div class="flex bg-slate-100 p-1 rounded-xl">
                            <button onclick="setChartView('categories')" id="btn-cat" class="px-3 py-1.5 text-xs font-bold rounded-lg transition-all bg-white shadow-sm text-indigo-600">Categorias</button>
                            <button onclick="setChartView('annual')" id="btn-ann" class="px-3 py-1.5 text-xs font-bold rounded-lg transition-all text-slate-500 hover:text-slate-700">Evolução Anual</button>
                        </div>
                    </div>
                    <div id="chart-legend" class="flex flex-wrap gap-2 mb-4 min-h-[24px]"></div>
                    <div class="chart-container flex justify-center items-center" id="chart">
                        <div id="no-data-msg" class="text-slate-400 text-sm flex flex-col items-center py-10">
                            <svg class="w-16 h-16 mb-2 opacity-10" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="1" d="M9 19v-6a2 2 0 00-2-2H5a2 2 0 00-2 2v6a2 2 0 002 2h2a2 2 0 002-2zm0 0V9a2 2 0 012-2h2a2 2 0 012 2v10m-6 0a2 2 0 002 2h2a2 2 0 002-2m0 0V5a2 2 0 012-2h2a2 2 0 012 2v14a2 2 0 01-2 2h-2a2 2 0 01-2-2z"></path></svg>
                            Adicione despesas para visualizar os gráficos
                        </div>
                    </div>
                </div>

                <!-- Lista de Despesas -->
                <div class="bg-white rounded-2xl border border-slate-200 shadow-sm overflow-hidden">
                    <div class="p-6 border-b border-slate-100 flex justify-between items-center">
                        <h3 class="text-lg font-semibold text-slate-900">Histórico de Gastos</h3>
                        <span class="text-[10px] bg-slate-100 text-slate-500 px-2 py-0.5 rounded-full font-bold uppercase tracking-widest">Recentes Primeiro</span>
                    </div>
                    <div class="max-h-[500px] overflow-y-auto">
                        <table class="w-full text-left border-collapse">
                            <thead>
                                <tr class="text-[10px] uppercase tracking-wider text-slate-400 bg-slate-50 sticky top-0 z-10">
                                    <th class="px-6 py-3 font-bold">Data</th>
                                    <th class="px-6 py-3 font-bold">Descrição</th>
                                    <th class="px-6 py-3 font-bold text-right">Valor</th>
                                    <th class="px-6 py-3 font-bold text-center w-12"></th>
                                </tr>
                            </thead>
                            <tbody id="expense-list-body" class="divide-y divide-slate-100">
                                <!-- Linhas dinâmicas -->
                            </tbody>
                        </table>
                        <div id="empty-list" class="p-12 text-center text-slate-400 hidden">
                            Nenhuma despesa registada ainda.
                        </div>
                    </div>
                </div>
            </div>
        </main>
    </div>

    <!-- Sistema de Toast -->
    <div id="toast" class="fixed bottom-6 right-6 transform translate-y-20 opacity-0 transition-all duration-300 bg-slate-900 text-white px-6 py-3 rounded-xl shadow-2xl flex items-center gap-3 z-50">
        <span id="toast-msg"></span>
    </div>

    <script>
        // --- Gestão de Estado e Persistência ---
        let expenses = JSON.parse(localStorage.getItem('finance_flow_data')) || [];
        let currentView = 'categories';
        
        const CATEGORY_COLORS = {
            'Renda de Casa': '#1e293b',
            'Luz': '#f59e0b',
            'Água': '#0ea5e9',
            'Telecomunicações': '#6366f1',
            'Alimentação': '#8b5cf6',
            'Transporte': '#f97316',
            'Lazer': '#ec4899',
            'Saúde': '#ef4444',
            'Outros': '#94a3b8'
        };

        const form = document.getElementById('expense-form');
        const listBody = document.getElementById('expense-list-body');
        const totalDisplay = document.getElementById('total-monthly-amount');
        const countDisplay = document.getElementById('expense-count');
        const emptyListMsg = document.getElementById('empty-list');
        const chartDiv = document.getElementById('chart');
        const noDataMsg = document.getElementById('no-data-msg');
        const legendDiv = document.getElementById('chart-legend');
        const chartTitle = document.getElementById('chart-title');
        const chartSubtitle = document.getElementById('chart-subtitle');

        function saveToLocal() {
            localStorage.setItem('finance_flow_data', JSON.stringify(expenses));
        }

        /**
         * Atualiza toda a Interface
         */
        function updateUI() {
            // 1. Calcular Total (Considerando o mês atual ou total acumulado)
            const total = expenses.reduce((sum, exp) => sum + exp.amount, 0);
            totalDisplay.textContent = total.toLocaleString('pt-PT', { style: 'currency', currency: 'EUR' });
            
            // 2. Atualizar Contagem
            countDisplay.textContent = `${expenses.length} ${expenses.length === 1 ? 'despesa' : 'despesas'}`;

            // 3. Renderizar Lista
            listBody.innerHTML = '';
            if (expenses.length === 0) {
                emptyListMsg.classList.remove('hidden');
            } else {
                emptyListMsg.classList.add('hidden');
                
                // Ordenar por data descendente
                const sortedExpenses = [...expenses].sort((a, b) => new Date(b.date) - new Date(a.date));
                
                sortedExpenses.forEach(exp => {
                    const row = document.createElement('tr');
                    row.className = 'expense-row group hover:bg-slate-50 transition-colors';
                    row.innerHTML = `
                        <td class="px-6 py-4 text-xs font-semibold text-slate-500">${new Date(exp.date).toLocaleDateString('pt-PT')}</td>
                        <td class="px-6 py-4">
                            <div class="flex flex-col">
                                <span class="text-sm font-bold text-slate-900">${exp.desc}</span>
                                <span class="text-[9px] text-slate-400 font-bold uppercase tracking-widest">${exp.category}</span>
                            </div>
                        </td>
                        <td class="px-6 py-4 text-right text-sm font-black text-slate-900">
                            ${exp.amount.toLocaleString('pt-PT', { minimumFractionDigits: 2 })}€
                        </td>
                        <td class="px-6 py-4 text-right">
                            <button onclick="deleteExpense('${exp.id}')" class="delete-btn opacity-0 group-hover:opacity-100 p-2 text-slate-300 hover:text-red-500 transition-all">
                                <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"></path></svg>
                            </button>
                        </td>
                    `;
                    listBody.appendChild(row);
                });
            }

            // 4. Atualizar Gráfico
            renderChart();
            saveToLocal();
        }

        window.setChartView = function(view) {
            currentView = view;
            const btnCat = document.getElementById('btn-cat');
            const btnAnn = document.getElementById('btn-ann');
            
            if (view === 'categories') {
                btnCat.className = 'px-3 py-1.5 text-xs font-bold rounded-lg transition-all bg-white shadow-sm text-indigo-600';
                btnAnn.className = 'px-3 py-1.5 text-xs font-bold rounded-lg transition-all text-slate-500 hover:text-slate-700';
                chartTitle.textContent = 'Distribuição Mensal';
                chartSubtitle.textContent = 'Gasto por categoria';
            } else {
                btnAnn.className = 'px-3 py-1.5 text-xs font-bold rounded-lg transition-all bg-white shadow-sm text-indigo-600';
                btnCat.className = 'px-3 py-1.5 text-xs font-bold rounded-lg transition-all text-slate-500 hover:text-slate-700';
                chartTitle.textContent = 'Resumo Anual';
                chartSubtitle.textContent = 'Total gasto por mês';
            }
            renderChart();
        };

        function renderChart() {
            chartDiv.querySelectorAll('svg').forEach(s => s.remove());
            legendDiv.innerHTML = '';

            if (expenses.length === 0) {
                noDataMsg.style.display = 'flex';
                return;
            }
            noDataMsg.style.display = 'none';

            if (currentView === 'categories') {
                renderDonutChart();
            } else {
                renderBarChart();
            }
        }

        function renderDonutChart() {
            const dataMap = d3.rollup(expenses, v => d3.sum(v, d => d.amount), d => d.category);
            const data = Array.from(dataMap, ([name, value]) => ({ name, value }));

            const width = 280;
            const height = 280;
            const radius = Math.min(width, height) / 2;

            const svg = d3.select("#chart")
                .append("svg")
                .attr("width", width)
                .attr("height", height)
                .attr("viewBox", `0 0 ${width} ${height}`)
                .append("g")
                .attr("transform", `translate(${width / 2}, ${height / 2})`);

            const pie = d3.pie().value(d => d.value).sort(null);
            const arc = d3.arc().innerRadius(radius * 0.65).outerRadius(radius).cornerRadius(6);

            svg.selectAll("path")
                .data(pie(data))
                .enter()
                .append("path")
                .attr("d", arc)
                .attr("fill", d => CATEGORY_COLORS[d.data.name] || '#cbd5e1')
                .attr("stroke", "white")
                .style("stroke-width", "4px")
                .transition().duration(800)
                .attrTween("d", function(d) {
                    var i = d3.interpolate(d.startAngle + 0.1, d.endAngle);
                    return t => { d.endAngle = i(t); return arc(d); };
                });

            // Adicionar valor total no centro
            const totalSum = d3.sum(data, d => d.value);
            svg.append("text")
                .attr("text-anchor", "middle")
                .attr("dy", "0.35em")
                .attr("class", "font-black text-slate-900")
                .style("font-size", "20px")
                .text(totalSum.toLocaleString('pt-PT') + "€");

            data.forEach(d => {
                const item = document.createElement('div');
                item.className = 'flex items-center gap-1.5 text-[10px] font-bold text-slate-500 bg-slate-50 px-2 py-1 rounded-lg border border-slate-100 uppercase tracking-tighter';
                item.innerHTML = `<span class="w-2 h-2 rounded-full" style="background-color: ${CATEGORY_COLORS[d.name]}"></span>${d.name}`;
                legendDiv.appendChild(item);
            });
        }

        function renderBarChart() {
            const months = ['Jan', 'Fev', 'Mar', 'Abr', 'Mai', 'Jun', 'Jul', 'Ago', 'Set', 'Out', 'Nov', 'Dez'];
            const yearData = months.map((m, i) => ({ month: m, value: 0 }));
            
            const currentYear = new Date().getFullYear();
            expenses.forEach(exp => {
                const d = new Date(exp.date);
                if (d.getFullYear() === currentYear) {
                    yearData[d.getMonth()].value += exp.amount;
                }
            });

            const margin = { top: 20, right: 10, bottom: 30, left: 45 };
            const width = chartDiv.clientWidth - margin.left - margin.right;
            const height = 260 - margin.top - margin.bottom;

            const svg = d3.select("#chart")
                .append("svg")
                .attr("width", width + margin.left + margin.right)
                .attr("height", height + margin.top + margin.bottom)
                .append("g")
                .attr("transform", `translate(${margin.left},${margin.top})`);

            const x = d3.scaleBand().range([0, width]).domain(months).padding(0.3);
            const y = d3.scaleLinear().range([height, 0]).domain([0, d3.max(yearData, d => d.value) * 1.2 || 100]);

            svg.append("g")
                .attr("transform", `translate(0,${height})`)
                .call(d3.axisBottom(x).tickSize(0).tickPadding(10))
                .selectAll("text")
                .attr("class", "font-bold text-slate-400")
                .style("font-size", "9px");

            svg.append("g")
                .call(d3.axisLeft(y).ticks(5).tickFormat(d => d + "€").tickSize(-width))
                .selectAll(".tick line").attr("stroke", "#f1f5f9");
            
            svg.selectAll(".domain").remove();

            svg.selectAll("rect")
                .data(yearData)
                .enter()
                .append("rect")
                .attr("x", d => x(d.month))
                .attr("width", x.bandwidth())
                .attr("fill", "#6366f1")
                .attr("rx", 4)
                .attr("y", height)
                .attr("height", 0)
                .transition().duration(800)
                .attr("y", d => y(d.value))
                .attr("height", d => Math.max(0, height - y(d.value)));
        }

        // --- Handlers de Interação ---

        function showToast(msg) {
            const toast = document.getElementById('toast');
            const toastMsg = document.getElementById('toast-msg');
            toastMsg.textContent = msg;
            toast.classList.remove('translate-y-20', 'opacity-0');
            setTimeout(() => {
                toast.classList.add('translate-y-20', 'opacity-0');
            }, 3000);
        }

        form.addEventListener('submit', (e) => {
            e.preventDefault();
            
            const newExpense = {
                id: crypto.randomUUID(),
                desc: document.getElementById('desc').value,
                amount: parseFloat(document.getElementById('amount').value),
                category: document.getElementById('category').value,
                date: document.getElementById('date').value
            };

            expenses.push(newExpense);
            form.reset();
            document.getElementById('date').valueAsDate = new Date();
            
            showToast('Despesa adicionada!');
            updateUI();
        });

        window.deleteExpense = function(id) {
            if (confirm('Tem a certeza que deseja remover esta despesa?')) {
                expenses = expenses.filter(e => e.id !== id);
                showToast('Despesa removida.');
                updateUI();
            }
        };

        window.clearAllData = function() {
            if (confirm('ATENÇÃO: Isto apagará TODOS os dados permanentemente. Continuar?')) {
                expenses = [];
                updateUI();
                showToast('Todos os dados foram limpos.');
            }
        };

        window.exportData = function() {
            if (expenses.length === 0) return showToast('Sem dados para exportar.');
            const dataStr = "data:text/json;charset=utf-8," + encodeURIComponent(JSON.stringify(expenses, null, 2));
            const downloadAnchorNode = document.createElement('a');
            downloadAnchorNode.setAttribute("href", dataStr);
            downloadAnchorNode.setAttribute("download", "backup_financeflow_" + new Date().toISOString().split('T')[0] + ".json");
            document.body.appendChild(downloadAnchorNode);
            downloadAnchorNode.click();
            downloadAnchorNode.remove();
        };

        window.onload = () => {
            document.getElementById('date').valueAsDate = new Date();
            document.getElementById('current-date-display').textContent = new Date().toLocaleDateString('pt-PT', { day: 'numeric', month: 'long', year: 'numeric' });
            updateUI();
        };

        window.onresize = () => renderChart();
    </script>
</body>
</html>
