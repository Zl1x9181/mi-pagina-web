<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Agenda</title>
    <!-- Meta etiquetas necesarias -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <!-- Enlace a Bootstrap CSS -->
    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css">
    <!-- Estilos personalizados -->
    <style>
        body {
            background-color: #f8f9fa;
        }
        .container {
            margin-top: 50px;
            margin-bottom: 50px;
        }
        h1 {
            margin-bottom: 40px;
            font-weight: bold;
        }
        .task-item {
            position: relative;
            margin-bottom: 20px;
            background-color: #ffffff;
            border-radius: 10px;
            padding: 20px;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
        }
        .delete-btn {
            position: absolute;
            top: 20px;
            right: 20px;
            color: #dc3545;
            font-size: 1.5rem;
            cursor: pointer;
        }
        .delete-btn:hover {
            color: #bd2130;
        }
        .task-time {
            font-size: 1rem;
            color: #6c757d;
        }
        .task-notes {
            margin-top: 10px;
            font-size: 1rem;
            color: #343a40;
        }
        .btn-primary {
            background-color: #007bff;
            border: none;
            padding: 15px;
            font-size: 1.1rem;
            border-radius: 50px;
        }
        .btn-primary:hover {
            background-color: #0056b3;
        }
        /* Estilos para la barra de progreso de las tareas */
        .progress {
            height: 10px;
            border-radius: 50px;
            margin-top: 15px;
        }
        .progress-bar {
            background-color: #28a745;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1 class="text-center">Agenda</h1>
        <!-- Formulario -->
        <form id="task-form">
            <div class="form-row align-items-end">
                <div class="form-group col-md-3">
                    <label for="task-time">Hora</label>
                    <input type="time" id="task-time" class="form-control" required>
                </div>
                <div class="form-group col-md-7">
                    <label for="task-name">Título de la Tarea</label>
                    <input type="text" id="task-name" class="form-control" placeholder="Escribe el título de la tarea" required>
                </div>
                <div class="form-group col-md-2">
                    <button type="submit" class="btn btn-primary btn-block">Agregar</button>
                </div>
            </div>
            <div class="form-group">
                <label for="task-notes">Notas Adicionales</label>
                <textarea id="task-notes" class="form-control" rows="3" placeholder="Detalles importantes..."></textarea>
            </div>
        </form>
        <!-- Lista de tareas -->
        <div id="task-list" class="mt-4"></div>
    </div>

    <!-- Enlaces a scripts de Bootstrap y jQuery -->
    <script src="https://code.jquery.com/jquery-3.5.1.slim.min.js"></script>
    <!-- Enlace a Popper.js y Bootstrap JS -->
    <script src="https://cdn.jsdelivr.net/npm/popper.js@1.16.1/dist/umd/popper.min.js"></script>
    <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/js/bootstrap.min.js"></script>
    <!-- Script de la aplicación -->
    <script>
        const taskForm = document.getElementById('task-form');
        const taskList = document.getElementById('task-list');
        let tasks = JSON.parse(localStorage.getItem('tasks')) || [];

        displayTasks();
        checkAlarms();
        requestNotificationPermission();

        // Evento para agregar una nueva tarea
        taskForm.addEventListener('submit', function(e) {
            e.preventDefault();

            const time = document.getElementById('task-time').value;
            const name = document.getElementById('task-name').value.trim();
            const notes = document.getElementById('task-notes').value.trim();
            const alarmSet = false; // Indicador de alarma

            if (name === '') {
                alert('Por favor, ingresa el título de la tarea.');
                return;
            }

            const task = { time, name, notes, alarmSet };
            tasks.push(task);
            saveTasks();
            displayTasks();
            taskForm.reset();
        });

        // Función para mostrar las tareas en la interfaz
        function displayTasks() {
            taskList.innerHTML = '';

            // Ordenar las tareas por hora
            tasks.sort((a, b) => a.time.localeCompare(b.time));

            tasks.forEach((task, index) => {
                const taskItem = document.createElement('div');
                taskItem.classList.add('task-item');

                const deleteBtn = document.createElement('span');
                deleteBtn.innerHTML = '&times;';
                deleteBtn.classList.add('delete-btn');
                deleteBtn.addEventListener('click', () => {
                    if (confirm('¿Deseas eliminar esta tarea?')) {
                        tasks.splice(index, 1);
                        saveTasks();
                        displayTasks();
                    }
                });

                const taskTitle = document.createElement('h5');
                taskTitle.textContent = task.name;

                const taskTime = document.createElement('p');
                taskTime.textContent = `⏰ ${task.time}`;
                taskTime.classList.add('task-time');

                if (task.notes) {
                    const taskNotes = document.createElement('p');
                    taskNotes.textContent = task.notes;
                    taskNotes.classList.add('task-notes');
                    taskItem.appendChild(taskNotes);
                }

                taskItem.appendChild(deleteBtn);
                taskItem.appendChild(taskTitle);
                taskItem.appendChild(taskTime);

                taskList.appendChild(taskItem);
            });
        }

        // Función para guardar las tareas en localStorage
        function saveTasks() {
            localStorage.setItem('tasks', JSON.stringify(tasks));
        }

        // Función para verificar y activar las alarmas
        function checkAlarms() {
            setInterval(() => {
                const now = new Date();
                const currentTime = now.toTimeString().substr(0,5);

                tasks.forEach((task, index) => {
                    if (!task.alarmSet && task.time === currentTime) {
                        // Mostrar notificación
                        if (Notification.permission === 'granted') {
                            new Notification('¡Alarma!', {
                                body: `Es hora de: ${task.name}`,
                                icon: 'https://img.icons8.com/color/96/000000/alarm.png'
                            });
                        } else {
                            alert(`¡Es hora de: ${task.name}!`);
                        }
                        tasks[index].alarmSet = true;
                        saveTasks();
                    }
                });
            }, 60000); // Verificar cada minuto
        }

        // Solicitar permiso para mostrar notificaciones
        function requestNotificationPermission() {
            if ('Notification' in window) {
                if (Notification.permission !== 'granted') {
                    Notification.requestPermission();
                }
            } else {
                alert('Tu navegador no soporta notificaciones de escritorio.');
            }
        }
    </script>
</body>
