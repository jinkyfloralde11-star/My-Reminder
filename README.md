!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>RemindMe Dashboard ✨ AI</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap');
        
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f8fafc;
        }

        .glass-card {
            background: rgba(255, 255, 255, 0.9);
            backdrop-filter: blur(10px);
            border: 1px solid rgba(255, 255, 255, 0.2);
        }

        .task-card {
            transition: transform 0.2s ease, box-shadow 0.2s ease;
        }

        .task-card:hover {
            transform: translateY(-2px);
            box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1);
        }

        .priority-high { border-left: 4px solid #ef4444; }
        .priority-medium { border-left: 4px solid #f59e0b; }
        .priority-low { border-left: 4px solid #10b981; }

        #notification-container {
            position: fixed;
            bottom: 24px;
            right: 24px;
            z-index: 100;
            display: flex;
            flex-direction: column;
            gap: 12px;
            pointer-events: none;
        }

        .toast {
            pointer-events: auto;
            min-width: 280px;
            padding: 16px;
            border-radius: 12px;
            background: white;
            box-shadow: 0 10px 25px -5px rgba(0,0,0,0.1);
            display: flex;
            align-items: center;
            gap: 12px;
            animation: slideIn 0.3s ease-out forwards;
            border-left: 4px solid #6366f1;
        }

        @keyframes slideIn { from { transform: translateX(100%); opacity: 0; } to { transform: translateX(0); opacity: 1; } }
        .toast.fade-out { animation: slideOut 0.3s ease-in forwards; }
        @keyframes slideOut { from { transform: translateX(0); opacity: 1; } to { transform: translateX(100%); opacity: 0; } }

        /* Loader Animation */
        .spinner {
            border: 2px solid #f3f3f3;
            border-top: 2px solid #6366f1;
            border-radius: 50%;
            width: 14px;
            height: 14px;
            animation: spin 1s linear infinite;
            display: inline-block;
        }
        @keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }
    </style>
</head>
<body class="min-h-screen text-slate-800">

    <div id="notification-container"></div>

    <div class="max-w-5xl mx-auto px-4 py-8">
        <!-- Header -->
        <header class="flex flex-col md:flex-row md:items-start justify-between mb-8 gap-4">
            <div>
                <div class="flex items-center gap-2">
                    <h1 class="text-3xl font-bold text-slate-900 tracking-tight">Daily Reminders</h1>
                    <span id="overdue-badge" class="hidden bg-rose-100 text-rose-600 text-xs font-bold px-2 py-1 rounded-full animate-pulse">
                        Action Required
                    </span>
                </div>
                <p class="text-slate-500 mt-1" id="current-date"></p>
            </div>
            <div class="flex flex-wrap gap-2">
                <button onclick="generateAIFocusPlan()" class="bg-indigo-50 border border-indigo-200 text-indigo-600 px-4 py-3 rounded-xl font-medium flex items-center justify-center gap-2 transition-all hover:bg-indigo-100">
                    <span id="ai-plan-icon">✨</span>
                    <span>AI Focus Plan</span>
                </button>
                <button onclick="toggleModal(true)" class="bg-indigo-600 hover:bg-indigo-700 text-white px-6 py-3 rounded-xl font-medium flex items-center justify-center gap-2 transition-all shadow-lg shadow-indigo-200">
                    <i class="fas fa-plus"></i>
                    <span>Add Reminder</span>
                </button>
            </div>
        </header>

        <!-- Stats Dashboard -->
        <div class="grid grid-cols-2 lg:grid-cols-4 gap-4 mb-10">
            <div class="glass-card p-5 rounded-2xl shadow-sm border border-slate-200">
                <p class="text-slate-500 text-sm font-medium">Total Tasks</p>
                <p class="text-2xl font-bold mt-1" id="stat-total">0</p>
            </div>
            <div class="glass-card p-5 rounded-2xl shadow-sm border border-slate-200">
                <p class="text-amber-600 text-sm font-medium">Pending</p>
                <p class="text-2xl font-bold mt-1" id="stat-pending">0</p>
            </div>
            <div class="glass-card p-5 rounded-2xl shadow-sm border border-slate-200">
                <p class="text-emerald-600 text-sm font-medium">Completed</p>
                <p class="text-2xl font-bold mt-1" id="stat-completed">0</p>
            </div>
            <div class="glass-card p-5 rounded-2xl shadow-sm border border-slate-200">
                <p class="text-rose-600 text-sm font-medium">Overdue</p>
                <p class="text-2xl font-bold mt-1" id="stat-overdue">0</p>
            </div>
        </div>

        <!-- AI Plan Result Container (Hidden by default) -->
        <div id="ai-plan-box" class="hidden mb-8 bg-indigo-600 text-white p-6 rounded-3xl shadow-xl animate-in fade-in slide-in-from-top-4 duration-500">
            <div class="flex justify-between items-start mb-4">
                <div class="flex items-center gap-2">
                    <span class="text-2xl">✨</span>
                    <h3 class="text-lg font-bold">Your AI Focus Plan</h3>
                </div>
                <button onclick="closeAIPlan()" class="text-indigo-200 hover:text-white">
                    <i class="fas fa-times"></i>
                </button>
            </div>
            <div id="ai-plan-content" class="text-indigo-50 text-sm leading-relaxed space-y-2 italic">
                Analyzing your tasks...
            </div>
        </div>

        <!-- Task List Container -->
        <div class="space-y-4" id="task-list">
            <div id="empty-state" class="text-center py-20 bg-white rounded-3xl border-2 border-dashed border-slate-200">
                <div class="bg-slate-50 w-16 h-16 rounded-full flex items-center justify-center mx-auto mb-4">
                    <i class="fas fa-clipboard-list text-slate-300 text-2xl"></i>
                </div>
                <h3 class="text-slate-600 font-medium">No reminders yet</h3>
                <p class="text-slate-400 text-sm">Tap "Add Reminder" to start organizing.</p>
            </div>
        </div>
    </div>

    <!-- Modal for Adding Tasks -->
    <div id="task-modal" class="fixed inset-0 bg-slate-900/50 backdrop-blur-sm z-50 hidden flex items-center justify-center p-4">
        <div class="bg-white rounded-2xl w-full max-w-md shadow-2xl overflow-hidden animate-in fade-in zoom-in duration-200">
            <div class="p-6 border-b border-slate-100 flex justify-between items-center">
                <h2 class="text-xl font-bold text-slate-800">New Reminder</h2>
                <button onclick="toggleModal(false)" class="text-slate-400 hover:text-slate-600">
                    <i class="fas fa-times"></i>
                </button>
            </div>
            <form id="reminder-form" class="p-6 space-y-4">
                <div>
                    <label class="block text-sm font-medium text-slate-700 mb-1">Task Title</label>
                    <input type="text" id="task-title" required placeholder="e.g., Plan summer vacation" class="w-full px-4 py-2 rounded-lg border border-slate-200 focus:outline-none focus:ring-2 focus:ring-indigo-500">
                </div>
                <div class="grid grid-cols-2 gap-4">
                    <div>
                        <label class="block text-sm font-medium text-slate-700 mb-1">Due Date</label>
                        <input type="date" id="task-date" required class="w-full px-4 py-2 rounded-lg border border-slate-200 focus:outline-none focus:ring-2 focus:ring-indigo-500">
                    </div>
                    <div>
                        <label class="block text-sm font-medium text-slate-700 mb-1">Priority</label>
                        <select id="task-priority" class="w-full px-4 py-2 rounded-lg border border-slate-200 focus:outline-none focus:ring-2 focus:ring-indigo-500">
                            <option value="low">Low</option>
                            <option value="medium" selected>Medium</option>
                            <option value="high">High</option>
                        </select>
                    </div>
                </div>
                <button type="submit" class="w-full bg-indigo-600 text-white py-3 rounded-xl font-bold mt-4 hover:bg-indigo-700 transition-colors">
                    Save Reminder
                </button>
            </form>
        </div>
    </div>

    <script>
        const apiKey = ""; // Provided by environment
        let tasks = [];

        const modal = document.getElementById('task-modal');
        const form = document.getElementById('reminder-form');
        const taskList = document.getElementById('task-list');
        const emptyState = document.getElementById('empty-state');
        const notificationContainer = document.getElementById('notification-container');
        const aiPlanBox = document.getElementById('ai-plan-box');
        const aiPlanContent = document.getElementById('ai-plan-content');

        // Gemini API Helper with Exponential Backoff
        async function callGemini(prompt, systemInstruction = "") {
            const url = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=${apiKey}`;
            const payload = {
                contents: [{ parts: [{ text: prompt }] }],
                systemInstruction: systemInstruction ? { parts: [{ text: systemInstruction }] } : undefined
            };

            for (let i = 0; i < 5; i++) {
                try {
                    const response = await fetch(url, {
                        method: 'POST',
                        headers: { 'Content-Type': 'application/json' },
                        body: JSON.stringify(payload)
                    });
                    if (!response.ok) throw new Error('API request failed');
                    const data = await response.json();
                    return data.candidates?.[0]?.content?.parts?.[0]?.text || "No response generated.";
                } catch (error) {
                    if (i === 4) throw error;
                    await new Promise(r => setTimeout(r, Math.pow(2, i) * 1000));
                }
            }
        }

        // ✨ AI Integration 1: Focus Plan
        async function generateAIFocusPlan() {
            const pendingTasks = tasks.filter(t => !t.completed);
            if (pendingTasks.length === 0) {
                showNotification("No pending tasks to analyze!", "info");
                return;
            }

            aiPlanBox.classList.remove('hidden');
            aiPlanContent.innerHTML = '<div class="flex items-center gap-2"><div class="spinner border-white border-t-indigo-200"></div> AI is calculating your strategy...</div>';
            
            const taskContext = pendingTasks.map(t => `- ${t.title} (Priority: ${t.priority}, Due: ${t.date})`).join('\n');
            const prompt = `Here are my tasks:\n${taskContext}\n\nPlease give me a 2-sentence highly encouraging strategic plan on what to focus on first and why. Be brief.`;
            const systemPrompt = "You are a professional productivity coach. You give concise, warm, and highly effective advice.";

            try {
                const advice = await callGemini(prompt, systemPrompt);
                aiPlanContent.innerText = advice;
            } catch (err) {
                aiPlanContent.innerText = "The AI is resting right now. Try again in a moment!";
            }
        }

        // ✨ AI Integration 2: Task Breakdown
        async function breakdownTask(id) {
            const task = tasks.find(t => t.id === id);
            const btn = document.getElementById(`breakdown-btn-${id}`);
            const originalContent = btn.innerHTML;
            btn.innerHTML = '<div class="spinner"></div>';
            btn.disabled = true;

            const prompt = `The task is: "${task.title}". Provide exactly 3 very short, actionable sub-tasks (max 5 words each) to help complete this. Return as a plain list.`;
            
            try {
                const subtasks = await callGemini(prompt, "You are a helpful assistant that breaks down large tasks into small, bite-sized actions.");
                showNotification(`✨ AI suggested steps for "${task.title}"`, 'info');
                const container = document.getElementById(`ai-result-${id}`);
                container.innerHTML = `<div class="mt-2 text-[10px] bg-indigo-50 text-indigo-700 p-2 rounded-lg border border-indigo-100 animate-in fade-in slide-in-from-top-1">
                    <strong class="block mb-1">✨ Suggested Steps:</strong>
                    ${subtasks.replace(/\n/g, '<br>')}
                </div>`;
            } catch (err) {
                showNotification("AI Breakdown failed.", "delete");
            } finally {
                btn.innerHTML = originalContent;
                btn.disabled = false;
            }
        }

        // ✨ AI Integration 3: Smart Grocery List
        async function generateShoppingList(id) {
            const task = tasks.find(t => t.id === id);
            const btn = document.getElementById(`shop-btn-${id}`);
            const originalContent = btn.innerHTML;
            btn.innerHTML = '<div class="spinner"></div>';
            btn.disabled = true;

            const prompt = `Based on the reminder "${task.title}", generate a short, categorized shopping list (e.g., Dairy, Produce, Pantry) with 2-3 items in each. Keep it helpful and concise.`;
            
            try {
                const list = await callGemini(prompt, "You are a shopping assistant. You create organized grocery lists based on general shopping reminders.");
                showNotification(`✨ AI generated a shopping list!`, 'info');
                const container = document.getElementById(`ai-result-${id}`);
                container.innerHTML = `<div class="mt-2 text-[10px] bg-emerald-50 text-emerald-700 p-2 rounded-lg border border-emerald-100 animate-in fade-in slide-in-from-top-1">
                    <strong class="block mb-1 font-bold"><i class="fas fa-shopping-basket mr-1"></i> AI Suggested List:</strong>
                    ${list.replace(/\n/g, '<br>')}
                </div>`;
            } catch (err) {
                showNotification("AI Shopping List failed.", "delete");
            } finally {
                btn.innerHTML = originalContent;
                btn.disabled = false;
            }
        }

        function closeAIPlan() {
            aiPlanBox.classList.add('hidden');
        }

        function showNotification(message, type = 'success') {
            const toast = document.createElement('div');
            toast.className = 'toast';
            let icon = 'fa-check-circle text-emerald-500';
            if (type === 'delete') icon = 'fa-trash-alt text-rose-500';
            else if (type === 'info') icon = 'fa-sparkles text-indigo-500';
            toast.innerHTML = `<i class="fas ${icon} text-lg"></i><span class="text-sm font-medium text-slate-700">${message}</span>`;
            notificationContainer.appendChild(toast);
            setTimeout(() => {
                toast.classList.add('fade-out');
                setTimeout(() => toast.remove(), 300);
            }, 3000);
        }

        function toggleModal(show) {
            modal.classList.toggle('hidden', !show);
            if (show) document.getElementById('task-title').focus();
        }

        function updateDashboard() {
            const now = new Date();
            now.setHours(0,0,0,0);
            const total = tasks.length;
            const completed = tasks.filter(t => t.completed).length;
            const pending = total - completed;
            const overdueCount = tasks.filter(t => !t.completed && new Date(t.date) < now).length;
            document.getElementById('stat-total').innerText = total;
            document.getElementById('stat-pending').innerText = pending;
            document.getElementById('stat-completed').innerText = completed;
            document.getElementById('stat-overdue').innerText = overdueCount;
            const badge = document.getElementById('overdue-badge');
            if (overdueCount > 0) { badge.classList.remove('hidden'); badge.innerText = `${overdueCount} Overdue`; }
            else { badge.classList.add('hidden'); }
            emptyState.classList.toggle('hidden', total > 0);
        }

        function renderTasks() {
            const sortedTasks = [...tasks].sort((a, b) => {
                if (a.completed !== b.completed) return a.completed ? 1 : -1;
                return new Date(a.date) - new Date(b.date);
            });
            const existingTasks = taskList.querySelectorAll('.task-card-item');
            existingTasks.forEach(t => t.remove());
            
            sortedTasks.forEach(task => {
                const card = document.createElement('div');
                card.className = `task-card-item task-card glass-card p-5 rounded-2xl shadow-sm border border-slate-200 priority-${task.priority} ${task.completed ? 'opacity-60' : ''}`;
                const isOverdue = !task.completed && new Date(task.date) < new Date().setHours(0,0,0,0);
                
                // Detect if it's a shopping task
                const isShopping = /shop|grocery|buy|market/i.test(task.title);

                card.innerHTML = `
                    <div class="flex items-start justify-between gap-4">
                        <div class="flex items-start gap-4 flex-1">
                            <button onclick="toggleComplete('${task.id}')" class="mt-1 w-6 h-6 rounded-full border-2 flex items-center justify-center transition-colors ${task.completed ? 'bg-emerald-500 border-emerald-500 text-white' : 'border-slate-300 hover:border-indigo-500'}">
                                ${task.completed ? '<i class="fas fa-check text-xs"></i>' : ''}
                            </button>
                            <div class="min-w-0 flex-1">
                                <h3 class="font-semibold text-slate-800 truncate ${task.completed ? 'line-through text-slate-400' : ''}">${task.title}</h3>
                                <div class="flex flex-wrap items-center gap-3 mt-1 text-xs">
                                    <span class="flex items-center gap-1 ${isOverdue ? 'text-rose-500 font-bold' : 'text-slate-500'}">
                                        <i class="far fa-calendar"></i> ${formatDate(task.date)}
                                    </span>
                                    <span class="uppercase tracking-wider font-bold text-[10px] px-2 py-0.5 rounded-full bg-slate-100 text-slate-600">
                                        ${task.priority}
                                    </span>
                                    ${!task.completed ? `
                                        <button id="breakdown-btn-${task.id}" onclick="breakdownTask('${task.id}')" class="text-indigo-600 hover:text-indigo-800 font-bold flex items-center gap-1">✨ Break Down</button>
                                        ${isShopping ? `<button id="shop-btn-${task.id}" onclick="generateShoppingList('${task.id}')" class="text-emerald-600 hover:text-emerald-800 font-bold flex items-center gap-1">✨ Generate List</button>` : ''}
                                    ` : ''}
                                </div>
                                <div id="ai-result-${task.id}"></div>
                            </div>
                        </div>
                        <button onclick="deleteTask('${task.id}')" class="text-slate-300 hover:text-rose-500 transition-colors p-2">
                            <i class="far fa-trash-alt"></i>
                        </button>
                    </div>
                `;
                taskList.appendChild(card);
            });
            updateDashboard();
        }

        function formatDate(dateStr) {
            const options = { month: 'short', day: 'numeric' };
            return new Date(dateStr).toLocaleDateString(undefined, options);
        }

        form.onsubmit = (e) => {
            e.preventDefault();
            const newTask = { id: Date.now().toString(), title: document.getElementById('task-title').value, date: document.getElementById('task-date').value, priority: document.getElementById('task-priority').value, completed: false };
            tasks.push(newTask);
            showNotification(`Added: ${newTask.title}`);
            form.reset();
            document.getElementById('task-date').valueAsDate = new Date();
            toggleModal(false);
            renderTasks();
        };

        window.toggleComplete = (id) => {
            const task = tasks.find(t => t.id === id);
            if (task) { task.completed = !task.completed; showNotification(task.completed ? 'Task completed!' : 'Task reopened', 'info'); renderTasks(); }
        };

        window.deleteTask = (id) => {
            const task = tasks.find(t => t.id === id);
            tasks = tasks.filter(t => t.id !== id);
            showNotification(`Deleted: ${task?.title || 'Reminder'}`, 'delete');
            renderTasks();
        };

        document.getElementById('current-date').innerText = new Date().toLocaleDateString(undefined, { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' });
        document.getElementById('task-date').valueAsDate = new Date();
        updateDashboard();
    </script>
</body>
</html>
