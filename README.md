
1. Создание папки:
mkdir docker-nginx; cd docker-nginx

2. Создание образа по написанному коду
docker build . -t nginx-project

3. Создание и запуск контейнера
docker run -d -p 8888:80 nginx-project

код:

Dockerfile:

FROM nginx
COPY nginx.conf /etc/nginx/nginx.conf
COPY index.html /usr/share/nginx/html/
COPY script.js  /usr/share/nginx/html/
CMD ["nginx", "-g", "daemon off;"]

Image:

<img width="1254" height="1254" alt="image-part" src="https://github.com/user-attachments/assets/79b4b766-4a64-4950-be0b-8c2f6c1a1aec" />

Index.html:

<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AI Чат</title>
    <style>
        body {
            background-color: #050810;
            margin: 0;
            padding: 20px;
            min-height: 100vh;
            position: relative;
        }

        .image {
            display: block;
            margin: 0 auto 40px;
        }

        #response {
            position: absolute;
            top: 45%;
            left: 50%;
            transform: translate(-50%, -50%);
            
            min-height: 350px;
            max-width: 800px;
            width: 90%;
            padding: 20px;
            font-family: 'Franklin Gothic Medium', 'Arial Narrow', Arial, sans-serif;
            color: #ffffff;
            box-sizing: border-box;
        }

        .input-wrapper {
            position: absolute;
            bottom: 60px;
            left: 50%;
            transform: translateX(-50%);
            
            width: 70%;
            max-width: 1050px;
        }

        #userInput {
            background-color: #0c1322;
            color: #ffffff;
            width: 100%;
            height: 50px;
            padding: 14px 24px;
            font-size: 16px;
            border: 1px solid #ffffff;
            border-radius: 25px; /* Красивое скругление */
            font-family: 'Franklin Gothic Medium', 'Arial Narrow', Arial, sans-serif;
            box-sizing: border-box;
            outline: none; /* Убираем стандартную обводку при фокусе */
        }

        #userInput:focus {
            border-color: #4a90d9; /* Подсветка рамки при наборе текста */
        }
    </style>
</head>
<body>
    <img class="image" src="https://i.ibb.co/L(z)hDTt86/image-pa(r)t.png" 
         alt="лого" 
         style="width: 150px; height: 150px;">
    
    <div id="response">
        <p style="text-align: center; margin: 0;">Ответ ИИ появится здесь...</p>
    </div>

    <div class="input-wrapper">
        <input
            type="text"
            id="userInput"
            placeholder="Напишите вопрос и нажмите Enter..."
            autocomplete="off"
        />
    </div>

    <script src="script.js"></script>
</body>
</html>

Nginx.conf:

events {
    worker_connections 1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    server {
        listen 80;
        server_name localhost;

        # Раздача статики (ваш HTML и JS)
        location / {
            root /usr/share/nginx/html;
            index index.html;
            try_files $uri $uri/ =404;
        }

        # Прокси для API
        location /api/proxy {
            # Внутренний адрес Provod AI
            proxy_pass https://api.provod.ai/v1/chat/completions;
            
            # Обязательные заголовки для проксирования
            proxy_ssl_server_name on;
            proxy_set_header Host api.provod.ai;
            proxy_set_header Content-Type application/json;
            
            # ВОТ ЗДЕСЬ МЫ ПРЯЧЕМ КЛЮЧ
            # Пользователь не видит этот файл, поэтому ключ скрыт от него
            proxy_set_header Authorization "Bearer sk_e960d8af9438c98fe67d303391798c789c68a9b81b14a577";
            
            # Передаем тело запроса от клиента
            proxy_pass_request_body on;
            proxy_set_header Content-Length $content_length;
        }
    }
}

Requirements:

# Этот проект написан на JavaScript и не использует Python.
# Зависимости отсутствуют.

Script.js:

const API_URL = "/api/proxy"; 
const MODEL = "deepseek-v4-flash";

const SYSTEM_PROMPT = "Тебя зовут Коннор. Ты — наставник, который учит понимать, а не списывать. Учеба: Никогда не давай готовых решений. Объясняй принцип, разбирай похожий пример и проси пользователя решить самому. Проверяй только его попытки. Факты: Отвечай сразу, добавляя интересный контекст. Стиль: Простой, дружелюбный, без сложных терминов и снисходительности. Не используй смайлики";

const userInput = document.getElementById("userInput");
const responseBox = document.getElementById("response");

userInput.addEventListener("keydown", function (event) {
    if (event.key === "Enter") {
        event.preventDefault();
        const text = userInput.value.trim();
        if (!text) return;
        sendToAI(text);
    }
});

async function sendToAI(userText) {
    responseBox.innerHTML = '<p style="color: #4a90d9; text-align: center;">ИИ думает...</p>';
    userInput.value = "";

    try {
        // Отправляем запрос на наш Nginx прокси
        const response = await fetch(API_URL, {
            method: "POST",
            headers: {
                "Content-Type": "application/json"
                // Заголовок Authorization больше не нужен здесь, Nginx сам его добавит
            },
            body: JSON.stringify({
                model: MODEL,
                messages: [
                    { role: "system", content: SYSTEM_PROMPT },
                    { role: "user", content: userText }
                ],
                temperature: 0.7
            })
        });

        if (!response.ok) {
            const errorData = await response.json().catch(() => null);
            const errMsg = errorData?.error?.message || `Ошибка ${response.status}`;
            throw new Error(errMsg);
        }

        const data = await response.json();
        const aiReply = data.choices?.[0]?.message?.content ?? "(пустой ответ)";
        responseBox.textContent = aiReply;

    } catch (error) {
        responseBox.innerHTML = `<p style="color: #e74c3c;">Ошибка: ${error.message}</p>`;
    }
}






