<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Онлайн Чат</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
        }
        .container {
            width: 90%;
            max-width: 800px;
            height: 80vh;
            background: white;
            border-radius: 16px;
            box-shadow: 0 20px 60px rgba(0,0,0,0.3);
            display: flex;
            flex-direction: column;
            overflow: hidden;
        }
        .header {
            background: #4a5568;
            color: white;
            padding: 20px;
            text-align: center;
            font-size: 20px;
            font-weight: bold;
        }
        #loginScreen, #chatScreen {
            flex: 1;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            padding: 20px;
        }
        #chatScreen {
            display: none;
            flex-direction: column;
            overflow: hidden;
        }
        .login-box {
            background: #f7fafc;
            padding: 40px;
            border-radius: 12px;
            text-align: center;
            width: 100%;
            max-width: 350px;
        }
        .login-box h2 {
            margin-bottom: 20px;
            color: #2d3748;
        }
        input {
            width: 100%;
            padding: 12px;
            margin: 10px 0;
            border: 2px solid #e2e8f0;
            border-radius: 8px;
            font-size: 16px;
            transition: border-color 0.3s;
        }
        input:focus {
            outline: none;
            border-color: #667eea;
        }
        button {
            width: 100%;
            padding: 12px;
            background: #667eea;
            color: white;
            border: none;
            border-radius: 8px;
            font-size: 16px;
            cursor: pointer;
            transition: background 0.3s;
        }
        button:hover {
            background: #5a67d8;
        }
        .messages-area {
            flex: 1;
            overflow-y: auto;
            padding: 20px;
            background: #f7fafc;
        }
        .message {
            margin-bottom: 12px;
            padding: 10px 15px;
            border-radius: 18px;
            max-width: 70%;
            word-wrap: break-word;
            animation: fadeIn 0.3s ease;
        }
        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(10px); }
            to { opacity: 1; transform: translateY(0); }
        }
        .message.sent {
            background: #667eea;
            color: white;
            margin-left: auto;
            text-align: right;
        }
        .message.received {
            background: white;
            color: #2d3748;
            border: 1px solid #e2e8f0;
        }
        .message.system {
            background: #edf2f7;
            color: #4a5568;
            text-align: center;
            font-style: italic;
            font-size: 14px;
            max-width: 90%;
            margin: 8px auto;
        }
        .message-header {
            font-size: 12px;
            margin-bottom: 5px;
            opacity: 0.8;
        }
        .input-area {
            padding: 20px;
            background: white;
            border-top: 1px solid #e2e8f0;
            display: flex;
            gap: 10px;
        }
        .input-area input {
            flex: 1;
            margin: 0;
        }
        .input-area button {
            width: auto;
            padding: 12px 24px;
        }
        .online-users {
            background: #edf2f7;
            padding: 10px 20px;
            border-bottom: 1px solid #e2e8f0;
            font-size: 14px;
            color: #4a5568;
        }
        .error {
            color: #e53e3e;
            font-size: 14px;
            margin-top: 10px;
        }
    </style>
</head>
<body>
<div class="container">
    <div class="header">
        💬 Онлайн Чат
    </div>
    
    <!-- Экран входа -->
    <div id="loginScreen">
        <div class="login-box">
            <h2>Вход в чат</h2>
            <input type="text" id="username" placeholder="Введите ваше имя" autocomplete="off">
            <button onclick="joinChat()">Присоединиться</button>
            <div id="loginError" class="error"></div>
        </div>
    </div>
    
    <!-- Экран чата -->
    <div id="chatScreen">
        <div class="online-users" id="onlineUsers">
            👥 Онлайн: 1
        </div>
        <div class="messages-area" id="messagesArea">
            <div class="message system">
                💡 Добро пожаловать! Сообщения не сохраняются и видны только текущим участникам.
            </div>
        </div>
        <div class="input-area">
            <input type="text" id="messageInput" placeholder="Введите сообщение..." autocomplete="off">
            <button onclick="sendMessage()">Отправить</button>
        </div>
    </div>
