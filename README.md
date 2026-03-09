<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Google Forms Simulation</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/chartjs-plugin-datalabels@2"></script>
    <style>
        body { background-color: #f0ebf8; font-family: 'Google Sans', Roboto, Arial, sans-serif; display: flex; flex-direction: column; align-items: center; padding: 40px; margin: 0; }
        .container { width: 100%; max-width: 800px; }
        
        .card { background: white; border-radius: 8px; border: 1px solid #dadce0; padding: 32px 32px 48px 32px; margin-bottom: 20px; position: relative; box-shadow: 0 1px 2px rgba(0,0,0,0.1); }
        
        .title { font-size: 20px; color: #202124; margin: 0 0 5px 0; outline: none; font-weight: 400; }
        .response-count { font-size: 14px; color: #70757a; margin-bottom: 30px; outline: none; }
        
        .chart-flex { display: flex; align-items: center; justify-content: flex-start; gap: 60px; }
        .canvas-box { width: 320px; height: 320px; flex-shrink: 0; }
        
        .legend-list { list-style: none; padding: 0; margin: 0; flex-grow: 1; }
        .legend-item { display: flex; align-items: center; margin-bottom: 12px; font-size: 14px; color: #3c4043; }
        .dot { height: 12px; width: 12px; border-radius: 50%; margin-right: 12px; flex-shrink: 0; }
        
        /* Edit UI Elements */
        .edit-ui { display: none; align-items: center; }
        .data-input { border: none; border-bottom: 1px solid #1a73e8; width: 50px; text-align: center; font-family: inherit; font-size: 14px; outline: none; background: #f8f9fa; font-weight: bold; margin: 0 5px; }
        .btn-remove { color: #ea4335; cursor: pointer; margin-left: 10px; font-weight: bold; font-size: 12px; }

        /* Admin Control Panel */
        .admin-panel { position: fixed; bottom: 20px; right: 20px; background: white; padding: 15px; border-radius: 12px; box-shadow: 0 4px 20px rgba(0,0,0,0.15); z-index: 999; border: 1px solid #ddd; }
        .btn { border: none; padding: 8px 16px; border-radius: 6px; cursor: pointer; font-size: 13px; font-weight: 500; margin: 4px; display: block; width: 150px; text-align: center; }
        .btn-edit { background: #673ab7; color: white; }
        .btn-add-card { background: #e8f0fe; color: #1a73e8; border: 1px solid #1a73e8; }
        .btn-add-opt { background: #34a853; color: white; margin-top: 10px; }
        
        .show-editing .edit-ui { display: inline-flex; }
    </style>
</head>
<body id="body-tag">

<div class="admin-panel">
    <button class="btn btn-edit" onclick="toggleEdit()">Toggle Edit Mode</button>
    <button class="btn btn-add-card" onclick="addNewCard()">Add Question</button>
    <div class="edit-ui" style="flex-direction: column;">
        <button class="btn btn-add-opt" onclick="addOptionToActive()">+ Add Option</button>
    </div>
</div>

<div class="container" id="wrapper">
    <div class="card" id="card-1">
        <div class="title" contenteditable="true">What's the content creation category?</div>
        <div class="response-count" contenteditable="true">1,395 responses</div>

        <div class="chart-flex">
            <div class="canvas-box">
                <canvas id="chart-1"></canvas>
            </div>

            <div class="legend-list" id="list-1">
                <div class="legend-item">
                    <span class="dot" style="background-color: #4285f4;"></span>
                    <span contenteditable="true">Video</span>
                    <span class="edit-ui"> : <input type="number" class="data-input" value="33.5" oninput="updateChart(1)"> %</span>
                </div>
                <div class="legend-item">
                    <span class="dot" style="background-color: #ea4335;"></span>
                    <span contenteditable="true">Post/Article/Blog</span>
                    <span class="edit-ui"> : <input type="number" class="data-input" value="30.4" oninput="updateChart(1)"> %</span>
                </div>
                <div class="legend-item">
                    <span class="dot" style="background-color: #fbbc04;"></span>
                    <span contenteditable="true">Infographic</span>
                    <span class="edit-ui"> : <input type="number" class="data-input" value="36.1" oninput="updateChart(1)"> %</span>
                </div>
            </div>
        </div>
    </div>
</div>

<script>
    Chart.register(ChartDataLabels);
    let charts = {};
    const colors = ['#4285f4', '#ea4335', '#fbbc04', '#34a853', '#ff6e01', '#a142f4', '#24c1e0', '#f439a0'];

    function initChart(id) {
        const ctx = document.getElementById(`chart-${id}`).getContext('2d');
        const inputs = document.querySelectorAll(`#card-${id} .data-input`);
        const values = Array.from(inputs).map(i => parseFloat(i.value) || 0);

        charts[id] = new Chart(ctx, {
            type: 'pie',
            data: {
                datasets: [{
                    data: values,
                    backgroundColor: colors,
                    borderWidth: 2,
                    borderColor: '#ffffff'
                }]
            },
            options: {
                responsive: true,
                maintainAspectRatio: false,
                plugins: {
                    legend: { display: false },
                    datalabels: {
                        color: '#fff',
                        font: { weight: 'bold', size: 15 },
                        formatter: (val) => val > 0 ? val + '%' : ''
                    }
                }
            }
        });
    }

    function updateChart(id) {
        const inputs = document.querySelectorAll(`#card-${id} .data-input`);
        charts[id].data.datasets[0].data = Array.from(inputs).map(i => parseFloat(i.value) || 0);
        charts[id].update();
    }

    function toggleEdit() {
        document.getElementById('body-tag').classList.toggle('show-editing');
    }

    function addOptionToActive() {
        // Adds to the first card for simplicity, can be expanded to target specific cards
        const list = document.querySelector('.legend-list');
        const cardId = list.id.split('-')[1];
        const index = list.children.length;
        const color = colors[index % colors.length];

        const item = document.createElement('div');
        item.className = 'legend-item';
        item.innerHTML = `
            <span class="dot" style="background-color: ${color}"></span>
            <span contenteditable="true">New Option</span>
            <span class="edit-ui"> : <input type="number" class="data-input" value="10" oninput="updateChart(${cardId})"> % 
            <span class="btn-remove" onclick="this.parentElement.parentElement.remove(); updateChart(${cardId})">✕</span></span>
        `;
        list.appendChild(item);
        updateChart(cardId);
    }

    function addNewCard() {
        const id = Date.now();
        const wrapper = document.getElementById('wrapper');
        const div = document.createElement('div');
        div.className = 'card';
        div.id = `card-${id}`;
        div.innerHTML = `
            <div class="title" contenteditable="true">New Question Title</div>
            <div class="response-count" contenteditable="true">0 responses</div>
            <div class="chart-flex">
                <div class="canvas-box"><canvas id="chart-${id}"></canvas></div>
                <div class="legend-list" id="list-${id}">
                    <div class="legend-item"><span class="dot" style="background-color: #4285f4;"></span><span contenteditable="true">Option 1</span><span class="edit-ui"> : <input type="number" class="data-input" value="100" oninput="updateChart(${id})"> %</span></div>
                </div>
            </div>`;
        wrapper.appendChild(div);
        initChart(id);
    }

    window.onload = () => initChart(1);
</script>

</body>
</html>