</div>

<script>
    // Хранилище в памяти (не сохраняется между вкладками/браузерами)
    let currentUser = null;
    let users = []; // Список пользователей в этом окне
    let messages = []; // Локальные сообщения
    
    // DOM элементы
    const loginScreen = document.getElementById('loginScreen');
    const chatScreen = document.getElementById('chatScreen');
    const messagesArea = document.getElementById('messagesArea');
    const onlineUsersSpan = document.getElementById('onlineUsers');
    const usernameInput = document.getElementById('username');
    const messageInput = document.getElementById('messageInput');
    const loginError = document.getElementById('loginError');
    
    // Функция добавления сообщения в чат
    function addMessage(text, type, sender = null) {
        const messageDiv = document.createElement('div');
        messageDiv.className = `message ${type}`;
        
        if (type === 'system') {
            messageDiv.textContent = text;
        } else {
            const header = document.createElement('div');
            header.className = 'message-header';
            header.textContent = type === 'sent' ? 'Вы' : sender;
            messageDiv.appendChild(header);
            
            const content = document.createElement('div');
            content.textContent = text;
            messageDiv.appendChild(content);
        }
        
        messagesArea.appendChild(messageDiv);
        messagesArea.scrollTop = messagesArea.scrollHeight;
        
        // Сохраняем в локальный массив (только для истории текущей сессии)
        messages.push({ text, type, sender, timestamp: Date.now() });
    }
    
    // Системное сообщение о входе/выходе
    function systemMessage(text) {
        addMessage(text, 'system');
    }
    
    // Обновление списка онлайн пользователей
    function updateOnlineUsers() {
        const count = users.length;
        onlineUsersSpan.innerHTML = `👥 Онлайн: ${count} ${getUserList()}`;
    }
    
    function getUserList() {
        if (users.length === 0) return '';
        const names = users.map(u => u.name).join(', ');
        return `— ${names}`;
    }
    
    // Отправка сообщения всем (в текущем окне)
    function broadcastMessage(message, sender, isSystem = false) {
        if (isSystem) {
            systemMessage(message);
            return;
        }
        
        // Отображаем у отправителя
        if (sender === currentUser.name) {
            addMessage(message, 'sent');
        } else {
            addMessage(message, 'received', sender);
        }
    }
    
    // Отправка сообщения
    function sendMessage() {
        const text = messageInput.value.trim();
        if (!text) return;
        
        broadcastMessage(text, currentUser.name);
        messageInput.value = '';
        messageInput.focus();
    }
    
    // Присоединение к чату
    function joinChat() {
        const name = usernameInput.value.trim();
        if (!name) {
            loginError.textContent = 'Пожалуйста, введите имя';
            return;
        }
        
        if (name.length > 20) {
            loginError.textContent = 'Имя не должно превышать 20 символов';
            return;
        }
        
        currentUser = { name: name, id: Date.now() };
        users.push(currentUser);
        
        // Переключаем экраны
        loginScreen.style.display = 'none';
        chatScreen.style.display = 'flex';
        
        // Системное сообщение о входе
        systemMessage(`✨ ${currentUser.name} присоединился к чату ✨`);
        updateOnlineUsers();
        
        // Обработка отправки по Enter
        messageInput.addEventListener('keypress', (e) => {
            if (e.key === 'Enter') sendMessage();
        });
    }
    
    // Обработка закрытия вкладки (выход из чата)
    window.addEventListener('beforeunload', () => {
        if (currentUser) {
            // При закрытии показываем системное сообщение (только в этом окне)
            systemMessage(`👋 ${currentUser.name} покинул чат 👋`);
        }
    });
    
    // Начальная установка
    function init() {
        // Фокус на поле ввода имени
        usernameInput.focus();
        
        // Обработка Enter на экране входа
        usernameInput.addEventListener('keypress', (e) => {
            if (e.key === 'Enter') joinChat();
        });
    }
    
    init();
</script>
</body>
</html>
